# mit-6.828-2019fall

## Tools

参考https://pdos.csail.mit.edu/6.828/2019/tools.html

选择自行编译所有的工具。

> ### Other Linux distributions (i.e. compiling your own toolchain)
>
> We assume that you are installing the toolchain into `/usr/local` on a modern Ubuntu installation.  You will need a fair amount of disk space to compile the tools (around 9GiB).  If you don't have that much space, consider using an MIT Athena machine.
>
> First, clone the repository for the RISC-V GNU Compiler Toolchain:
>
> ```
> $ git clone --recursive https://github.com/riscv/riscv-gnu-toolchain
> ```
>
> Next, make sure you have the packages needed to compile the toolchain:
>
> ```
> $ sudo apt-get install autoconf automake autotools-dev curl libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev
> ```
>
>  Configure and build the toolchain:
>
> ```
> $ cd riscv-gnu-toolchain
> $ ./configure --prefix=/usr/local
> $ sudo make
> $ cd ..
> ```
>
>  Next, retrieve and extract the source for QEMU 4.1:
>
> ```
> $ wget https://download.qemu.org/qemu-4.1.0.tar.xz
> $ tar xf qemu-4.1.0.tar.xz
> ```
>
>  Build QEMU for riscv64-softmmu:
>
> ```
> $ cd qemu-4.1.0
> $ ./configure --disable-kvm --disable-werror --prefix=/usr/local --target-list="riscv64-softmmu"
> $ make
> $ sudo make install
> $ cd ..
> ```

我这里将qemu和toolchain当作submodule添加进了git仓库，然后qemu也是从官方的github镜像编译的。

在编译qemu的过程中我遇到了一些缺少依赖的问题。通过运行

```
sudo apt install build-essential zlib1g-dev pkg-config libglib2.0-dev binutils-dev libboost-all-dev autoconf libtool libssl-dev libpixman-1-dev libpython-dev python-pip python-capstone virtualenv
```

完成了qemu的依赖安装。

最终进行测试

>   Testing your Installation 
>
> To test your installation, you should be able to check the following:
>
> ```
> $ riscv64-unknown-elf-gcc --version
> riscv64-unknown-elf-gcc (GCC) 9.2.0
> $ qemu-system-riscv64 --version
> QEMU emulator version 4.1.0
> ```
>
> You should also be able to compile and run xv6:
>
> ```
> # in the xv6 directory
> $ make qemu
> # ... lots of output ...
> init: starting sh
> $
> ```

的确是正常的。

### 注意事项

这个过程中要下载的东西非常的多。没有得设法加速，不然慢到怀疑人生。

## Lab1 Xv6 and Unix utilities

参考 https://pdos.csail.mit.edu/6.828/2019/labs/util.html

### Sleep

