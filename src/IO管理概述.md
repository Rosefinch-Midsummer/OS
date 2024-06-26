# I/O 管理概述

<!-- toc -->

## I/O 设备的基本概念与分类

“l/O”就是“输入/输出”（Input/Output)

I/O 设备就是可以将数据输入到计算机，或者可以接收计算机输出数据的外部设备，属于计算机中的
硬件部件。

UNIX系统将外部设备抽象为一种特殊的文件，用户可以使用
与文件操作相同的方式对外部设备进行操作。

Write操作：向外部设备写出数据

Read操作：：从外部设备读入数据

I/O 设备的分类一一按使用特性

- 人机交互类外部设备：数据传输速度慢
- 存储设备：数据传输速度快
- 网络通信设备：数据传输速度介于上述二者之间

I/O 设备的分类一一按传输速率分类

- 低速设备：鼠标、键盘等一一传输速率为每秒几个到几百字节
- 中速设备：如激光打印机等——传输速率为每秒数千至上万个字节
- 高速设备：如磁盘等一传输速率为每秒数千字节至千兆字节的设备

I/O 设备的分类一一按信息交换的单位分类

- 块设备，传输速率较高，可寻址，即对它可随机地读/写任一块I/O设备按信息交换的单位分类，块设备：如磁盘等一一数据传输的基本单位是“块”。
- 字符设备，传输速率较慢，不可寻址，在输入/输出时常采用中断驱动方式。字符设备：鼠标、键盘等，数据传输的基本单位是字符。


![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240523160637.png)

## I/O 控制器

I/O设备的机械部件主要用来执行具体I/O操作。

如我们看得见摸得着的鼠标/键盘的按钮；显示器的LED屏；移动硬盘的磁臂、磁盘盘面。

I/O设备的电子部件通常是一块插入主板扩充槽的印刷电路板。

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240523161411.png)


![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240523162802.png)

## I/O 控制方式

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240523164758.png)

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240523165013.png)

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240523165432.png)

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240523165703.png)

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240523165928.png)

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240523170143.png)

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240523170425.png)

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240523170553.png)


![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240523173359.png)

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240523173613.png)

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240523173840.png)

## IO软件层次结构

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240523175043.png)

## 输入输出应用程序接口和驱动程序接口


![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240523180016.png)

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240523180046.png)


阻塞/非阻塞I/O

阻塞I/O：应用程序发出I/0系统调用，进程需转为阻塞态等待。

eg：字符设备接口－－从键盘读一个字符 get

非阻塞I/O：应用程序发出I/O系统调用，系统调用可迅速返回，进程无需阻塞等待。

eg：块设备接口－一往磁盘写数据write

![](https://cdn.jsdelivr.net/gh/Rosefinch-Midsummer/MyImagesHost03/img/20240523180527.png)