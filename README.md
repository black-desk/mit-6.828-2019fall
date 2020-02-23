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
