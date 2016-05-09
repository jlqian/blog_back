---
layout: post
title:  "Ghost Linux System"
date:   2016-04-02 23:46:15 +0800
categories: jekyll update
---
对于Windows操作系统，我们可以通过PE工具对其进行Ghost操作，在还原的时候通过镜像就可以还原到备份时的状态，一般的用途是批量对机器进行安装操作系统，另一个作用是个人不想对系统各种调试安装，在系统出问题之后直接进行还原操作。

对于Linux操作系统，WINPE对其没有作用，WINPE对Linux的文件系统Ext3及以上都不识别，所以没有办法对Linux系统进行备份，对于备份Linux操作系统，有很多软件可以选择，比如：Clonezilla、RedoBackup等

如果是对分区进行备份建议使用RedoBackup，带有非常好的UI界面，通过鼠标点击就能够完成分区的备份与还原。

如果是对硬盘进行备份，使用Clonezilla，但是在硬盘拷贝时，源盘要小于等于目标盘，否则拷贝将会出现异常。

步骤：

1、使用Clonezilla官方建议的启动盘制作软件，测试LinuxLive USB Creator可用，像其它常用的启动盘制作软件UltraISO制作的启动盘没有办法启动。

clonezilla-live-2.4.5-23-amd64.iso clonezilla-live-20160210-wily-amd64.iso 区别在于后者是 uEFI secure boot

制作启动盘过程：

1.1	安装LinuxLive USB Creator软件

1.2 插入U盘，打开软件

1.3 制作启动盘

    1.3.1 选择U盘
    
    1.3.2 选择克隆镜像clonezilla
    
    1.3.3 Live Mode 不用操作
    
    1.3.4 勾选Format the key
    
    1.3.5 点击闪电标示，开始制作，直到出现Your LinuxLive Key is now up and ready!
    
2、克隆Linux系统

2.1 进入BIOS，选择U盘启动进入Clonezilla
	<img src="/img/Clonezilla-1.jpg" />
2.2 选择语言
	<img src="/img/Clonezilla-2.jpg" />
	

3、恢复Linux系统

[百度网盘][百度网盘]
[github][github]

[百度网盘]: http://pan.baidu.com/s/1bpDMJwV
[github]: https://github.com/jlqian


