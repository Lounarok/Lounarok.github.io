---
layout: post
title: Evaluate linux context switch on linux
tag: linux thread context switch benchmark measuring 
---

My next porject is porting a system from wince (Yes...it's WinCE!) to linux.
On WinCE there is a special design memory model for compressing object memory for every non driver library. ( Read the memory section about this [post](https://docs.microsoft.com/en-us/archive/msdn-magazine/2000/november/windows-ce-3-0-enhanced-real-time-features-provide-sophisticated-thread-handling) ).
My first thought is to evaluate context switch time because the communication between UI and underlying logic would switch a lot.
Luckily found Eli Bendersky's blog about [Measuring context switching and memory overheads for Linux threads](https://eli.thegreenplace.net/2018/measuring-context-switching-and-memory-overheads-for-linux-threads/).


## Environment
* i7-4720HQ @ 2.6GHz-2.6GHz Gigabyte P34G
* Host OS: Win10 1803
* VM: VMWare-player with single processor
* Guest OS: Ubuntu 20.04 with *single* processor
* Test is run under GUI environment.

Reference [Measuring context switching and memory overheads for Linux threads](https://eli.thegreenplace.net/2018/measuring-context-switching-and-memory-overheads-for-linux-threads/).
[Test code](https://github.com/eliben/code-for-blog/tree/master/2018/threadoverhead)

Using http://kinolien.github.io/gitzip/ download files

## Test result

### ./thread-switch-pipe
```
measure_self_pipe: 117184600 ns for 100000 iterations (1171.85 ns / iter)
Thread 1410209600 ping_pong
  readfd 7; writefd 6
Thread 1410205440 ping_pong
  readfd 5; writefd 8
200000 context switches in 5757654236 ns (28788.3ns / switch)
From getrusage:
  voluntary switches = 199842
  involuntary switches = 116
```
Average context switch costs 28788.3 ns ~= 28.8 us.

### ./thread-switch-condvar
```
Running for 100000 iterations
200000 context switches in 5426563841 ns (27132.8ns / switch)
From getrusage:
  voluntary switches = 199969
  involuntary switches = 108
```
Average context switch costs 27132.8 ns ~= 27.1 us.


### ./thread-pipe-msgpersec
```
200000 iterations took 9124217647 ns. 21919.69 iters/sec
```
One iteration ~= 45.6 us. Nearly twice of context switch time, legit.

## Conclusion

  1. Context switch time on vm is more than twice of Bendersky's result.
While Bendersky using i7-4771, and my test laptop is i7-4720HQ. i7-4720HQ's performance is just slighter lower than i7-4771.

  2. Considering my laptop might underclocking its cpu when overheat. 
After checking by [CpuFrequenz](http://www.softwareok.com/?Download=CpuFrequenz), the frequncy is nearly 2.6GHz.
So vm might be the difference. 

  3. Need to update with a native machine or try it on WSL2.