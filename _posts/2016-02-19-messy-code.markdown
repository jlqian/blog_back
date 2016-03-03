---
layout: post
title:  "解决兼容各种浏览器下载文件名称乱码问题"
date:   2016-02-19 17:51:46 +0800
categories: jekyll update
---
JavaWEB项目中，下载文件时，不同的浏览器可能会出现文件名称乱码的问题，有些干脆直接使用非中文字符作为文件名称（如：时间戳），其实事情解决方式很简单，无非是编码的问题，除非操作系统不支持中文...
以Tomcat为例服务器为例:

请求乱码：

默认情况下Tomcat的编码集为ISO-8859-1，这意味着，GET请求和POST请求都将采用ISO-8859-1进行编码，POST乱码很好解决。GET请求地址栏的内容将会被ISO-8859-1字符集编码，然后再解码，才是我们所得到的内容。这就是为什么GET请求中的中文，我们得到的是乱码。可以全部改为POST请求，利用Spring的CharacterEncodingFilter拦截器设置编码字符集。或者是在Tomcat的server.xml中修改配置：
{% highlight mxml %}
<Connector connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443"/>
{% endhighlight %}
修改为：
{% highlight mxml %}
<Connector connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443" URIEncoding="UTF-8" />
{% endhighlight %}

对于下载文件，其文件名为中文时，出现乱码的原因很多，一般情况下，每个浏览器解析的方式不一样，需要考虑兼容性的问题：

由于编码集是ISO-8859-1,下载有中文名称的文件时，需要对其进行转码。其流程为：Tomcat得到文件名进行ISO-8859-1的编码，然后浏览器端根据相同编码集进行解码。但是中文ISO-8859-1经过这些操作就会乱码，这也就是乱码的原因。

我们可以先使用GBK对中文进行编码，然后再使用ISO-8859-1对其进行解码。这样再加上上述过程。浏览器端使用GBK进行解码后，就能得到中文。

{% highlight java %}
    @RequestMapping("/download")
    public void download(HttpServletRequest request , HttpServletResponse response){
    	response.setCharacterEncoding("GBK");
        response.setContentType("text/plain;charset=GBK");
        
    	try 
    	{
			String downLoadName = new String("你好！".getBytes("GBK"),"ISO-8859-1");
			response.setHeader("Content-Disposition", "attachment;fileName="+downLoadName);
			ServletOutputStream outputStream = response.getOutputStream();
			outputStream.write(new String("中文内容。。。").getBytes("GBK"));
		} 
    	catch (UnsupportedEncodingException e)
        {
            logger.error("UnsupportedEncodingException错误", e);
        }
        catch (IOException e)
        {
            logger.error("IOException错误", e);
        }
    }
{% endhighlight %}

经过测试在IE Chrome Firefox 都能够正常显示，示例代码中的编码字符集不是随意定义的，对于Chrome Firefox 可以随意定义（需要浏览器支持的编码），对于IE,只能够采用GBK,GB2312等中文编码集。

[github][git_hub]

[github]: https://github.com/jlqian


