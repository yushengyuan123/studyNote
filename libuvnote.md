### timers

用到的数据结构是一个小顶堆

### pending

用到一个万能队列

### idle，check，prepare

它们都是在 [loop-watcher.c](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fgoing-merry0%2Flibuv%2Fblob%2Ffeature%2Flearn%2Fsrc%2Funix%2Floop-watcher.c "https://github.com/going-merry0/libuv/blob/feature/learn/src/unix/loop-watcher.c") 中通过宏生成的，因为它们的操作都是一样的 - 从各自的队列中取出 handle 然后执行即可

从其他博客学习知道，这个idle和prepare阶段是node的内部调用，实际上并不知道干了什么，check阶段是执行setImmediate这种定时任务

我觉得 prepare 是取「为了下面的 `uv__io_poll` 做准备」之意，所以如果是为了 io_poll 做准备的 handle，那么可以添加到 prepare 队列中，其余则可以添加到 idle 之中

```cpp
  int uv_##name##_init(uv_loop_t* loop, uv_##name##_t* handle) {              \
    uv__handle_init(loop, (uv_handle_t*)handle, UV_##type);                   \
    handle->name##_cb = NULL;                                                 \
    return 0;                                                                 \
  }                                                                           \
                                                                              \
  int uv_##name##_start(uv_##name##_t* handle, uv_##name##_cb cb) {           \
    if (uv__is_active(handle)) return 0;                                      \
    if (cb == NULL) return UV_EINVAL;                                         \
    QUEUE_INSERT_HEAD(&handle->loop->name##_handles, &handle->queue);         \
    handle->name##_cb = cb;                                                   \
    uv__handle_start(handle);                                                 \
    return 0;                                                                 \
  }                                                                           \
                                                                              \
  int uv_##name##_stop(uv_##name##_t* handle) {                               \
    if (!uv__is_active(handle)) return 0;                                     \
    QUEUE_REMOVE(&handle->queue);                                             \
    uv__handle_stop(handle);                                                  \
    return 0;                                                                 \
  }                                                                           \
                                                                              \
  void uv__run_##name(uv_loop_t* loop) {                                      \
    uv_##name##_t* h;                                                         \
    QUEUE queue;                                                              \
    QUEUE* q;                                                                 \
    QUEUE_MOVE(&loop->name##_handles, &queue);                                \
    while (!QUEUE_EMPTY(&queue)) {                                            \
      q = QUEUE_HEAD(&queue);                                                 \
      h = QUEUE_DATA(q, uv_##name##_t, queue);                                \
      QUEUE_REMOVE(q);                                                        \
      QUEUE_INSERT_TAIL(&loop->name##_handles, q);                            \
      h->name##_cb(h);                                                        \
    }                                                                         \
  }                                                                           \
                                                                              \
  void uv__##name##_close(uv_##name##_t* handle) {                            \
    uv_##name##_stop(handle);                                                 \
  }
```

他的run方法有点想pending，但是又有不同。他定义了一个队列，也是把队列移动到了一个新的队列中，不断取出队列里面的一些node，把它移除，同时又把原来的头节点插入到队列末尾，不知道为什么要这样做。最后执行他的回调函数。

关键是`loop->name##_handles`这个东西存放的是什么任务

### io poll

利用了epoll的高校能力来完成网络请求

# libuv api

## 运行三种模式

- UV_RUN_DEFAULT：运行事件循环直到没有更加多的激活的或者饮用的handles和requests，返回一个非0数字，如果调用了uv_stop并且还有一些handles和requests要跑的话，那么就返回0

- UV_RUN_ONCE：轮询io一次，注意这个函数在没有pending回调的时候会被阻塞。当完成了所以handle和request之后这个会返回0，如果有回调还没有执行的话（意味着你应该再次执行事件循环的时候）这个就会返回一个非0

- UV_RUN_NOWAIT：轮询一次io，但是在pending阶段没有回调的时候不会发生阻塞。当完成了所以handle和request之后这个会返回0，如果有回调还没有执行的话（意味着你应该再次执行事件循环的时候）这个就会返回一个非0
