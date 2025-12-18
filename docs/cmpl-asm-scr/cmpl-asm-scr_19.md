# 附录 A 运行 ARM 程序

> 原文：[`keleshev.com/compiling-to-assembly-from-scratch/appendix-a-running-arm-programs`](https://keleshev.com/compiling-to-assembly-from-scratch/appendix-a-running-arm-programs)

从头开始编译汇编

由 Vladimir Keleshev

## 32 位 ARM Linux（例如 Raspberry Pi）

如果您是幸运的 ARM 板（如 Raspberry Pi）的拥有者，您不需要任何特殊步骤。您可以使用内置的`gcc`工具链并本地运行生成的可执行文件，正如书中所讨论的那样。只需确保运行 32 位操作系统即可。

例如，假设您从*第一部分*中有一个`hello.s`汇编列表，您可以像这样汇编它：

```js
$ gcc hello.s -o hello
```

然后，您就可以本地运行生成的可执行文件：

```js
$ ./hello
Hello, assembly!
```

即使现代的 Raspberry Pi 板具有 64 位处理器，它们也可以完美地运行 32 位操作系统。

如果您想在 Raspberry Pi 上运行 64 位操作系统，您仍然可以执行 32 位代码。在这种情况下，以下说明适用。

## 64 位 ARM Linux

如果您在 ARM 设备上运行 64 位 Linux，以下说明适用：

+   更新的 Raspberry Pi，

+   一款 ARM 笔记本电脑，例如 Pinebook Pro，

+   一款 ARM 开发板，

+   一款 ARM 服务器或计算实例。

好消息是，大多数 64 位 ARM 处理器都是向后兼容的，并且可以本地运行 32 位 ARM 可执行文件。

> **实际上，……**
> 
> 有时候，芯片制造商为了节省成本而禁用了 32 位支持。最著名的是苹果的 Silicon 芯片。

注意，在 64 位 ARM Linux 的情况下，内置的`gcc`工具链仅支持 ARM 汇编的 64 位版本，这与本书中使用的 32 位汇编不同。这意味着您需要安装一个针对 32 位 ARM 的`gcc`工具链版本。假设您有一个基于 Debian 的 Linux 发行版和`apt-get`包管理器，您可以按照以下方式安装：

```js
$ sudo apt-get install gcc-arm-linux-gnueabihf
```

现在，每当书中使用`gcc`时，您需要使用`arm-linux-gnueabihf-gcc -static`代替。例如，假设您从*第一部分*中有一个`hello.s`汇编列表，您可以像这样汇编它：

```js
$ arm-linux-gnueabihf-gcc -static hello.s -o hello
```

但然后，你可以本地运行生成的可执行文件：

```js
$ ./hello
Hello, assembly!
```

## 在 x86-64 上使用 QEMU 的 Linux

本指南假设您无法访问像 Raspberry Pi 或 Pinebook Pro 这样的 ARM Linux 机器，而只有基于 AMD 或 Intel 处理器的老式 x86-64 计算机。

> 以下操作已在 Ubuntu 20.04 LTS 上测试过，但应在所有基于 Debian 的 Linux 发行版上以相同的方式工作。对于其他发行版，您可能需要使用不同的包管理器。包名可能会有所不同。

在 x86-64 Linux 机器上，默认的`gcc`工具链期望 x86-64 汇编，这与本书中描述的 32 位汇编有很大不同。这意味着您需要安装一个针对 32 位 ARM 的`gcc`工具链版本。假设您有一个基于 Debian 的 Linux 发行版和`apt-get`包管理器，您可以按照以下方式安装：

```js
$ sudo apt-get install gcc-arm-linux-gnueabihf
```

现在，每当书中使用`gcc`时，你需要使用`arm-linux-gnueabihf-gcc -static`代替。例如，假设你从*第一部分*中有一个汇编列表`hello.s`，你可以这样汇编它：

```js
$ arm-linux-gnueabihf-gcc -static hello.s -o hello
```

然而，你不能直接运行这个可执行文件，因为它是为 ARM 处理器制作的：

```js
$ ./hello
-bash: ./hello: cannot execute binary file: Exec form…
```

要运行它，我们需要安装 QEMU——一款允许模拟不同处理器的软件，包括 ARM。使用它的方法有好几种。`qemu-user`软件包允许对单个可执行文件进行模拟，而不是模拟整个操作系统。

```js
$ sudo apt-get install qemu-user
```

现在，下一步没有错误：

```js
$ ./hello
Hello, assembly!
```

刚才发生了什么？我们是如何在 x86-64 上运行 ARM 二进制的？结果是`qemu-user`有一个智能机制：当安装时，它会配置自己来处理 ARM 二进制文件（这和我们像运行二进制文件一样运行 shell 脚本的方式类似）。然而，在某些 Linux 配置中，这不会工作，你需要更明确地调用 QEMU：

```js
$ qemu-arm ./hello
Hello, assembly!
```

注意，尽管软件包的名称是`qemu-user`，但我们感兴趣的可执行文件名称是`qemu-arm`，因为 ARM 是 QEMU 的几个可能目标之一。

## 在 x86-64 的 Windows 上使用 WSL 和 QEMU

按照步骤启用 Windows Subsystem for Linux (WSL)。Ubuntu 20.04 LTS 已为此目的进行了测试。

*[`ubuntu.com/wsl`](https://ubuntu.com/wsl)*

现在，打开 WSL 终端，按照上一节中描述的步骤操作，*在 x86-64 上使用 QEMU 的 Linux*。

* * *