[sleep.c](https://github.com/black-desk/xv6-riscv-fall19/blob/b79af849131e667f6a2466f765804c055dd93048/user/sleep.c)

根据提示，我们需要去使用一个叫sleep的系统调用。

那我们首先看一下其他的user目录下的程序是如何使用系统调用的：

打开[`user/echo.c`](https://github.com/black-desk/xv6-riscv-fall19/blob/c0beeccb1b46cfba7762740b901ca266adb65a6f/user/echo.c),可以看到其引用了三个头文件，以及下面引用了一个名为write的函数，看起来像是个系统调用。

可以在[`user/user.h`](https://github.com/black-desk/xv6-riscv-fall19/blob/64b93d175ac6eb739036b394fbb0766fbf06f5b7/user/user.h)中找到这个函数，也可以看到这里有一大堆系统调用的声明。

根据提示不难发现这些系统调用的实现在[`user/usys.S`](https://github.com/black-desk/xv6-riscv-fall19/blob/util/user/usys.S)中，可以看出形如SYS_sleep的宏是系统调用号。可以在[`kernel/syscall.h`](https://github.com/black-desk/xv6-riscv-fall19/blob/87696bad0d7c92cf573e7fd5eb9a8178ba686c4e/kernel/syscall.h)中找到它们的具体数值。

根据提示我们可以发现这些系统调用通过[`kernel/syscall.c`](https://github.com/black-desk/xv6-riscv-fall19/blob/f6ec8d09bcef5c303cc55014eff359ae07ac417f/kernel/syscall.c) 最终运行了位于文件[`kernel/sysproc.c`](https://github.com/black-desk/xv6-riscv-fall19/blob/67702cf706bce7adef472f0caa48d81ddfaeb33a/kernel/sysproc.c)中的相关代码。

这些函数的相关使用我们可以参考linux的man手册中的说明，行为基本是一致的。

参考其他的user程序简单写完sleep之后，我们将sleep添加到[`Makefile`的120行](https://github.com/black-desk/xv6-riscv-fall19/blob/b79af849131e667f6a2466f765804c055dd93048/Makefile#L120)处的UPROGS中，然后编译运行即可看到效果。

### Uptime

[uptime.c](https://github.com/black-desk/xv6-riscv-fall19/blob/b79af849131e667f6a2466f765804c055dd93048/user/uptime.c)

与sleep类似实现uptime即可。有相应的系统调用。

### Pingpong

[pingpong.c](https://github.com/black-desk/xv6-riscv-fall19/blob/b79af849131e667f6a2466f765804c055dd93048/user/pingpong.c)

这里要求我们创建一个程序，父进程通过管道向子进程写消息“ping”，子进程收到后向父进程发消息“pong”。

提示告诉我们可以使用pipe系统调用创建一个管道，然后使用fork系统调用创建一个子进程，使用read系统调用从管道中读，使用write系统调用向管道中写。同时可以使用getid（）来获取程序的pid。

我们可以在[`kernel/proc.c`的272行](https://github.com/black-desk/xv6-riscv-fall19/blob/23ecf5bb68de26059addd4499ae384ca6eee0c1a/kernel/proc.c#L272)处看到，子进程的fork（）的结果是0

这里要注意的是如果[27行处](https://github.com/black-desk/xv6-riscv-fall19/blob/b79af849131e667f6a2466f765804c055dd93048/user/pingpong.c#L27)不加sleep(1)，子进程可能会来不及写pong。

### Primes

[primes.c](https://github.com/black-desk/xv6-riscv-fall19/blob/b79af849131e667f6a2466f765804c055dd93048/user/primes.c)

这里要求我们根据https://swtch.com/~rsc/thread/创建一个找质数的程序。

大体的思路如下：

```
p = get a number from left neighbor
print p
loop:
    n = get a number from left neighbor
    if (p does not divide n)
        send n to right neighbor
```

![算法示意图](https://swtch.com/~rsc/thread/sieve.gif)

需要注意的地方在lab的说明中已经很详细了。

### Find

[find.c](https://github.com/black-desk/xv6-riscv-fall19/blob/31bed5a737aebcf99e251f8419292ff575aa85cf/user/find.c)

复制[ls.c](https://github.com/black-desk/xv6-riscv-fall19/blob/64b93d175ac6eb739036b394fbb0766fbf06f5b7/user/ls.c)的代码来魔改即可.

提示告诉我们在[grep.c](https://github.com/black-desk/xv6-riscv-fall19/blob/fc337af2b6275d8b0b8bc41b5e2eb3619eb47bf1/user/grep.c)里面有正则相关的代码,也复制过来用就行了.

这里有个坑:

回忆之前的Primes中的情况,求质数的过程中会停下来是因为这个系统对同时打开的文件数量有限制.

>  Since xv6 has limited number of file descriptors and processes, the first process can stop at 35.

所以这里如果我们写find的时候open了一个文件而没有close,会导致之后的文件无法打开.

要注意及时关闭文件.

### Xargs

[xargs.c](https://github.com/black-desk/xv6-riscv-fall19/blob/util/user/xargs.c)

需要从一个程序中执行另一个程序, 我们会想到 exec() 系统调用, 但是由于一个程序执行了 exec() 之后, 会被所调用的那个程序代替, 所以我们不能直接运行 exec() 这个函数, 根据提示, 我们需要先 fork() 之后用子进程去 exec().

提示中写的还算清楚, 只需要控制一下读到回车再执行程序就行了.