---
layout: post
title: Linux Thread Hands on
tag: linux thread races condition data lightweight process
---

### Content
1. Basic concept of thread.
2. Basic history of Linux thread.
3. Hands-On for thread creation.

### What is thread? What is Process? What's the difference
1. Both of them are indepenent sequences of execution.
1. Threads has shared memory( code segment, data segment ) and open files while Process exists indepenently.
1. Thread is lightweight process. 
    - A process could has many thread.
    - Thread takes less time in creation, context switch, and takes less memory.
    - Thread has shared memory
    - Easier the communicate across thread by shared memory. Note: this is a double-edge.
    - Easier to get race condition and data race so synchronization techniques are required (mutex, semaphore..etc).

References: 
* [What is the difference between a process and a thread?](https://stackoverflow.com/questions/200469/what-is-the-difference-between-a-process-and-a-thread)
* [What is thread?](https://www.tutorialspoint.com/operating_system/os_multi_threading.htm)
* [Difference between process and thread.](https://www.guru99.com/difference-between-process-and-thread.html)
* [Is data race and race condition the same?](https://stackoverflow.com/questions/11276259/are-data-races-and-race-condition-actually-the-same-thing-in-context-of-conc)

### History of Thread in Linux
1. [LinuxThread](https://en.wikipedia.org/wiki/LinuxThreads) first came out but it has some problems (Eg: process ID is the same so signal handling is complex).
2. Then [Native POSIX Thread Library(NPTL)](https://en.wikipedia.org/wiki/Native_POSIX_Thread_Library)
3. After linux kernel 2.6, the implementation of `pthread` is NPTL.

### Hands-On
Online compiler https://godbolt.org/z/MKKzhe
```
#include <sys/types.h>
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <pthread.h>

void printids(const char *s)
{
    pid_t pid;
    pthread_t tid;
 
    pid = getpid();
    tid = pthread_self();
    printf("%s pid %lu tid %lu (0x%lx)\n", s, (unsigned long)pid, (unsigned long)tid, (unsigned long)tid);
}
 
void *thr_fn(void *arg)
{
    printids("new thread: ");
    return((void *)0);
}
 
int main(void)
{
    int err;
    pthread_t ntid;
    
    err = pthread_create(&ntid, NULL, thr_fn, NULL);
    if (err != 0)
    {
        printf( "Can't create thread, err code %d", err );
    }
 
    printids("main thread:");
    pthread_join( ntid, NULL ); // Blocking until pthread finished.
    exit(0);
}
```

References:
*[`pthread_create`](https://man7.org/linux/man-pages/man3/pthread_create.3.html)
*[`pthread_attr_t` and `pthread_attr_init`](https://man7.org/linux/man-pages/man3/pthread_attr_init.3.html)

*[Usage of pthread(llnl)](https://computing.llnl.gov/tutorials/pthreads/)

Overall Reference:
*[Thread (CHT)](https://www.kshuang.xyz/doku.php/course:nctu-%E9%AB%98%E7%AD%89unix%E7%A8%8B%E5%BC%8F%E8%A8%AD%E8%A8%88:chapter11)
*[Prority Inversion(Wiki)](http://en.wikipedia.org/wiki/Priority_inversion)
*[Prority Inversion(CHT)](http://blog.linux.org.tw/~jserv/archives/001299.html)