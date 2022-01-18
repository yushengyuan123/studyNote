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

# 非网络的模拟

## 线程池

以一个uv_fs_read为例子，可以看到它有一个POST方法uv__work_submit方法，这个方法里面本质上在初始化一个线程池

```c
void uv__work_submit(uv_loop_t* loop,
                     struct uv__work* w,
                     enum uv__work_kind kind,
                     void (*work)(struct uv__work* w),
                     void (*done)(struct uv__work* w, int status)) {
  uv_once(&once, init_once);
  w->loop = loop;
  w->work = work;
  w->done = done;
  post(&w->wq, kind);
}
```

init_once本质上是一`init_threads();`函数，这个函数的作用就是初始化线程池

```c
// 线程池的初始化
static void init_threads(void) {
  unsigned int i;
  const char* val;
  uv_sem_t sem;

  nthreads = ARRAY_SIZE(default_threads);
  // 根据用户的环境变量修改创建的线程池个数
  val = getenv("UV_THREADPOOL_SIZE");
  if (val != NULL)
    nthreads = atoi(val);
  if (nthreads == 0)
    nthreads = 1;
  if (nthreads > MAX_THREADPOOL_SIZE)
    nthreads = MAX_THREADPOOL_SIZE;

  // static uv_thread_t default_threads[4];
  threads = default_threads;
  if (nthreads > ARRAY_SIZE(default_threads)) {
    threads = uv__malloc(nthreads * sizeof(threads[0]));
    if (threads == NULL) {
      nthreads = ARRAY_SIZE(default_threads);
      threads = default_threads;
    }
  }

  if (uv_cond_init(&cond))
    abort();

  if (uv_mutex_init(&mutex))
    abort();
  // static QUEUE wq;wq是一个全局的队列
  QUEUE_INIT(&wq);
  QUEUE_INIT(&slow_io_pending_wq);
  QUEUE_INIT(&run_slow_work_message);

  if (uv_sem_init(&sem, 0))
    abort();

  for (i = 0; i < nthreads; i++)
    // 创建线程池，他是线程池的执行内容
    if (uv_thread_create(threads + i, worker, &sem))
      abort();

  for (i = 0; i < nthreads; i++)
    uv_sem_wait(&sem);

  uv_sem_destroy(&sem);
}
```

可以看到这个函数做了一些事情，初始化了线程池状态pthread，初始化了锁的状态，初始化了三个队列，暂时不知道这三个队列存放一些什么消息。for循环创建线程`uv_thread_create`本质上是pthread_create创建线程

再看post函数

```c
static void post(QUEUE* q, enum uv__work_kind kind) {
  // 加锁
  uv_mutex_lock(&mutex);
  if (kind == UV__WORK_SLOW_IO) {
    /* Insert into a separate queue. */
    QUEUE_INSERT_TAIL(&slow_io_pending_wq, q);
    // 将任务插入到 `wq` 这个线程共享的队列中
    if (!QUEUE_EMPTY(&run_slow_work_message)) {
      /* Running slow I/O tasks is already scheduled => Nothing to do here.
         The worker that runs said other task will schedule this one as well. */
      uv_mutex_unlock(&mutex);
      return;
    }
    q = &run_slow_work_message;
  }
  // 把这个q任务插入到wq的队尾，这个时候已经收到了任务，在work函数中，线程由于
  // 没有任务可做，都被wait发生了阻塞，这里就是来任务了，所以要唤醒线程让他们干活
  QUEUE_INSERT_TAIL(&wq, q);
  // 如果有空闲线程，则通知它们开始工作
  if (idle_threads > 0)
    // 唤醒wait的线程
    uv_cond_signal(&cond);
  uv_mutex_unlock(&mutex);
}
```

这个post就是开始添加任务，同时唤醒初始化的时候被阻塞的线程，让他们开始干活了

## 总结一下流程

就是在fs操作里面，有一个submit操作，会去初始化线程池，但是呢这个时候任务队列里面什么都是没有的，所有的线程都处于wait，被阻塞的状态。

