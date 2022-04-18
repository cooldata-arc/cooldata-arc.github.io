---
layout: post
date:   2022-04-15 09:20:00
title:  "Linux-Namespace-And-Cgroups"
categories: Linux
tags:  Namespace cgroups Container
mathjax: true
---

* content
{:toc}

Linux 容器的基石 Namespace 和 Cgroups. Namespace 限制了容器能看到和能使用什么；Cgroups(Control groups) 限制了容器可以各种资源的上线。

# 1. Namespace

> ref-url: https://man7.org/linux/man-pages/man7/namespaces.7.html  

命名空间将全局系统资源包装在一个抽象中，这使得命名空间中的进程似乎拥有自己的独立的全局资源实例。对全局资源的更改对作为命名空间成员的其他进程是可见的，但对其他进程是不可见的。

## 1.1 Namespace types

* 第二列显示了用于指定各种api中的名称空间类型的标志值。
* 第三列标识提供命名空间类型详细信息页面
* 最后一列是由命名空间类型隔离的资源汇总

| Namespace   | Flag        | Page        | Isolates(隔离资源汇总)      |
| :---        |    :----:   |    :----:   | :--- |
| Cgroup | CLONE_NEWCGROUP | cgroup_namespaces(7) | Cgroup root directory |
| IPC | CLONE_NEWIPC | ipc_namespaces(7) | System V IPC</br> POSIX message queues |
| Network | CLONE_NEWNET | network_namespaces(7) | Network devices, stacks, ports, etc. |
| Mount | CLONE_NEWNS | mount_namespaces(7) | Mount points |
| PID | CLONE_NEWPID | pid_namespaces(7) | Process IDs |
| Time | CLONE_NEWTIME | time_namespaces(7) | Boot and monotonic clocks |
| User | CLONE_NEWUSER | user_namespaces(7) | User and group IDs |
| UTS | CLONE_NEWUTS | uts_namespaces(7) | Hostname and NIS domain name |

### 1.1.1 调用方式
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; clone 系统调用创建一个新进程。如果调用的flags参数指定了一个或多个CLONE_NEW*标志，那么将为每个标志创建新的命名空间，子进程将成为这些命名空间的成员。
``` c++
int pid = clone(main_function, stack_size, CLONE_NEWCGROUP | CLONE_NEWIPC | CLONE_NEWNET | CLONE_NEWNS | CLONE_NEWPID | CLONE_NEWTIME | CLONE_NEWUSER | CLONE_NEWUTS | SIGCHLD, NULL); 
```

## 1.2 Namespace lifetime
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果没有任何其他因素，当名称空间中的最后一个进程终止或离开该名称空间时，该名称空间将自动拆除。但是，即使名称空间没有成员进程，也有许多其他因素可能会将其固定下来。这些因素包括:
* 命名空间是分层的(例如，PID或用户命名空间)，并且有一个子命名空间。
* 用户命名空间，拥有一个或多个非用户命名空间。
* 它是一个PID命名空间，有一个进程通过/proc/[PID]/ns/pid_for_children符号链接引用这个命名空间。
* 它是一个时间命名空间，有一个进程通过/proc/[pid]/ns/time_for_children符号链接引用这个命名空间。
* 它是一个IPC命名空间，对应的mqueue文件系统的挂载(参见mq_overview(7))引用这个命名空间。
* 它是一个PID命名空间，proc(5)文件系统对应的挂载引用这个命名空间。

# 2. Cgroups - Control Groups
> Ref_URL: https://man7.org/linux/man-pages/man7/cgroups.7.html </br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;控制组(Cgroups)是Linux内核的一种特性，它允许将进程组织成分层的组，然后可以限制和监视对各种类型资源的使用。内核的cgroup接口是通过一个名为cgroupfs的伪文件系统提供的。分组是在核心cgroup内核代码中实现的，而资源跟踪和限制是在一组每个资源类型的子系统（内存、CPU等）中实现的。</br>


## 2.1 术语
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cgroup是一组进程的集合，它们被绑定到通过cgroup文件系统定义的一组限制或参数上。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;子系统是修改cgroup中进程行为的内核组件。已经实现了各种子系统，可以限制cgroup可用的CPU时间和内存，计算cgroup使用的CPU时间，冻结和恢复执行cgroup中的进程。子系统有时也称为资源控制器(或简单地说，控制器)。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;控制器的cgroup被安排在一个层次结构中。这个层次结构是通过在cgroup文件系统中创建、删除和重命名子目录来定义的。在层次结构的每一层，属性(例如，限制)都可以被定义。cgroup提供的限制、控制和计算通常会影响cgroup下面定义属性的子层次结构。因此，例如，在层次结构中更高层次上的cgroup的限制不能被后代cgroup所超越。

