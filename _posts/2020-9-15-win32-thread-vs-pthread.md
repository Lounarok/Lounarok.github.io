---
layout: post
title: Thread Difference Between Winapi and Linux (Pthread)
tag: linux thread mutex semaphore winapi pthread 
---

Found Robert Sayegh's [PThreads Vs Win32 Threads](https://www.slideshare.net/abufayez/pthreads-vs-win32-threads)
which is a good material for a side by side comparison. Quoting some of its conclusion here.

## Mutex
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
[Slide P.13](https://www.slideshare.net/abufayez/pthreads-vs-win32-threads/13)

1. Winapi has manual/auto reset. Linux has only auto-reset
1. Winapi allowed thread/process synchization. Pthreads is inter-thread based.
1. Winapi event objects have initial signaled state. Pthreads does not have initial state.
1. Winapi event objects are asynchronous, requires [WaitForSingleObject or other WaitForXXX](https://docs.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-waitforsingleobject)
1. Both timeout could be specified.