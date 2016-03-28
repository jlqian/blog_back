---
layout: post
title:  "Chrome Run Android Apk"
date:   2016-03-18 16:41:15 +0800
categories: jekyll update
---
当我们在Linux下时，想使用各种window下可以使用的软件，可以使用wine，但是兼容性不是太好，Linux下能够运行Android程序，那是最好不过了。

目前apk文件能够通过编译成crx文件，运行在chrome中，尽管效果不是太好，但是总比没有好……

1.下载安装ARChon
你可以在<a href="http://archon-runtime.github.io/">http://archon-runtime.github.io/</a>网站中下载，或者在下面的网盘中下载。目前最新的版本为：2.1.0 Beta (ARC 41.4410.238.0)

下载时，选择与浏览器位数相同的版本。将下载的文件解压，打开浏览器的扩展程序，选中开发者模式，点击“加载已解压的扩展程序”，选中你解压的目录。插件将会安装到浏览器中，会有两个警告，不用担心，不影响使用。

2.你可以下载chromeos-apk，nodejs，将apk转为chrome的插件，目前很多的apk能够安装成功，但是插件崩溃和闪退占绝大部分，不建议使用这种方式，太折磨人，但方式我还是要提供的：

2.1自己动手

首先要下载<a href="https://nodejs.org/">nodejs/</a>，window下下载msi格式的安装包，linux下载tar.gz，对于windows下的.exe文件，我们需要的东西没有提供。

另外需要下载<a href="https://github.com/vladikoff/chromeos-apk">Chromeos-Apk</a>，下载完成后，将其中的内容复制到nodejs下的node_modules\npm\node_modules\chromeos-apk文件夹中

然后进入nodejs目录下，执行npm install chromeos-apk安装chromeos-apk,安装后会在自己的家目录出现一些东西npm\chromeos-apk，将apk文件复制到这个目录，然后执行chromeos-apk xx.apk –archon 生成一个文件夹，然后将这个文件夹作为插件安装到浏览器就可以了。

在启动app时，最常见的错误为no "message" element for key extName

修改转换apk后的文件夹\_locales\en目录下的messages.json文件，在"description": "Extension name"后加英文逗号，回车，添加"message": "XXXX"   XXXX为转换apk后的文件夹名，比如你这个应该就是"message": "com.tecentqq.android"  保存后重新加载应该就可以了

2.2Twerk转化

还可以通过Twerk插件进行转化

2.3中科大在线转换

最好的做法是在中科大提供的网站上进行转换，另外可能别人已经转换过了，你可以直接下载crx格式的插件，还有对于软件的评价，地址为：<a href="http://huodongweb.com/Crx">http://huodongweb.com/Crx</a>顺便说一下，需要翻墙……

将转换的插件安装到chrome浏览器，就可以使用App了

[github][github]

[github]: https://github.com/jlqian


