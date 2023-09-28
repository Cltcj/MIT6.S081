我尝试在三个不同的地方进行搭建：

# 一、虚拟机：

- Windows系统下安装虚拟机

- 在虚拟机里面安装ubuntu系统
- 在ubuntu系统里面搭建实验环境

# 二、通过Windows子系统进行搭建

Windows搭建子系统步骤：

1、将 Windows 子系统功能添加到当前正在运行的 Windows 操作系统中。具体为：以管理员身份打开 PowerShell 终端或命令提示符窗口，并输入该命令，并重启：

Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
或者，你可以直接打开以下界面，并将`适用于Linux的Windows子系统`打上勾，具体如何打开可以参考：安装 Docker Desktop 时遇到 "WSL 2 installation is incomplete" 的解决方法


2、在Microsoft Store中搜索Ununtu，然后下载并安装Ubuntu20.04


3、安装成功之后，进行设置用户名和密码，这就是Ubuntu那一套东西了。这里不多赘述。

# 三、在云服务器上进行安装

从云服务供应商手中，白嫖或者租一台丐版的云服务器，选用Ubuntu系统进行操作即可，这里不多赘述。

# 四、MIT6.S081实验环境搭建

可以参考官方文档：6.S081 / Fall 2021

1、在Ubuntu20.04系统的终端中输入以下命令：

```
sudo apt-get install git build-essential gdb-multiarch qemu-system-misc gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu
```

该命令将安装以下软件包：

1. git: 一个版本控制系统，用于管理源代码。

2. build-essential: 一个软件包的集合，包含了编译和构建软件所需的基本工具和库。

3. gdb-multiarch: 一个支持多种架构的 GDB 调试器。

4. qemu-system-misc: QEMU 模拟器的一部分，提供了一些基本的虚拟机设备和功能。

5. gcc-riscv64-linux-gnu: 用于 RISC-V 64 位架构的交叉编译器。

6. binutils-riscv64-linux-gnu: 用于 RISC-V 64 位架构的二进制工具集。

2、下载xv6的源码或实验版的代码：

xv6源码:

```
git clone git://github.com/mit-pdos/xv6-riscv.git
```

实验版:

```
git clone git://g.csail.mit.edu/xv6-labs-2021
```

然后进行xv6文件夹内，并执行以下两条指令：

```
git checkout util
make qemu
```

然后会出现以下输入出：当出现init: starting sh时你就成功了。

```
qemu-system-riscv64 -machine virt -bios none -kernel kernel/kernel -m 128M -smp 3 -nographic -drive file=fs.img,if=none,format=raw,id=x0 -device virtio-blk-device,drive=x0,bus=virtio-mmio-bus.0

xv6 kernel is booting

hart 2 starting
hart 1 starting
init: starting sh
```

这里需要注意的是必须用Ubuntu20.04版本，我使用过22.04的会卡在这一句：

```
qemu-system-riscv64 -machine virt -bios none -kernel kernel/kernel -m 128M -smp 3 -nographic -drive file=fs.img,if=none,format=raw,id=x0 -device virtio-blk-device,drive=x0,bus=virtio-mmio-bus.0 
```

改用20.04就没出现这问题了。
