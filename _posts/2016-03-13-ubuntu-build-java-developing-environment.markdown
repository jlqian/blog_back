---
layout: post
title:  "Ubuntu搭建JAVAEE开发环境"
date:   2016-03-13 00:33:37 +0800
categories: jekyll update
---
在工作中可能会要求使用Linux系统进行开发，对于不熟悉Linux的人来说，搭建开发环境确实很不容易，写这篇日志的目的主要也是如此。

安装Ubuntu操作系统就不再多说，几乎和Windows差不多了。

搭建开发环境步骤：(注：安装软件均在/usr/local/下)

1.安装openssh-server方便发送文件
{% highlight c %}
$sudo apt-get install openssh-server
{% endhighlight %}
查看是否启动
{% highlight c %}
$ps -ef | grep sshd
{% endhighlight %}
如果没有启动，则执行
{% highlight c %}
$sudo service ssh start
{% endhighlight %}

2.安装vim编辑器，vi编辑器有问题
{% highlight c %}
$sudo apt-get install vim
{% endhighlight %}

3.安装jdk，以6u45为例
{% highlight c %}
$sudo sh ./jdk-6u45-linux-x64.bin
{% endhighlight %}
添加JAVA_HOME的环境变量
{% highlight c %}
$vim ~/.profile
{% endhighlight %}
增加：
{% highlight c %}
export JAVA_HOME=/usr/local/jdk1.6.0_45
PATH=$JAVA_HOME/bin:$PATH
{% endhighlight %}
重新加载.profile文件
{% highlight c %}
$source ~/.profile
{% endhighlight %}
现在就可以使用Java了，java -version打印java的版本信息

4.安装Tomcat，以Tomcat7为例
{% highlight c %}
$sudo tar -zxvf apache-tomcat-7.0.67.tar.gz
{% endhighlight %}
添加CATALINA_HOME的环境变量
{% highlight c %}
$vim ~/.profile
{% endhighlight %}
修改：
{% highlight c %}
export JAVA_HOME=/usr/local/jdk1.6.0_45
export CATALINA_HOME=/usr/local/apache-tomcat-7.0.67
PATH=$JAVA_HOME/bin:$CATALINA_HOME/bin:$PATH
{% endhighlight %}
重新加载.profile文件
{% highlight c %}
$source ~/.profile
{% endhighlight %}
现在就可以使用Tomcat了

5.安装MySQL,以MySQL5为例
{% highlight c %}
$sudo tar -zxvf mysql-5.7.10-linux-glibc2.5-x86_64.tar.gz
$sudo ln -s mysql-5.7.10-linux-glibc2.5-x86_64/ mysql 
{% endhighlight %}
添加MYSQL_HOME的环境变量
{% highlight c %}
$vim ~/.profile
{% endhighlight %}
修改：
{% highlight c %}
export JAVA_HOME=/usr/local/jdk1.6.0_45
export CATALINA_HOME=/usr/local/apache-tomcat-7.0.67
export MYSQL_HOME=/usr/local/mysql
PATH=$JAVA_HOME/bin:$CATALINA_HOME/bin:$MYSQL_HOME/bin:$PATH
{% endhighlight %}
重新加载.profile文件
{% highlight c %}
$source ~/.profile
{% endhighlight %}
安装过程完全参考MySQL自带的INSTALL-BINNARY

5.1安装libaio依赖 
{% highlight c %}
$apt-cache search libaio # search for info
$sudo apt-get install libaio1 # install library 
{% endhighlight %}
5.2添加mysql用户和组
{% highlight c %}
$sudo groupadd mysql
$sudo useradd -r -g mysql -s /bin/false mysql
{% endhighlight %}
5.3创建data与mysql-files目录
{% highlight c %}
$cd mysql
$sudo mkdir data mysql-files
$sudo chmod 770 data/ mysql-files/
{% endhighlight %}
5.4改变mysql目录的用户和组
{% highlight c %}
$sudo chown -R mysql .
$sudo chgrp -R mysql .
{% endhighlight %}
5.5初始化数据库，会生成root密码
{% highlight c %}
$sudo bin/mysqld --initialize --user=mysql    #记住root密码
$sudo bin/mysql_ssl_rsa_setup
$sudo chown -R root .
$sudo chown -R mysql data mysql-files
$sudo bin/mysqld_safe --user=mysql &
{% endhighlight %}
5.6加入Mysql服务
{% highlight c %}
$sudo cp support-files/mysql.server /etc/init.d/mysql.server
{% endhighlight %}
5.7修改root密码

