[TOC]

### 什么是 Init 系统,init 系统的历史和现状

Linux 操作系统的启动首先从 BIOS 开始，接下来进入 bootloader，由 bootloader 载入内核，进行内核初始化。内核初始化的最后一步就是启动 pid 为 1 的 init 进程。这个进程是系统的第一个进程。它负责产生其他所有用户进程。

init 以守护进程方式存在，是所有其他进程的祖先。init 进程非常独特，能够完成其他进程无法完成的任务。

Init 系统能够定义、管理和控制 init 进程的行为。它负责组织和运行许多独立的或相关的始化工作(因此被称为 init 系统)，从而让计算机系统进入某种用户预订的运行模式。

仅仅将内核运行起来是毫无实际用途的，必须由 init 系统将系统代入可操作状态。比如启动外壳 shell 后，便有了人机交互，这样就可以让计算机执行一些预订程序完成有实际意义的任务。或者启动 X 图形系统以便提供更佳的人机界面，更加高效的完成任务。这里，字符界面的 shell 或者 X 系统都是一种预设的运行模式。

大多数 Linux 发行版的 init 系统是和 System V 相兼容的，被称为 sysvinit。这是人们最熟悉的 init 系统。一些发行版如 Slackware 采用的是 BSD 风格 Init 系统，这种风格使用较少，本文不再涉及。其他的发行版如 Gentoo 是自己定制的。Ubuntu 和 RHEL 采用 upstart 替代了传统的 sysvinit。而 Fedora 从版本 15 开始使用了一个被称为 systemd 的新 init 系统。

可以看到不同的发行版采用了不同的 init 实现，本系列文章就是打算讲述三个主要的 Init 系统：sysvinit，UpStart 和 systemd。了解它们各自的设计特点，并简要介绍它们的使用。

在 Linux 主要应用于服务器和 PC 机的时代，SysVinit 运行非常良好，概念简单清晰。它主要依赖于 Shell 脚本，这就决定了它的**最大弱点：启动太慢**。在很少重新启动的 Server 上，这个缺点并不重要。而当 Linux 被应用到移动终端设备的时候，启动慢就成了一个大问题。为了更快地启动，人们开始改进 sysvinit，先后出现了 upstart 和 systemd 这两个主要的新一代 init 系统。Upstart 已经开发了 8 年多，在不少系统中已经替换 sysvinit。Systemd 出现较晚，但发展更快，大有取代 upstart 的趋势。

### Sysvinit 概况

Sysvinit 就是 system V 风格的 init 系统，顾名思义，它源于 System V 系列 UNIX。它提供了比 BSD 风格 init 系统更高的灵活性。是已经风行了几十年的 UNIX init 系统，一直被各类 Linux 发行版所采用。

### 运行级别

Sysvinit 用术语 runlevel 来定义"预订的运行模式"。Sysvinit 检查 '/etc/inittab' 文件中是否含有 'initdefault' 项。 这告诉 init 系统是否有一个默认运行模式。如果没有默认的运行模式，那么用户将进入系统控制台，手动决定进入何种运行模式。

sysvinit 中运行模式描述了系统各种预订的运行模式。通常会有 8 种运行模式，即运行模式 0 到 6 和 S 或者 s。

每种 Linux 发行版对运行模式的定义都不太一样。但 0，1，6 却得到了大家的一致赞同：

- 0 关机
- 1 单用户模式
- 6 重启

通常在 /etc/inittab 文件中定义了各种运行模式的工作范围。比如 RedHat 定义了 runlevel 3 和 5。运行模式 3 将系统初始化为字符界面的 shell 模式；运行模式 5 将系统初始化为 GUI 模式。无论是命令行界面还是 GUI，运行模式 3 和 5 相对于其他运行模式而言都是完整的正式的运行状态，计算机可以完成用户需要的任务。而模式 1，S 等往往用于系统故障之后的排错和恢复。

很显然，这些不同的运行模式下系统需要初始化运行的进程和需要进行的初始化准备都是不同的。比如运行模式 3 不需要启动 X 系统。用户只需要指定需要进入哪种模式，sysvinit 将负责执行所有该模式所必须的初始化工作。

### sysvinit 运行顺序

Sysvinit 巧妙地用脚本，文件命名规则和软链接来实现不同的 runlevel。首先，sysvinit 需要读取/etc/inittab 文件。分析这个文件的内容，它获得以下一些配置信息：

- 系统需要进入的 runlevel
- 捕获组合键的定义
- 定义电源 fail/restore 脚本
- 启动 getty 和虚拟控制台

得到配置信息后，sysvinit 顺序地执行以下这些步骤，从而将系统初始化为预订的 runlevel X。

- /etc/rc.d/rc.sysinit
- /etc/rc.d/rc 和/etc/rc.d/rcX.d/ (X 代表运行级别 0-6)
- /etc/rc.d/rc.local
- X Display Manager（如果需要的话）

首先，运行 rc.sysinit 以便执行一些重要的系统初始化任务。在 RedHat 公司的 RHEL5 中(RHEL6 已经使用 upstart 了)，rc.sysinit 主要完成以下这些工作。

- 激活 udev 和 selinux
- 设置定义在/etc/sysctl.conf 中的内核参数
- 设置系统时钟
- 加载 keymaps
- 使能交换分区
- 设置主机名(hostname)
- 根分区检查和 remount
- 激活 RAID 和 LVM 设备
- 开启磁盘配额
- 检查并挂载所有文件系统
- 清除过期的 locks 和 PID 文件

完成了以上这些工作之后，sysvinit 开始运行/etc/rc.d/rc 脚本。根据不同的 runlevel，rc 脚本将打开对应该 runlevel 的 rcX.d 目录(X 就是 runlevel)，找到并运行存放在该目录下的所有启动脚本。每个 runlevel X 都有一个这样的目录，目录名为/etc/rc.d/rcX.d。

在这些目录下存放着很多不同的脚本。文件名以 S 开头的脚本就是启动时应该运行的脚本，S 后面跟的数字定义了这些脚本的执行顺序。在/etc/rc.d/rcX.d 目录下的脚本其实都是一些软链接文件，真实的脚本文件存放在/etc/init.d 目录下。如下所示：
#### rc5.d 目录下的脚本

```
[root@www ~]# ll /etc/rc5.d/
lrwxrwxrwx 1 root root 16 Sep 4 2008 K02dhcdbd -> ../init.d/dhcdbd
....(中间省略)....
lrwxrwxrwx 1 root root 14 Sep 4 2008 K91capi -> ../init.d/capi
lrwxrwxrwx 1 root root 23 Sep 4 2008 S00microcode_ctl -> ../init.d/microcode_ctl
lrwxrwxrwx 1 root root 22 Sep 4 2008 S02lvm2-monitor -> ../init.d/lvm2-monitor
....(中间省略)....
lrwxrwxrwx 1 root root 17 Sep 4 2008 S10network -> ../init.d/network
....(中间省略)....
lrwxrwxrwx 1 root root 11 Sep 4 2008 S99local -> ../rc.local
lrwxrwxrwx 1 root root 16 Sep 4 2008 S99smartd -> ../init.d/smartd
....(底下省略)....
```
当所有的初始化脚本执行完毕。Sysvinit 运行/etc/rc.d/rc.local 脚本。

rc.local 是 Linux 留给用户进行个性化设置的地方。您可以把自己私人想设置和启动的东西放到这里，一台 Linux Server 的用户一般不止一个，所以才有这样的考虑。

### Sysvinit 和系统关闭

Sysvinit 不仅需要负责初始化系统，还需要负责关闭系统。在系统关闭时，为了保证数据的一致性，需要小心地按顺序进行结束和清理工作。

比如应该先停止对文件系统有读写操作的服务，然后再 umount 文件系统。否则数据就会丢失。

这种顺序的控制这也是依靠/etc/rc.d/rcX.d/目录下所有脚本的命名规则来控制的，在该目录下所有以 K 开头的脚本都将在关闭系统时调用，字母 K 之后的数字定义了它们的执行顺序。

这些脚本负责安全地停止服务或者其他的关闭工作。

### Sysvinit 的管理和控制功能

此外，在系统启动之后，管理员还需要对已经启动的进程进行管理和控制。原始的 sysvinit 软件包包含了一系列的控制启动，运行和关闭所有其他程序的工具。
** halt **
停止系统。
** init **
这个就是 sysvinit 本身的 init 进程实体，以 pid1 身份运行，是所有用户进程的父进程。最主要的作用是在启动过程中使用/etc/inittab 文件创建进程。
** killall5 **
就是 SystemV 的 killall 命令。向除自己的会话(session)进程之外的其它进程发出信号，所以不能杀死当前使用的 shell。
** last **
回溯/var/log/wtmp 文件(或者-f 选项指定的文件)，显示自从这个文件建立以来，所有用户的登录情况。
** lastb **
作用和 last 差不多，默认情况下使用/var/log/btmp 文件，显示所有失败登录企图。
** mesg **
控制其它用户对用户终端的访问。
** pidof **
找出程序的进程识别号(pid)，输出到标准输出设备。
** poweroff **
等于 shutdown -h –p，或者 telinit 0。关闭系统并切断电源。
** reboot **
等于 shutdown –r 或者 telinit 6。重启系统。
** runlevel **
读取系统的登录记录文件(一般是/var/run/utmp)把以前和当前的系统运行级输出到标准输出设备。
** shutdown **
以一种安全的方式终止系统，所有正在登录的用户都会收到系统将要终止通知，并且不准新的登录。
** sulogin **
当系统进入单用户模式时，被 init 调用。当接收到启动加载程序传递的-b 选项时，init 也会调用 sulogin。
** telinit **
实际是 init 的一个连接，用来向 init 传送单字符参数和信号。
** utmpdump **
以一种用户友好的格式向标准输出设备显示/var/run/utmp 文件的内容。
** wall **
向所有有信息权限的登录用户发送消息。

