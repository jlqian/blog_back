---
layout: post
title:  "CentOS 搭建 JavaEE 运行环境"
date:   2016-05-09 17:39:46 +0800
categories: jekyll update
---

对于JavaEE服务器的环境部署，有时候确实挺棘手，写这篇日志目的是自己搭建一下环境，学习Linux命令

对于CentOS的安装不再叙述，采取最小化安装

1.设置静态IP地址，允许远程登录
{% highlight c %}
//1.1设置静态IP
cd  /etc/sysconfig/network-scripts/
vi ifcfg-eth0 
//修改内容为：
DEVICE=eth0
HWADDR=00:0C:29:96:EA:D5
TYPE=Ethernet
UUID=061964d0-5219-42ed-b423-634d2700d200
ONBOOT=yes
NM_CONTROLLED=yes
BOOTPROTO=static
IPADDR=192.168.1.101
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
//1.2设置DNS
vi /etc/resolv.conf
//增加DNS服务，可以填写路由IP：
nameserver 192.168.1.1
//1.3重启网络服务
service network restart
//1.4防火墙默认开启22端口，可根据IP直接远程登录
{% endhighlight %}

2.建立本地yum源
{% highlight c %}
//2.1修改Base库名称，使其不再生效
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
//2.2修改CentOS-Media.repo
vi /etc/yum.repos.d/CentOS-Media.repo
//修改目的是file可以指向挂载的cdrom
baseurl=file:///media/CentOS/
        file:///media/cdrom/
        file:///media/cdrecorder/
//2.3挂载cdrom
mkdir /media/CentOS/
mount /dev/cdrom /media/CentOS/
{% endhighlight %}

3.安装vim编辑器
{% highlight c %}
yum install vim
{% endhighlight %}

4.由于root权限太大，首先创建用户：
{% highlight c %}
useradd jlqian
//默认会创建用户主目录
{% endhighlight %}

修改创建用户的密码
{% highlight c %}
passwd jlqian
//输入两遍密码即可
{% endhighlight %}

5.安装lrzsz，方便与windows上传下载文件
{% highlight c %}
yum install lrzsz
{% endhighlight %}

6.安装JDK
{% highlight c %}
//6.1上传JDK包
cd /usr/local/
rz //选择JDK安装包
//6.2解压
sh ./jdk-6u45-linux-x64.bin
//6.3配置环境变量
//环境变量的配置可以在当前用户的.profile或全局的/etc/profile，使用全局环境变量
vim /etc/profile
//修改export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL行为
JAVA_HOME=/usr/local/jdk1.6.0_45
PATH=$JAVA_HOME/bin:$PATH
export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL JAVA_HOME
//重新加载环境变量配置文件
source /etc/profile
{% endhighlight %}

7.安装Tomcat
{% highlight c %}
//7.1上传Tomcat包
cd /usr/local/
rz //选择Tomcat安装包
//7.2解压
tar -zxvf apache-tomcat-7.0.67.tar.gz 
//7.3配置环境变量
//环境变量的配置可以在当前用户的.profile或全局的/etc/profile，使用全局环境变量
vim /etc/profile
//修改export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL行为
JAVA_HOME=/usr/local/jdk1.6.0_45
CATALINA_HOME=/usr/local/apache-tomcat-7.0.67
PATH=$JAVA_HOME/bin:$CATALINA_HOME/bin:$PATH
export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL JAVA_HOME CATALINA_HOME
//重新加载环境变量配置文件
source /etc/profile
{% endhighlight %}

8.安装MySQL
{% highlight c %}
//8.1上传MySQL包
cd /usr/local/
rz //选择MySQL安装包
//8.2解压
tar -zxvf apache-tomcat-7.0.67.tar.gz 
//8.3配置环境变量
//环境变量的配置可以在当前用户的.profile或全局的/etc/profile，使用全局环境变量
vim /etc/profile
//修改export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL行为
JAVA_HOME=/usr/local/jdk1.6.0_45
CATALINA_HOME=/usr/local/apache-tomcat-7.0.67
PATH=$JAVA_HOME/bin:$CATALINA_HOME/bin:$PATH
export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL JAVA_HOME CATALINA_HOME
//重新加载环境变量配置文件
source /etc/profile
{% endhighlight %}
9.安装nginx

详细信息可以参考github [mysqlbackup][mysqlbackup]

[mysqlbackup]: https://github.com/jlqian/mysqlbakup

