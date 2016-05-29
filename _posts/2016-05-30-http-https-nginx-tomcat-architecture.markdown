---
layout: post
title:  "Nginx代理Https与http，https转发到Tomcat"
date:   2016-05-30 01:18:46 +0800
categories: jekyll update
---
JavaEE项目中，对于静态资源如：html css js img等资源，通过http协议获取，对于Java项目的后台数据，通过https协议获取，避免了服务器进行没有必要的运算。

1.静态资源

1.1Nginx配置

对于静态资源，可以配置Nginx80端口的Server
{% highlight c %}
server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location /html/ {
            	alias   D:/workspace/html/;
			index   index.html;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
}
{% endhighlight %}

1.2静态资源请求动态数据

使用Ajax请求https跨域数据(jQuery版)
{% highlight javascript %}
	$.ajax({
		url:url,
		data:date,
		xhrFields: {
            withCredentials: true
		},
		dataType:"json",
		type:"POST",
		crossDomain: true,
		success:function(data){
			if(data!=null){
				sessionStorage.setItem('dict',JSON.stringify(data));
			}
		}
	});
{% endhighlight %}

上面的例子就是发送一个跨域请求，withCredentials: true目的是跨域请求维持会话（带上JESSIONID的COOKIE）;crossDomain: true: 跨域时默认会加上，为了显示表明这是个跨域请求。

2.动态资源

2.1Nginx配置，配置端口为443的Server
{% highlight c %}
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

2.2返回动态资源，需要添加额外响应头

对于动态资源，在跨域请求获取时，需要加上响应头：Access-Control-Allow-Origin，只加上这个响应头还不够，维持会话需要需要js发送过来的Credentials，所以需要Access-Control-Allow-Credentials头，响应头既可以在java代码中添加，也可以在Nginx配置中添加。

2.2.1Java代码中添加

可以通过springmvc的拦截器进行添加，创建拦截器AllowCrossDomainInterceptor
{% highlight java %}
public class AllowCrossDomainInterceptor implements HandlerInterceptor {

	@Override
	public boolean preHandle(HttpServletRequest request,
			HttpServletResponse response, Object handler) throws Exception {

		response.addHeader("Access-Control-Allow-Origin", request.getHeader("Origin"));//被允许跨域访问这个资源的网站，测试阶段可以这样写，允许所有的网站进行跨域访问；上限需要修改为自己的域名http://www.xxx.com
		response.addHeader("Access-Control-Allow-Credentials", "true");//是否允许浏览器携带 Cookie 来访问这个资源。
		response.addHeader("Access-Control-Allow-Methods", "POST, GET");
		return true;
	}

	@Override
	public void postHandle(HttpServletRequest request,
			HttpServletResponse response, Object handler,
			ModelAndView modelAndView) throws Exception {
	}

	@Override
	public void afterCompletion(HttpServletRequest request,
			HttpServletResponse response, Object handler, Exception ex)
			throws Exception {
	}
}
{% endhighlight %}

配置拦截器
{% highlight xml %}
  <mvc:interceptors>
  	<mvc:interceptor>
  		<mvc:mapping path="/**"/>
	  	<bean class="com.xxx.java.controller.aop.AllowCrossDomainInterceptor"></bean>
  	</mvc:interceptor>
  </mvc:interceptors>
{% endhighlight %}

2.2.2在Nginx中配置
{% highlight c %}
		location /java/ {
			proxy_pass http://127.0.0.1:8080;
			add_header Access-Control-Allow-Origin $http_origin;
			add_header Access-Control-Allow-Credentials true;
			add_header Access-Control-Allow-Methods GET,POST;
			proxy_set_header Host $host; 
			proxy_set_header X-Real-IP  $remote_addr;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
			proxy_set_header SSL_CERT $ssl_client_cert;
        }
{% endhighlight %}

[github][github]

[github]: https://github.com/jlqian