然后执行到了一个叫做post的函数，这个post函数就是向这个任务队列里面去塞东西的，然后去signal线程，线程就被唤醒了。然后就可以开始运行了。

运行完了之后，线程就会向loop这个数据结构里面去塞东西，给祝线程调用。然后唤醒祝线程去执行他的业务逻辑。

## 线程池通知主线程

在loop init的过程中，初始化了这个消息通知

```c
rr = uv_async_init(loop, &loop->wq_async, uv__work_done);
```

```c
int uv_async_init(uv_loop_t* loop, uv_async_t* handle, uv_async_cb async_cb) {
  int err;
  // 在win平台中是没有这句话的
  err = uv__async_start(loop);
  if (err)
    return err;

  uv__handle_init(loop, (uv_handle_t*)handle, UV_ASYNC);
  handle->async_cb = async_cb;
  handle->pending = 0;

  QUEUE_INSERT_TAIL(&loop->async_handles, &handle->queue);
  uv__handle_start(handle);

  return 0;
}
```

看下这个`uv__async_start`

```c
static int uv__async_start(uv_loop_t* loop) {
  int pipefd[2];
  int err;

  if (loop->async_io_watcher.fd != -1)
    return 0;

#ifdef __linux__
  // `eventfd` 可以创建一个 epoll 内部维护的 fd，该 fd 可以和其他真实的 fd（比如 socket fd）一样
  // 添加到 epoll 实例中，可以监听它的可读事件，也可以对其进行写入操作，因此就用户代码就可以借助这个
  // 看似虚拟的 fd 来实现的事件订阅了
  err = eventfd(0, EFD_CLOEXEC | EFD_NONBLOCK);
  if (err < 0)
    return UV__ERR(errno);

  pipefd[0] = err;
  pipefd[1] = -1;
#else
  err = uv__make_pipe(pipefd, UV_NONBLOCK_PIPE);
  if (err < 0)
    return err;
#endif
//  void uv__io_init(uv__io_t* w, uv__io_cb cb, int fd) {
//      assert(cb != NULL);
//      assert(fd >= -1);
//      QUEUE_INIT(&w->pending_queue);
//      QUEUE_INIT(&w->watcher_queue);
//      w->cb = cb;
//      w->fd = fd;
//      w->events = 0;
//      w->pevents = 0;
//
//#if defined(UV_HAVE_KQUEUE)
//      w->rcount = 0;
//      w->wcount = 0;
//#endif /* defined(UV_HAVE_KQUEUE) */
//  }
  // 可以看到这个init初始化了pending队列，是观察者队列，uv__async_io
  // 这是一个回调函数
  uv__io_init(&loop->async_io_watcher, uv__async_io, pipefd[0]);
  uv__io_start(loop, &loop->async_io_watcher, POLLIN);
  loop->async_wfd = pipefd[1];

  return 0;
}
```

这个start函数我们可以看到了他使用了eventfd去虚拟出一个fd，后面在poll run中会把它加入到epoll中

然后后面执行`uv__io_start`函数

```c
void uv__io_start(uv_loop_t* loop, uv__io_t* w, unsigned int events) {
  assert(0 == (events & ~(POLLIN | POLLOUT | UV__POLLRDHUP | UV__POLLPRI)));
  assert(0 != events);
  assert(w->fd >= 0);
  assert(w->fd < INT_MAX);

  w->pevents |= events;
  maybe_resize(loop, w->fd + 1);

#if !defined(__sun)
  /* The event ports backend needs to rearm all file descriptors on each and
   * every tick of the event loop but the other backends allow us to
   * short-circuit here if the event mask is unchanged.
   */
  if (w->events == w->pevents)
    return;
#endif
  // 大家可以翻到上面 `uv__io_poll` 的部分，会发现其中有遍历 `loop->watcher_queue`
  // 将其中的 fd 都加入到 epoll 实例中，以订阅它们的事件的动作
  if (QUEUE_EMPTY(&w->watcher_queue))
    // 在io run中loop->watcher_queue会从这个数据结构中提取观察者执行，所以我们
    // 要把它加入到观察者队列中
    QUEUE_INSERT_TAIL(&loop->watcher_queue, &w->watcher_queue);
  // 将 fd 和对应的任务关联的操作，同样可以翻看上面的 `uv__io_poll`，当接收到事件
  // 通知后，会有从 `loop->watchers` 中根据 fd 取出任务并执行其完成回调的动作
  // 另外，根据 fd 确保 watcher 不会被重复添加
  if (loop->watchers[w->fd] == NULL) {
    loop->watchers[w->fd] = w;
    loop->nfds++;
  }
}
```

