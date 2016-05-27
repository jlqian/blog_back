---
layout: post
title:  "Centos下Https访问Nginx并通过Http转发到Tomcat"
date:   2016-05-27 11:12:46 +0800
categories: jekyll update
---
Https访问Nginx并通过Http转发到Tomcat

1.安装Nginx或Nginx增加SSL模块

1.1如果没有安装Nginx
{% highlight c %}
#1.1.1上传Nginx包
cd /usr/local/
rz #选择Nginx安装包
#1.1.2解压
tar -zxvf nginx-1.9.9.tar.gz
#1.1.3编译安装
#1.1.3.1安装依赖gcc、openssl-devel、pcre-devel、zlib-devel
yum -y install gcc openssl-devel pcre-devel zlib-devel
#1.1.3.2 configure
cd nginx-1.9.9
./configure --with-http_ssl_module --with-http_stub_status_module --with-http_gzip_static_module --prefix=/usr/local/nginx
#1.1.3.3 make
make
#1.1.3.4 makeinstall
makeinstall
#1.1.4配置环境变量
#环境变量的配置可以在当前用户的.profile或全局的/etc/profile，使用全局环境变量
vim /etc/profile
#修改export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL行为
JAVA_HOME=/usr/local/jdk1.6.0_45
CATALINA_HOME=/usr/local/apache-tomcat-7.0.67
NGINX_HOME=/usr/local/nginx
PATH=$JAVA_HOME/bin:$CATALINA_HOME/bin:$MYSQL_HOME/bin:$NGINX_HOME/sbin:$PATH
export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL JAVA_HOME CATALINA_HOME NGINX_HOME
#重新加载环境变量配置文件
source /etc/profile
#1.1.5开放80端口
/sbin/iptables -I INPUT -p tcp --dport 80 -j ACCEPT
/etc/rc.d/init.d/iptables save
#查看防火墙状态
/etc/init.d/iptables status #service iptables status
{% endhighlight %}

1.2若已经安装过NGINX,但是没有安装SSL模块
{% highlight c %}
nginx -V #查看安装时使用的参数
cd /usr/local/nginx-1.9.9
./configure --with-http_ssl_module --with-http_stub_status_module --with-http_gzip_static_module --prefix=/usr/local/nginx #重新编译
cp ./objs/nginx /usr/local/nginx/sbin/ #覆盖二进制文件
{% endhighlight %}

2.证书制作

注：使用工具OpenSSL

2.1生成根证书
{% highlight c %}
OpenSSL> req -x509 -nodes -days 365 -newkey rsa:2048 -keyout root-key.key -out root-cert.pem #无密码，需要密码去掉参数 -nodes
#Generating a 2048 bit RSA private key
#...+++
#............+++
#writing new private key to 'root-key.key'
#-----
#You are about to be asked to enter information that will be incorporated
#into your certificate request.
#What you are about to enter is what is called a Distinguished Name or a DN.
#There are quite a few fields but you can leave some blank
#For some fields there will be a default value,
#If you enter '.', the field will be left blank.
#-----
#Country Name (2 letter code) [AU]:CN
#State or Province Name (full name) [Some-State]:
#Locality Name (eg, city) []:
#Organization Name (eg, company) [Internet Widgits Pty Ltd]:CFCA
#Organizational Unit Name (eg, section) []:
#Common Name (e.g. server FQDN or YOUR name) []:
#Email Address []:
{% endhighlight %}

2.2生成用户私钥
{% highlight c %}
OpenSSL> genrsa -out user-key.key 2048
#Generating RSA private key, 2048 bit long modulus
#................................................................................
#.............................................................................+++
#
#................................................+++
#e is 65537 (0x10001)
{% endhighlight %}

2.3生成用户证书请求文件
{% highlight c %}
OpenSSL> req -new -out user-req.csr -key user-key.key
#You are about to be asked to enter information that will be incorporated
#into your certificate request.
#What you are about to enter is what is called a Distinguished Name or a DN.
#There are quite a few fields but you can leave some blank
#For some fields there will be a default value,
#If you enter '.', the field will be left blank.
#-----
#Country Name (2 letter code) [AU]:CN
#State or Province Name (full name) [Some-State]:
#Locality Name (eg, city) []:
#Organization Name (eg, company) [Internet Widgits Pty Ltd]:YHST
#Organizational Unit Name (eg, section) []:
#Common Name (e.g. server FQDN or YOUR name) []:LOCALHOST
#Email Address []:
#
#Please enter the following 'extra' attributes
#to be sent with your certificate request
#A challenge password []:
#An optional company name []:
{% endhighlight %}

注：Common Name 为网站域名，不能使用IP地址，测试的话，使用LOCALHOST

2.4为用户颁发证书
{% highlight c %}
OpenSSL> x509 -req -in user-req.csr -out user-cert.cer -CA root-cert.cer -CAkey root-key.key -CAcreateserial -days 365
#Signature ok
#subject=/C=CN/ST=Some-State/O=YHST
#Getting CA Private Key
{% endhighlight %}

3.Nginx配置反向代理

{% highlight c %}
cd /usr/local/nginx/conf
vim nginx.conf
#增加内容为：
	server {
		listen       443;
		server_name  localhost;
		
		ssl on;
		ssl_protocols SSLv2 SSLv3 TLSv1;
		
		ssl_certificate      user-cert.cer;
		ssl_certificate_key  user-key.key;
		
		ssl_session_cache    shared:SSL:1m;
		ssl_session_timeout  5m;
		
		ssl_ciphers  HIGH:!aNULL:!MD5;
		ssl_prefer_server_ciphers  on;
		
		location /java/ {
		        proxy_pass http://127.0.0.1:8080;
		        proxy_set_header Host $host; 
		        proxy_set_header X-Real-IP  $remote_addr;
		        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		        proxy_set_header SSL_CERT $ssl_client_cert;
		}
    }
{% endhighlight %}

注：将上一步生成的证书放到/usr/local/nginx/conf目录下

4.浏览器端安装证书

备注：

5.Https直接访问Tomcat如何配置：
{% highlight c %}
    <Connector port="8443"
               maxThreads="150" SSLEnabled="true" scheme="https" secure="true"
               sslProtocol="TLS" 
               SSLCertificateFile="user-cert.cer"
               SSLCertificateKeyFile="user-key.key"/>
{% endhighlight %}


