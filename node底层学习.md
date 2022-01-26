# node启动流程

node是一个进程，那么启动进程肯定是有一个main函数的，这个main函数就在node.cc中定义

梳理了node初始化的流程，它主要干了这些事情：

1. 初始化了v8执行的环境

2. 初始化了c++的核心模块，形成一个链表，如果我们js模块想找的话就从这里面去找

3. 执行了一些启动的js，这些js在internal/bootstrap目录下，其中包括了loader.js

4. loader.js暴露出了require等等函数

5. 初始化node1的全局环境

6. 初始化了libuv，其中包括了事件循环

# 结合libuv对node性能的理解

看了下libuv可以发现，node在对于一些操作系统的调用（例如fs操作）是采用c++多线程的方式去做的，默认开启4线程，那么如果我们想加快这些系统调用的速度我们可以通过环境变量开启更多的线程。

但是我们留意到这些任务的回调函数都是通过主线程事件循环进行回调的，所以node执行回调函数的效率是低下的。

总结：所以说node是擅长做一些fs这种操作的，因为底层是多线程是做的，但是对于回调函数来说不太适应做一些操作很重的任务，因为这些回调只有主线程在跑，会阻塞的

所以说node不行我觉得就是libuv不行，是事件循环这套体系不行。

所以说我觉得node当初设计的初衷就不是用来搞服务器的。

# 继续探索的问题

1. 探索一下node的进程，cluster模块和线程是怎么实现的

2. 继续探索libuv在node中的使用，把整一套事件循环串联起来

# child_process

child__process是ChildProcess类实例，他js层是继承了EventEmitter事件中心

## spawn

```cpp
function spawn(file, args, options) {
  options = normalizeSpawnArguments(file, args, options);
  validateTimeout(options.timeout);
  validateAbortSignal(options.signal, 'options.signal');
  const killSignal = sanitizeKillSignal(options.killSignal);
  // 
  const child = new ChildProcess();
}
```

ChildProcess是一个js类，里面有一个handle属性，是一个c++类，创建出来的进程对象。

这个c++对象有spawn方法，本质上调用了libuv的方法uv_spawn方法创建子进程，里面本质就是c语言的fork函数

## c语言的进程通信

### pipe函数

```cpp
#include<unistd.h>
int pipe(int fd[2]);
```

pipe函数可用于创建一个管道，已实现进程通信

pipe函数定义中的fd参数是一个大小为2的一个数组类型的指针。该函数成功时返回0，并将一对打开的文件描述符值填入fd参数指向的数组。失败时返回 -1并设置errno。

通过pipe函数创建的这两个文件描述符 fd[0] 和 fd[1] 分别构成管道的两端，往 fd[1] 写入的数据可以从 fd[0] 读出。并且 fd[1] 一端只能进行写操作，fd[0] 一端只能进行读操作，不能反过来使用。要实现双向数据传输，可以使用两个管道。

### fork函数

```c
   #include <sys/types.h>

    #include <unistd.h>

    pid_t fork(void);
```

由f o r k创建的新进程被称为子进程（ child process）。该函数被调用一次，但返回两次。两次返回的区别是子进程的返回值是0，而父进程的返回值则是新子进程的进程I D。将子进程I D返回给父进程的理由是：因为一个进程的子进程可以多于一个，所以没有一个函数使一个进程可以获得其所有子进程的进程I D。f o r k使子进程得到返回值0的理由是：一个进程只会有一个父进程，所以子进程总是可以调用g e t p p i d以获得其父进程的进程I D (进程ID 0总是由交换进程使用，所以一个子进程的进程I D不可能为0 )。

### 普通管道实现

```c
#include <stdio.h>
  2 #include <unistd.h>
  3 #include <string.h>
  4 #include <stdlib.h>
  5
  6 int main(void)
  7 {
  8     int fds[2];
  9     if(pipe(fds)==-1)
10         perror("pipe"),exit(1);
11     pid_t pid=fork();
12     if(pid==-1) perror("fork"),exit(1);
13
14     if(pid==0){//子进程
15         close(fds[0]);//关闭读端
16         sleep(1);
17         write(fds[1],"abj",3);//在写端上写入abj
18         close(fds[1]);//再关闭写端
19         exit(0);
20     }else{//父进程
21         close(fds[1]);//关闭写端
22         char buf[100]={};
23         int r=read(fds[0],buf,100);//将管道中的数据读到buf中，返回值是实际读取的字节数
24         if(r==0)//读取的自己为0，代表读取文件结束
25         printf("read EOF\n");
26         else if(r==-1){
27             perror("read"),exit(1);
28         }
29         else if(r>0)
30             printf("buf=[%s]\n",buf);
31         close(fds[0]);//读取成功，关闭读端
32         exit(0);
33     }
34 }
35
```

## fork方法原理

fork方法本质上就是spawn，底层还是调用了spawn方法，但是只是把它的几个参数写死了。例如第一个参数spawn命令行参数置为node，第二个参数帮你写死了路径。



# worker_thread

˙这个模块可以帮助我们创建nodejs的工作者线程。工作者线程和执行线程有很多类似的地方。个人理解的他们的还是有一点区别的，即使他们的用法和功能上是差不多的。

区别在于：

1. 工作者线程比执行线程更加重，原因在于工作者线程也需要自己有一套事件循环的系统在里面

## 为什么js就不能够像别的语言那样开一种执行线程呢

node的代码和c++代码不是1对1的关系，中间调用c++底层接口都是结果libuv的操作

个人觉得由于node底层libuv的限制。因为libuv就限制了你的代码执行必须走我的这一套流程。

万一不创建出由于js执行环境类似的线程，他不久乱套了吗，那些代码接口有回调函数的，异步的你怎么跑呢。所以个人觉得你出来的线程必须要遵循libuv的规范