这个的作用就是把我们的这个watcher_queue加入到loop->watcher_queue中。

### 总结下这个流程

在loop init过程中进行初始化，创建了eventfd这个虚拟fd，同时把观察这队列也加入到了loop观察者队列中。

那么当执行到run poll的时候，他就会从这个观察者队列中，取出东西，并且加入到epoll中

```c
while (!QUEUE_EMPTY(&loop->watcher_queue)) {
    q = QUEUE_HEAD(&loop->watcher_queue);
    QUEUE_REMOVE(q);
    QUEUE_INIT(q);

    w = QUEUE_DATA(q, uv__io_t, watcher_queue);
    assert(w->pevents != 0);
    assert(w->fd >= 0);
    assert(w->fd < (int) loop->nwatchers);

    e.events = w->pevents;
    // 把描述符加入到epoll中,在async中我们的fs操作通过eventfd，也搞出了一个fd
    // 就是这个w->fd，这里就是把这个fd加入到了epoll中
    e.data.fd = w->fd;

    if (w->events == 0)
      op = EPOLL_CTL_ADD;
    else
      op = EPOLL_CTL_MOD;

    /* XXX Future optimization: do EPOLL_CTL_MOD lazily if we stop watching
     * events, skip the syscall and squelch the events after epoll_wait().
     */
    if (epoll_ctl(loop->backend_fd, op, w->fd, &e)) {
      if (errno != EEXIST)
        abort();

      assert(op == EPOLL_CTL_ADD);

      /* We've reactivated a file descriptor that's been watched before. */
      if (epoll_ctl(loop->backend_fd, EPOLL_CTL_MOD, w->fd, &e))
        abort();
    }

    w->events = w->pevents;
  }
```

在我们的线程池里面线程，当完成了任务的时候就会执行一个叫做`uv__async_send`函数，这个函数就是会执行write方法，向fd去写入东西。

在poll run的过程中，会取出epoll event。执行他的回调函数，对于fs的回调函数，他的回调函数就是`uv__async_io`函数，会从fd去read东西，从而实现文件读取

## eventfd介绍

eventfd是一个linux的系统调用，用于事件通知

## 了解select函数