## 2.2 Cgroup version 1 and version 2
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在 cgroups v1下，每个控制器可以挂载到一个单独的group文件系统上，该文件系统提供了系统中进程的自己的层次结构。也可以对同一个cgroup文件系统计算多个（甚至全部）cgroup v1控制器，这意味着计算的控制器管理相同的进程层次结构。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对于每个挂载的层次结构，目录树反映了控制组的层次结构。每个控制组由一个目录标识，其每个控制组cgroup表示为一个子目录。例如：/user/kevin/1.session , 它是cgroup kevin的子节点，而 kevin 又是 /user 的子节点。在每个cgroup目录下都有一组可读或可写的文件，反映了资源限制和一些通用的cgroup属性。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在cgroups v1中，进程和任务（Task）是有区别的。在这个视图中，一个进程可以由多个任务组成(从用户空间的角度来看，通常称为线程，在本手册的其余部分中称为线程)。在cgroups v1中，可以独立地操作进程中线程的cgroup成员关系。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cgroups v1在不同cgroups之间分割线程的能力在某些情况下会导致问题。例如，它对内存控制器没有意义，因为一个进程的所有线程共享一个地址空间。由于这些问题，在最初的cgroups v2实现中删除了独立操作进程中线程的cgroup成员关系的能力，随后以更有限的形式恢复了这种能力(请参阅下面关于“线程模式”的讨论)。

### 2.2.1 Cgroups v1
#### 2.2.1.1 Mounting and Unmounting v1 Controllers

``` sheel 
# mounting v1 controllers
mount -t cgroup -o cpu none /sys/fs/cgroup/cpu
or
mount -t cgroup -o cpu,cpuacct none /sys/fs/cgroup/cpu,cpuacct
or
mount -t cgroup -o all cgroup /sys/fs/cgroup

# unmounting v1 controllers
umount /sys/fs/cgroup/pids
```
*cgroups v1 controllers 见总表。*

#### 2.2.1.2 Creating cgroups and moving processes
一个 cgroup 文件系统最初包含一个根 cgroup "/"，所有进程都属于这个跟 cgroup。通过在 cgroup 文件系统中创建一个目录来创建一个新的 cgroup:
``` sh
# create a new empty cgroup
mkdir /sys/fs/cgroup/cpu/cg1

# 一个进程可以通过将其 PID 写入 cgroup 的 cgroup 来移动到这个 cgroup. 如：
echo $$ > /sys/fs/cgroup/cpu/cg1/cgroup.procs
```

#### 2.2.1.3 Removing cgroups
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;要删除cgroup，*它必须首先没有子cgroup，并且不包含(非僵尸)进程*。只要是这种情况，就可以简单地删除相应的目录路径名。注意，cgroup目录下的文件不能也不需要删除。

### 2.2.2 Cgroups v2
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在cgroups v2中，所有挂载的控制器都位于一个统一的层次结构中。虽然(不同的)控制器可以同时挂载在v1和v2层次结构下，但不可能同时挂载同一个控制器在v1和v2层次结构下。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这里总结了cgroup v2中的新行为，在某些情况下，将在以下小节中详细阐述。
1. Cgroups v2提供了一个统一的层次结构，所有控制器都根据这个层次安装。
2. 不允许“内部”进程。除了根cgroup之外，进程只能驻留在叶节点中(cgroup本身不包含子cgroup)。细节比这要微妙得多，下文将对此进行描述。
3. 激活的cgroups必须通过文件cgroup指定。控制器和cgroup.subtree_control。
4. 任务文件已被删除。此外，cgroup。cpuset控制器使用的Clone_children文件已被移除。
5. cgroup提供了一种改进的通知空cgroup的机制。事件的文件。

*上面列出的一些新行为后来在Linux 4.14中增加了“线程模式”(如下所述)。*

