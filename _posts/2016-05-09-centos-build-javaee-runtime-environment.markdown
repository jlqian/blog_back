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
#1.1设置静态IP
cd  /etc/sysconfig/network-scripts/
vi ifcfg-eth0 
#修改内容为：
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
#1.2设置DNS
vi /etc/resolv.conf
#增加DNS服务，可以填写路由IP：
nameserver 192.168.1.1
#1.3重启网络服务
service network restart
#1.4防火墙默认开启22端口，可根据IP直接远程登录
{% endhighlight %}

2.建立本地yum源
{% highlight c %}
#2.1修改Base库名称，使其不再生效
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
#2.2修改CentOS-Media.repo
vi /etc/yum.repos.d/CentOS-Media.repo
#修改目的是file可以指向挂载的cdrom
baseurl=file:#/media/CentOS/
        file:#/media/cdrom/
        file:#/media/cdrecorder/
#2.3挂载cdrom
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
#默认会创建用户主目录
{% endhighlight %}

修改创建用户的密码
{% highlight c %}
passwd jlqian
#输入两遍密码即可
{% endhighlight %}

5.安装lrzsz，方便与windows上传下载文件
{% highlight c %}
yum install lrzsz
{% endhighlight %}

6.安装JDK
{% highlight c %}
#6.1上传JDK包
cd /usr/local/
rz #选择JDK安装包
#6.2解压
sh ./jdk-6u45-linux-x64.bin
#6.3配置环境变量
#环境变量的配置可以在当前用户的.profile或全局的/etc/profile，使用全局环境变量
vim /etc/profile
#修改export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL行为
JAVA_HOME=/usr/local/jdk1.6.0_45
PATH=$JAVA_HOME/bin:$PATH
export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL JAVA_HOME
#重新加载环境变量配置文件
source /etc/profile
{% endhighlight %}

7.安装Tomcat
{% highlight c %}
#7.1上传Tomcat包
cd /usr/local/
rz #选择Tomcat安装包
#7.2解压
tar -zxvf apache-tomcat-7.0.67.tar.gz 
#7.3配置环境变量
#环境变量的配置可以在当前用户的.profile或全局的/etc/profile，使用全局环境变量
vim /etc/profile
#修改export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL行为
JAVA_HOME=/usr/local/jdk1.6.0_45
CATALINA_HOME=/usr/local/apache-tomcat-7.0.67
PATH=$JAVA_HOME/bin:$CATALINA_HOME/bin:$PATH
export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL JAVA_HOME CATALINA_HOME
#重新加载环境变量配置文件
source /etc/profile
{% endhighlight %}

8.安装MySQL
{% highlight c %}
#8.1上传MySQL包
cd /usr/local/
rz #选择MySQL安装包
#8.2解压
tar -zxvf mysql-5.7.10-linux-glibc2.5-x86_64.tar.gz
#8.3创建mysql链接
ln -s mysql-5.7.10-linux-glibc2.5-x86_64/ mysql
#8.3配置环境变量
#环境变量的配置可以在当前用户的.profile或全局的/etc/profile，使用全局环境变量
vim /etc/profile
#修改export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL行为
JAVA_HOME=/usr/local/jdk1.6.0_45
CATALINA_HOME=/usr/local/apache-tomcat-7.0.67
MYSQL_HOME=/usr/local/mysql
PATH=$JAVA_HOME/bin:$CATALINA_HOME/bin:$MYSQL_HOME/bin:$PATH
export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL JAVA_HOME CATALINA_HOME MYSQL_HOME
#重新加载环境变量配置文件
source /etc/profile
#8.4安装
#8.4.1安装libaio依赖
yum search libaio  # search for info
yum install libaio # install library
#8.4.2增加mysql组
groupadd mysql
#8.4.3增加mysql用户
useradd -r -g mysql -s /bin/false mysql
#8.4.4数据访问权限
cd mysql
mkdir mysql-files
chmod 770 mysql-files
chown -R mysql .
chgrp -R mysql .
#8.4.5数据库初始化
bin/mysqld --initialize --user=mysql #记住生成的密码ffyCvD1O!ojj
bin/mysql_ssl_rsa_setup
#8.4.6改变数据访问权限             
chown -R root .
chown -R mysql mysql-files
#8.4.7安全方式启动数据库
bin/mysqld_safe --user=mysql &
#8.4.8增加mysql.server服务
cp support-files/mysql.server /etc/init.d/mysql.server
#8.4.9出现ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)
#可能是因为服务没有启动起来，ps -ef | grep mysql 查看一下
#另外是/tmp/mysql.sock不是mysql.server的socket，socket默认值为：/var/lib/mysql/mysql.sock（/etc/my.cnf）
ln -s /var/lib/mysql/mysql.sock /tmp/mysql.sock #可以解决这个问题
#8.4.10修改密码
#利用上面的密码登录之后，无论什么操作都会提示：ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('123456');
#8.4.11忘记root密码，修改密码
service mysql.server stop
cd /usr/local/mysql/bin/
./mysqld_safe --skip-grant-tables
mysql  #另开shell
mysql> use mysql;
mysql> UPDATE user SET authentication_string=PASSWORD('1234567') WHERE user='root';
mysql> FLUSH PRIVILEGES;
mysql> exit
service mysql.server restart
{% endhighlight %}

9.安装Nginx
{% highlight c %}
#9.1上传Nginx包
cd /usr/local/
rz #选择Nginx安装包
#9.2解压
tar -zxvf nginx-1.9.9.tar.gz
#9.3编译安装
#9.3.1安装依赖gcc、openssl-devel、pcre-devel、zlib-devel
yum -y install gcc openssl-devel pcre-devel zlib-devel
#9.3.2 configure
cd nginx-1.9.9
./configure --with-http_stub_status_module --with-http_gzip_static_module --prefix=/usr/local/nginx
#9.3.3 make
make
#9.3.4 makeinstall
makeinstall
#9.4配置环境变量
#环境变量的配置可以在当前用户的.profile或全局的/etc/profile，使用全局环境变量
vim /etc/profile
#修改export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL行为
JAVA_HOME=/usr/local/jdk1.6.0_45
CATALINA_HOME=/usr/local/apache-tomcat-7.0.67
MYSQL_HOME=/usr/local/mysql
NGINX_HOME=/usr/local/nginx-1.9.9
PATH=$JAVA_HOME/bin:$CATALINA_HOME/bin:$MYSQL_HOME/bin:$NGINX_HOME/sbin:$PATH
export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL JAVA_HOME CATALINA_HOME MYSQL_HOME NGINX_HOME
#重新加载环境变量配置文件
source /etc/profile
#9.5开放80端口
/sbin/iptables -I INPUT -p tcp --dport 80 -j ACCEPT
/etc/rc.d/init.d/iptables save
#查看防火墙状态
/etc/init.d/iptables status #service iptables status
{% endhighlight %}

上面用到的软件都可以在百度网盘找到 [百度网盘] [百度网盘]

[百度网盘]: http://pan.baidu.com/s/1pKpWv6j