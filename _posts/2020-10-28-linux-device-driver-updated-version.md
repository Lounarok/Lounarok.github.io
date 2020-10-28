---
layout: post
title: Modern Linux device driver: Where to start and what not should be used for hardware register read/write, gpio and irq handling.
tag: linux device driver kernel module gpio gpiod ioremap request_irq request_threaded_irq irq handling
---


# Preface

It's been more than 15 years after Linux Device Driver v3. 
There are many good books for it.
This article would focus on what you should find and what should not be used.

# What would you learn from this article

2. Appropriate apis for hardware registers read/write.
1. Appropriate apis for GPIO controlling.
3. Deprecated apis.
4. https://www.kernel.org is your good friend.

References

- [Kernel.org documentation](https://www.kernel.org/doc/html/latest/index.html)
- [Unreliable Guide To Hacking The Linux Kernel](https://www.kernel.org/doc/html/latest/kernel-hacking/hacking.html)
- [Linux Kernel Labs](https://linux-kernel-labs.github.io/refs/heads/master/)
- [Elixir bootlin](https://elixir.bootlin.com/linux/latest/source) is your good firend to dive into details.

# Controlling hardware registers read/write

1. [LDD3 CH9](https://static.lwn.net/images/pdf/LDD3/ch09.pdf) is still applied.
2. `ioremap()` is the way.
3. Don't forget `request_mem_region()` and `release_mem_region()`
4. Do NOT use ioremap as a memory mapping api. It would failed terribly.

References:

- [How to use ioremap](https://stackoverflow.com/questions/16935041/how-to-write-register-from-linux-kernel-module-cpu-arm)
- [Beware of perfomance](https://stackoverflow.com/questions/4452400/memory-access-after-ioremap-very-slow)
- [Access device memory from userspace](https://stackoverflow.com/questions/44312599/write-and-read-memory-mapped-device-registers-in-linux-on-arm)
- [Access devie memory from kernel space](https://stackoverflow.com/questions/16935041/how-to-write-register-from-linux-kernel-module-cpu-arm)
- [Beware to check register clock](https://stackoverflow.com/questions/10322169/cant-read-and-write-memory-mapped-hardware-register)
- [Linux device driver ch15](http://www.makelinux.net/ldd3/)

# Controlling GPIO

1. Use gpio descriptor interface `gpiod_*()` instead of legacy `gpio_*`

References:

- https://www.kernel.org/doc/html/latest/driver-api/gpio/board.html
- https://www.kernel.org/doc/html/latest/driver-api/gpio/consumer.html

# IRQ handling

1. Use `request_threaded_irq()` instead of `request_irq()`. Actually `request_irq()` is [a special case](https://elixir.bootlin.com/linux/latest/source/include/linux/interrupt.h#L157) of `request_threaded_irq` interface.
2. Understand interrupt context and process context.
3. Signal mask and unmask.

References

- https://www.kernel.org/doc/html/latest/core-api/genericirq.html