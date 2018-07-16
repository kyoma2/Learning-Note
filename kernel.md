# 1、Linux内核准备
## 1.1 获取内核源码
- Linux内核官方网站：http://www.kernel.org
- git：
``` shell
git clone https://github.com/torvalds/linux.git
```
- - - - -
## 1.2 安装内核源码
``` shell
$ cd /usr/src/kernel
$ tar xzvf linux-2.6.32.tar.gz
```
- - - - -
## 1.3 使用补丁
``` shell
diff -Nur newfile oldfile
patch -p1 < ../patch-x.y.z
```
``` shell
for i in `ls ../patch`; \ 
do patch -p1 < ../patch/"$i";done
```
- - - - -
## 1.4 内核源码树

| 目录 | 描述 |
| :---: | :---: |
| arch | 特定体系结构的源码 |
| block | 块设备I/O层 |
| crypto | 加密API |
| Documention | 内核源码文档 |
| drivers | 设备驱动程序 |
| firmware | 使用某些驱动程序而需要的设备固件 |
| fs | VFS和各种文件系统 |
| include | 内核头文件 |
| init | 内核引导和初始化 |
| ipc | 进程间通信 |
| kernel | 像调度程序这样的核心子系统 |
| lib | 管理内核函数 |
| mm | 内核管理子系统和VM |
| net | 网络子系统 |
| samples | 实例，实例代码 |
| scripts | 编译内核使用的脚本 |
| security | Linux安全模块 |
| sound | 语音子系统 |
| usr | 早期用户空间代码（initramfs）|
| tools | 在Linux开发中有用的工具 |
| virt | 虚拟化基础结构 |
- - - - -
## 1.5 编译内核
内核配置
- `make menuconfig`
- `make config`
- `make xconfig`
- `make defconfig`

- `make clean`
- `make distclean`

配置之后会产生.config文件  
内核编译：
- `make`
- `make modules`
- `make modules_install`
- `make install`
- `/sbin/depmod 2.6.31.8`
- `make -jn`

模块安装到 `/lib/modues`
- - - - -
## 1.6 内核开发的特点
- 内核编程时既不能访问C库也不能访问标准的C头文件；
- 内核编程时必须使用GNU C；
- 内核编程时缺乏想用户空间那样的内存保护机制；
- 内核编程时难以执行浮点运算；
- 内核给每个进程只有一个很小的定长堆栈；
- 由于内核支持异步中断、抢占和SMP，因此必须时刻注意同步和并发；
- 要考虑可移植性的重要性。

