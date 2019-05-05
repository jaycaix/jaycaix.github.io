---
layout:     post
title:      "Docker 背后的容器管理——Libcontainer 深度解析"
date:       2016-04-19 10:00:00 +0800
author:     "Jay Cai"
header-style: text
catalog: true
tags:
    - Docker
    - Libcontainer
---

# Docker 背后的容器管理——Libcontainer 深度解析

Libcontainer 是 Docker 中用于容器管理的包，它基于 Go 语言实现，通过管理`namespaces`、`cgroups`、`capabilities`以及文件系统来进行容器控制。你可以使用 Libcontainer 创建容器，并对容器进行生命周期管理。

> 容器是一个可管理的执行环境，与主机系统共享内核，可与系统中的其他容器进行隔离。

在 2013 年 Docker 刚发布的时候，它是一款基于 LXC 的开源容器管理引擎。把 LXC 复杂的容器创建与使用方式简化为 Docker 自己的一套命令体系。随着 Docker 的不断发展，它开始有了更为远大的目标，那就是反向定义容器的实现标准，将底层实现都抽象化到 Libcontainer 的接口。这就意味着，底层容器的实现方式变成了一种可变的方案，无论是使用 namespace、cgroups 技术抑或是使用 systemd 等其他方案，只要实现了 Libcontainer 定义的一组接口，Docker 都可以运行。这也为 Docker 实现全面的跨平台带来了可能。

## 1\. Libcontainer 特性

目前版本的 Libcontainer，功能实现上涵盖了包括 namespaces 使用、cgroups 管理、Rootfs 的配置启动、默认的 Linux capability 权限集、以及进程运行的环境变量配置。内核版本最低要求为`2.6`，最好是`3.8`，这与内核对 namespace 的支持有关。

目前除 user namespace 不完全支持以外，其他五个 namespace 都是默认开启的，通过`clone`系统调用进行创建。

### 1.1 建立文件系统

文件系统方面，容器运行需要`rootfs`。所有容器中要执行的指令，都需要包含在`rootfs`中。所有挂载在容器销毁时都会被卸载，因为 mount namespace 会在容器销毁时一同消失。为了容器可以正常执行命令，以下文件系统必须在容器运行时挂载到`rootfs`中。

| 路径 | 类型 | 参数 | 权限及数据 |
| /proc | proc | MS_NOEXEC,MS_NOSUID,MS_NODEV |   |
| /dev | tmpfs | MS_NOEXEC,MS_STRICTATIME | mode=755 |
| /dev/shm | shm | MS_NOEXEC,MS_NOSUID,MS_NODEV | mode=1777,size=65536k |
| /dev/mqueue | mqueue | MS_NOEXEC,MS_NOSUID,MS_NODEV |   |
| /dev/pts | devpts | MS_NOEXEC,MS_NOSUID | newinstance,ptmxmode=0666,mode=620,gid5 |
| /sys | sysfs | MS_NOEXEC,MS_NOSUID,MS_NODEV,MS_RDONLY |   |

当容器的文件系统刚挂载完毕时，`/dev`文件系统会被一系列设备节点所填充，所以`rootfs`不应该管理`/dev`文件系统下的设备节点，Libcontainer 会负责处理并正确启动这些设备。设备及其权限模式如下。

| 路径 | 模式 | 权限 |
| /dev/null | 0666 | rwm |
| /dev/zero | 0666 | rwm |
| /dev/full | 0666 | rwm |
| /dev/tty  | 0666 | rwm |
| /dev/random | 0666 | rwm |
| /dev/urandom | 0666 | rwm |
| /dev/fuse | 0666 | rwm |

容器支持伪终端`TTY`，当用户使用时，就会建立`/dev/console`设备。其他终端支持设备，如`/dev/ptmx`则是宿主机的`/dev/ptmx` 链接。容器中指向宿主机 `/dev/null`的 IO 也会被重定向到容器内的 `/dev/null`设备。当`/proc`挂载完成后，`/dev/`中与 IO 相关的链接也会建立，如下表。

