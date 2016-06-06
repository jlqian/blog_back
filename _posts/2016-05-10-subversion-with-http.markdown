---
layout: post
title:  "通过Http访问Subversion服务"
date:   2016-05-10 21:34:46 +0800
categories: jekyll update
---

## 1.安装subversion和mod_dav_svn
{% highlight shell %}
yum install subversion
yum install mod_dav_svn
{% endhighlight %}

## 2.创建用户权限与密码文件
{% highlight shell %}
cd /var/www/
mkdir svn
cd svn/
vim authz #用户权限文件
#内容为：
[groups]
[svn:/]
touch passwd #用户认证文件
{% endhighlight %}

## 3.修改subversion.conf
{% highlight shell %}
cd /etc/httpd/conf.d/
vim subversion.conf 
#修改内容为：
<Location /repos/> #/repos/加上后面的"/"才能访问目录，否则访问目录时为403
   DAV svn
   SVNListParentPath on
   SVNParentPath /var/www/svn
   AuthType Basic
   AuthName "Subversion repos"
   AuthUserFile /var/www/svn/passwd
   AuthzSVNAccessFile /var/www/svn/authz
   Require valid-user
</Location>
{% endhighlight %}

## 4.修改httpd.conf
{% highlight shell %}
cd /etc/httpd/conf/
vim httpd.conf
#修改默认端口号：Listen 80（Nginx端口号冲突）
#为Listen 81（SELinux开放 80, 81, 443, 488, 8008, 8009, 8443, 9000）
#增加：
ServerName 127.0.0.1:81
{% endhighlight %}

## 5.修改Nginx配置文件
{% highlight shell %}
cd /usr/local/nginx/conf/
vim nginx.conf
#增加的server内容为：
location /svn {
        proxy_pass http://127.0.0.1:81/repos;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header REMOTE-HOST $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
{% endhighlight %}

## 6.创建SVN仓库
{% highlight shell %}
cd /var/www/svn/
svnadmin create repo1
{% endhighlight %}

## 7.仓库授权
{% highlight shell %}
cd /var/www/svn/
#用户
vim authz 
#内容为：
[groups]
admin = jlqian
[/]	#所有仓库全选，包括访问目录
@admin = rw
[repo1:/]	#repo1仓库的权限
@admin = rw
#密码
htpasswd -m /var/www/svn/passwd jlqian
#输入两遍密码即可
{% endhighlight %}

## 8.修改仓库的用户和组为apache
{% highlight shell %}
chown -R apache /var/www/svn/
chgrp -R apache /var/www/svn/
{% endhighlight %}

## 9.启动httpd与Nginx服务
{% highlight shell %}
service httpd start
nginx
{% endhighlight %}