- 查看内核打印消息：`dmesg`或`cat /var/log/message`
- 清空当前内核消息：`dmesg -c`
- 插入内核模块：`insmod HelloWorld.ko`
- 查看内核模块：`lsmod`
- 删除内核模块：`rmmod HelloWorld.ko`
- - - - -
## 1.7 第一个内核模块程序
`HelloWord.c`
``` C
#Include <linux/init.h>
#Include <linux/module.h>
MODULE_LICENSE("Dual BAD/GPL")
static int hello_init(void) {
    printk(KERN_INFO "Hello, world!\n");
    printk("Hello: module loaded at 0x%p", hello_init);
    printk("HZ = %d\n", HZ);
}
static void hello_exit(void) {
    printk(KERN_INFO "Goodbye, cruel word!\n"); 
    printk("By: module loaded at 0x%p\n", hello_exit);
}
module_init(hello_init);
module_exit(hello_exit);
MODULE_AUTHOR("harlonxl@gmail.com");
MODULE_DESCRIPTION("Hello world, first kernel module");
```
- - - - -
# 2、进程管理
## 2.1 进程
- 进程就是处于运行期的程序以及相关资源的总称，是操作系统进行资源分配和调度的基本单位，进程通常包含一些资源，例如打开的文件描述符，挂起的信号，内核内部数据，处理器状态，一个或者多个具有内存映射的内存地址空间及一个或多个执行线程，还有用来存放全局变量的数据段等。
- 执行线程是在进程中的活动对象，是CPU调度的最小单位，每个线程拥有独立的程序计数器，进程栈和一组进程寄存器。
- linux系统下查看进程：
``` shell
# ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1  43272  3672 ?        Ss   Jun07   0:26 /usr/lib/systemd/systemd --switched-root --system --deserialize 21
root         2  0.0  0.0      0     0 ?        S    Jun07   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        S    Jun07   0:21 [ksoftirqd/0]
root         5  0.0  0.0      0     0 ?        S<   Jun07   0:00 [kworker/0:0H]
root         7  0.0  0.0      0     0 ?        S    Jun07   0:00 [migration/0]
root         8  0.0  0.0      0     0 ?        S    Jun07   0:00 [rcu_bh]
root         9  0.0  0.0      0     0 ?        R    Jun07   2:29 [rcu_sched]
root        10  0.0  0.0      0     0 ?        S    Jun07   0:10 [watchdog/0]
root        12  0.0  0.0      0     0 ?        S    Jun07   0:00 [kdevtmpfs]
root        13  0.0  0.0      0     0 ?        S<   Jun07   0:00 [netns]
root        14  0.0  0.0      0     0 ?        S    Jun07   0:00 [khungtaskd]
root        15  0.0  0.0      0     0 ?        S<   Jun07   0:00 [writeback]
root        16  0.0  0.0      0     0 ?        S<   Jun07   0:00 [kintegrityd]
root        17  0.0  0.0      0     0 ?        S<   Jun07   0:00 [bioset]
root        18  0.0  0.0      0     0 ?        S<   Jun07   0:00 [kblockd]
root        19  0.0  0.0      0     0 ?        S<   Jun07   0:00 [md]
root        25  0.0  0.0      0     0 ?        S    Jun07   0:00 [kswapd0]
root        26  0.0  0.0      0     0 ?        SN   Jun07   0:00 [ksmd]
...
```
`ps`命令参数含义：
- `a`：打印所有进程
- `u`：以用户为主打印进程
- `x`：显示所有进程，不区分终端
- - - - -
## 2.1 进程描述符及任务结果
进程描述符`task_struct`，该结构定义在`<linux/sched.h>`中，放在一个双向循环循环链表。
``` C
struct task_struct {
    /** 进程状态 */
    volatile long state;	 /* -1 unrunnable, 0 runnable, >0 stopped */
    ...
    /** 进程描述符链表 */
    struct list_head tasks;
	struct plist_node pushable_tasks;
    /** 虚拟内存结构 */
	struct mm_struct *mm, *active_mm;
    ...
    /** 进程PID */    
    pid_t pid; 
	pid_t tgid;
	...
	struct task_struct *real_parent; /* real parent process */
	/** 父进程进程描述符 */
	struct task_struct *parent;
	/** 子进程链表 */
	struct list_head children;
	...
	/** 文件系统信息 */
	struct fs_struct *fs;
    /** 进程打开文件的信息 */
	struct files_struct *files;
    /** 命名空间 */
	struct nsproxy *nsproxy;
    ...
}
```
查看linux系统中`pid`的最大值：
``` shell
# cat /proc/sys/kernel/pid_max 
32768
```
- - - - -
### 2.1.1 分配进程描述符
linux是通过`slab`分期器来分配 `task_struct`结构，这样能达到对象复用和缓存着色`(cache coloring)`的目的。
- - - - -
### 2.1.2 进程描述符的存放
在内核中，访问任务通常要获得指向task_struct的指针，可以通过`current`宏查找当前正在运行进程的进程描述符。硬件体系不同，这个宏的实现是不同的，例如：`PowerPC`中，专门有一个寄存器来存放该宏，而对于`x86`这种寄存器比较缺的体系结构，只能在栈的尾端创建`thread_info`结构，通过计算偏移间接地查找 `task_sturct`结构。
在`x86`系统中，`current`把栈指针的后13个有效位屏蔽掉，用来计算`thread_info`的偏移，该操作是通过`current_thread_info()`函数来完成的。
```C
static inline struct thread_info *current_thread_info(void)
{
	return (struct thread_info *)
		(current_stack_pointer & ~(THREAD_SIZE - 1));
}
```
`thread_info`结构体如下：
``` C
struct thread_info {
	struct pcb_struct	pcb;		/* palcode state */

	struct task_struct	*task;		/* main task structure */
	unsigned int		flags;		/* low level flags */
	unsigned int		ieee_state;	/* see fpu.h */

	struct exec_domain	*exec_domain;	/* execution domain */
	mm_segment_t		addr_limit;	/* thread address space */
	unsigned		cpu;		/* current CPU */
	int			preempt_count; /* 0 => preemptable, <0 => BUG */

	int bpt_nsaved;
	unsigned long bpt_addr[2];		/* breakpoint handling  */
	unsigned int bpt_insn[2];

	struct restart_block	restart_block;
};
```
`thread_info`的地址在栈顶，如下图所示：
<div align=center><img width="500" src="https://github.com/Harlonxl/Learning-Note/blob/master/image/thread_info.png"/></div>

- - - - -

### 2.1.3 进程状态
进程描述符`state`域描述了当前进程的状态，系统中的每个进程必然处于五种进程状态中的一种：
- `TASK_RUNING`：运行状态，进程时可运行的，它或者正在执行，或者在运行队列找那个等待运行。
- `TASK_INTERRUPTIBLE`：可中断，进程正在睡眠，等待某些条件的达成。一旦条件达成，内核就会把进程状态置为运行，处于此状态的进程也会因为接收到信号而提前被唤醒并随时准备投入运行。
- `TASK_UNINTERRUPTIBLE`：不可屏蔽，除了就算接收到信号也不会被唤醒或准备投入运行外，这个状态与可中断状态相同。
- `_TASK_TRACED`：被其他进程跟踪的进程，例如通过`ptrace`对调试程序进行跟踪。
- `_TASK_STOPPED`：停止状态，进程停止运行。
<div align=center><img width="800" src="https://github.com/Harlonxl/Learning-Note/blob/master/image/task_state.jpeg"/></div>
<div align=center>进程状态转化图</div>

- - - - -
### 2.1.4 设置当前进程状态
通过函数`set_task_state(task, state)`函数来设置，必要的时候设置内存屏障来强制其他处理器作重新排序。否则，等同于：
``` C
task->state = state;
```
- - - - -
### 2.1.5 进程上下文
当一个程序执行了系统调用或者触发了某个异常，就陷入了内核空间，此时，我们称内核“代表进程执行”并处于进程上下文中，在此上下文中`current`宏是有效的。