使用生成的root密码登录后任何操作都会提示：You must reset your password using ALTER USER statement before executing this statement.
{% highlight c %}
mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('123456');
{% endhighlight %}
如果忘记root密码，可以--skip-grant-tables的启动方式进行修改
{% highlight c %}
$sudo service mysql.server stop
$cd /usr/local/mysql/bin/
$sudo ./mysqld_safe --skip-grant-tables
$mysql  #另开shell
mysql> use mysql;
mysql> UPDATE user SET authentication_string=PASSWORD('1234567') WHERE user='root';
mysql> FLUSH PRIVILEGES;
mysql> exit
$sudo service mysql.server restart
{% endhighlight %}

6.安装NGINX
{% highlight c %}
$sudo tar -zxvf nginx-1.9.9.tar.gz
{% endhighlight %}
6.1安装依赖:(pcre，zlib)
{% highlight c %}
$sudo apt-get install libpcre3-dev #也可以到PCRE官网下载进行安装
$sudo sudo apt-get install zlib1g-dev
{% endhighlight %}
6.2编译安装
{% highlight c %}
$sudo ./configure --prefix=/usr/local/nginx
$sudo make
$sudo make install
{% endhighlight %}
6.3添加NGINX_HOME的环境变量
{% highlight c %}
$vim ~/.profile
{% endhighlight %}
增加：
{% highlight c %}
export JAVA_HOME=/usr/local/jdk1.6.0_45
export CATALINA_HOME=/usr/local/apache-tomcat-7.0.67
export MYSQL_HOME=/usr/local/mysql
export NGINX_HOME=/usr/local/nginx
PATH=$JAVA_HOME/bin:$CATALINA_HOME/bin:$MYSQL_HOME/bin:$NGINX_HOME/sbin:$PATH
{% endhighlight %}
重新加载.profile文件
{% highlight c %}
$source ~/.profile
{% endhighlight %}

7.安装Eclipse
{% highlight c %}
$sudo tar -zxvf eclipse-jee-luna-SR2-linux-gtk-x86_64.tar.gz
{% endhighlight %}
创建桌面快捷方式
{% highlight c %}
$sudo vim /usr/share/applications/eclipse.desktop
{% endhighlight %}
内容为：
{% highlight c %}
#!/usr/bin/env xdg-open
[Desktop Entry]
Name=Eclipse
Comment=Eclipse IDE
Keywords=documentation;information;manual;
OnlyShowIn=GNOME;Unity;
Exec=/usr/local/eclipse/eclipse
Icon=/usr/local/eclipse/icon.xpm
StartupNotify=true
Terminal=false
Type=Application
Categories=GNOME;GTK;Core;Documentation;Utility;
{% endhighlight %}
复制一份到桌面是上即可

8.安装Lantern
{% highlight c %}
$sudo dpkg -i lantern-installer-beta-64-bit.deb
{% endhighlight %}
复制快捷方式
{% highlight c %}
$cp /usr/share/applications/lantern.desktop ~/Desktop/
$chmod +x ~/Desktop/lantern.desktop 
{% endhighlight %}

9.安装搜狗输入法
{% highlight c %}
$sudo add-apt-repository ppa:fcitx-team/nightly
$sudo apt-get update
$sudo apt-get install fcitx
$sudo dpkg -i sogoupinyin_2.0.0.0068_amd64.deb 
{% endhighlight %}
打开Language Support提示没有安装完成，直接Install就行了，如果没有sogou拼音，打开Input Method Configuration-->Add Input Method-->Only Show Current Language 去掉，就会显示sogoupinyin，添加就可以了。

10.安装adobe flash player
{% highlight c %}
$sudo tar -zxvf install_flash_player_11_linux.x86_64.tar.gz 
$sudo cp libflashplayer.so /usr/lib/mozilla/plugins/
$sudo chmod 755 /usr/lib/mozilla/plugins/libflashplayer.so
{% endhighlight %}

[本文中的所有软件均可以在百度网盘下载][baiduyun]
[附上本人的GitHub地址][github]

[github]: https://github.com/jlqian
[baiduyun]: http://pan.baidu.com/s/1pKpWv6j