---
layout: post
title:  发布项目到Maven中心仓库
date:   2016-11-20 13:26:46 +0800
categories: jekyll update
---
  
如果将自己的项目开源，在github或者osChina中尽管可以，但是对使用maven构建工具依然不是很方便，所以发布到Maven中心仓库是更方便开放项目的使用。除Maven之外，其它工具如Ivy和Gradle也使用Maven中央仓库。目前来说，http://repo1.maven.org/maven2/是真正的Maven中央仓库的地址，该地址内置在Maven的源码中，其它地址包括著名的ibiblio.org，都是镜像。

注：中央仓库不是Apache的资源，中央仓库是由Sonatype出资维护的。

## 1.注册Sonatype的账户

访问：https://issues.sonatype.org/secure/Signup!default.jspa 注册用户，记住申请的用户名、密码
<img src="/img/2016-11-20/register.jpg" />
<img src="/img/2016-11-20/register_success.jpg" />
	
## 2.创建一个issue

Sonatype使用了JIRA来管理流程，需要创建一个issue，用于审核项目

其中Summary可以填写项目名，Description填写项目介绍，Group Id填写倒置域名，Project URL填写项目地址，SCM url填写源码地址
<img src="/img/2016-11-20/create_issue.jpg" />
<img src="/img/2016-11-20/issue_details.jpg" />

由于时区不同，审核时间在晚上，所以第二天我们会看到审核结果

<img src="/img/2016-11-20/domain_response.jpg" />

询问是否域名属于自己，回复Yes , of course，会收到审核通过的回复

<img src="/img/2016-11-20/configuration_prepared.jpg" />

	
## 3.使用 GPG 生成密钥对

### 3.1安装密钥对生成软件

Windows下可以通过Gpg4win-Vanilla 软件来生成密钥对，下载地址为：https://www.gpg4win.org/download.html
<img src="/img/2016-11-20/gunpg.jpg" />

Linux下直接安装gpg软件包就行。

### 3.2生成密钥

在命令行模式下：
{% highlight shell %}
gpg --gen-key
gpg (GnuPG) 2.0.30; Copyright (C) 2015 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
Your selection?
{% endhighlight %}
生成的签名类型：默认即可
{% highlight shell %}
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048)
{% endhighlight %}
密钥长度：默认
{% highlight shell %}
Requested keysize is 2048 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0)
{% endhighlight %}
密钥有效期：根据需要，（默认）
{% highlight shell %}
Key does not expire at all
Is this correct? (y/N)
{% endhighlight %}
确认上面的信息是否正确。
{% highlight shell %}
GnuPG needs to construct a user ID to identify your key.

Real name: qianjl
Email address: qianjl.cn@gmail.com
Comment:
You selected this USER-ID:
    "qianjl <qianjl.cn@gmail.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit?
{% endhighlight %}
确定之后会弹出密码框：
<img src="/img/2016-11-20/passphrase_box.jpg" />
记住密码，以后publish项目会用到。
{% highlight shell %}
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: key 616D288C marked as ultimately trusted
public and secret key created and signed.

gpg: checking the trustdb
gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
gpg: depth: 0  valid:   2  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 2u
pub   2048R/616D288C 2016-11-20
      Key fingerprint = 6245 CE50 CFD3 040B 286D  5BAB 05B7 CADA 616D 288C
uid       [ultimate] qianjl <qianjl.cn@gmail.com>
sub   2048R/F588D38C 2016-11-20
{% endhighlight %}

### 3.3上传公钥
{% highlight shell %}
gpg --keyserver hkp://pool.sks-keyservers.net --send-keys 616D288C
{% endhighlight %}
显示：
{% highlight shell %}
gpg: sending key 616D288C to hkp server pool.sks-keyservers.net
{% endhighlight %}

如果不知道公钥可以通过下面命令：
{% highlight shell %}
gpg --list-keys
{% endhighlight %}

此后，可使用本地的私钥来对上传构件进行数字签名，而下载该构件的用户可通过上传的公钥来验证签名，也就是说，大家可以验证这个构件是否由本人上传的，因为有可能该构件被坏人给篡改了。

### 3.4查询公钥是否发布成功
{% highlight shell %}
gpg --keyserver hkp://pool.sks-keyservers.net --recv-keys 616D288C
{% endhighlight %}
发布成功显示：
{% highlight shell %}
gpg: requesting key 616D288C from hkp server pool.sks-keyservers.net
gpg: key 616D288C: "qianjl <qianjl.cn@gmail.com>" not changed
gpg: Total number processed: 1
gpg:              unchanged: 1
{% endhighlight %}

## 4.Maven配置文件

### 4.1配置settings.xml文件
{% highlight xml %}
<servers>
    <server>
        <id>oss</id>
        <username>用户名</username>
        <password>密码</password>
    </server>
</servers>
{% endhighlight %}
这里的id是将来要在pom.xml里面使用的，所以务必记好，用户名和密码就是在Sonatype上面注册的用户名和密码。