[select函数及fd_set介绍 - cs_wu - 博客园](https://www.cnblogs.com/wuyepeng/p/9745573.html)

在网络变成中经常遇到一些read这中阻塞函数，当函数不能成功执行的时候，程序就会一直阻塞在这里，无法执行下面的代码，这时候就需要用到非阻塞方法，select函数就可以实现非阻塞编程

select函数是一个轮询函数，轮询文件节点，可设置超时，超时了代码就会跳过往下执行

### 函数定义

```c
int select(int nfds,  fd_set* readset,  fd_set* writeset,  f_set* exceptset,  struct timeval* timeout);
```

 nfds           需要检查的文件描述字个数  
 readset     用来检查可读性的一组文件描述字。  
 writeset     用来检查可写性的一组文件描述字。  
 exceptset  用来检查是否有异常条件出现的文件描述字。(注：错误不包括在异常条件之内)  
 timeout      超时，填NULL为阻塞，填0为非阻塞，其他为一段超时时间

返回值：返回fd的总数，如果是-1说明出错了，0说明超时了

timeout == NULL  等待无限长的时间。等待可以被一个信号中断。当有一个描述符做好准备或者是捕获到一个信号时函数会返回。如果捕获到一个信号， select函数将返回 -1,并将变量 erro设为 EINTR。

 timeout->tv_sec == 0 &&timeout->tv_usec == 0不等待，直接返回。加入描述符集的描述符都会被测试，并且返回满足要求的描述符的个数。这种方法通过轮询，无阻塞地获得了多个文件描述符状态。

 timeout->tv_sec !=0 || timeout->tv_usec!= 0 等待指定的时间。当有描述符符合条件或者超过超时时间的话，函数返回。在超时时间即将用完但又没有描述符合条件的话，返回 0。对于第一种情况，等待也会被信号所中断。

#### 使用select实现事件通知

```c
#include <iostream>
#include <assert.h>
#include <poll.h>
#include <signal.h>
#include <sys/eventfd.h>
#include <unistd.h>
#include <string.h>
#include <thread>

static int s_efd = 0;

int createEventfd()
{
    int evtfd = eventfd(0, EFD_NONBLOCK | EFD_CLOEXEC);

    std::cout << "createEventfd() fd : " << evtfd << std::endl;

    if (evtfd < 0)
    {
        std::cout << "Failed in eventfd\n";
        abort();
    }

    return evtfd;
}

void testThread()
{
    int timeout = 0;
    while(timeout < 3) {
        sleep(1);
        timeout++;
    }

    uint64_t one = 1;
    ssize_t n = write(s_efd, &one, sizeof one);
    if(n != sizeof one)
    {
        std::cout << " writes " << n << " bytes instead of 8\n";
    }
}

int main()
{
    s_efd = createEventfd();

    fd_set rdset;
    FD_ZERO(&rdset);
    FD_SET(s_efd, &rdset);

    struct timeval timeout;
    timeout.tv_sec = 1;
    timeout.tv_usec = 0;

    std::thread t(testThread);

    while(1)
    {
        // 关键在这句话，select返回值为0说明超时了，那么就不执行我们的业务，否则就是
        // fd有事件返回，这个时候就可以执行我们的业务逻辑
        if(select(s_efd + 1, &rdset, NULL, NULL, &timeout) == 0)
        {
            std::cout << "timeout\n";
            timeout.tv_sec = 1;
            timeout.tv_usec = 0;
            FD_SET(s_efd, &rdset);
            continue;
        }

        uint64_t one = 0;

        ssize_t n = read(s_efd, &one, sizeof one);
        if(n != sizeof one)
        {
            std::cout << " read " << n << " bytes instead of 8\n";
        }

        std::cout << " wakeup ！\n";

        break;
    }

    t.join();
    close(s_efd);

    return 0;
}
```

### fd_set结构体

这个其实是一个数组的宏定义，实际上是一long类数组，每个元素都能与一开大的文件句柄建立联系。当调用select时候，由内核根据io状态修改fd_set内容，由此来通知调用select进程哪一个句柄可读

```c
FD_SET(int fd, fd_set *fdset);       //将fd加入set集合，但是注意我们的fd表示，这个数组这个位中的二进制是多少
FD_CLR(int fd, fd_set *fdset);       //将fd从set集合中清除
FD_ISSET(int fd, fd_set *fdset);     //检测fd是否在set集合中，不在则返回0，存放返回这个fd的二进制
FD_ZERO(fd_set *fdset);              //将set清零使集合中不含任何fd
```

```c
int main()
{   
    fd_set fdset;   
    FD_ZERO(&fdset);   
    FD_SET(1, &fdset);   
    FD_SET(2, &fdset);   
    FD_SET(3, &fdset);   
    FD_SET(7, &fdset);   
    int isset = FD_ISSET(3, &fdset);   
    printf("isset = %d\n", isset);   
    FD_CLR(3, &fdset);   
    isset = FD_ISSET(3, &fdset);   
    printf("isset = %d\n", isset);   
    return 0;
}
```

## timeval结构体

```c
struct timeval {
    long    tv_sec;         /* seconds */
    long    tv_usec;        /* and microseconds */
};
```