不同的 Linux 发行版在这些 sysvinit 的基本工具基础上又开发了一些辅助工具用来简化 init 系统的管理工作。比如 RedHat 的 RHEL 在 sysvinit 的基础上开发了 initscripts 软件包，包含了大量的启动脚本 (如 rc.sysinit) ，还提供了 service，chkconfig 等命令行工具，甚至一套图形化界面来管理 init 系统。其他的 Linux 发行版也有各自的 initscript 或其他名字的 init 软件包来简化 sysvinit 的管理。

只要您理解了 sysvinit 的机制，在一个最简的仅有 sysvinit 的系统下，您也可以直接调用脚本启动和停止服务，手动创建 inittab 和创建软连接来完成这些任务。因此理解 sysvinit 的基本原理和命令是最重要的。您甚至也可以开发自己的一套管理工具。

### Sysvinit 的小结

Sysvinit 的优点是概念简单。Service 开发人员只需要编写启动和停止脚本，概念非常清楚；将 service 添加/删除到某个 runlevel 时，只需要执行一些创建/删除软连接文件的基本操作；这些都不需要学习额外的知识或特殊的定义语法(UpStart 和 Systemd 都需要用户学习新的定义系统初始化行为的语言)。

其次，sysvinit 的另一个重要优点是确定的执行顺序：脚本严格按照启动数字的大小顺序执行，一个执行完毕再执行下一个，这非常有益于错误排查。UpStart 和 systemd 支持并发启动，导致没有人可以确定地了解具体的启动顺序，排错不易。

但是串行地执行脚本导致 sysvinit 运行效率较慢，在新的 IT 环境下，启动快慢成为一个重要问题。此外动态设备加载等 Linux 新特性也暴露出 sysvinit 设计的一些问题。针对这些问题，人们开始想办法改进 sysvinit，以便加快启动时间，并解决 sysvinit 自身的设计问题。

- - -

### Upstart 简介

假如您使用的 Linux 发行版是 Ubuntu，很可能会发现在您的计算机上找不到/etc/inittab 文件了，这是因为 Ubuntu 使用了一种被称为 upstart 的新型 init 系统。

### 开发 Upstart 的缘由

大约在 2006 年或者更早的时候， Ubuntu 开发人员试图将 Linux 安装在笔记本电脑上。在这期间技术人员发现经典的 sysvinit 存在一些问题：它不适合笔记本环境。这促使程序员 Scott James Remnant 着手开发 upstart。

当 Linux 内核进入 2.6 时代时，内核功能有了很多新的更新。新特性使得 Linux 不仅是一款优秀的服务器操作系统，也可以被用于桌面系统，甚至嵌入式设备。桌面系统或便携式设备的一个特点是经常重启，而且要频繁地使用硬件热插拔技术。 在现代计算机系统中，硬件繁多、接口有限，人们并非将所有设备都始终连接在计算机上，比如 U 盘平时并不连接电脑，使用时才插入 USB 插口。因此，当系统上电启动时，一些外设可能并没有连接。而是在启动后当需要的时候才连接这些设备。在 2.6 内核支持下，一旦新外设连接到系统，内核便可以自动实时地发现它们，并初始化这些设备，进而使用它们。这为便携式设备用户提供了很大的灵活性。

可是这些特性为 sysvinit 带来了一些挑战。当系统初始化时，需要被初始化的设备并没有连接到系统上；比如打印机。为了管理打印任务，系统需要启动 CUPS 等服务，而如果打印机没有接入系统的情况下，启动这些服务就是一种浪费。Sysvinit 没有办法处理这类需求，它必须一次性把所有可能用到的服务都启动起来，即使打印机并没有连接到系统，CUPS 服务也必须启动。

还有网络共 享盘的挂载问题。在/etc/fstab 中，可以指定系统自动挂载一个网络盘，比如 NFS，或者 iSCSI 设备。在本文的第一部分 sysvinit 的简介中可以看到，sysvinit 分析/etc/fstab 挂载文件系统这个步骤是在网络启动之前。可是如果网络没有启动，NFS 或者 iSCSI 都不可访问，当然也无法进行挂载操作。Sysvinit 采用 netdev 的方式来解决这个问题，即/etc/fstab 发现 netdev 属性挂载点的时候，不尝试挂载它，在网络初始化并使能之后，还有一个专门的 netfs 服务来挂载所有这些网络盘。这是一个不得已的补救方法，给管理员带来不便。部分新手管理员甚至从来也没有听说过 netdev 选项，因此经常成为系统管理的一个陷阱。

针对以上种种情况，Ubuntu 开发人员在评估了当时的几个可选 init 系统之后，决定重新设计和开发一个全新的 init 系统，即 UpStart。UpStart 基于事件机制，比如 U 盘插入 USB 接口后，udev 得到内核通知，发现该设备，这就是一个新的事件。UpStart 在感知到该事件之后触发相应的等待任务，比如处理/etc/fstab 中存在的挂载点。采用这种事件驱动的模式，upstart 完美地解决了即插即用设备带来的新问题。

此外，采用事件驱动机制也带来了一些其它有益的变化，比如加快了系统启动时间。sysvinit 运行时是同步阻塞的。一个脚本运行的时候，后续脚本必须等待。这意味着所有的初始化步骤都是串行执行的，而实际上很多服务彼此并不相关，完全可以并行启 动，从而减小系统的启动时间。在 Linux 大量应用于服务器的时代，系统启动时间也许还不那么重要；然而对于桌面系统和便携式设备，启动时间的长短对用户体验影响很大。此外云计算等新的 Server 端技术也往往需要单个设备可以更加快速地启动。

UpStart 满足了这些需求，目前不仅桌面系统 Ubuntu 采用了 UpStart，甚至企业级服务器级的 RHEL 也默认采用 UpStart 来替换 sysvinit 作为 init 系统。

### Upstart 的特点

UpStart 解决了之前提到的 sysvinit 的缺点。采用事件驱动模型，UpStart 可以：

- 更快地启动系统
- 当新硬件被发现时动态启动服务
- 硬件被拔除时动态停止服务

这些特点使得 UpStart 可以很好地应用在桌面或者便携式系统中，处理这些系统中的动态硬件插拔特性。

### Upstart 概念和术语

Upstart 的基本概念和设计清晰明确。UpStart 主要的概念是 job 和 event。Job 就是一个工作单元，用来完成一件工作，比如启动一个后台服务，或者运行一个配置命令。每个 Job 都等待一个或多个事件，一旦事件发生，upstart 就触发该 job 完成相应的工作。

#### Job

Job 就是一个工作的单元，一个任务或者一个服务。可以理解为 sysvinit 中的一个服务脚本。有三种类型的工作：

- task job；
- service job；
- abstract job；

task job 代表在一定时间内会执行完毕的任务，比如删除一个文件；

service job 代表后台服务进程，比如 apache httpd。这里进程一般不会退出，一旦开始运行就成为一个后台精灵进程，由 init 进程管理，如果这类进程退出，由 init 进程重新启动，它们只能由 init 进程发送信号停止。它们的停止一般也是由于所依赖的停止事件而触发的，不过 upstart 也提供命令行工具，让管理人员手动停止某个服务；

Abstract job 仅由 upstart 内部使用，仅对理解 upstart 内部机理有所帮助。我们不用关心它。

除了以上的分类之外，还有另一种工作（Job）分类方法。Upstart 不仅可以用来为整个系统的初始化服务，也可以为每个用户会话（session）的初始化服务。系统的初始化任务就叫做 system job，比如挂载文件系统的任务就是一个 system job；用户会话的初始化服务就叫做 session job。

#### Job 生命周期

Upstart 为每个工作都维护一个生命周期。一般来说，工作有开始，运行和结束这几种状态。为了更精细地描述工作的变化，Upstart 还引入了一些其它的状态。比如开始就有开始之前(pre-start)，即将开始(starting)和已经开始了(started)几种不同的状态，这 样可以更加精确地描述工作的当前状态。

工作从某种初始状态开始，逐渐变化，或许要经历其它几种不同的状态，最终进入另外一种状态，形成一个状态机。在这个过程中，当工作的状态即将发生变化的时候，init 进程会发出相应的事件（event）。

表 1.Upstart 中 Job 的可能状态