### 4.2pom.xml文件
{% highlight xml %}
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.yuanheng100</groupId>
	<artifactId>chinaregion</artifactId>
	<version>1.1-RELEASE</version>
	<packaging>jar</packaging>
	<name>chinaregion</name>
	<description>
		China Region is a lightweight Java Framework for providing region name by region code.
	</description>
	<url>http://www.yuanheng100.com</url>
	<licenses>
		<license>
			<name>The Apache Software License, Version 2.0</name>
			<url>http://www.apache.org/licenses/LICENSE-2.0.txt</url>
		</license>
	</licenses>
	<developers>
		<developer>
			<name>Bai Song</name>
			<email>baisong@yuanheng100.com</email>
		</developer>
	</developers>
	<scm>
		<connection>
			scm:git:git@git.oschina.net:yuanheng100/ChinaRegion.git
		</connection>
		<developerConnection>
			scm:git:git@git.oschina.net:yuanheng100/ChinaRegion.git
		</developerConnection>
		<url>git@git.oschina.net:yuanheng100/ChinaRegion.git</url>
	</scm>
	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.javassist</groupId>
			<artifactId>javassist</artifactId>
			<version>3.20.0-GA</version>
		</dependency>
		<dependency>
			<groupId>org.jsoup</groupId>
			<artifactId>jsoup</artifactId>
			<version>1.10.1</version>
		</dependency>
	</dependencies>
	<profiles>
		<profile>
			<id>release</id>
			<activation>
            	<activeByDefault>true</activeByDefault>
        	</activation>
			<build>
				<plugins>
					<!--  Source  -->
					<plugin>
						<groupId>org.apache.maven.plugins</groupId>
						<artifactId>maven-source-plugin</artifactId>
						<version>2.2.1</version>
						<executions>
							<execution>
								<phase>package</phase>
								<goals>
									<goal>jar-no-fork</goal>
								</goals>
							</execution>
						</executions>
					</plugin>
					<!--  Javadoc  -->
					<plugin>
						<groupId>org.apache.maven.plugins</groupId>
						<artifactId>maven-javadoc-plugin</artifactId>
						<version>2.9.1</version>
						<executions>
							<execution>
								<phase>package</phase>
								<goals>
									<goal>jar</goal>
								</goals>
							</execution>
						</executions>
					</plugin>
					<!--  GPG  -->
					<plugin>
						<groupId>org.apache.maven.plugins</groupId>
						<artifactId>maven-gpg-plugin</artifactId>
						<version>1.5</version>
						<executions>
							<execution>
								<phase>verify</phase>
								<goals>
									<goal>sign</goal>
								</goals>
							</execution>
						</executions>
					</plugin>
				</plugins>
			</build>
		<distributionManagement>
			<snapshotRepository>
				<id>oss</id>
				<url>
					https://oss.sonatype.org/content/repositories/snapshots/
				</url>
			</snapshotRepository>
			<repository>
				<id>oss</id>
				<url>
					https://oss.sonatype.org/service/local/staging/deploy/maven2/
				</url>
			</repository>
		</distributionManagement>
		</profile>
	</profiles>
</project>
{% endhighlight %}

## 5.上传
{% highlight xml %}
mvn deploy
{% endhighlight %}
上传时需要输入密码，会弹出密码输入框，输入之前使用GunPG是输入的密码


### 5.1上传有可能出现问题，通过插件获取问题详情：
{% highlight xml %}
<plugin>
    <groupId>org.sonatype.plugins</groupId>
    <artifactId>nexus-staging-maven-plugin</artifactId>
    <version>1.6.3</version>
    <extensions>true</extensions>
    <configuration>
        <serverId>oss</serverId>
        <nexusUrl>https://oss.sonatype.org/</nexusUrl>
        <autoReleaseAfterClose>true</autoReleaseAfterClose>
    </configuration>
</plugin>
{% endhighlight %}
若出现：you have no permissions to stage against profile with ID "13fc88d70cfa2b"? Get to Nexus admin...

表明中央仓库没有赋予当前登录仓库的用户权限，需要在Jira提出，他们会赋予用户权限

## 6.发布
登陆：https://oss.sonatype.org

在左侧菜单选择：Staging Repositories

<img src="/img/2016-11-20/staging_repositories.jpg" />

搜索自己要发布的项目：

<img src="/img/2016-11-20/repositories_search.jpg" />

选中要发布的项目，Close，刷新，如果下方的校验都通过，就可以点击Release按钮，进行项目发布。

## 7.通知sonatype的工作人员关闭issue

回到issue系统，找到你的那个申请发布构件的issue，在下面回复工作人员，说明构件已经发布，待工作人员确认后，会关闭这个issue，如：My first release was promoted and is ready to be synced ,thank you !

发布后中心仓库很快就有了，其他仓库需要进行同步。

1. https://oss.sonatype.org/content/groups/staging/（项目发布地址）

2. http://mvnrepository.com/（这个地址比较方便复制dependency）

3. http://repo1.maven.org/maven2（Maven插件使用地址）

## 8.同一个groupId发布其它构件

执行4、5、6步骤即可。



[github][github]

[github]: https://github.com/jlqian