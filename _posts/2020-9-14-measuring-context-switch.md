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

### `./thread-switch-pipe`
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

### `./thread-switch-condvar`
```
Running for 100000 iterations
200000 context switches in 5426563841 ns (27132.8ns / switch)
From getrusage:
  voluntary switches = 199969
  involuntary switches = 108
```
Average context switch costs 27132.8 ns ~= 27.1 us.


### `./thread-pipe-msgpersec`
```
200000 iterations took 9124217647 ns. 21919.69 iters/sec
```
One iteration ~= 45.6 us. Nearly twice of context switch time, legit.


## WSL2 Ubuntu 20.04
  
### `./thread-switch-pipe`
```
measure_self_pipe: 377100100 ns for 100000 iterations (3771.00 ns / iter)
Thread 2522220352 ping_pong
  readfd 7; writefd 6
Thread 2522154752 ping_pong
  readfd 5; writefd 8
200000 context switches in 1561146400 ns (7805.7ns / switch)
From getrusage:
  voluntary switches = 0
  involuntary switches = 0
```
7805.7ns / switch ~= 7.8us/switch

### `./thread-switch-condvar`
```
Running for 100000 iterations
200000 context switches in 1094506300 ns (5472.5ns / switch)
From getrusage:
  voluntary switches = 0
  involuntary switches = 0
```
5472.5ns / switch ~= 5.5us/switch

### `./thread-pipe-msgpersec`
```
200000 iterations took 3164816200 ns. 63194.82 iters/sec
```
63194.82 iters/sec ~= 15.8 us/iter 


## MSI GF638RC 
* ( i7-8750H ) 2.2GHz
* Host OS: Win10 2004 19041.508
* VM: VMWare-player with single processor
* Guest OS: Ubuntu 20.04 with *single* processor
* Test is run under GUI environment.

### `./thread-switch-pipe`
```
measure_self_pipe: 107464989 ns for 100000 iterations (1074.65 ns / iter)
Thread 1691105088 ping_pong
  readfd 7; writefd 6
Thread 1691100928 ping_pong
  readfd 5; writefd 8
200000 context switches in 452141890 ns (2260.7ns / switch)
From getrusage:
  voluntary switches = 192383
  involuntary switches = 7689
```
2260.7ns / switch ~= 2.26us/switch

### `./thread-switch-condvar`
```
Running for 100000 iterations
200000 context switches in 463729508 ns (2318.6ns / switch)
From getrusage:
  voluntary switches = 200006
  involuntary switches = 4620

```
2318.6ns / switch ~= 2.32 us /switch
### `./thread-pipe-msgpersec`
```
200000 iterations took 913885385 ns. 218845.82 iters/sec
```
218845.82 iters/sec~= 4.57 us /iter


## Conclusion

  1. Context switch time on vm is more than twice of Bendersky's result.
While Bendersky using i7-4771, and my test laptop is i7-4720HQ. i7-4720HQ's performance is just slighter lower than i7-4771.

  2. Considering my laptop might underclocking its cpu when overheat. 
After checking by [CpuFrequenz](http://www.softwareok.com/?Download=CpuFrequenz), the frequncy is nearly 2.6GHz.
So vm might be the difference. 

  3. Gigabyte laptop might be too old to hold.. or vmplayer version is too old?
  