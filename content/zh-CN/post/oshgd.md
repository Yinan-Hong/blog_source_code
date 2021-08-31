+++
title = "哈工大操作系统网课笔记"
date = "2021-08-03T17:25:57+08:00"
categories = [
    "笔记"
]
tags = [
    "操作系统"
]
toc = true
+++

## 开机

x86PC上电后，硬件使cs=oxFFFF，ip=0x0000

把0磁道0扇区（引导扇区）读入0x7c00处

![Snipaste_2021-08-03_15-59-06](C:\Users\Tutoo\newsite\content\zh-CN\post\oshgdPic\Snipaste_2021-08-03_15-59-06.png)

### 引导扇区代码

bootsect.s，把读入的引导扇区代码移动到0x9000处，为操作系统读入腾出空间。调用int 0x13中断，继续读入四个扇区（setup扇区），读到boot后面0x9020。

![Snipaste_2021-08-03_16-00-57](C:\Users\Tutoo\newsite\content\zh-CN\post\oshgdPic\Snipaste_2021-08-03_16-00-57.png)

读入光标位置，屏幕打印“loading system”信息等等，继续读入os扇区等等，然后进入setup.s

![Snipaste_2021-08-03_16-01-36](C:\Users\Tutoo\newsite\content\zh-CN\post\oshgdPic\Snipaste_2021-08-03_16-01-36.png)



### setup.s代码

读入扩展内存大小等硬件参数。do_move：把操作系统代码移动到0地址处。etc。

![Snipaste_2021-08-03_15-57-24](C:\Users\Tutoo\newsite\content\zh-CN\post\oshgdPic\Snipaste_2021-08-03_15-57-24.png)



setup最后进入32位模式（保护模式）。把cr0置为1，使cpu进入保护模式的电路。

![Snipaste_2021-08-03_16-13-27](C:\Users\Tutoo\newsite\content\zh-CN\post\oshgdPic\Snipaste_2021-08-03_16-13-27.png)



### 保护模式下的寻址和中断：

全局描述符表GDT（Global Descriptor Table），cs：查表的下标，ip：偏移地址。在setup中会初始化gdt的表项。中断函数处理也有一个IDT表。

![Snipaste_2021-08-03_16-18-24](C:\Users\Tutoo\newsite\content\zh-CN\post\oshgdPic\Snipaste_2021-08-03_16-18-24.png)

setup最后jmpi 0,8 ，读gdt表项可以得到其实就是跳转到0地址。

![Snipaste_2021-08-03_16-42-25](C:\Users\Tutoo\newsite\content\zh-CN\post\oshgdPic\Snipaste_2021-08-03_16-42-25.png)

![Snipaste_2021-08-03_16-43-07](C:\Users\Tutoo\newsite\content\zh-CN\post\oshgdPic\Snipaste_2021-08-03_16-43-07.png)

然后进入head.s 然后进入main.c

将参数压栈（虽然没啥用），将返回地址压栈，将main地址压栈，然后jmp到设置页表，设置页表返回后ret到main.c，main.c不会退出，否则会进入L6死循环。虽然咱也不知道为啥要这样写。

![Snipaste_2021-08-03_16-53-09](C:\Users\Tutoo\newsite\content\zh-CN\post\oshgdPic\Snipaste_2021-08-03_16-53-09.png)





### main.c

《linux0.01》61页

开头一堆init。举了个mem_init的例子，进行分页。

![Snipaste_2021-08-03_16-57-37](C:\Users\Tutoo\newsite\content\zh-CN\post\oshgdPic\Snipaste_2021-08-03_16-57-37.png)

最后mian进入一个循环等待的函数or桌面whatever



----------



## 操作系统接口

与操作系统的交互，就是通过命令行or图形界面之类的。这本质上其实也只是调用了一下打印函数，图形界面函数之类之类的。

其实都是一些c函数，传入相应的参数之类的。这些接口又叫做system call（系统调用）

![Snipaste_2021-08-03_17-53-38](C:\Users\Tutoo\newsite\content\zh-CN\post\oshgdPic\Snipaste_2021-08-03_17-53-38.png)

main fork出循环等待的线程，linux的shell

![Snipaste_2021-08-03_22-49-58](C:\Users\Tutoo\newsite\content\zh-CN\post\oshgdPic\Snipaste_2021-08-03_22-49-58.png)

系统调用统一标准（unix）：POSIX

可以去查需要提供哪些接口，方便自己写个os

![Snipaste_2021-08-03_17-59-25](C:\Users\Tutoo\newsite\content\zh-CN\post\oshgdPic\Snipaste_2021-08-03_17-59-25.png)

### 关于系统调用的实现

将系统代码和用户代码分离，是硬件设计好的

在gdt表中，每个段都有个特权级DPL，数值越小权限越大

CPL：current privilege level

DPL：description privilege level

只有CPL<=DPL时，才能调用对应的程序

（弹幕说现在的os基本不依靠段进行权限检查，而是以页保护为主进行权限检查

![Snipaste_2021-08-03_22-57-48](C:\Users\Tutoo\newsite\content\zh-CN\post\oshgdPic\Snipaste_2021-08-03_22-57-48.png)

