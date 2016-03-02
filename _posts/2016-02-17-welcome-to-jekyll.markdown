---
layout: post
title:  "Java 实现 MySQL数据库备份"
date:   2016-02-17 17:51:46 +0800
categories: jekyll update
---
在Java项目中，可能会定时对数据库进行备份。

可以使用Mysql自带的mysqldump命令，具体命令为：
{% highlight c %}
mysqldump [options] [db_name [tbl_name ...]]
{% endhighlight %}
在备份时通常备份一个数据库

所以可以使用命令如下：
{% highlight java %}
mysqldump -h127.0.0.1 -P3306 -uroot -proot --default-character-setutf8 -Bdatabase -r export_file
{% endhighlight %}
使用Java调用系统命令：
{% highlight java %}
Runtime runtime = Runtime.getRuntime();
Process exec = runtime.exec(command);
try {
	exec.waitFor();
} catch (InterruptedException e) {
	logger.error("MySQL数据库进行备份，命令没有正常退出！");
}
{% endhighlight %}
尽管mysqldump命令的参数 -c 可以对备份进行压缩，但是需要客户端和服务端都支持。
可以使用Java自带的压缩工具对其进行压缩：

Java中自带ZipOutputStream和GZIPOutputStream类，可以根据不同的操作系统，利用其对导出结果进行压缩，windows下压缩为.zip;linux下压缩为.gz。

详细信息可以参考github [mysqlbackup][mysqlbackup]

[mysqlbackup]: https://github.com/jlqian/mysqlbakup

