---
layout: post
title: Thread Difference Between Winapi and Linux (Pthread)
tag: linux thread mutex semaphore winapi pthread sched setinheritsched setschedpolicy
---

Found Robert Sayegh's [PThreads Vs Win32 Threads](https://www.slideshare.net/abufayez/pthreads-vs-win32-threads)
which is a good material for a side by side comparison. Quoting some of its conclusion here.

##Thread Comparison

![Signal actions ](https://image.slidesharecdn.com/pthreadsvswin32threads-13037976979078-phpapp02/95/pthreads-vs-win32-threads-10-728.jpg?cb=1415371209)

[P.10](https://www.slideshare.net/abufayez/pthreads-vs-win32-threads)

1.  SetThreadPriority → [pthread\_attr\_setschedparam](https://www.man7.org/linux/man-pages/man3/pthread_attr_setschedparam.3.html)
    1.  [pthread\_attr\_setinheritsched](https://www.man7.org/linux/man-pages/man3/pthread_attr_setinheritsched.3.html) need to be set to PTHREAD\_EXPLICIT\_SCHED. Because default value is PTHREAD\_INHERIT\_SCHED, which is inherited from creator and ignoring [pthread\_attr\_setschedparam](https://www.man7.org/linux/man-pages/man3/pthread_attr_setschedparam.3.html)
    2.  [sched\_setscheduler](https://man7.org/linux/man-pages/man2/sched_setscheduler.2.html)
    3.  [pthread\_attr\_setschedpolicy](https://www.man7.org/linux/man-pages/man3/pthread_attr_setschedpolicy.3.html) could set the scheduler policy. Details: [sched\_setscheduler](https://man7.org/linux/man-pages/man2/sched_setscheduler.2.html).
        1.  Realtime schduler
            1.  `SCHED_FIFO` : Run until end or yield. → Similar to CeSetThreadQuantum set to 0.
            2.  `SCHED_RR` : Run with time slice.
        2.  `SCHED_OTHER`: linux normal schduler. Time sharing + round robin.
        3.  Referece: [https://stackoverflow.com/a/16730634/4123703](https://stackoverflow.com/a/16730634/4123703)
    4.  [pthread\_attr\_setschedparam](https://www.man7.org/linux/man-pages/man3/pthread_attr_setschedparam.3.html) would set the priority. Details in the [setpriority](https://man7.org/linux/man-pages/man2/setpriority.2.html)
        1.  For `SCHED_OTHER`, priority must be set to 0.
        2.  For realtime scheduler, set to 1~99. Higher number with higher priority. See [sched\_get\_priority\_max](https://man7.org/linux/man-pages/man2/sched_get_priority_max.2.html) 


## Mutex
![](https://image.slidesharecdn.com/pthreadsvswin32threads-13037976979078-phpapp02/95/pthreads-vs-win32-threads-15-728.jpg?cb=1415371209)

![](https://image.slidesharecdn.com/pthreadsvswin32threads-13037976979078-phpapp02/95/pthreads-vs-win32-threads-16-728.jpg?cb=1415371209)
[Slide P.16](https://www.slideshare.net/abufayez/pthreads-vs-win32-threads/16)

2. Winapi mutex could be locked when create while pthread mutex need to be *explicit* locked after creation.
1. Winapi Named Mutex for thread/process synchronize. While Pthread mutex is interthread only. System V Semaphores could be used for across processes.
3. Winapi mutex is recursive by default while Pthreads need to *explicit* initialized it ( some linux systems might not support recursive mutex ).
4. Linux mutex trylock has no timeout but Winapi has.

References: 
* [Named Object (Mutex)](https://docs.microsoft.com/en-us/windows/win32/sync/using-named-objects)
* [System V Semaphore](https://www.softprayog.in/programming/system-v-semaphores)
* [Recursive Mutex](https://en.wikipedia.org/wiki/Reentrant_mutex)

## Events (Signal)
![](https://image.slidesharecdn.com/pthreadsvswin32threads-13037976979078-phpapp02/95/pthreads-vs-win32-threads-13-728.jpg?cb=1415371209)

![](https://image.slidesharecdn.com/pthreadsvswin32threads-13037976979078-phpapp02/95/pthreads-vs-win32-threads-14-728.jpg?cb=1415371209)
[Slide P.13](https://www.slideshare.net/abufayez/pthreads-vs-win32-threads/13)

1. Winapi has manual/auto reset. Linux has only auto-reset
1. Winapi allowed thread/process synchization. Pthreads is inter-thread based.
1. Winapi event objects have initial signaled state. Pthreads does not have initial state.
1. Winapi event objects are asynchronous, requires [WaitForSingleObject or other WaitForXXX](https://docs.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-waitforsingleobject)
1. Both timeout could be specified.

## Eample

### Winapi

```
#include <iostream>
#include <windows.h>

DWORD WINAPI helloworld(LPVOID prm) {
    DWORD cpuId = GetCurrentProcessorNumber();
    std::cout << "Hello thread on cpu "<< cpuId <<std::endl;
    return 0;
}

int main()
{
    std::cout << "WINAPI create instant run\n";
    HANDLE hInstantRun = CreateThread(
        NULL,
        0,
        &helloworld,
        NULL,
        0,// immediately run it
        NULL
    );
    WaitForSingleObject(hInstantRun, INFINITE);
    std::cout << "WINAPI create instant run done\n";
    CloseHandle(hInstantRun);

    std::cout << "WINAPI create suspend run\n";
    HANDLE hSuspendRun = CreateThread(
        NULL,
        0,
        &helloworld,
        NULL,
        CREATE_SUSPENDED,// immediately run it
        NULL
    );
    SetThreadAffinityMask(hSuspendRun, 0x01);
    std::cout << "WINAPI resume and always should be cpu 0 \n";
    ResumeThread(hSuspendRun);
    WaitForSingleObject(hSuspendRun, INFINITE);
    std::cout << "WINAPI resume done \n";
    CloseHandle(hSuspendRun);

    return 0;
}
```

Output
```
WINAPI create instant run
Hello thread on cpu 6  // <== random is machine has more than 1 core
WINAPI create instant run done
WINAPI create suspend run
WINAPI resume and always should be cpu 0
Hello thread on cpu 0  // <== always 0 because of affinity
WINAPI resume done
```

### Pthread

```
#include <pthread.h>
#include <sched.h>
#include <iostream>
#include <sys/resource.h> // to use getrusage

// This function is copied from C example in https://github.com/eliben/code-for-blog/tree/master/2018/threadoverhead
static inline unsigned long long time_ns() {
  struct timespec ts;
  if (clock_gettime(CLOCK_REALTIME, &ts)) {
    exit(1);
  }
  return ((unsigned long long)ts.tv_sec) * 1000000000LLU +
         (unsigned long long)ts.tv_nsec;
}

 // rusage using is copied from  https://github.com/eliben/code-for-blog/tree/master/2018/threadoverhead as well.
static inline void check_contextswtich(){
   
    struct rusage ru;
    if (getrusage(RUSAGE_SELF, &ru)) {
        perror("getrusage");
    } else {
        printf("From getrusage:\n");
        printf("  voluntary switches = %ld\n", ru.ru_nvcsw);
        printf("  involuntary switches = %ld\n", ru.ru_nivcsw);
    }
}

void* helloworld ( void *arg ){

    int cpuid = sched_getcpu();
    int sum = 0;
    const auto start = time_ns();
    check_contextswtich();

    // Pretending heavy works
    for (size_t i = 0; i < 100000; i++)
    {
        sum++;
    }
    check_contextswtich();
    std::cout<< "Hello pthread on cpu " << cpuid << std::endl;

    const auto end = time_ns();
    std::cout<< "Time cost in ns: " << (end-start);
    
    // TODO: for some reason, even using FIFO with max priority would get voluntary context switched
    return NULL;
}

int main(void){
    std::cout<<" pthread create\n";
    pthread_t handle;
    pthread_attr_t attr;
    cpu_set_t cpus;
    struct sched_param sparam;
    int error = 0;

    // Setting SCHED_FIFO must run program in sudo
    int policy = SCHED_FIFO;
    CPU_ZERO( &cpus );

    // Set bit cpu0 to enable in mask.
    CPU_SET( 1,  &cpus );

    pthread_attr_init( &attr );

    // _np means "non-portable"
    pthread_attr_setaffinity_np( &attr,sizeof(cpu_set_t), &cpus );
    error = pthread_attr_setinheritsched(&attr, PTHREAD_EXPLICIT_SCHED);
    std::cout << "setinheritsched: " << error << std::endl;

    error = pthread_attr_setschedpolicy(&attr, policy);
    std::cout << "setschedpolicy: " << error << std::endl;

    sparam.sched_priority = sched_get_priority_max(policy);
    error = pthread_attr_setschedparam(&attr ,&sparam);
    std::cout << "setschedparam: " << error << std::endl;
    error = pthread_create( &handle, &attr , &helloworld, NULL );
    //pthread_detach( handle );

    // This one would not pin the thread
    // int error = pthread_create( &handle, NULL , &helloworld, NULL ); 
    std::cout<< "pthread_create " << error << std::endl;
    void* retvalue;
    pthread_join( handle, &retvalue );

    pthread_attr_destroy( &attr );
    return 0;
}
```

Output
```
pthread create
setinheritsched: 0
setschedpolicy: 0
setschedparam: 0
pthread_create 0
From getrusage:
voluntary switches = 1
involuntary switches = 4
From getrusage:
voluntary switches = 1
involuntary switches = 4
Hello pthread on cpu 1
```

## References
1.  [If an API named xxxx\_np, means it's non-portable.](https://stackoverflow.com/questions/2238564/pthread-functions-np-suffix)
2.  [Pthread on windows](http://www.sourceware.org/pthreads-win32/)
3.  [How to include pthread properly on windows.](https://stackoverflow.com/questions/48894212/visual-studio-2017-how-to-make-include-pthread-h-work)
4.  [Using nuget in cmake](https://stackoverflow.com/questions/58488575/including-nuget-packages-in-vs2019-c-cross-platform-program)
5.  [Get processor # on linux](https://man7.org/linux/man-pages/man3/sched_getcpu.3.html)