> ![博客封面](https://raw.githubusercontent.com/QinMou000/pic/main/a46182e6318c4593a5c674f2bf9439d4.jpeg) 
>
> ✨✨所属专栏：[Linux](https://blog.csdn.net/2301_80194476/category_12799988.html)✨✨
>
> ✨✨作者主页：[嶔某](https://blog.csdn.net/2301_80194476?spm=1000.2115.3001.5343)✨✨

# Linux：线程

## 概念

> 在一个程序里的一个执行路线就叫做线程`thread`。更准确一点：线程是“一个进程内部的控制序列”
>
> 一切进程都至少有一个线程
>
> 线程在进程内部运行，本质是在进程地址空间运行
>
> 在`Linux`系统中，在`CPU`眼中，看到的PCB都要比传统的要轻量化
>
> 透过进程虚拟地址空间，可以看到进程的大部分资源，将进程资源合理分配给每个执行流，就形成了线程执行流。

![image-20250408221044041](https://raw.githubusercontent.com/QinMou000/pic/main/image-20250408221044041.png)

## 分页式储存管理

如果没有虚拟内存和分页机制，每一个用户在物理内存上的空间必须是连续的，

![8b127986cf552877b7077b3a82fedaa](https://raw.githubusercontent.com/QinMou000/pic/main/8b127986cf552877b7077b3a82fedaa.jpg)

## Linux进程VS线程

## Linux线程控制



