初始化时，系统代码段的DPL会设为0，执行用户程序时CPL会设为3，所以上层程序不可以直接调用系统程序因为没有权限。

硬件提供唯一的进入内核的方法是中断int 0x80。

通过系统接口进入内核基本都是c函数进行宏展开，调用内嵌汇编，调用int 0x80中断

![Snipaste_2021-08-03_23-16-36](C:\Users\Tutoo\newsite\content\zh-CN\post\oshgdPic\Snipaste_2021-08-03_23-16-36.png)

调用int 0x80时 set_system_gate(0x80, &system_call); 第二个参数是内核程序system_call的地址

宏展开后，把int 0x80的IDT表项的DPL设为3，因为当前的CPL肯定是3，然后把段地址设为8，偏移地址设为上面的第二个参数，这样就进入内核

这时cs=8，cs的最后两位是0，所以CPL是0，就可以顺利调用内核程序

（中断返回时CPL会置回3

![Snipaste_2021-08-03_23-32-35](C:\Users\Tutoo\newsite\content\zh-CN\post\oshgdPic\Snipaste_2021-08-03_23-32-35.png)

system_call程序会es fs置为内核数据段

eax存放系统调用号（从用户代码传进来，但没讲在哪里传给eax的...），在_sys_call_table里存各种函数指针，一个指针大小是4个字节，call到表里的第四个指针

![Snipaste_2021-08-03_23-36-47](C:\Users\Tutoo\newsite\content\zh-CN\post\oshgdPic\Snipaste_2021-08-03_23-36-47.png)

于是就进入到sys_write函数

![Snipaste_2021-08-03_23-39-01](C:\Users\Tutoo\newsite\content\zh-CN\post\oshgdPic\Snipaste_2021-08-03_23-39-01.png)



## 多进程图像

进程：运行着的程序

PCB：process control block

进程放在PCB里，由操作系统控制分配运行

schedule函数：进程交替

![Snipaste_2021-08-06_18-34-13](C:\Users\Tutoo\newsite\content\zh-CN\post\oshgdPic\Snipaste_2021-08-06_18-34-13.png)

getNext：调度 找下一个进程

switch_to()：保护现场（汇编

![image-20210806183750255](C:\Users\Tutoo\AppData\Roaming\Typora\typora-user-images\image-20210806183750255.png)

用映射表进行内存的分离，避免一个进程改了内存里其他内存的内容

不同进程访问的同一地址的实际物理地址不同

![Snipaste_2021-08-06_18-40-39](C:\Users\Tutoo\newsite\content\zh-CN\post\oshgdPic\Snipaste_2021-08-06_18-40-39.png)

进程同步（生产者消费者

锁

## 用户级线程

进程切换包括切换指令流和切换资源。

进行指令切换，不进行资源切换，是切换线程

切换进程开销比切换线程要大

![Snipaste_2021-08-06_18-51-15](C:\Users\Tutoo\newsite\content\zh-CN\post\oshgdPic\Snipaste_2021-08-06_18-51-15.png)

TCB：thread control block

yield()：切到另一线程，将返回地址入栈

每个线程分别用自己的个栈，否则会ret回错的地方

![Snipaste_2021-08-06_19-05-09](C:\Users\Tutoo\newsite\content\zh-CN\post\oshgdPic\Snipaste_2021-08-06_19-05-09.png)

用户级线程和核心级线程

核心级线程并发性更好，完全由操作系统进行调度

用户级进程的缺陷：如果进程的某个线程进入内核并发生阻塞，比如访问网卡但网络阻塞，那整个进程就会阻塞，其他线程也不会继续运行

![Snipaste_2021-08-06_19-12-12](C:\Users\Tutoo\newsite\content\zh-CN\post\oshgdPic\Snipaste_2021-08-06_19-12-12.png)

核心级线程：

调度用schedule()，对用户不可见

![Snipaste_2021-08-06_19-14-16](C:\Users\Tutoo\newsite\content\zh-CN\post\oshgdPic\Snipaste_2021-08-06_19-14-16.png)



## 内核级线程

MMU：memory management unit

一般服务器架构是多处理器

不过弹幕又说多核处理器每个核心都有自己的MMU，所以多核情况访问MMU的情况要再查一下

linux都是进程，所以资源分配是怎样的也没在这里体现出来

![Snipaste_2021-08-06_19-22-13](C:\Users\Tutoo\newsite\content\zh-CN\post\oshgdPic\Snipaste_2021-08-06_19-22-13.png)

内核级线程切换，要切换一套栈

中断进入内核，触发当前线程的中断，切换内核栈，切换用户栈

![Snipaste_2021-08-06_21-48-10](C:\Users\Tutoo\newsite\content\zh-CN\post\oshgdPic\Snipaste_2021-08-06_21-48-10.png)



![Snipaste_2021-08-06_21-51-50](C:\Users\Tutoo\newsite\content\zh-CN\post\oshgdPic\Snipaste_2021-08-06_21-51-50.png)

## 内核级线程的实现

