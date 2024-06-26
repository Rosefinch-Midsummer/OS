# I/O 核心子系统 

<!-- toc -->

## I/O 核心子系统概述

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240525153639.png)

操作系统需要实现文件保护功能，不同的用户对各个文件有不同的访问权限（如：只读、读和写等）。

在UNIX系统中，设备被看做是一种特殊的文件，每个设备也会有对应的FCB。当用户请求访问某个设备时，系统根据FCB中记录的信息来判断该用户是否有相应的访问权限，以此实现“设备保护”的功能。（参考“文件保护”小节)



## SPOOLing 技术（假脱机技术）

什么是脱机技术？

批处理阶段引入了脱机输入/输出技术（用磁带完成）。

在外围控制机的控制下，慢速输入设备的数据先被输入到更快速的磁带上。之后主机可以从快速的磁带上读入数据，从而缓解了速度矛盾

Tips：为什么称为“脱机”——脱离主机的控制进行的输入/输出操作。

引入脱机技术后，缓解了CPU与慢速I/O设备的速度矛盾。另一方面，即使CPU在忙碌，也可以提前将数据输入到磁带；即使慢速的输出设备正在忙碌，也可以提前将数据输出到磁带。

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240525154717.png)

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240525155250.png)

虽然系统中只有一个台打印机，但每个进程提出打印请求时，系统都会为在输出井中为其分配一个存储区（相当于分配了一个逻辑设备），使每个用户进程都觉得自己在独占一台打印机，从而实现对打印机的共享。

SPOOLing 技术可以把一台物理设备虚拟成逻辑上的多台设备，可将独占式设备改造成共享设备。

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240525155413.png)


## 设备的分配与回收

设备分配时应考虑的因素

- 设备的固有属性
- 设备分配算法
- 设备分配中的安全性

设备的固有属性可分为三种：独占设备、共享设备、虚拟设备。

独占设备——一个时段只能分配给一个进程（如打印机）

共享设备——可同时分配给多个进程使用（如磁盘），各进程往往是宏观上同时共享使用设备，而微观上交替使用。

虚拟设备一一采用SPOOLing技术将独占设备改造成虚拟的共享设备，可同时分配给多个进程使用（如采用 SPOOLing技术实现的共享打印机）

设备的分配算法：先来先服务、优先级高者优先、短任务优先……

从进程运行的安全性上考虑，设备分配有两种方式：

安全分配方式：为进程分配一个设备后就将进程阻塞，本次I/O完成后才将进程唤醒。（eg：考虑进程请求打印机打印输出的例子）

一个时段内每个进程只能使用一个设备

优点：破坏了“请求和保持”条件，不会死锁

缺点：对于一个进程来说，CPU和I/O设备只能串行工作

不安全分配方式：进程发出I/O请求后，系统为其分配I/O设备，进程可继续执行，之后还可以发出新的I/O请求。只有某个I/O请求得不到满足时才将进程阻塞。

一个进程可以同时使用多个设备

优点：进程的计算任务和/O任务可以并行处理，使进程迅速推进

缺点：有可能发生死锁（死锁避免、死锁的检测和解除）

静态分配：进程运行前为其分配全部所需资源，运行结束后归还资源

破坏了“请求和保持”条件，不会发生死锁

动态分配：进程运行过程中动态申请设备资源


![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240525160734.png)


![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240525160927.png)

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240525161237.png)

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240525161325.png)


![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240525161627.png)

设备分配的步骤

①根据进程请求的物理设备名查找SDT（注：物理设备名是进程请求分配设备时提供的参数）

②根据SDT找到DCT，若设备忙碌则将进程PCB挂到设备等待队列中，不忙碌则将设备分配给进程。

③根据DCT找到COCT，若控制器忙碌则将进程PCB挂到控制器等待队列中，不忙碌则将控制器分配给进程。

④根据COCT找到CHCT，若通道忙碌则将进程PCB挂到通道等待队列中，不忙碌则将通道分配给进程。

注：只有设备、控制器、通道三者都分配成功时，这次设备分配才算成功，之后便可启动I/O设备进行数据传送

缺点：

①用户编程时必须使用“物理设备名”，底层细节对用户不透明，不方便编程

②若换了一个物理设备，则程序无法运行

③若进程请求的物理设备正在忙碌，则即使系统中还有同类型的设备，进程也必须阻塞等待

改进方法：建立逻辑设备名与物理设备名的映射机制，用户编程时只需提供逻辑设备名

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240525162108.png)

逻辑设备表（LUT）建立了逻辑设备名与物理设备名之间的映射关系。

某用户进程第一次使用设备时使用逻辑设备名向操作系统发出请求，操作系统根据用户进程指定的设备类型（逻辑设备名）查找系统设备表，找到一个空闲设备分配给进程，并在LUT中增加相应表项。

如果之后用户进程再次通过相同的逻辑设备名请求使用设备：则操作系统通过LUT表即可知道用户进程实际要使用的是哪个物理设备了，并且也能知道该设备的驱动程序入口地址。

逻辑设备表的设置问题：

整个系统只有一张LUT：各用户所用的逻辑设备名不允许重复，适用于单用户操作系统

每个用户一张LUT：不同用户的逻辑设备名可重复，适用于多用户操作系统

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240525162417.png)

## 缓冲区管理

什么是缓冲区？有什么作用？

缓冲区是一个存储区域，可以由专门的硬件寄存器组成，也可利用内存作为缓冲区。

使用硬件作为缓冲区的成本较高，容量也较小，一般仅用在对速度要求非常高的场合（如存储器管理中所用的联想寄存器，由于对页表的访问频率极高，因此使用速度很快的联想寄存器来存放页表项的副本)

一般情况下，更多的是利用内存作为缓冲区，“设备独立性软件”的缓冲区管理就是要组织管理
好这些缓冲区。

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240525164149.png)

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240525170735.png)
### 单缓冲

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240525165200.png)

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240525165100.png)

### 双缓冲

双缓冲

假设某用户进程请求某种块设备读入若干块的数据。若采用双缓冲的策略，操作系统会在主存中为其分配两个缓冲区（若题目中没有特别说明，一个缓冲区的大小就是一个块）

双缓冲题目中，假设初始状态为：工作区空，其中一个缓冲区满，另一个缓冲区空

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240525165748.png)

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240525165820.png)

结论：采用双缓冲策略，处理一个数据块的平均耗时为 Max(T,C+M)

### 使用单/双缓冲在通信时的区别

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240525170119.png)

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240525170200.png)

### 循环缓冲区

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240525170315.png)

### 缓冲池

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240525170640.png)