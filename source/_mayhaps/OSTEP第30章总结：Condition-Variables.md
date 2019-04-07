---
title: OSTEP第30章总结：Condition Variables
urlname: ostep-ch-30-summary-condition-variables
toc: true
date: 2018-04-26 17:02:25
tags: [OS, OSTEP]
---

本章主要讲了除锁之外的另一个重要的同步原语：**条件变量**（condition variable）。
* 条件变量简介
* 用条件变量实现`pthread_join`功能
* 用条件变量解决生产者-消费者问题
  * 错误1：发送信号的语义
  * 错误2：唤醒进程的范围
  * 正确的实现方法
  * 完整的解决方案
* 内存分配问题和广播式条件变量

## 条件变量简介
锁的功能已经很强大了，但是在某些应用场景下并不适合。比如，线程在继续执行之前需要检查某个条件是否满足，如子线程是否已经结束运行了。这是可以用共享变量和锁实现的，但是比较低效。为了满足这种应用场景，我们就需要条件变量。

条件变量是一个队列，这个队列中存放着正在等待某些状态的一些线程。当其他线程改变这一状态时，可以通过发信号的方式唤醒这个队列中的某个线程，使它继续执行。

条件变量的两个相关操作：
* `wait()`：线程希望在某个条件变量的队列中睡眠
* `signal()`：线程修改了某些系统状态，希望唤醒一个在队列中睡眠的进程

条件变量一般需要配合一个互斥锁使用：在修改共享状态变量、发出信号和进行等待的时候，都需要上锁。这是为了防止竞争条件的出现。

在POSIX标准下，声明并静态初始化条件变量的语句是
```
pthread_cond_t c = PTHREAD_COND_INITIALIZER;
```

上述操作分别对应的代码是
```
pthread_cond_wait(pthread_cond_t *c, pthread_mutex_t *m);
pthread_cond_signal(pthread_cond_t *c);
```

可以发现，`wait()`调用的参数之一是该条件变量对应的互斥锁，这是因为它假定调用`wait()`时已经获得该锁，它会释放该锁，并使线程进入睡眠状态（上述操作显然是原子的）；当线程被唤醒时，从`wait()`中返回之前，它会重新获得锁。因此把锁作为参数是必要的。至于为什么在睡眠之前要释放锁，这是显然的，否则就死锁了。

## 用条件变量实现`pthread_join`功能
代码如下。
```
int done = 0;
pthread_mutex_t m = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t c = PTHREAD_COND_INITIALIZER;

void thr_exit() {
    Pthread_mutex_lock(&m);
    done = 1;
    Pthread_cond_signal(&c);
    Pthread_mutex_unlock(&m);
}

void *child(void *arg) {
    printf("child\n");
    thr_exit();
    return NULL;
}

void thr_join() {
    Pthread_mutex_lock(&m);
    while (done == 0)
        Pthread_cond_wait(&c, &m);
    Pthread_mutex_unlock(&m);
}

int main(int argc, char *argv[]) {
    printf("parent: begin\n");
    pthread_t p;
    Pthread_create(&p, NULL, child, NULL);
    thr_join();
    printf("parent: end\n");
    return 0;
}
```

书上给出了两种错误的实现，用来说明代码中的每一句话的重要性：
* `done`状态变量是必需的。假如没有这一状态变量，且创建子线程之后，子进程立刻开始运行，结束之后父线程才执行`thr_join()`，那么父线程就接收不到信号，会一直等待。
* 在`signal`和`wait`操作周围加锁是必要的。否则仍然会形成竞争条件。如果父线程恰好在检查状态之后被打断，状态变量被修改，则父线程恢复之后会一直睡眠。
  * 事实上，虽然很多时候看起来不需要在调用`signal`时加锁，但还是加上比较好
  * 而`wait`的语义表明，在调用它的时候必须加锁


## 用条件变量解决生产者-消费者问题

### 错误1：发送信号的语义
### 错误2：唤醒进程的范围
### 正确的实现方法
### 完整的解决方案

## 内存分配问题和广播式条件变量