|	 状态名	 |		 含义		 |
|--------|--------|
|	Waiting		|	初始状态	|
|	Starting	|	Job 即将开始|
|	pre-start	|	执行 pre-start 段，即任务开始前应该完成的工作	|
|	Spawned		|	准备执行 script 或者 exec 段	|
|	post-start	|	执行 post-start 动作	|
|	Running		|	interim state set after post-start section processed denoting job is running (But it may have no associated PID!)	|
|	pre-stop	|	执行 pre-stop 段	|
|	Stopping	|	interim state set after pre-stop section processed	|
|	Killed		|	任务即将被停止	|
|	post-stop	|	执行 post-stop 段	|

下图展示了 Job 的状态机。

Job’s life cycle

![](http://www.ibm.com/developerworks/cn/linux/1407_liuming_init2/image003.jpg)

其中有四个状态会引起 init 进程发送相应的事件，表明该工作的相应变化：

- Starting

- Started

- Stopping

- Stopped

而其它的状态变化不会发出事件。那么我们接下来就来看看事件的详细含义吧。

#### 事件 Event

顾名思义，Event 就是一个事件。事件在 upstart 中以通知消息的形式具体存在。一旦某个事件发生了，Upstart 就向整个系统发送一个消息。没有任何手段阻止事件消息被 upstart 的其它部分知晓，也就是说，事件一旦发生，整个 upstart 系统中所有工作和其它的事件都会得到通知。

Event 可以分为三类: signal，methods 或者 hooks。

** Signals **
Signal 事件是非阻塞的，异步的。发送一个信号之后控制权立即返回。

** Methods **
Methods 事件是阻塞的，同步的。

** Hooks **
Hooks 事件是阻塞的，同步的。它介于 Signals 和 Methods 之间，调用发出 Hooks 事件的进程必须等待事件完成才可以得到控制权，但不检查事件是否成功。

事件是个非常抽象的概念，下面我罗列出一些常见的事件，希望可以帮助您进一步了解事件的含义：

- 系统上电启动，init 进程会发送"start"事件

- 根文件系统可写时，相应 job 会发送文件系统就绪的事件

- 一个块设备被发现并初始化完成，发送相应的事件

- 某个文件系统被挂载，发送相应的事件

- 类似 atd 和 cron，可以在某个时间点，或者周期的时间点发送事件

- 另外一个 job 开始或结束时，发送相应的事件

- 一个磁盘文件被修改时，可以发出相应的事件

- 一个网络设备被发现时，可以发出相应的事件

- 缺省路由被添加或删除时，可以发出相应的事件

不同的 Linux 发行版对 upstart 有不同的定制和实现，实现和支持的事件也有所不同，可以用man 7 upstart-events来查看事件列表。

#### Job 和 Event 的相互协作

Upstart 就是由事件触发工作运行的一个系统，每一个程序的运行都由其依赖的事件发生而触发的。

系统初始化的过程是在工作和事件的相互协作下完成的，可以大致描述如下：系统初始化时，init 进程开始运行，init 进程自身会发出不同的事件，这些最初的事件会触发一些工作运行。每个工作运行过程中会释放不同的事件，这些事件又将触发新的工作运行。如此反复，直到整个 系统正常运行起来。

究竟哪些事件会触发某个工作的运行？这是由工作配置文件定义的。

#### 工作配置文件

任何一个工作都是由一个工作配置文件（Job Configuration File）定义的。这个文件是一个文本文件，包含一个或者多个小节（stanza）。每个小节是一个完整的定义模块，定义了工作的一个方面，比如 author 小节定义了工作的作者。工作配置文件存放在/etc/init 下面，是以.conf 作为文件后缀的文件。

##### 一个最简单的工作配置文件
```
#This is a simple demo of Job Configure file
#This line is comment, start with #

#Stanza 1, The author
author “Liu Ming”

#Stanza 2, Description
description “This job only has author and description, so no use, just a demo”
```
上面的例子不会产生任何作用，一个真正的工作配置文件会包含很多小节，其中比较重要的小节有以下几个：

** "expect" Stanza **

Upstart 除了负责系统的启动过程之外，和 SysVinit 一样，Upstart 还提供一系列的管理工具。当系统启动之后，管理员可能还需要进行维护和调整，比如启动或者停止某项系统服务。或者将系统切换到其它的工作状态，比如改变运 行级别。本文后续将详细介绍 Upstart 的管理工具的使用。

为了启动，停止，重启和查询某个系统服务。Upstart 需要跟踪该服务所对应的进程。比如 httpd 服务的进程 PID 为 1000。当用户需要查询 httpd 服务是否正常运行时，Upstart 就可以利用 ps 命令查询进程 1000，假如它还在正常运行，则表明服务正常。当用户需要停止 httpd 服务时，Upstart 就使用 kill 命令终止该进程。为此，Upstart 必须跟踪服务进程的进程号。

部分服务进程为了将自己变成后台精灵进程(daemon)， 会采用两次派生(fork)的技术，另外一些服务则不会这样做。假如一个服务派生了两次，那么 UpStart 必须采用第二个派生出来的进程号作为服务的 PID。但是，UpStart 本身无法判断服务进程是否会派生两次，为此在定义该服务的工作配置文件中必须写明 expect 小节，告诉 UpStart 进程是否会派生两次。

Expect 有两种，"expect fork"表示进程只会 fork 一次；"expect daemonize"表示进程会 fork 两次。

** "exec" Stanza 和"script" Stanza **

一个 UpStart 工作一定需要做些什么，可能是运行一条 shell 命令，或者运行一段脚本。用"exec"关键字配置工作需要运行的命令；用"script"关键字定义需要运行的脚本。

##### 显示了 exec 和 script 的用法：

script 例子
```
# mountall.conf
description “Mount filesystems on boot”
start on startup
stop on starting rcS
...
script
  . /etc/default/rcS
  [ -f /forcefsck ] && force_fsck=”--force-fsck”
  [ “$FSCKFIX”=”yes” ] && fsck_fix=”--fsck-fix”
   
  ...
  
  exec mountall –daemon $force_fsck $fsck_fix
end script
...
```
这是 mountall 的例子，该工作在系统启动时运行，负责挂载所有的文件系统。该工作需要执行复杂的脚本，由"script"关键字定义；在脚本中，使用了 exec 来执行 mountall 命令。

** "start on" Stanza 和"stop on" Stanza **

"start on"定义了触发工作的所有事件。"start on"的语法很简单，如下所示：

`start on EVENT [[KEY=]VALUE]... [and|or...]`

EVENT 表示事件的名字，可以在 start on 中指定多个事件，表示该工作的开始需要依赖多个事件发生。多个事件之间可以用 and 或者 or 组合，"表示全部都必须发生"或者"其中之一发生即可"等不同的依赖条件。除了事件发生之外，工作的启动还可以依赖特定的条件，因此在 start on 的 EVENT 之后，可以用 KEY=VALUE 来表示额外的条件，一般是某个环境变量(KEY)和特定值(VALUE)进行比较。如果只有一个变量，或者变量的顺序已知，则 KEY 可以省略。

"stop on"和"start on"非常类似，只不过是定义工作在什么情况下需要停止。

#####  "start on"和"stop on"的一个例子。

start on/ stop on 例子
```
#dbus.conf
description     “D-Bus system message bus”

start on local-filesystems
stop on deconfiguring-networking
…
```
D-Bus 是一个系统消息服务，上面的配置文件表明当系统发出 local-filesystems 事件时启动 D-Bus；当系统发出 deconfiguring-networking 事件时，停止 D-Bus 服务。

#### Session Init

UpStart 还可以用于管理用户会话的初始化。在我写这篇文章的今天，多数 Linux 发行版还没有使用 UpStart 管理会话。只有在 Ubuntu Raring 版本中，使用 UpStart 管理用户会话的初始化过程。

首先让我们了解一下 Session 的概念。Session 就是一个用户会话，即用户从远程或者本地登入系统开始工作，直到用户退出。这整个过程就构成一个会话。

每个用户的使用习惯和使用方法都不相同，因此用户往往需要为自己的会话做一个定制，比如添加特定的命令别名，启动特殊的应用程序或者服务，等等。这些工作都属于对特定会话的初始化操作，因此可以被称为 Session Init。

用 户使用 Linux 可以有两种模式：字符模式和图形界面。在字符模式下，会话初始化相对简单。用户登录后只能启动一个 Shell，通过 shell 命令使用系统。各种 shell 程序都支持一个自动运行的启动脚本，比如~/.bashrc。用户在这些脚本中加入需要运行的定制化命令。字符会话需求简单，因此这种现有的机制工作的很 好。

在图形界面下，事情就变得复杂一些。用户登录后看到的并不是一个 shell 提示符，而是一个桌面。一个完整的桌面环境由很多组件组成。

一 个桌面环境包括 window manager，panel 以及其它一些定义在/usr/share/gnome-session/sessions/下面的基本组件；此外还有一些辅助的应用程序，共同帮助构成一 个完整的方便的桌面，比如 system monitors，panel applets，NetworkManager，Bluetooth，printers 等。当用户登录之后，这些组件都需要被初始化，这个过程比字符界面要复杂的多。目前启动各种图形组件和应用的工作由 gnome-session 完成。过程如下：

以 Ubuntu 为例，当用户登录 Ubuntu 图形界面后，显示管理器(Display Manager)lightDM 启动 Xsession。Xsession 接着启动 gnome-session，gnome-session 负责其它的初始化工作，然后就开始了一个 desktop session。

传统 desktop session 启动过程
```
init
 |- lightdm
 |   |- Xorg
 |   |- lightdm ---session-child
 |        |- gnome-session --session=ubuntu
 |             |- compiz
 |             |- gwibber
 |             |- nautilus
 |             |- nm-applet
 |             :
 |             :
 |
 |- dbus-daemon --session
 |
 :
 :
```

这个过程有一些缺点（和 sysVInit 类似）。一些应用和组件其实并不需要在会话初始化过程中启动，更好的选择是在需要它们的时候才启动。比如 update-notifier 服务，该服务不停地监测几个文件系统路径，一旦这些路径上发现可以更新的软件包，就提醒用户。这些文件系统路径包括新插入的 DVD 盘等。Update-notifier 由 gnome-session 启动并一直运行着，在多数情况下，用户并不会插入新的 DVD，此时 update-notifier 服务一直在后台运行并消耗系统资源。更好的模式是当用户插入 DVD 的时候再运行 update-notifier。这样可以加快启动时间，减小系统运行过程中的内存等系统资源的开销。对于移动，嵌入式等设备等这还意味着省电。除了 Update-notifier 服务之外，还有其它一些类似的服务。比如 Network Manager，一天之内用户很少切换网络设备，所以大部分时间 Network Manager 服务仅仅是在浪费系统资源；再比如 backup manager 等其它常驻内存，后台不间断运行却很少真正被使用的服务。

用 UpStart 的基于事件的按需启动的模式就可以很好地解决这些问题，比如用户插入网线的时候才启动 Network Manager，因为用户插入网线表明需要使用网络，这可以被称为按需启动。

下图描述了采用 UpStart 之后的会话初始化过程。

采用 Upstart 的 Desktop session init 过程
```
init
 |- lightdm
 |   |- Xorg
 |   |- lightdm ---session-child
 |        |- session-init # <-- upstart running as normal user
 |             |- dbus-daemon --session
 |             |- gnome-session --session=ubuntu
 |             |- compiz
 |             |- gwibber
 |             |- nautilus
 |             |- nm-applet
 |             :
 |             :
 :
 :
 ```
### UpStart 使用

有两种人员需要了解 Upstart 的使用。第一类是系统开发人员，比如 MySQL 的开发人员。它们需要了解如何编写工作配置文件，以便用 UpStart 来管理服务。比如启动，停止 MySQL 服务。

另外一种情况是系统管理员，它们需要掌握 Upstart 的管理命令以便配置和管理系统的初始化，管理系统服务。

#### 系统开发人员需要了解的 UpStart 知识

系统开发人员不仅需要掌握工作配置文件的写法，还需要了解一些针对服务进程编程上的要求。本文仅列出了少数工作配置文件的语法。要全面掌握工作配置文件的写 法，需要详细阅读 Upstart 的手册。这里让我们来分析一下如何用 Upstart 来实现传统的运行级别，进而了解如何灵活使用工作配置文件。

** Upstart 系统中的运行级别 **

Upstart 的运作完全是基于工作和事件的。工作的状态变化和运行会引起事件，进而触发其它工作和事件。

而传统的 Linux 系统初始化是基于运行级别的，即 SysVInit。因为历史的原因，Linux 上的多数软件还是采用传统的 SysVInit 脚本启动方式，并没有为 UpStart 开发新的启动脚本，因此即便在 Debian 和 Ubuntu 系统上，还是必须模拟老的 SysVInit 的运行级别模式，以便和多数现有软件兼容。

虽然 Upstart 本身并没有运行级别的概念，但完全可以用 UpStart 的工作模拟出来。让我们完整地考察一下 UpStart 机制下的系统启动过程。

** 系统启动过程 **

下图描述了 UpStart 的启动过程。

UpStart 启动过程

![](http://www.ibm.com/developerworks/cn/linux/1407_liuming_init2/image004.png)


系统上电后运行 GRUB 载入内核。内核执行硬件初始化和内核自身初始化。在内核初始化的最后，内核将启动 pid 为 1 的 init 进程，即 UpStart 进程。

Upstart 进程在执行了一些自身的初始化工作后，立即发出"startup"事件。上图中用红色方框加红色箭头表示事件，可以在左上方看到"startup"事件。

所有依赖于"startup"事件的工作被触发，其中最重要的是 mountall。mountall 任务负责挂载系统中需要使用的文件系统，完成相应工作后，mountall 任务会发出以下事件：local-filesystem，virtual-filesystem，all-swaps，

其中 virtual-filesystem 事件触发 udev 任务开始工作。任务 udev 触发 upstart-udev-bridge 的工作。Upstart-udev-bridge 会发出 net-device-up IFACE=lo 事件，表示本地回环 IP 网络已经准备就绪。同时，任务 mountall 继续执行，最终会发出 filesystem 事件。

此时，任务 rc-sysinit 会被触发，因为 rc-sysinit 的 start on 条件如下：

`start on filesystem and net-device-up IFACE=lo`
任务 rc-sysinit 调用 telinit。Telinit 任务会发出 runlevel 事件，触发执行/etc/init/rc.conf。

rc.conf 执行/etc/rc$.d/目录下的所有脚本，和 SysVInit 非常类似，读者可以参考本文第一部分的描述。

** 程序开发时需要注意的事项 **

作为程序开发人员，在编写系统服务时，需要了解 UpStart 的一些特殊要求。只有符合这些要求的软件才可以被 UpStart 管理。

** 规则一，派生次数需声明。 **

很多 Linux 后台服务都通过派生两次的技巧将自己变成后台服务程序。如果您编写的服务也采用了这个技术，就必须通过文档或其它的某种方式明确地让 UpStart 的维护人员知道这一点，这将影响 UpStart 的 expect stanza，我们在前面已经详细介绍过这个 stanza 的含义。

** 规则二，派生后即可用。**

后台程序在完成第二次派生的时候，必须保证服务已经可用。因为 UpStart 通过派生计数来决定服务是否处于就绪状态。

** 规则三，遵守 SIGHUP 的要求。 **

UpStart 会给精灵进程发送 SIGHUP 信号，此时，UpStart 希望该精灵进程做以下这些响应工作：

- 完成所有必要的重新初始化工作，比如重新读取配置文件。这是因为 UpStart 的命令"initctl reload"被设计为可以让服务在不重启的情况下更新配置。

- 精灵进程必须继续使用现有的 PID，即收到 SIGHUP 时不能调用 fork。如果服务必须在这里调用 fork，则等同于派生两次，参考上面的规则一的处理。这个规则保证了 UpStart 可以继续使用 PID 管理本服务。

** 规则四，收到 SIGTEM 即 shutdown。 **

- 当收到 SIGTERM 信号后，UpStart 希望精灵进程进程立即干净地退出，释放所有资源。如果一个进程在收到 SIGTERM 信号后不退出，Upstart 将对其发送 SIGKILL 信号。

**系统管理员需要了解的 Upstart 命令 **

作为系统管理员，一个重要的职责就是管理系统服务。比如系统服务的监控，启动，停止和配置。UpStart 提供了一系列的命令来完成这些工作。其中的核心是initctl，这是一个带子命令风格的命令行工具。

比如可以用 initctl list 来查看所有工作的概况：
```
$initctl list
alsa-mixer-save stop/waiting
avahi-daemon start/running, process 690
mountall-net stop/waiting
rc stop/waiting
rsyslog start/running, process 482
screen-cleanup stop/waiting
tty4 start/running, process 859
udev start/running, process 334
upstart-udev-bridge start/running, process 304
ureadahead-other stop/waiting
```
这是在 Ubuntu10.10 系统上的输出，其它的 Linux 发行版上的输出会有所不同。第一列是工作名，比如 rsyslog。第二列是工作的目标；第三列是工作的状态。

此外还可以用 initctl stop 停止一个正在运行的工作；用 initctl start 开始一个工作；还可以用 initctl status 来查看一个工作的状态；initctl restart 重启一个工作；initctl reload 可以让一个正在运行的服务重新载入配置文件。这些命令和传统的 service 命令十分相似。

service 命令和 initctl 命令对照表

|Service 命令	|UpStart initctl 命令|
|------------|---------------|
|service start	|initctl start|
|service stop	|initctl stop|
|service restart	|initctl restart|
|service reload	|initctl reload|

很多情况下管理员并不喜欢子命令风格，因为需要手动键入的字符太多。UpStart 还提供了一些快捷命令来简化 initctl，实际上这些命令只是在内部调用相应的 initctl 命令。比如 reload，restart，start，stop 等等。启动一个服务可以简单地调用
`start <job>`
这和执行 `initctl start <job>` 是一样的效果。

一些命令是为了兼容其它系统(主要是 sysvinit)，比如显示 runlevel 用/sbin/runlevel 命令：
```
$runlevel
N 2
```
这个输出说明当前系统的运行级别为 2。而且系统没有之前的运行级别，也就是说在系统上电启动进入预定运行级别之后没有再修改过运行级别。

那么如何修改系统上电之后的默认运行级别呢？

在 Upstart 系统中，需要修改/etc/init/rc-sysinti.conf 中的 DEFAULT_RUNLEVEL 这个参数，以便修改默认启动运行级别。这一点和 sysvinit 的习惯有所不同，大家需要格外留意。

还有一些随 UpStart 发布的小工具，用来帮助开发 UpStart 或者诊断 UpStart 的问题。比如 init-checkconf 和 upstart-monitor

还可以使用 initctl 的 emit 命令从命令行发送一个事件。
`#initctl emit <event>`
这一般是用于 UpStart 本身的排错。

### Upstart 小结

可以看到，UpStart 的设计比 SysVInit 更加先进。多数 Linux 发行版上已经不再使用 SysVInit，一部分发行版采用了 UpStart，比如 Ubuntu；而另外一些比如 Fedora，采用了一种被称为 systemd 的 init 系统。Systemd 出现的比 UpStart 更晚，但发展迅速，虽然 UpStart 也还在积极开发并被越来越多地应用，但 systemd 似乎发展更快，我将在下一篇文章中再介绍 systemd。


- - -

### Systemd 的简介和特点

Systemd 是 Linux 系统中最新的初始化系统（init），它主要的设计目标是克服 sysvinit 固有的缺点，提高系统的启动速度。systemd 和 ubuntu 的 upstart 是竞争对手，预计会取代 UpStart，实际上在作者写作本文时，已经有消息称 Ubuntu 也将采用 systemd 作为其标准的系统初始化系统。

Systemd 的很多概念来源于苹果 Mac OS 操作系统上的 launchd，不过 launchd 专用于苹果系统，因此长期未能获得应有的广泛关注。Systemd 借鉴了很多 launchd 的思想，它的重要特性如下：

#### 同 SysVinit 和 LSB init scripts 兼容

Systemd 是一个"新来的"，Linux 上的很多应用程序并没有来得及为它做相应的改变。和 UpStart 一样，systemd 引入了新的配置方式，对应用程序的开发也有一些新的要求。如果 systemd 想替代目前正在运行的初始化系统，就必须和现有程序兼容。任何一个 Linux 发行版都很难为了采用 systemd 而在短时间内将所有的服务代码都修改一遍。

Systemd 提供了和 Sysvinit 以及 LSB initscripts 兼容的特性。系统中已经存在的服务和进程无需修改。这降低了系统向 systemd 迁移的成本，使得 systemd 替换现有初始化系统成为可能。

#### 更快的启动速度

Systemd 提供了比 UpStart 更激进的并行启动能力，采用了 socket / D-Bus activation 等技术启动服务。一个显而易见的结果就是：更快的启动速度。

为了减少系统启动时间，systemd 的目标是：

- 尽可能启动更少的进程

- 尽可能将更多进程并行启动

同样地，UpStart 也试图实现这两个目标。UpStart 采用事件驱动机制，服务可以暂不启动，当需要的时候才通过事件触发其启动，这符合第一个设计目标；此外，不相干的服务可以并行启动，这也实现了第二个目标。

下面的图形演示了 UpStart 相对于 SysVInit 在并发启动这个方面的改进：

** UpStart 对 SysVinit 的改进 **

![](http://www.ibm.com/developerworks/cn/linux/1407_liuming_init3/image003.jpg)

假设有 7 个不同的启动项目， 比如 JobA、Job B 等等。在 SysVInit 中，每一个启动项目都由一个独立的脚本负责，它们由 sysVinit 顺序地，串行地调用。因此总的启动时间为 T1+T2+T3+T4+T5+T6+T7。其中一些任务有依赖关系，比如 A,B,C,D。

而 Job E 和 F 却和 A,B,C,D 无关。这种情况下，UpStart 能够并发地运行任务{E，F，(A,B,C,D)}，使得总的启动时间减少为 T1+T2+T3。

这无疑增加了系统启动的并行性，从而提高了系统启动速度。但是在 UpStart 中，有依赖关系的服务还是必须先后启动。比如任务 A,B,(C,D)因为存在依赖关系，所以在这个局部，还是串行执行。

让我们例举一些例子， Avahi 服务需要 D-Bus 提供的功能，因此 Avahi 的启动依赖于 D-Bus，UpStart 中，Avahi 必须等到 D-Bus 启动就绪之后才开始启动。类似的，livirtd 和 X11 都需要 HAL 服务先启动，而所有这些服务都需要 syslog 服务记录日志，因此它们都必须等待 syslog 服务先启动起来。然而 httpd 和他们都没有关系，因此 httpd 可以和 Avahi 等服务并发启动。

Systemd 能够更进一步提高并发性，即便对于那些 UpStart 认为存在相互依赖而必须串行的服务，比如 Avahi 和 D-Bus 也可以并发启动。从而实现如下图所示的并发启动过程：

** systemd 的并发启动 **

![](http://www.ibm.com/developerworks/cn/linux/1407_liuming_init3/image005.jpg)

所有的任务都同时并发执行，总的启动时间被进一步降低为 T1。

可见 systemd 比 UpStart 更进一步提高了并行启动能力，极大地加速了系统启动时间。

#### systemd 提供按需启动能力

当 sysvinit 系统初始化的时候，它会将所有可能用到的后台服务进程全部启动运行。并且系统必须等待所有的服务都启动就绪之后，才允许用户登录。这种做法有两个缺点：首先是启动时间过长；其次是系统资源浪费。

某些服务很可能在很长一段时间内，甚至整个服务器运行期间都没有被使用过。比如 CUPS，打印服务在多数服务器上很少被真正使用到。您可能没有想到，在很多服务器上 SSHD 也是很少被真正访问到的。花费在启动这些服务上的时间是不必要的；同样，花费在这些服务上的系统资源也是一种浪费。

Systemd 可以提供按需启动的能力，只有在某个服务被真正请求的时候才启动它。当该服务结束，systemd 可以关闭它，等待下次需要时再次启动它。

#### Systemd 采用 Linux 的 Cgroup 特性跟踪和管理进程的生命周期

init 系统的一个重要职责就是负责跟踪和管理服务进程的生命周期。它不仅可以启动一个服务，也必须也能够停止服务。这看上去没有什么特别的，然而在真正用代码实现的时候，您或许会发现停止服务比一开始想的要困难。

服务进程一般都会作为精灵进程（daemon）在后台运行，为此服务程序有时候会派生(fork)两次。在 UpStart 中，需要在配置文件中正确地配置 expect 小节。这样 UpStart 通过对 fork 系统调用进行计数，从而获知真正的精灵进程的 PID 号。

** 找到正确 pid **

![](http://www.ibm.com/developerworks/cn/linux/1407_liuming_init3/image007.jpg)

如果 UpStart 找错了，将 p1作为服务进程的 Pid，那么停止服务的时候，UpStart 会试图杀死 p1进程，而真正的 p1进程则继续执行。换句话说该服务就失去控制了。

还有更加特殊的情况。比如，一个 CGI 程序会派生两次，从而脱离了和 Apache 的父子关系。当 Apache 进程被停止后，该 CGI 程序还在继续运行。而我们希望服务停止后，所有由它所启动的相关进程也被停止。

为了处理这类问题，UpStart 通过 strace 来跟踪 fork、exit 等系统调用，但是这种方法很笨拙，且缺乏可扩展性。systemd 则利用了 Linux 内核的特性即 CGroup 来完成跟踪的任务。当停止服务时，通过查询 CGroup，systemd 可以确保找到所有的相关进程，从而干净地停止服务。

CGroup 已经出现了很久，它主要用来实现系统资源配额管理。CGroup 提供了类似文件系统的接口，使用方便。当进程创建子进程时，子进程会继承父进程的 CGroup。因此无论服务如何启动新的子进程，所有的这些相关进程都会属于同一个 CGroup，systemd 只需要简单地遍历指定的 CGroup 即可正确地找到所有的相关进程，将它们一一停止即可。

#### 启动挂载点和自动挂载的管理

传统的 Linux 系统中，用户可以用/etc/fstab 文件来维护固定的文件系统挂载点。这些挂载点在系统启动过程中被自动挂载，一旦启动过程结束，这些挂载点就会确保存在。这些挂载点都是对系统运行至关重要 的文件系统，比如 HOME 目录。和 sysvinit 一样，Systemd 管理这些挂载点，以便能够在系统启动时自动挂载它们。Systemd 还兼容/etc/fstab 文件，您可以继续使用该文件管理挂载点。

有时候用户还需要动态挂载点，比如打算访问 DVD 内容时，才临时执行挂载以便访问其中的内容，而不访问光盘时该挂载点被取消(umount)，以便节约资源。传统地，人们依赖 autofs 服务来实现这种功能。

Systemd 内建了自动挂载服务，无需另外安装 autofs 服务，可以直接使用 systemd 提供的自动挂载管理能力来实现 autofs 的功能。

#### 实现事务性依赖关系管理

系统启动过程是由很多的独立工作共同组成的，这些工作之间可能存在依赖关系，比如挂载一个 NFS 文件系统必须依赖网络能够正常工作。Systemd 虽然能够最大限度地并发执行很多有依赖关系的工作，但是类似"挂载 NFS"和"启动网络"这样的工作还是存在天生的先后依赖关系，无法并发执行。对于这些任务，systemd 维护一个"事务一致性"的概念，保证所有相关的服务都可以正常启动而不会出现互相依赖，以至于死锁的情况。

#### 能够对系统进行快照和恢复

systemd 支持按需启动，因此系统的运行状态是动态变化的，人们无法准确地知道系统当前运行了哪些服务。Systemd 快照提供了一种将当前系统运行状态保存并恢复的能力。

比如系统当前正运行服务 A 和 B，可以用 systemd 命令行对当前系统运行状况创建快照。然后将进程 A 停止，或者做其他的任意的对系统的改变，比如启动新的进程 C。在这些改变之后，运行 systemd 的快照恢复命令，就可立即将系统恢复到快照时刻的状态，即只有服务 A，B 在运行。一个可能的应用场景是调试：比如服务器出现一些异常，为了调试用户将当前状态保存为快照，然后可以进行任意的操作，比如停止服务等等。等调试结 束，恢复快照即可。

这个快照功能目前在 systemd 中并不完善，似乎开发人员也没有特别关注它，因此有报告指出它还存在一些使用上的问题，使用时尚需慎重。

#### 日志服务

systemd 自带日志服务 journald，该日志服务的设计初衷是克服现有的 syslog 服务的缺点。比如：

syslog 不安全，消息的内容无法验证。每一个本地进程都可以声称自己是 Apache PID 4711，而 syslog 也就相信并保存到磁盘上。

数据没有严格的格式，非常随意。自动化的日志分析器需要分析人类语言字符串来识别消息。一方面此类分析困难低效；此外日志格式的变化会导致分析代码需要更新甚至重写。

Systemd Journal 用二进制格式保存所有日志信息，用户使用 journalctl 命令来查看日志信息。无需自己编写复杂脆弱的字符串分析处理程序。

Systemd Journal 的优点如下：

- 简单性：代码少，依赖少，抽象开销最小。

- 零维护：日志是除错和监控系统的核心功能，因此它自己不能再产生问题。举例说，自动管理磁盘空间，避免由于日志的不断产生而将磁盘空间耗尽。

- 移植性：日志 文件应该在所有类型的 Linux 系统上可用，无论它使用的何种 CPU 或者字节序。

- 性能：添加和浏览 日志 非常快。

- 最小资源占用：日志 数据文件需要较小。

- 统一化：各种不同的日志存储技术应该统一起来，将所有的可记录事件保存在同一个数据存储中。所以日志内容的全局上下文都会被保存并且可供日后查询。例如一条 固件记录后通常会跟随一条内核记录，最终还会有一条用户态记录。重要的是当保存到硬盘上时这三者之间的关系不会丢失。Syslog 将不同的信息保存到不同的文件中，分析的时候很难确定哪些条目是相关的。

- 扩展性：日志的适用范围很广，从嵌入式设备到超级计算机集群都可以满足需求。

- 安全性：日志 文件是可以验证的，让无法检测的修改不再可能。

### Systemd 的基本概念

#### 单元的概念

系统初始化需要做的事情非常多。需要启动后台服务，比如启动 SSHD 服务；需要做配置工作，比如挂载文件系统。这个过程中的每一步都被 systemd 抽象为一个配置单元，即 unit。可以认为一个服务是一个配置单元；一个挂载点是一个配置单元；一个交换分区的配置是一个配置单元；等等。systemd 将配置单元归纳为以下一些不同的类型。然而，systemd 正在快速发展，新功能不断增加。所以配置单元类型可能在不久的将来继续增加。

- service ：代表一个后台服务进程，比如 mysqld。这是最常用的一类。

- socket ：此类配置单元封装系统和互联网中的一个 套接字 。当下，systemd 支持流式、数据报和连续包的 AF_INET、AF_INET6、AF_UNIX socket 。每一个套接字配置单元都有一个相应的服务配置单元 。相应的服务在第一个"连接"进入套接字时就会启动(例如：nscd.socket 在有新连接后便启动 nscd.service)。

- device ：此类配置单元封装一个存在于 Linux 设备树中的设备。每一个使用 udev 规则标记的设备都将会在 systemd 中作为一个设备配置单元出现。

- mount ：此类配置单元封装文件系统结构层次中的一个挂载点。Systemd 将对这个挂载点进行监控和管理。比如可以在启动时自动将其挂载；可以在某些条件下自动卸载。Systemd 会将/etc/fstab 中的条目都转换为挂载点，并在开机时处理。

- automount ：此类配置单元封装系统结构层次中的一个自挂载点。每一个自挂载配置单元对应一个挂载配置单元 ，当该自动挂载点被访问时，systemd 执行挂载点中定义的挂载行为。

- swap: 和挂载配置单元类似，交换配置单元用来管理交换分区。用户可以用交换配置单元来定义系统中的交换分区，可以让这些交换分区在启动时被激活。

- target ：此类配置单元为其他配置单元进行逻辑分组。它们本身实际上并不做什么，只是引用其他配置单元而已。这样便可以对配置单元做一个统一的控制。这样就可以实 现大家都已经非常熟悉的运行级别概念。比如想让系统进入图形化模式，需要运行许多服务和配置命令，这些操作都由一个个的配置单元表示，将所有这些配置单元 组合为一个目标(target)，就表示需要将这些配置单元全部执行一遍以便进入目标所代表的系统运行状态。 (例如：multi-user.target 相当于在传统使用 SysV 的系统中运行级别 5)

- timer：定时器配置单元用来定时触发用户定义的操作，这类配置单元取代了 atd、crond 等传统的定时服务。

- snapshot ：与 target 配置单元相似，快照是一组配置单元。它保存了系统当前的运行状态。

每个配置单元都有一个对应的配置文件，系统管理员的任务就是编写和维护这些不同的配置文件，比如一个 MySQL 服务对应一个 mysql.service 文件。这种配置文件的语法非常简单，用户不需要再编写和维护复杂的系统 5 脚本了。

#### 依赖关系

虽然 systemd 将大量的启动工作解除了依赖，使得它们可以并发启动。但还是存在有些任务，它们之间存在天生的依赖，不能用"套接字激活"(socket activation)、D-Bus activation 和 autofs 三大方法来解除依赖（三大方法详情见后续描述）。比如：挂载必须等待挂载点在文件系统中被创建；挂载也必须等待相应的物理设备就绪。为了解决这类依赖问 题，systemd 的配置单元之间可以彼此定义依赖关系。

Systemd 用配置单元定义文件中的关键字来描述配置单元之间的依赖关系。比如：unit A 依赖 unit B，可以在 unit B 的定义中用"require A"来表示。这样 systemd 就会保证先启动 A 再启动 B。

#### Systemd 事务

Systemd 能保证事务完整性。Systemd 的事务概念和数据库中的有所不同，主要是为了保证多个依赖的配置单元之间没有环形引用。比如 unit A、B、C，假如它们的依赖关系为:

** Unit 的循环依赖 **

![](http://www.ibm.com/developerworks/cn/linux/1407_liuming_init3/image009.jpg)


存在循环依赖，那么 systemd 将无法启动任意一个服务。此时 systemd 将会尝试解决这个问题，因为配置单元之间的依赖关系有两种：required 是强依赖；want 则是弱依赖，systemd 将去掉 wants 关键字指定的依赖看看是否能打破循环。如果无法修复，systemd 会报错。

Systemd 能够自动检测和修复这类配置错误，极大地减轻了管理员的排错负担。

#### Target 和运行级别

systemd 用目标（target）替代了运行级别的概念，提供了更大的灵活性，如您可以继承一个已有的目标，并添加其它服务，来创建自己的目标。下表列举了 systemd 下的目标和常见 runlevel 的对应关系：

** Sysvinit 运行级别和 systemd 目标的对应表 **

|	Sysvinit 运行级别	|	Systemd 目标	|	备注	|
|--------------|--------------|--------------|
|0		|	runlevel0.target, poweroff.target		|	关闭系统。	|
|1, s, single	|	runlevel1.target, rescue.target		|	单用户模式。	|
|2, 4		|	runlevel2.target, runlevel4.target, multi-user.target	|	用户定义/域特定运行级别。默认等同于 3。	|
|3		|	runlevel3.target, multi-user.target		|	多用户，非图形化。用户可以通过多个控制台或网络登录。	|
|5		|	runlevel5.target, graphical.target	|	多用户，图形化。通常为所有运行级别 3 的服务外加图形化登录。	|
|6		|	runlevel6.target, reboot.target		|	重启	|
|emergency		|	emergency.target	|	紧急 Shell	|


### Systemd 的并发启动原理

如前所述，在 Systemd 中，所有的服务都并发启动，比如 Avahi、D-Bus、livirtd、X11、HAL 可以同时启动。乍一看，这似乎有点儿问题，比如 Avahi 需要 syslog 的服务，Avahi 和 syslog 同时启动，假设 Avahi 的启动比较快，所以 syslog 还没有准备好，可是 Avahi 又需要记录日志，这岂不是会出现问题？

Systemd 的开发人员仔细研究了服务之间相互依赖的本质问题，发现所谓依赖可以分为三个具体的类型，而每一个类型实际上都可以通过相应的技术解除依赖关系。

#### 并发启动原理之一：解决 socket 依赖

绝大多数的服务依赖是套接字依赖。比如服务 A 通过一个套接字端口 S1 提供自己的服务，其他的服务如果需要服务 A，则需要连接 S1。因此如果服务 A 尚未启动，S1 就不存在，其他的服务就会得到启动错误。所以传统地，人们需要先启动服务 A，等待它进入就绪状态，再启动其他需要它的服务。Systemd 认为，只要我们预先把 S1 建立好，那么其他所有的服务就可以同时启动而无需等待服务 A 来创建 S1 了。如果服务 A 尚未启动，那么其他进程向 S1 发送的服务请求实际上会被 Linux 操作系统缓存，其他进程会在这个请求的地方等待。一旦服务 A 启动就绪，就可以立即处理缓存的请求，一切都开始正常运行。

那么服务如何使用由 init 进程创建的套接字呢？

Linux 操作系统有一个特性，当进程调用 fork 或者 exec 创建子进程之后，所有在父进程中被打开的文件句柄 (file descriptor) 都被子进程所继承。套接字也是一种文件句柄，进程 A 可以创建一个套接字，此后当进程 A 调用 exec 启动一个新的子进程时，只要确保该套接字的 close_on_exec 标志位被清空，那么新的子进程就可以继承这个套接字。子进程看到的套接字和父进程创建的套接字是同一个系统套接字，就仿佛这个套接字是子进程自己创建的一 样，没有任何区别。

这个特性以前被一个叫做 inetd 的系统服务所利用。Inetd 进程会负责监控一些常用套接字端口，比如 Telnet，当该端口有连接请求时，inetd 才启动 telnetd 进程，并把有连接的套接字传递给新的 telnetd 进程进行处理。这样，当系统没有 telnet 客户端连接时，就不需要启动 telnetd 进程。Inetd 可以代理很多的网络服务，这样就可以节约很多的系统负载和内存资源，只有当有真正的连接请求时才启动相应服务，并把套接字传递给相应的服务进程。

和 inetd 类似，systemd 是所有其他进程的父进程，它可以先建立所有需要的套接字，然后在调用 exec 的时候将该套接字传递给新的服务进程，而新进程直接使用该套接字进行服务即可。

#### 并发启动原理之二：解决 D-Bus 依赖

D-Bus 是 desktop-bus 的简称，是一个低延迟、低开销、高可用性的进程间通信机制。它越来越多地用于应用程序之间通信，也用于应用程序和操作系统内核之间的通信。很多现代的服务 进程都使用D-Bus 取代套接字作为进程间通信机制，对外提供服务。比如简化 Linux 网络配置的 NetworkManager 服务就使用 D-Bus 和其他的应用程序或者服务进行交互：邮件客户端软件 evolution 可以通过 D-Bus 从 NetworkManager 服务获取网络状态的改变，以便做出相应的处理。

D-Bus 支持所谓"bus activation"功能。如果服务 A 需要使用服务 B 的 D-Bus 服务，而服务 B 并没有运行，则 D-Bus 可以在服务 A 请求服务 B 的 D-Bus 时自动启动服务 B。而服务 A 发出的请求会被 D-Bus 缓存，服务 A 会等待服务 B 启动就绪。利用这个特性，依赖 D-Bus 的服务就可以实现并行启动。

#### 并发启动原理之三：解决文件系统依赖

系统启动过程中，文件系统相关的活动是最耗时的，比如挂载文件系统，对文件系统进行磁盘检查（fsck），磁盘配额检查等都是非常耗时的操作。在等待这些工 作完成的同时，系统处于空闲状态。那些想使用文件系统的服务似乎必须等待文件系统初始化完成才可以启动。但是 systemd 发现这种依赖也是可以避免的。

Systemd 参考了 autofs 的设计思路，使得依赖文件系统的服务和文件系统本身初始化两者可以并发工作。autofs 可以监测到某个文件系统挂载点真正被访问到的时候才触发挂载操作，这是通过内核 automounter 模块的支持而实现的。比如一个 open()系统调用作用在"/misc/cd/file1"的时候，/misc/cd 尚未执行挂载操作，此时 open()调用被挂起等待，Linux 内核通知 autofs，autofs 执行挂载。这时候，控制权返回给 open()系统调用，并正常打开文件。

Systemd 集成了 autofs 的实现，对于系统中的挂载点，比如/home，当系统启动的时候，systemd 为其创建一个临时的自动挂载点。在这个时刻/home 真正的挂载设备尚未启动好，真正的挂载操作还没有执行，文件系统检测也还没有完成。可是那些依赖该目录的进程已经可以并发启动，他们的 open()操作被内建在 systemd 中的 autofs 捕获，将该  open()调用挂起（可中断睡眠状态）。然后等待真正的挂载操作完成，文件系统检测也完成后，systemd 将该自动挂载点替换为真正的挂载点，并让 open()调用返回。由此，实现了那些依赖于文件系统的服务和文件系统本身同时并发启动。

当然对于"/"根目录的依赖实际上一定还是要串行执行，因为 systemd 自己也存放在/之下，必须等待系统根目录挂载检查好。

不过对于类似/home 等挂载点，这种并发可以提高系统的启动速度，尤其是当/home 是远程的 NFS 节点，或者是加密盘等，需要耗费较长的时间才可以准备就绪的情况下，因为并发启动，这段时间内，系统并不是完全无事可做，而是可以利用这段空余时间做更多 的启动进程的事情，总的来说就缩短了系统启动时间。

### Systemd 的使用

下面针对技术人员的不同角色来简单地介绍一下 systemd 的使用。本文只打算给出简单的描述，让您对 systemd 的使用有一个大概的理解。具体的细节内容太多，即无法在一篇短文内写全，本人也没有那么强大的能力。还需要读者自己去进一步查阅 systemd 的文档。

#### 系统软件开发人员

开发人员需要了解 systemd 的更多细节。比如您打算开发一个新的系统服务，就必须了解如何让这个服务能够被 systemd 管理。这需要您注意以下这些要点：

- 后台服务进程代码不需要执行两次派生来实现后台精灵进程，只需要实现服务本身的主循环即可。

- 不要调用 setsid()，交给 systemd 处理

- 不再需要维护 pid 文件。

- Systemd 提供了日志功能，服务进程只需要输出到 stderr 即可，无需使用 syslog。

- 处理信号 SIGTERM，这个信号的唯一正确作用就是停止当前服务，不要做其他的事情。

- SIGHUP 信号的作用是重启服务。

- 需要套接字的服务，不要自己创建套接字，让 systemd 传入套接字。

- 使用 sd_notify()函数通知 systemd 服务自己的状态改变。一般地，当服务初始化结束，进入服务就绪状态时，可以调用它。

** Unit 文件的编写 **

对于开发者来说，工作量最大的部分应该是编写配置单元文件，定义所需要的单元。

举例来说，开发人员开发了一个新的服务程序，比如 httpd，就需要为其编写一个配置单元文件以便该服务可以被 systemd 管理，类似 UpStart 的工作配置文件。在该文件中定义服务启动的命令行语法，以及和其他服务的依赖关系等。

此外我们之前已经了解到，systemd 的功能繁多，不仅用来管理服务，还可以管理挂载点，定义定时任务等。这些工作都是由编辑相应的配置单元文件完成的。我在这里给出几个配置单元文件的例子。

下面是 SSH 服务的配置单元文件，服务配置单元文件以.service 为文件名后缀。
```
  #cat /etc/system/system/sshd.service
  [Unit]
  Description=OpenSSH server daemon
  [Service]
  EnvironmentFile=/etc/sysconfig/sshd
  ExecStartPre=/usr/sbin/sshd-keygen
  ExecStart=/usrsbin/sshd –D $OPTIONS
  ExecReload=/bin/kill –HUP $MAINPID
  KillMode=process
  Restart=on-failure
  RestartSec=42s
  [Install]
  WantedBy=multi-user.target
```
文件分为三个小节。第一个是[Unit]部分，这里仅仅有一个 描述信息。第二部分是 Service 定义，其中，ExecStartPre 定义启动服务之前应该运行的命令；ExecStart 定义启动服务的具体命令行语法。第三部分是[Install]，WangtedBy 表明这个服务是在多用户模式下所需要的。

那我们就来看下 multi-user.target 吧：
```
  #cat multi-user.target
  [Unit]
  Description=Multi-User System
  Documentation=man.systemd.special(7)
  Requires=basic.target
  Conflicts=rescue.service rescure.target
  After=basic.target rescue.service rescue.target
  AllowIsolate=yes
  [Install]
  Alias=default.target
```
第一部分中的 Requires 定义表明 multi-user.target 启动的时候 basic.target 也必须被启动；另外 basic.target 停止的时候，multi-user.target 也必须停止。如果您接着查看 basic.target 文件，会发现它又指定了 sysinit.target 等其他的单元必须随之启动。同样 sysinit.target 也会包含其他的单元。采用这样的层层链接的结构，最终所有需要支持多用户模式的组件服务都会被初始化启动好。

在[Install]小节中有 Alias 定义，即定义本单元的别名，这样在运行 systemctl 的时候就可以使用这个别名来引用本单元。这里的别名是 default.target，比 multi-user.target 要简单一些。。。

此外在/etc/systemd/system 目录下还可以看到诸如*.wants 的目录，放在该目录下的配置单元文件等同于在[Unit]小节中的 wants 关键字，即本单元启动时，还需要启动这些单元。比如您可以简单地把您自己写的 foo.service 文件放入 multi-user.target.wants 目录下，这样每次都会被默认启动了。

最后，让我们来看看 sys-kernel-debug.mout 文件，这个文件定义了一个文件挂载点：
```
#cat sys-kernel-debug.mount
[Unit]
Description=Debug File Syste
DefaultDependencies=no
ConditionPathExists=/sys/kernel/debug
Before=sysinit.target
[Mount]
What=debugfs
Where=/sys/kernel/debug
Type=debugfs
```
这个配置单元文件定义了一个挂载点。挂载配置单元文件有一个[Mount]配置小节，里面配置了 What，Where 和 Type 三个数据项。这都是挂载命令所必须的，例子中的配置等同于下面这个挂载命令：

`mount –t debugfs /sys/kernel/debug debugfs`

配置单元文件的编写需要很多的学习，必须参考 systemd 附带的 man 等文档进行深入学习。希望通过上面几个小例子，大家已经了解配置单元文件的作用和一般写法了。

#### 系统管理员

systemd 的主要命令行工具是 systemctl。

多数管理员应该都已经非常熟悉系统服务和 init 系统的管理，比如 service、chkconfig 以及 telinit 命令的使用。systemd 也完成同样的管理任务，只是命令工具 systemctl 的语法有所不同而已，因此用表格来对比 systemctl 和传统的系统管理命令会非常清晰。

** Systemd 命令和 sysvinit 命令的对照表 **

|	Sysvinit 命令		|	Systemd 命令	|	备注	|
|-------------|------------|---------------|
|service foo start	|	systemctl start foo.service	|	用来启动一个服务 (并不会重启现有的)	|
|service foo stop	|	systemctl stop foo.service	|	用来停止一个服务 (并不会重启现有的)。|
|service foo restart	|	systemctl restart foo.service	|	用来停止并启动一个服务。	|
|service foo reload		|	systemctl reload foo.service	|	当支持时，重新装载配置文件而不中断等待操作。	|
|service foo condrestart	|	systemctl condrestart foo.service	|	如果服务正在运行那么重启它。|
|service foo status		|	systemctl status foo.service	|	汇报服务是否正在运行。	|
|ls /etc/rc.d/init.d/	|	systemctl list-unit-files --type=service	|	用来列出可以启动或停止的服务列表。	|
|chkconfig foo on	|	systemctl enable foo.service	|	在下次启动时或满足其他触发条件时设置服务为启用	|
|chkconfig foo off	|	systemctl disable foo.service	|	在下次启动时或满足其他触发条件时设置服务为禁用	|
|chkconfig foo	|	systemctl is-enabled foo.service	|	用来检查一个服务在当前环境下被配置为启用还是禁用。	|
|chkconfig –list	|	systemctl list-unit-files --type=service	|	输出在各个运行级别下服务的启用和禁用情况	|
|chkconfig foo –list	|	ls /etc/systemd/system/*.wants/foo.service	|	用来列出该服务在哪些运行级别下启用和禁用。	|
|chkconfig foo –add		|	systemctl daemon-reload	|	当您创建新服务文件或者变更设置时使用。	|
|telinit 3	|	systemctl isolate multi-user.target (OR systemctl isolate runlevel3.target OR telinit 3)	|	改变至多用户运行级别。	|

表中列出的常见用法，系统管理员还需要了解其他一些系统配置和管理任务的改变。

首先我们了解 systemd 如何处理电源管理，命令如下表所示：

** systemd 电源管理命令 **

|命令	|操作 |
|----|----|
|systemctl reboot	| 重启机器|
|systemctl poweroff	|关机|
|systemctl suspend	|待机|
|systemctl hibernate	|休眠|
|systemctl hybrid-sleep	混合休眠模式（同时休眠到硬盘并待机）

关机不是每个登录用户在任何情况下都可以执行的，一般只有管理员才可以关机。正常情况下系统 不应该允许 SSH 远程登录的用户执行关机命令。否则其他用户正在工作，一个用户把系统关了就不好了。为了解决这个问题，传统的 Linux 系统使用 ConsoleKit 跟踪用户登录情况，并决定是否赋予其关机的权限。现在 ConsoleKit 已经被 systemd 的 logind 所替代。

logind 不是 pid-1 的 init 进程。它的作用和 UpStart 的 session init 类似，但功能要丰富很多，它能够管理几乎所有用户会话(session)相关的事情。logind 不仅是 ConsoleKit 的替代，它可以：

- 维护，跟踪会话和用户登录情况。如上所述，为了决定关机命令是否可行，系统需要了解当前用户登录情况，如果用户从 SSH 登录，不允许其执行关机命令；如果普通用户从本地登录，且该用户是系统中的唯一会话，则允许其执行关机命令；这些判断都需要 logind 维护所有的用户会话和登录情况。

- Logind 也负责统计用户会话是否长时间没有操作，可以执行休眠/关机等相应操作。

- 为用户会话的所有进程创建 CGroup。这不仅方便统计所有用户会话的相关进程，也可以实现会话级别的系统资源控制。

- 负责电源管理的组合键处理，比如用户按下电源键，将系统切换至睡眠状态。

- 多席位(multi-seat) 管理。如今的电脑，即便一台笔记本电脑，也完全可以提供多人同时使用的计算能力。多席位就是一台电脑主机管理多个外设，比如两个屏幕和两个鼠标/键盘。席 位一使用屏幕 1 和键盘 1；席位二使用屏幕 2 和键盘 2，但他们都共享一台主机。用户会话可以自由在多个席位之间切换。或者当插入新的键盘，屏幕等物理外设时，自动启动 gdm 用户登录界面等。所有这些都是多席位管理的内容。ConsoleKit 始终没有实现这个功能，systemd 的 logind 能够支持多席位。

以上描述的这些管理功能仅仅是 systemd 的部分功能，除此之外，systemd 还负责系统其他的管理配置，比如配置网络，Locale 管理，管理系统内核模块加载等，完整地描述它们已经超出了本人的能力。

### systemd 小结

作为系统初始化系统，systemd 的最大特点有两个：

- 令人惊奇的激进的并发启动能力，极大地提高了系统启动速度；

- 用 CGroup 统计跟踪子进程，干净可靠。

此外，和其前任不同的地方在于，systemd 已经不仅仅是一个初始化系统了。

Systemd 出色地替代了 sysvinit 的所有功能，但它并未就此自满。因为 init 进程是系统所有进程的父进程这样的特殊性，systemd 非常适合提供曾经由其他服务提供的功能，比如定时任务 (以前由 crond 完成) ；会话管理 (以前由 ConsoleKit/PolKit 等管理) 。仅仅从本文皮毛一样的介绍来看，Systemd 已经管得很多了，可它还在不断发展。它将逐渐成为一个多功能的系统环境，能够处理非常多的系统管理任务，有人甚至将它看作一个操作系统。

好 的一点是，这非常有助于标准化 Linux 的管理！从前，不同的 Linux 发行版各行其事，使用不同方法管理系统，从来也不会互相妥协。比如如何将系统进入休眠状态，不同的系统有不同的解决方案，即便是同一个 Linux 系统，也存在不同的方法，比如一个有趣的讨论：如何让 ubuntu 系统休眠， 可以使用底层的/sys/power/state 接口，也可以使用诸如 pm-utility 等高层接口。存在这么多种不同的方法做一件事情对像我这样的普通用户而言可不是件有趣的事情。systemd 提供统一的电源管理命令接口，这件事情的意义就类似全世界的人都说统一的语言，我们再也不需要学习外语了，多么美好！

如果所有的 Linux 发行版都采纳了 systemd，那么系统管理任务便可以很大程度上实现标准化。此外 systemd 有个很棒的承诺：接口保持稳定，不会再轻易改动。对于软件开发人员来说，这是多么体贴又让人感动的承诺啊！


- - -

### 结束语

文章从古老却简明稳定的 sysvinit 说起，接着简要描述了 UpStart 带来的清新改变，最后看到了充满野心和活力的新生代 systemd 系统逐渐统治 Linux 的各个版本。就好像在看我们这个世界，一代人老去，新的一代带着横扫一切的气概登上舞台，还没有喊出他们最有力的口号，更猛的一代已经把聚光灯和所有的目 光带走。Systemd 之后也许还有更新的 init 系统出现吧，让我们继续期待。。。