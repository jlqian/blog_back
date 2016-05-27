---
layout: post
title:  "Nginx Tomcat MySQL 优化"
date:   2016-05-27 11:59:46 +0800
categories: jekyll update
---

#1.Nginx优化
{% highlight c %}

##高层的配置

worker_processes  2;
worker_rlimit_nofile 1000000;
#worker_processes 定义了nginx对外提供web服务时的worker进程数。最优值取决于许多因素，包括（但不限于）CPU核的数量、存储数据的硬盘数量及负载模式。不能确定的时候，将其设置为可用的CPU内核数将是一个好的开始（设置为“auto”将尝试自动检测它）。
#worker_rlimit_nofile 更改worker进程的最大打开文件数限制。如果没设置的话，这个值为操作系统的限制。设置后你的操作系统和Nginx可以处理比“ulimit -a”更多的文件，所以把这个值设高，这样nginx就不会有“too many open files”问题了。

##Events模块

worker_connections 65535; 
multi_accept on; 
use epoll; 
#worker_connections 设置可由一个worker进程同时打开的最大连接数。如果设置了上面提到的worker_rlimit_nofile，我们可以将这个值设得很高。记住，最大客户数也由系统的可用socket连接数限制（~ 64K），所以设置不切实际的高没什么好处。
#multi_accept 告诉nginx收到一个新连接通知后接受尽可能多的连接。
#use 设置用于复用客户端线程的轮询方法。如果你使用Linux 2.6+，你应该使用epoll。如果你使用*BSD，你应该使用kqueue。值得注意的是如果你不知道Nginx该使用哪种轮询方法的话，它会选择一个最适合你操作系统的）

##HTTP 模块
sendfile        on;
access_log 		off; 
error_log 		logs/error.log crit; 
keepalive_timeout  20; 
open_file_cache max=65535 inactive=60s; 
open_file_cache_valid 80s; 
open_file_cache_min_uses 2; 
#sendfile 可以让sendfile()发挥作用。sendfile()可以在磁盘和TCP socket之间互相拷贝数据(或任意两个文件描述符)。Pre-sendfile是传送数据之前在用户空间申请数据缓冲区。之后用read()将数据从文件拷贝到这个缓冲区，write()将缓冲区数据写入网络。sendfile()是立即将数据从磁盘读到OS缓存。因为这种拷贝是在内核完成的，sendfile()要比组合read()和write()以及打开关闭丢弃缓冲更加有效(更多有关于sendfile)。
#access_log 设置nginx是否将存储访问日志。关闭这个选项可以让读取磁盘IO操作更快
#error_log 告诉nginx只能记录严重的错误
#open_file_cache 打开缓存的同时也指定了缓存最大数目，以及缓存的时间。我们可以设置一个相对高的最大时间，这样我们可以在它们不活动超过20秒后清除掉。
#open_file_cache_valid 在open_file_cache中指定检测正确信息的间隔时间。
#open_file_cache_min_uses 定义了open_file_cache中指令参数不活动时间期间里最小的文件数。

{% endhighlight %}

#2.Tomcat优化
##apache-tomcat/bin/catalina.sh
{% highlight c %}

JAVA_OPTS="-Xms4096m -Xmx4096m -Xss512K -XX:PermSize=2048m -XX:MaxPermSize=2048m"
#-Xms – 指定初始化时化的栈内存
#-Xmx – 指定最大栈内存
#-Xss – 指定每个线程的stack大小
#-Xms – 指定初始化时化的堆内存
#-Xmx – 指定最大堆内存

{% endhighlight %}

##apache-tomcat/conf/server.xml
###线程池配置
{% highlight xml %}

<Connector port="80" protocol="HTTP/1.1" maxThreads="1000" minSpareThreads="500" maxSpareThreads="800" acceptCount="2000"
enableLookups="false" connectionTimeout="20000" redirectPort="8443" URIEncoding="UTF-8"/>

<--
maxThreads="1000"       	最大线程数
minSpareThreads="500"		初始化时创建的线程数
maxSpareThreads="800"		最大空闲数，一旦创建的线程超过这个值，Tomcat就会关闭不再需要的socket线程
acceptCount="2000"			指定当所有可以使用的处理请求的线程数都被使用时，可以放到处理队列中的请求数，超过这个数的请求将不予处理
enableLookups="false"		是否反查域名，取值为：true或false。为了提高处理能力，应设置为false
connnectionTimeout="20000"	网络连接超时，单位：毫秒。设置为0表示永不超时，这样设置有隐患的，默认20000毫秒。
URIEncoding="UTF-8"			不属于优化，避免GET请求乱码
-->

{% endhighlight %}

#3.MySQL优化
{% highlight c %}

[mysqld]
port            = 3306
socket          = /tmp/mysql.sock
skip-name-resolve
back_log = 600 
max_connections = 1024
max_allowed_packet = 4M
skip-external-locking 
key_buffer_size = 256M
table_open_cache = 128
sort_buffer_size = 8M
net_buffer_length = 8K
read_buffer_size = 2M
read_rnd_buffer_size = 8M
myisam_sort_buffer_size = 8M
#skip-name-resolve	禁止MySQL对外部连接进行DNS解析，使用这一选项可以消除MySQL进行DNS解析的时间。但需要注意，如果开启该选项，则所有远程主机连接授权都要使用IP地址方式，否则MySQL将无法正常处理连接请求
#back_log	指出在MySQL暂时停止回答新请求之前的短时间内多少个请求可以被存在堆栈中。
#max_connections MySQL的最大连接数，如果服务器的并发连接请求量比较大，建议调高此值，以增加并行连接数量，当然这建立在机器能支撑的情况下，因为如果连接数越多，介于MySQL会为每个连接提供连接缓冲区，就会开销越多的内存，所以要适当调整该值，不能盲目提高设值。
#max_allowed_packet 接受的数据包大小；增加该变量的值十分安全，这是因为仅当需要时才会分配额外内存。
#skip-external-locking MySQL选项以避免外部锁定。该选项默认开启
#key_buffer_size指定用于索引的缓冲区大小，增加它可得到更好的索引处理性能。对于内存在4GB左右的服务器该参数可设置为256M或384M。注意：该参数值设置的过大反而会是服务器整体效率降低！
#table_open_cache MySQL每打开一个表，都会读入一些数据到table_open_cache缓存中，当MySQL在这个缓存中找不到相应信息时，才会去磁盘上读取。默认值64
#sort_buffer_size MySQL执行排序使用的缓冲大小。如果想要增加ORDER BY的速度，首先看是否可以让MySQL使用索引而不是额外的排序阶段。
#read_buffer_size MySQL读入缓冲区大小。对表进行顺序扫描的请求将分配一个读入缓冲区，MySQL会为它分配一段内存缓冲区。read_buffer_size变量控制这一缓冲区的大小。
#read_rnd_buffer_size MySQL的随机读缓冲区大小。当按任意顺序读取行时(例如，按照排序顺序)，将分配一个随机读缓存区。进行排序查询时，MySQL会首先扫描一遍该缓冲，以避免磁盘搜索，提高查询速度，如果需要排序大量数据，可适当调高该值。
#myisam_sort_buffer_size MyISAM设置恢复表之时使用的缓冲区的尺寸，当在REPAIR TABLE或用CREATE INDEX创建索引或ALTER TABLE过程中排序 MyISAM索引分配的缓冲区

{% endhighlight %}

