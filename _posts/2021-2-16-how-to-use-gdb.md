---
layout: post
title: (CHT) A memo to use gdb in traditional chinese
tag: linux gdb remote debug 
---

# Remote debug with gdb server 
- Non-extended remote最大限制：不能執行run command
- Target必須要有裝gdbserver

## On Target:

將想debug的檔案放到想要的資料夾
執行 `gdbserver :10000 ./executable.out`
`:10000`表示port number
`./xxxx.out`表示執行檔
此時即開始執行程式 
注意！ 使用ctrl+c是沒辦法關閉server的，一定要從別的地方連進去關閉

## On host

進入source code +執行檔的資料夾
執行$arm-none-linux-gnueabi-gdb ./executable.out
進到gdb後輸入以下命令
`(gdb) target extended-remote 10.10.40.56`
最後面就是ip
`(gdb) set remote exec-file ./executable.out`
設定target run的執行檔位置
進入後，程式已經開始執行（不能用run），只能下斷點然後下continue
測試完成之後，輸入`monitor exit`，讓target server自行關閉


# What is coredump

- https://wiki.archlinux.org/index.php/Core_dump
- 當user program非預期中止時，會產生core dump，會紀錄program中止當下的狀態
- 通常由linux kernel遇到user program非預期中止時產生
- core dump以下情況產生
- debugger要求產生
- program 非預期中止
- Kernel module有類似的工具嘛？
- kernel module有的不是core dump, 而是kdump/kexec or kgdb
- 參考 https://stackoverflow.com/a/33337987/4123703

# Cannot find coredump on ubuntu

- Ubuntu 18.04底下找不到core dump?!

    參考
    https://stackoverflow.com/a/18368068/4123703
    https://stackoverflow.com/a/16380374/4123703
    下$ ulimit -c 檢查core dump值是不是0, 如果是，下$ ulimit -c unlimited 讓它變成無限制
    $ ulimit -a 檢查其他value是否正確
    
- 還是找不到core dump

    參考 https://stackoverflow.com/a/18368068/4123703
    $ cat /proc/sys/kernel/core_pattern 發現內容如下
    |/usr/share/apport/apport %p %s %c %d %P %E
    
    
    表示它會把core dump redirect到apport底下
    發現ubuntu的system report log app: apport會影響到core dump的處理
    
    $ tail -n 10 /var/log/apport.log 顯示的內容為 
    
    ````
    syntecuser@ubuntu:~/codespace/pthread-cond-var$ tail -n 10 /var/log/apport.log
    ERROR: apport (pid 6622) Mon Nov 30 10:27:54 2020: called for pid 6621, signal 11, core limit 0, dump mode 1
    ERROR: apport (pid 6622) Mon Nov 30 10:27:54 2020: executable: /home/syntecuser/codespace/pthread-cond-var/named_user.out (command line "./named_user.out")
    ERROR: apport (pid 6622) Mon Nov 30 10:27:54 2020: executable does not belong to a package, ignoring
    ERROR: apport (pid 6790) Mon Nov 30 10:35:36 2020: called for pid 6789, signal 11, core limit 0, dump mode 1
    ERROR: apport (pid 6790) Mon Nov 30 10:35:36 2020: executable: /home/syntecuser/codespace/pthread-cond-var/named_user.out (command line "./named_user.out")
    ERROR: apport (pid 6790) Mon Nov 30 10:35:36 2020: executable does not belong to a package, ignoring
    ````
    
- $ systemctl stop apport暫時停掉

- TL;DR 解法：

    原因: Ubuntu apport會阻止core dump產生在當前資料夾
    執行 $ systemctl stop apport可以停下apport service
    再次執行core dump程式就可以出現core dump
    
    
# 如何確認gdb有讀到dynamically linked library debug info?

https://stackoverflow.com/a/6982726/4123703

先讓程式跑起來，break在執行的第一行
`(gdb) info sharedlibrary` 可以看見有讀到的library

如果library前面有星號，表示它沒讀到debug info 

````
(gdb) info sharedlibrary
From To Syms Read Shared Object Library
0x00007ffff7dd5f10 0x00007ffff7df4b50 Yes /lib64/ld-linux-x86-64.so.2
0x00007ffff7bd3770 0x00007ffff7bd39c2 Yes (*) ./test.so
0x00007ffff79cd200 0x00007ffff79d070c Yes /lib/x86_64-linux-gnu/librt.so.1
0x00007ffff76ce490 0x00007ffff777d9de Yes (*) /usr/lib/x86_64-linux-gnu/libstdc++.so.6
0x00007ffff742cac0 0x00007ffff743d36d Yes (*) /lib/x86_64-linux-gnu/libgcc_s.so.1
0x00007ffff7210bb0 0x00007ffff721f101 Yes /lib/x86_64-linux-gnu/libpthread.so.0
0x00007ffff6e3b2d0 0x00007ffff6fb3eac Yes /lib/x86_64-linux-gnu/libc.so.6
0x00007ffff6a87a80 0x00007ffff6b461d5 Yes /lib/x86_64-linux-gnu/libm.so.6
(*): Shared library is missing debugging information.
````

有可能debug的時候沒有把library丟到/usr
可以暫時使用 export LD_LIBRARY_PATH=./:$LD_LIBRARY_PATH