#### 2.2.2.1 Cgroups V2 统一层次结构
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在cgroups v1中，针对不同层次结构挂载不同控制器的能力旨在为应用程序设计提供极大的灵活性。但实际上，灵活性并没有预期的那么有用，而且在许多情况下还增加了复杂性。因此，在cgroups v2中，所有可用的控制器都挂载在一个层次结构上。可用的控制器会自动挂载，这意味着当使用如下命令挂载cgroup v2文件系统时，不需要(或可能)指定控制器:
``` sheel
mount -t cgroup2 none /mnt/cgroup2
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cgroup v2控制器只有在当前没有通过cgroup v1层次结构的挂载使用时才可用。或者，换句话说，在v1层次结构和统一v2层次结构上使用同一个控制器是不可能的。这意味着可能需要首先卸载v1控制器(如上所述)，然后才能在v2中使用该控制器。由于systemd(1)在默认情况下大量使用了一些v1控制器，因此在某些情况下禁用v1控制器可以更简单地引导系统。为此，在内核引导命令行上指定cgroup_no_v1=list选项;List是一个用逗号分隔的列表，包含了要禁用的控制器名称，或者单词all表示禁用所有v1控制器。(这种情况由systemd(1)正确处理，它回到没有指定控制器的情况下运行。)

#### 2.2.2.2 Cgroups V2 mount options
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The following options (mount -o) can be specified when mounting the group v2 filesystem:

 * **nsdelegate** (since Linux 4.15)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Treat cgroup namespaces as delegation boundaries.  For details, see below.

* **memory_localevents** (since Linux 5.2)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The memory.events should show statistics only for the  cgroup itself, and not for any descendant cgroups.  This  was the behavior before Linux 5.2.  Starting in Linux 5.2,  the default behavior is to include statistics for  descendant cgroups in memory.events, and this mount option  can be used to revert to the legacy behavior.  This option  is system wide and can be set on mount or modified through  remount only from the initial mount namespace; it is  silently ignored in noninitial namespaces.

#### 2.2.2.3 Cgroups v2 subtree control
v2 层次结构中每个 cgroup 都包含以下俩个文件：
* **cgroup.controllers** 这个只读文件公开了这个cgroup中可用的控制器的列表。这个文件的内容与cgroup的内容匹配。父目录cgroup中的子树控制文件。
* **cgroup.subtree_control** 这是cgroup中激活(启用)的控制器列表。这个文件中的控制器集是cgroup中控制器集的子集。这个cgroup的控制器。通过向该文件写入包含空格分隔的控制器名称的字符串来修改活动控制器集，每个字符串前面都有'+'(启用控制器)或'-'(禁用控制器)，如下所示:
``` shell
echo '+pids -memory' > x/y/cgroup.subtree_control
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;试图启用cgroup中不存在的控制器。当写入cgroup.subtree_control文件时，控制器导致ENOENT错误。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;因为cgroup.subtree_control中的控制器列是这些cgroup的子集。控制器，在层次结构中的一个cgroup中被禁用的控制器，永远不能在该cgroup下的子树中重新启用。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cgroup cgroup.subtree_control文件确定在子cgroup中执行的控制器集。当一个控制器(例如pid)在cgroup中出现时。父cgroup.subtree_control文件，然后相应的控制器接口文件(如pid .max)会自动在该cgroup的子cgroup中创建，并可用于对子cgroup进行资源控制。

| controllers </br> cgroups v1 | controllers </br> cgroups v2 | Linux version        | desc        |
| :--- |    :----:   |    :----:   |     :----   |
| cpu | cpu | Linux 5.2 and earlier | 当系统繁忙时，可以保证cgroup拥有最小数量的“CPU共享”。如果CPU不繁忙,这不会限制cgroup的CPU使用。</br> v2 中为 v1 中的 cpu 和 cpuacct 的继承者 | 
| cpuacct |  | Linux 5.2 and earlier | 提供了按进程组计算CPU使用情况的方法。 |
| cpuset | cpuset | Linux 5.2 and earlier | 该cgroup可用于将cgroup中的进程绑定到指定的cpu和NUMA节点集。 </br> v2 继承 v1 |
| memory | memory | Linux 5.2 and earlier | 内存控制器支持报告和限制cgroup使用的进程内存、内核内存和交换空间。</br> v2 是 v1 的继承 |
| devices |  | Linux 5.2 and earlier | 驱动控制器支持控制哪些进程可以创建(mknod)设备，以及打开它们进行读取或写入。策略可以指定为allow-lists和deny-lists。层次结构是强制的，所以新的规则不能违反目标或祖先cgroups的现有规则。 |
| freezer | freezer | Linux 5.2 and earlier | 冻结cgroup可以挂起和恢复cgroup中的所有进程。冻结cgroup/A也会导致其子进程被冻结，例如/A/B中的进程。</br> v2 继承 v1 |
| net_cls |  | Linux 5.2 and earlier | 这将在由cgroup创建的网络数据包上放置一个为cgroup指定的分类。这些分类可以在防火墙规则中使用，也可以使用tc(8)来塑造流量。这只适用于离开cgroup的报文，而不适用于到达cgroup的流量。 |
| blkio | io | Linux 5.2 and earlier | blkio cgroup通过对存储层次结构中的叶节点和中间节点应用节流和上限形式的IO控制来控制和限制对指定块设备的访问。</br>有两种策略可供选择。第一种是使用CFQ实现的基于时间的比例权重磁盘划分。这对使用CFQ的叶节点有效。第二种是 throttling policy 策略，它指定设备的I/O速率上限。 </br> v2 的 io 是 v1 blkio 的继承 |
| perf_event | perf_event | Linux 5.2 and earlier | 这个控制器允许对组在一个cgroup中的一组进程进行性能监视。 </br> v2 和 v1 相同 |
| net_prio |  | Linux 5.2 and earlier | 这允许为cgroups指定每个网络接口的优先级。 |
| hugetlb | hugetlb | Linux 5.2 and earlier | 这支持通过cgroups限制大页面的使用。 </br> v2 继承 v1 |
| pids | pids | Linux 5.2 and earlier | 这个控制器允许限制可以在一个cgroup(及其后代)中创建的进程的数量。 </br> v2 和 v1 相同 |
| rdma | rdma | Linux 5.2 and earlier | RDMA控制器允许限制每个cgroup使用RDMA/ ib特定资源。 </br> v2 和 v1 相同 |