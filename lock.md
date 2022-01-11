# pthread

有两个概念，一个是加锁和释放锁，另外一个是线程等待状态

## 如何加锁，释放锁

```c
pthread_mutex_t mutex=PTHREAD_MUTEX_INITIALIZER;//创建互斥锁并初始化

pthread_mutex_lock(&mutex)；//对线程上锁，此时其他线程阻塞等待该线程释放锁

----
要执行的代码段
----

pthread_mutex_unlock(&mutex);//执行完后释放锁
```

## 创建线程

```c
int number = 10;

void p(void)
{
    number*=2;
    printf("%d\n",number);
}
void q(void)
{
    number+=1;
    printf("%d\n",number);
}

int main(void)
{
    pthread_t tid1,tid2;
    pthread_create(&tid1,NULL,(void*)(&p),NULL);
    pthread_create(&tid2,NULL,(void*)(&q),NULL);
    pthread_join(tid1);
    pthread_join(tid2);

}
```

## 线程状态

pthread _cond_wait，pthread_cond_signal，同时还有用于pthread_cond_t初始化的pthread_cond_init，销毁的pthread_cond_destroy函数

wait和signal一般都是配对使用的，因为线程处于了等待状态，就需要别人通知他，唤醒他

```c
int number = 10;

void p(void)
{
    number*=2;
    printf("%d\n",number);
}
void q(void)
{
    number+=1;
    printf("%d\n",number);
}

int main(void)
{
    pthread_t tid1,tid2;
    pthread_create(&tid1,NULL,(void*)(&p),NULL);
    pthread_create(&tid2,NULL,(void*)(&q),NULL);
    pthread_join(tid1);
    pthread_join(tid2);

}
```

这段代码的打印结果是11 20 原因在于你创建了线程你没有上锁，他们是并发执行的。所以开始他们拿到的number值都是10，没有先后顺序

改造一下代码：

```c
#include<stdio.h>
#include<sys/types.h>
#include<stdlib.h>
#include<unistd.h>
#include<pthread.h>

int number = 10;

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;


void p(void)
{
  pthread_mutex_lock(&mutex);

  number*=2;

  printf("%d\n",number);

  pthread_cond_signal(&cond);

  pthread_mutex_unlock(&mutex);
}
void q(void)
{
    pthread_mutex_lock(&mutex);

    if (number == 10) {
      pthread_cond_wait(&cond, &mutex);
    }

    number+=1;
    printf("%d\n",number);
}

int main(void)
{
    pthread_t tid1,tid2;
    pthread_create(&tid1,NULL,(void*)(&p),NULL);
    pthread_create(&tid2,NULL,(void*)(&q),NULL);
    pthread_join(tid1, NULL);
    pthread_join(tid2, NULL);
}
```

这种情况下打印出来的就是20 21，执行就是有先后顺序的

## pthread_once

本函数使用初值为*PTHREAD_ONCE_INIT*的*once_control*变量保证*init_routine()*函数在本进程执行序列中仅执行一次。

使用范围，一般用于某个多线程调用的模块使用前初始化，但是无法判断那个线程会先调用他，从而不知道把代码写在哪里

使用例子：

```c
#include <semaphore.h>
#include <sys/types.h>
#include <dirent.h>
#include <pthread.h>
#include <errno.h>
#include <signal.h>
#include <time.h>

pthread_once_t once = PTHREAD_ONCE_INIT;

void once_run(void)
{
    printf("once_run in thread %u\n", pthread_self());
}

void* task1(void* arg)
{
    int tid = pthread_self();
    printf("thread1 enter %u\n", tid);
    pthread_once(&once, once_run);
    printf("thread1 returns %u\n", tid);

    return NULL;
}

void* task2(void* arg)
{
    int tid = pthread_self();
    printf("thread2 enter %u\n", tid);
    pthread_once(&once, once_run);
    printf("thread2 returns %u\n", tid);

    return NULL;
}

int main(int argc, char *argv[])
{
    pthread_t thrd1, thrd2;

    pthread_create(&thrd1, NULL, (void*)task1, NULL);
    pthread_create(&thrd2, NULL, (void*)task2, NULL);

    sleep(5);
    printf("Main thread exit...\n");

    return 0;
}
```

打印结果：

```
[root@robot ~]# ./thread_once
thread2 enter 3067722608
once_run in thread 3067722608
thread2 returns 3067722608
thread1 enter 3078212464
thread1 returns 3078212464
Main thread exit...
[root@robot ~]# 
```