| 源地址 | 目的地址 |
| /proc/1/fd | /dev/fd |
| /proc/1/fd/0 | /dev/stdin |
| /proc/1/fd/1 | /dev/stdout |
| /proc/1/fd/2 | /dev/stderr |

 `pivot_root` 则用于改变进程的根目录，这样可以有效的将进程控制在我们建立的`rootfs`中。如果`rootfs`是基于`ramfs`的（不支持`pivot_root`），那么会在`mount`时使用`MS_MOVE`标志位加上`chroot`来顶替。

当文件系统创建完毕后，`umask`权限被重新设置回`0022`。

### 1.2 资源管理

在[《Docker 背后的内核知识：cgroups 资源隔离》](http://www.infoq.com/cn/articles/docker-kernel-knowledge-cgroups-resource-isolation)一文中已经提到，Docker 使用 cgroups 进行资源管理与限制，包括设备、内存、CPU、输入输出等。

目前除网络外所有内核支持的子系统都被加入到 Libcontainer 的管理中，所以 Libcontainer 使用 cgroups 原生支持的统计信息作为资源管理的监控展示。

容器中运行的第一个进程`init`，必须在初始化开始前放置到指定的 cgroup 目录中，这样就能防止初始化完成后运行的其他用户指令逃逸出 cgroups 的控制。父子进程的同步则通过管道来完成，在随后的运行时初始化中会进行展开描述。

### 1.3 可配置的容器安全

容器安全一直是被广泛探讨的话题，使用 namespace 对进程进行隔离是容器安全的基础，遗憾的是，usernamespace 由于设计上的复杂性，还没有被 Libcontainer 完全支持。

Libcontainer 目前可通过配置[`capabilities`](http://linux.die.net/man/7/capabilities)、[`selinux`](http://selinuxproject.org/page/Main_Page)、[`apparmor`](http://selinuxproject.org/page/Main_Page) 以及[`seccomp`](http://en.wikipedia.org/wiki/Seccomp)进行一定的安全防范，目前除`seccomp`以外都有一份[默认的配置项](https://github.com/docker/libcontainer/blob/master/SPEC.md#security)提供给用户作为参考。

在本系列的后续文章中，我们将对容器安全进行更深入的探讨，敬请期待。

### 1.4 运行时与初始化进程

在容器创建过程中，父进程需要与容器的`init`进程进行同步通信，通信的方式则通过向容器中传入管道来实现。当`init`启动时，他会等待管道内传入`EOF`信息，这就给父进程完成初始化，建立 uid/gid 映射，并把新进程放进新建的 cgroup 一定的时间。

在 Libcontainer 中运行的应用（进程），应该是事先静态编译完成的。Libcontainer 在容器中并不提供任何类似 Unix init 这样的守护进程，用户提供的参数也是通过`exec`系统调用提供给用户进程。通常情况下容器中也没有长进程存在。

如果容器打开了伪终端，就会通过`dup2`把 console 作为容器的输入输出（STDIN, STDOUT, STDERR）对象。

除此之外，以下四个文件也会在容器运行时自动生成。

*   /etc/hosts
*   /etc/resolv.conf
*   /etc/hostname
*   /etc/localtime

### 1.5 在运行着的容器中执行新进程

用户也可以在运行着的容器中执行一条新的指令，就是我们熟悉的`docker exec`功能。同样，执行指令的二进制文件需要包含在容器的`rootfs`之内。

通过这种方式运行起来的进程会随容器的状态变化，如容器被暂停，进程也随之暂停，恢复也随之恢复。当容器进程不存在时，进程就会被销毁，重启也不会恢复。

### 1.6 容器热迁移（Checkpoint & Restore）

目前 libcontainer 已经集成了[CRIU](http://criu.org/Main_Page)作为容器检查点保存与恢复（通常也称为热迁移）的解决方案，应该在不久之后就会被 Docker 使用。也就是说，通过 libcontainer 你已经可以把一个正在运行的进程状态保存到磁盘上，然后在本地或其他机器中重新恢复当前的运行状态。这个功能主要带来如下几个好处。

*   服务器需要维护（如系统升级、重启等）时，通过热迁移技术把容器转移到别的服务器继续运行，应用服务信息不会丢失。
*   对于初始化时间极长的应用程序来说，容器热迁移可以加快启动时间，当应用启动完成后就保存它的检查点状态，下次要重启时直接通过检查点启动即可。
*   在高性能计算的场景中，容器热迁移可以保证运行了许多天的计算结果不会丢失，只要周期性的进行检查点快照保存就可以了。

要使用这个功能，需要保证机器上已经安装了 1.5.2 或更高版本的`criu`工具。不同 Linux 发行版都有`criu`的安装包，你也可以在 CRIU 官网上找到从[源码安装](http://criu.org/Installation)的方法。我们将会在`nsinit`的使用中介绍容器热迁移的使用方法。

CRIU（Checkpoint/Restore In Userspace）由 OpenVZ 项目于 2005 年发起，因为其涉及的内核系统繁多、代码多达数万行，其复杂性与向后兼容性都阻碍着它进入内核主线，几经周折之后决定在用户空间实现，并在 2012 年被 Linus 加并入内核主线，其后得以快速发展。

你可以在 CRIU 官网查看[其原理](http://criu.org/Checkpoint/Restore)，简单描述起来可以分为两部分，一是检查点的保存，其中分为 3 步。

1.  收集进程与其子进程构成的树，并冻结所有进程。
2.  收集任务（包括进程和线程）使用的所有资源，并保存。
3.  清理我们收集资源的相关寄生代码，并与进程分离。

第二部分自然是恢复，分为 4 步。

1.  读取快照文件并解析出共享的资源，对多个进程共享的资源优先恢复，其他资源则随后需要时恢复。
2.  使用 fork 恢复整个进程树，注意此时并不恢复线程，在第 4 步恢复。
3.  恢复所有基础任务（包括进程和线程）资源，除了内存映射、计时器、证书和线程。这一步主要打开文件、准备 namespace、创建 socket 连接等。
4.  恢复进程运行的上下文环境，恢复剩下的其他资源，继续运行进程。

至此，libcontainer 的基本特性已经预览完毕，下面我们将从使用开始，一步步深入 libcontainer 的原理。

## 2\. `nsinit`与 Libcontainer 的使用

俗话说，了解一个工具最好的入门方式就是去使用它，`nsinit`就是一个为了方便不通过 Docker 就可以直接使用`libcontainer`而开发的命令行工具。它可以用于启动一个容器或者在已有的容器中执行命令。使用`nsinit`需要有 rootfs 以及相应的配置文件。

### 2.1 `nsinit`的构建

使用`nsinit`需要`rootfs`，最简单最常用的是使用[`Docker busybox`](https://github.com/jpetazzo/docker-busybox)，相关配置文件则可以参考[`sample_configs`](https://github.com/docker/libcontainer/tree/master/sample_configs)目录，主要配置的参数及其作用将在**配置参数**一节中介绍。拷贝一份命名为`container.json`文件到你`rootfs`所在目录中，这份文件就包含了你对容器做的特定配置，包括运行环境、网络以及不同的权限。这份配置对容器中的所有进程都会产生效果。

具体的构建步骤在官方的[`README`](https://github.com/docker/libcontainer/blob/master/nsinit/README.md)文档中已经给出，在此为了节省篇幅不再赘述。

最终编译完成后生成`nsinit`二进制文件，将这个指令加入到系统的环境变量，在 busybox 目录下执行如下命令，即可使用，需要**root**权限。
`  -- -- . `

执行完成后会生成一个以容器 ID 命名的文件夹，上述命令没有指定容器 ID，默认名为”nsinit”，在“nsinit”文件夹下会生成一个`state.json`文件，表示容器的状态，其中的内容与配置参数中的内容类似，展示容器的状态。

### 2.2 `nsinit`的使用

目前`nsinit`定义了 9 个指令，使用`nsinit -h`就可以看到，对于每个单独的指令使用`--help`就能获得更详细的使用参数，如`nsinit config --help`。

`nsinit`这个命令行工具是通过[`cli.go`](https://github.com/codegangsta/cli)实现的，`cli.go`封装了命令行工具需要做的一些细节，包括参数解析、命令执行函数构建等等，这就使得`nsinit`本身的代码非常简洁明了。具体的命令功能如下。

*   **config**：使用内置的默认参数加上执行命令时用户添加的部分参数，生成一份容器可用的标准配置文件。
*   **exec**：启动容器并执行命令。除了一些共有的参数外，还有如下一些独有的参数。
    *   **–tty,-t**：为容器分配一个终端显示输出内容。
    *   **–config**：使用配置文件，后跟文件路径。
    *   **–id**：指定容器 ID，默认为`nsinit`。
    *   **–user,-u**：指定用户，默认为“root”.
    *   **–cwd**：指定当前工作目录。
    *   **–env**：为进程设置环境变量。
*   **init**：这是一个内置的参数，用户并不能直接使用。这个命令是在容器内部执行，为容器进行 namespace 初始化，并在完成初始化后执行用户指令。所以在代码中，运行`nsinit exec`后，传入到容器中运行的实际上是`nsinit init`，把用户指令作为配置项传入。
*   **oom**：展示容器的内存超限通知。
*   **pause**/**unpause**：暂停 / 恢复容器中的进程。
*   **stats**：显示容器中的统计信息，主要包括 cgroup 和网络。
*   **state**：展示容器状态，就是读取`state.json`文件。
*   **checkpoint**：保存容器的检查点快照并结束容器进程。需要填`--image-path`参数，后面是检查点保存的快照文件路径。完整的命令示例如下。nsinit checkpoint --image-path =/tmp/criu
*   **restore**：从容器检查点快照恢复容器进程的运行。参数同上。

总结起来，`nsinit`与 Docker execdriver 进行的工作基本相同，所以在 Docker 的源码中并不会涉及到`nsinit`包的调用，但是`nsinit`为 Libcontainer 自身的调试和使用带来了极大的便利。

## 3\. 配置参数解析

*   `no_pivot_root` ：这个参数表示用`rootfs`作为文件系统挂载点，不单独设置`pivot_root`。
*   `parent_death_signal`： 这个参数表示当容器父进程销毁时发送给容器进程的信号。
*   `pivot_dir`：在容器`root`目录中指定一个目录作为容器文件系统挂载点目录。
*   `rootfs`：容器根目录位置。
*   `readonlyfs`：设定容器根目录为只读。
*   `mounts`：设定额外的挂载，填充的信息包括原路径，容器内目的路径，文件系统类型，挂载标识位，挂载的数据大小和权限，最后设定共享挂载还是非共享挂载（独立于`mount_label`的设定起作用）。
*   `devices`：设定在容器启动时要创建的设备，填充的信息包括设备类型、容器内设备路径、设备块号（major，minor）、cgroup 文件权限、用户编号、用户组编号。
*   `mount_label`：设定共享挂载还是非共享挂载。
*   `hostname`：设定主机名。
*   `namespaces`：设定要加入的 namespace，每个不同种类的 namespace 都可以指定，默认与父进程在同一个 namespace 中。
*   `capabilities`：设定在容器内的进程拥有的`capabilities`权限，所有没加入此配置项的`capabilities`会被移除，即容器内进程失去该权限。
*   `networks`：初始化容器的网络配置，包括类型（loopback、veth）、名称、网桥、物理地址、IPV4 地址及网关、IPV6 地址及网关、Mtu 大小、传输缓冲长度`txqueuelen`、Hairpin Mode 设置以及宿主机设备名称。
*   `routes`：配置路由表。
*   `cgroups`：配置 cgroups 资源限制参数，使用的参数不多，主要包括允许的设备列表、内存、交换区用量、CPU 用量、块设备访问优先级、应用启停等。
*   `apparmor_profile`：配置用于 selinux 的 apparmor 文件。
*   `process_label`：同样用于 selinux 的配置。
*   `rlimits`：最大文件打开数量，默认与父进程相同。
*   `additional_groups`：设定`gid`，添加同一用户下的其他组。
*   `uid_mappings`：用于 User namespace 的 uid 映射。
*   `gid_mappings`：用户 User namespace 的 gid 映射。
*   `readonly_paths`：在容器内设定只读部分的文件路径。
*   `MaskPaths`：配置不使用的设备，通过绑定`/dev/null`进行路径掩盖。

## 4\. Libcontainer 实现原理

在 Docker 中，对容器管理的模块为`execdriver`，目前 Docker 支持的容器管理方式有两种，一种就是最初支持的 LXC 方式，另一种称为`native`，即使用 Libcontainer 进行容器管理。在孙宏亮的[《Docker 源码分析系列》](http://www.infoq.com/cn/articles/docker-source-code-analysis-part3)中，Docker Deamon 启动过程中就会对 execdriver 进行初始化，会根据驱动的名称选择使用的容器管理方式。

虽然在`execdriver`中只有 LXC 和 native 两种选择，但是 native（即`Libcontainer`）通过接口的方式定义了一系列容器管理的操作，包括处理容器的创建（Factory）、容器生命周期管理（Container）、进程生命周期管理（Process）等一系列接口，相信如果 Docker 的热潮一直像如今这般汹涌，那么不久的将来，Docker 必将实现其全平台通用的宏伟蓝图。本节也将从 Libcontainer 的这些抽象对象开始讲解，与你一同解开 Docker 容器管理之谜。在介绍抽象对象的具体实现过程中会与 Docker execdriver 联系起来，让你充分了解整个过程。

### 4.1 Factory 对象

Factory 对象为容器创建和初始化工作提供了一组抽象接口，目前已经具体实现的是 Linux 系统上的 Factory 对象。Factory 抽象对象包含如下四个方法，我们将主要描述这四个方法的工作过程，涉及到具体实现方法则以 LinuxFactory 为例进行讲解。

1.  **Create()**：通过一个`id`和一份配置参数创建容器，返回一个运行的进程。容器的`id`由字母、数字和下划线构成，长度范围为 1~1024。容器 ID 为每个容器独有，不能冲突。创建的最终返回一个 Container 类，包含这个`id`、状态目录（在 root 目录下创建的以`id`命名的文件夹，存`state.json`容器状态文件）、容器配置参数、初始化路径和参数，以及管理 cgroup 的方式（包含直接通过文件操作管理和 systemd 管理两个选择，默认选 cgroup 文件系统管理）。
2.  **Load()**：当创建的`id`已经存在时，即已经`Create`过，存在`id`文件目录，就会从`id`目录下直接读取`state.json`来载入容器。其中的参数在配置参数部分有详细解释。
3.  **Type()**：返回容器管理的类型，目前可能返回的有 libcontainer 和 lxc，为未来支持更多容器接口做准备。
4.  **StartInitialization()**：容器内初始化函数。
    *   这部分代码是在容器内部执行的，当容器创建时，如果`New`不加任何参数，默认在容器进程中运行的第一条命令就是`nsinit init`。在`execdriver`的初始化中，会向`reexec`注册初始化器，命名为`native`，然后在创建 Libcontainer 以后把`native`作为执行参数传递到容器中执行，这个初始化器创建的 Libcontainer 就是没有参数的。
    *   传入的参数是一个管道文件描述符，为了保证在初始化过程中，父子进程间状态同步和配置信息传递而建立。
    *   不管是纯粹新建的容器还是已经创建的容器执行新的命令，都是从这个入口做初始化。
    *   第一步，通过管道获取配置信息。
    *   第二步，从配置信息中获取环境变量并设置为容器内环境变量。
    *   若是已经存在的容器执行新命令，则只需要配置 cgroup、namespace 的 Capabilities 以及 AppArmor 等信息，最后执行命令。
    *   若是纯粹新建的容器，则还需要初始化网络、路由、namespace、主机名、配置只读路径等等，最后执行命令。

至此，容器就已经创建和初始化完毕了。

### 4.2 Container 对象

Container 对象主要包含了容器配置、控制、状态显示等功能，是对不同平台容器功能的抽象。目前已经具体实现的是 Linux 平台下的 Container 对象。每一个 Container 进程内部都是线程安全的。因为 Container 有可能被外部的进程销毁，所以每个方法都会对容器是否存在进行检测。

1.  **ID()**：显示 Container 的 ID，在 Factor 对象中已经说过，ID 很重要，具有唯一性。
2.  **Status()**：返回容器内进程是运行状态还是停止状态。通过执行“SIG=0”的 KILL 命令对进程是否存在进行检测。
3.  **State()**：返回容器的状态，包括容器 ID、配置信息、初始进程 ID、进程启动时间、cgroup 文件路径、namespace 路径。通过调用`Status()`判断进程是否存在。
4.  **Config()**：返回容器的配置信息，可在“配置参数解析”部分查看有哪些方面的配置信息。
5.  **Processes()**：返回 cgroup 文件`cgroup.procs`中的值，在[Docker 背后的内核知识：cgroups 资源限制](http://www.infoq.com/cn/articles/docker-kernel-knowledge-cgroups-resource-isolation)部分的讲解中我们已经提过，`cgroup.procs`文件会罗列所有在该 cgroup 中的线程组 ID（即若有线程创建了子线程，则子线程的 PID 不包含在内）。由于容器不断在运行，所以返回的结果并不能保证完全存活，除非容器处于“PAUSED”状态。
6.  **Stats()**：返回容器的统计信息，包括容器的 cgroups 中的统计以及网卡设备的统计信息。Cgroups 中主要统计了 cpu、memory 和 blkio 这三个子系统的统计内容，具体了解可以通过阅读[“cgroups 资源限制”](http://www.infoq.com/cn/articles/docker-kernel-knowledge-cgroups-resource-isolation)部分对于这三个子系统统计内容的介绍来了解。网卡设备的统计则通过读取系统中，网络网卡文件的统计信息文件`/sys/class/net/<EthInterface>/statistics`来实现。
7.  **Set()**：设置容器 cgroup 各子系统的文件路径。因为 cgroups 的配置是进程运行时也会生效的，所以我们可以通过这个方法在容器运行时改变 cgroups 文件从而改变资源分配。
8.  **Start()**：构建 ParentProcess 对象，用于处理启动容器进程的所有初始化工作，并作为父进程与新创建的子进程（容器）进行初始化通信。传入的 Process 对象可以帮助我们追踪进程的生命周期，Process 对象将在后文详细介绍。
    *   启动的过程首先会调用`Status()`方法的具体实现得知进程是否存活。
    *   创建一个**管道**（详见 Docker 初始化通信——管道）为后期父子进程通信做准备。
    *   配置子进程`cmd`命令模板，配置参数的值就是从`factory.Create()`传入进来的，包括命令执行的工作目录、命令参数、输入输出、根目录、子进程管道以及`KILL`信号的值。
    *   根据容器进程是否存在确定是在已有容器中执行命令还是创建新的容器执行命令。若存在，则把配置的命令构建成一个`exec.Cmd`对象、cgroup 路径、父子进程管道及配置保留到 ParentProcess 对象中；若不存在，则创建容器进程及相应 namespace，目前对 user namespace 有了一定的支持，若配置时加入 user namespace，会针对配置项进行映射，默认映射到宿主机的 root 用户，最后同样构建出相应的配置内容保留到 ParentProcess 对象中。通过在`cmd.Env`写入环境变量`_LIBCONTAINER_INITTYPE`来告诉容器进程采用的哪种方式启动。
    *   执行 ParentProcess 中构建的`exec.Cmd`内容，即执行`ParentProcess.start()`，具体的执行过程在 Process 部分介绍。
    *   最后如果是新建的容器进程，还会执行状态更新函数，把`state.json`的内容刷新。
9.  **Destroy()**：首先使用 cgroup 的 freezer 子系统暂停所有运行的进程，然后给所有进程发送`SIGKIL`信号（如果没有使用`pid namespace`就不对进程处理）。最后把 cgroup 及其子系统卸载，删除 cgroup 文件夹。
10.  **Pause()**：使用 cgroup 的 freezer 子系统暂停所有运行的进程。
11.  **Resume()**：使用 cgroup 的 freezer 子系统恢复所有运行的进程。
12.  **NotifyOOM()**：为容器内存使用超界提供只读的通道，通过向`cgroup.event_control`写入`eventfd`（用作线程间通信的消息队列）和`cgroup.oom_control`（用于决定内存使用超限后的处理方式）来实现。
13.  **Checkpoint()**：保存容器进程检查点快照，为容器热迁移做准备。通过使用 CRIU 的[SWRK 模式](http://lists.openvz.org/pipermail/criu/2015-March/019400.html)来实现，这种模式是 CRIU 另外两种模式 CLI 和 RPC 的结合体，允许用户需要的时候像使用命令行工具一样运行 CRIU，并接受用户远程调用的请求，即传入的热迁移检查点保存请求，传入文件形式以 Google 的 protobuf 协议保存。
14.  **Restore()**：恢复检查点快照并运行，完成容器热迁移。同样通过 CRIU 的 SWRK 模式实现，恢复的时候可以传入配置文件设置恢复挂载点、网络等配置信息。

至此，Container 对象中的所有函数及相关功能都已经介绍完毕，包含了容器生命周期的全部过程。

#### TIPs： Docker 初始化通信——管道

Libcontainer 创建容器进程时需要做初始化工作，此时就涉及到使用了 namespace 隔离后的两个进程间的通信。我们把负责创建容器的进程称为父进程，容器进程称为子进程。父进程`clone`出子进程以后，依旧是共享内存的。但是如何让子进程知道内存中写入了新数据依旧是一个问题，一般有四种方法。

*   发送信号通知（signal）
*   对内存轮询访问（poll memory）
*   sockets 通信（sockets）
*   文件和文件描述符（files and file-descriptors）

对于 Signal 而言，本身包含的信息有限，需要额外记录，namespace 带来的上下文变化使其不易理解，并不是最佳选择。显然通过轮询内存的方式来沟通是一个非常低效的做法。另外，因为 Docker 会加入 network namespace，实际上初始时网络栈也是完全隔离的，所以 socket 方式并不可行。

Docker 最终选择的方式就是打开的可读可写文件描述符——管道。

Linux 中，通过`pipe(int fd[2])`系统调用就可以创建管道，参数是一个包含两个整型的数组。调用完成后，在`fd[1]`端写入的数据，就可以从`fd[0]`端读取。
```c
#include <unistd.h>

int fd[2];

pipe(fd);

close(checkpoint[1]);
```

调用`pipe`函数后，创建的子进程会内嵌这个打开的文件描述符，对`fd[1]`写入数据后可以在`fd[0]`端读取。通过管道，父子进程之间就可以通信。通信完毕的奥秘就在于`EOF`信号的传递。大家都知道，当打开的文件描述符都关闭时，才能读到`EOF`信号，所以`libcontainer`中父进程先关闭自己这一端的管道，然后等待子进程关闭另一端的管道文件描述符，传来`EOF`表示子进程已经完成了初始化的过程。

### 4.3 Process 对象

Process 主要分为两类，一类在源码中就叫`Process`，用于容器内进程的配置和 IO 的管理；另一类在源码中叫`ParentProcess`，负责处理容器启动工作，与 Container 对象直接进行接触，启动完成后作为`Process`的一部分，执行等待、发信号、获得`pid`等管理工作。

**ParentProcess 对象**，主要包含以下六个函数，而根据”需要新建容器”和“在已经存在的容器中执行”的不同方式，具体的实现也有所不同。

*   **已有容器中执行命令**

    1.  **pid()**： 启动容器进程后通过管道从容器进程中获得，因为容器已经存在，与Ｄocker Ｄeamon 在不同的 pid namespace 中，从进程所在的 namespace 获得的进程号才有意义。
    2.  **start()**： 初始化容器中的执行进程。在已有容器中执行命令一般由`docker exec`调用，在 execdriver 包中，执行`exec`时会引入`nsenter`包，从而调用其中的 C 语言代码，执行`nsexec()`函数，该函数会读取配置文件，使用`setns()`加入到相应的 namespace，然后通过`clone()`在该 namespace 中生成一个子进程，并把子进程通过管道传递出去，使用`setns()`以后并没有进入 pid namespace，所以还需要通过加上`clone()`系统调用。
        *   开始执行进程，首先会运行`C`代码，通过管道获得进程 pid，最后等待`C`代码执行完毕。
        *   通过获得的 pid 把 cmd 中的 Process 替换成新生成的子进程。
        *   把子进程加入 cgroup 中。
        *   通过管道传配置文件给子进程。
        *   等待初始化完成或出错返回，结束。
*   **新建容器执行命令**

    1.  **pid()**：启动容器进程后通过`exec.Cmd`自带的`pid()`函数即可获得。
    2.  **start()**：初始化及执行容器命令。
        *   开始运行进程。
        *   把进程 pid 加入到 cgroup 中管理。
        *   初始化容器网络。（本部分内容丰富，将从本系列的后续文章中深入讲解）
        *   通过管道发送配置文件给子进程。
        *   等待初始化完成或出错返回，结束。
*   **实现方式类似的一些函数**

    *   **terminate()** ：发送`SIGKILL`信号结束进程。
    *   **startTime()** ：获取进程的启动时间。
    *   **signal()**：发送信号给进程。
    *   **wait()**：等待程序执行结束，返回结束的程序状态。

**Process 对象**，主要描述了容器内进程的配置以及 IO。包括参数`Args`，环境变量`Env`，用户`User`（由于 uid、gid 映射），工作目录`Cwd`，标准输入输出及错误输入，控制终端路径`consolePath`，容器权限`Capabilities`以及上述提到的 ParentProcess 对象`ops`（拥有上面的一些操作函数，可以直接管理进程）。

## 5\. 总结

本文主要介绍了 Docker 容器管理的方式 Libcontainer，从 Libcontainer 的使用到源码实现方式。我们深入到容器进程内部，感受到了 Libcontainer 较为全面的设计。总体而言，Libcontainer 本身主要分为三大块工作内容，一是容器的创建及初始化，二是容器生命周期管理，三则是进程管理，调用方为 Docker 的`execdriver`。容器的监控主要通过 cgroups 的状态统计信息，未来会加入进程追踪等更丰富的功能。另一方面，Libcontainer 在安全支持方面也为用户尽可能多的提供了支持和选择。遗憾的是，容器安全的配置需要用户对系统安全本身有足够高的理解，user namespace 也尚未支持，可见 Libcontainer 依旧有很多工作要完善。但是 Docker 社区的火热也自然带动了大家对 Libcontainer 的关注，相信在不久的将来，Libcontainer 就会变得更安全、更易用。

## 6\. 作者简介

孙健波，[浙江大学 SEL 实验室](http://www.sel.zju.edu.cn/)硕士研究生，目前在云平台团队从事科研和开发工作。浙大团队对 PaaS、Docker、大数据和主流开源云计算技术有深入的研究和二次开发经验，团队现将部分技术文章贡献出来，希望能对读者有所帮助。

## 原文链接

[Docker 背后的容器管理——Libcontainer 深度解析](https://infoq.cn/article/docker-container-management-libcontainer-depth-analysis)

* * *

感谢[郭蕾](http://www.infoq.com/cn/author/%E9%83%AD%E8%95%BE)对本文的策划和审校。

给 InfoQ 中文站投稿或者参与内容翻译工作，请邮件至[editors@cn.infoq.com](mailto:editors@cn.infoq.com)。也欢迎大家通过新浪微博（[@InfoQ](http://www.weibo.com/infoqchina)，[@丁晓昀](http://weibo.com/u/1451714913)），微信（微信号：[InfoQChina](http://weixin.sogou.com/gzh?openid=oIWsFt0HnZ93MfLi3pW2ggVJFRxY)）关注我们，并与我们的编辑和其他读者朋友交流（欢迎加入 InfoQ 读者交流群[![](https://static001.infoq.cn/resource/image/06/9f/06e1fec4a87eca3142d54d09844c629f.png)](http://shang.qq.com/wpa/qunwpa?idkey=cc82a73d7522f0090aa3cbb6a8f4bdafa8b82177f481014c976a8740d927997a)）。

 文章版权归极客邦科技 InfoQ 所有，未经许可不得转载。
