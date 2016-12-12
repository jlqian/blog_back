---
layout: post
title:  跨域请求的实现
date:   2016-12-05 10:03:36 +0800
categories: jekyll update
---

## 1.什么是跨域

违反浏览器同源策略的请求就是跨域请求，同源就是说HTML文档所在服务器的协议、域名、端口和所请求的资源必须一直，否则就是跨源

这是浏览器的一种安全策略

如：http://store.company.com/dir/page.html

<table class="standard-table" style="border-collapse: collapse;border: 2px solid #fff;border-width: 1px 0 0 1px;">
    <tbody>
        <tr>
             <th>URL</th>
             <th>Outcome</th>
             <th>Reason</th>
        </tr>
        <tr>
           <td><code>http://store.company.com/dir2/other.html</code></td>
           <td>Success</td>
           <td>&nbsp;</td>
        </tr>
        <tr>
           <td><code>http://store.company.com/dir/inner/another.html</code></td>
           <td>Success</td>
           <td>&nbsp;</td>
        </tr>
        <tr>
           <td><code>https://store.company.com/secure.html</code></td>
           <td>Failure</td>
           <td>Different protocol</td>
        </tr>
        <tr>
           <td><code>http://store.company.com:81/dir/etc.html</code></td>
           <td>Failure</td>
           <td>Different port</td>
        </tr>
        <tr>
           <td><code>http://news.company.com/dir/other.html</code></td>
           <td>Failure</td>
           <td>Different host</td>
        </tr>
    </tbody>
</table>
	
## 2.跨域请求方式

<table class="standard-table" style="border-collapse: collapse;border: 2px solid #fff;border-width: 1px 0 0 1px;">
    <tbody>
        <tr>
             <th>方式</th>
             <th>优势</th>
             <th>劣势</th>
        </tr>
        <tr>
           <td><code>CORS Cross-Origin Resource Sharing</code></td>
           <td> 1、支持GET POST 请求<br/>
                2、支持多种请求数据格式application/x-www-form-urlencoded, multipart/form-data等<br/>
                3、使用自定义请求头<br/>
                4、附带凭证信息（cookie）<br/>
           </td>
           <td>IE浏览器版本10以上支持<br/>[注：IE8、IE9 XDomainRequest 是阉割版的XMLHttpRequest,不能跨协议，不能附带凭证等]</td>
        </tr>
        <tr>
           <td><code>postMessage</code></td>
           <td> 1、支持GET POST 请求<br/>
                2、支持多种请求数据格式application/x-www-form-urlencoded, multipart/form-data等<br/>
                3、使用自定义请求头<br/>
                4、附带凭证信息（cookie）<br/>
            </td>
            <td>
                1、需要跨域的页面需要创建iframe<br/>
                2、被跨域的服务器需要创建代理页面<br/>
            </td>
        </tr>
        <tr>
           <td><code>webSocket</code></td>
           <td> 1、支持文本 字节流两种形式数据<br/>
                2、附带凭证信息（cookie）<br/>
           </td>
           <td>
               1、服务器、浏览器都需要支持【JDK1.7】<br/>
               2、服务器端配置较复杂<br/>
           </td>
        </tr>
        <tr>
           <td><code>JSONP</code></td>
           <td> 1、简单<br/>
           </td>
           <td>
               1、只能使用GET请求<br/>
               2、需要服务器支持<br/>
           </td>
        </tr>
    </tbody>
</table>

## 2.1.使用CORS

跨域资源共享（CORS）是一种网络浏览器的技术规范，它为Web服务器定义了一种方式，允许网页从不同的域访问其资源。而这种访问是被同源策略所禁止的。CORS系统定义了一种浏览器和服务器交互的方式来确定是否允许跨域请求。 它是一个妥协，有更大的灵活性，但比起简单地允许所有这些的要求来说更加安全。而W3C的官方文档目前还是工作草案，但是正在朝着W3C推荐的方向前进。

由域luantt.com访问qianjl.com数据

客户端
{% highlight html %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>跨域测试</title>
    <script src="script/jquery-1.12.4.min.js"></script>
    <script>
        (function( jQuery ) {
            if ( window.XDomainRequest ) {
                jQuery.ajaxTransport(function( s ) {
                    if ( s.crossDomain && s.async ) {
                        if ( s.timeout ) {
                            s.xdrTimeout = s.timeout;
                            delete s.timeout;
                        }
                        var xdr;
                        return {
                            send: function( _, complete ) {
                                function callback( status, statusText, responses, responseHeaders ) {
                                    xdr.onload = xdr.onerror = xdr.ontimeout = jQuery.noop;
                                    xdr = undefined;
                                    complete( status, statusText, responses, responseHeaders );
                                }
                                xdr = new window.XDomainRequest();
                                xdr.onload = function() {
                                    callback( 200, "OK", { text: xdr.responseText }, "Content-Type: " + xdr.contentType );
                                };
                                xdr.onerror = function() {
                                    callback( 404, "Not Found" );
                                };
                                xdr.onprogress = function() {};
                                if ( s.xdrTimeout ) {
                                    xdr.ontimeout = function() {
                                        callback( 0, "timeout" );
                                    };
                                    xdr.timeout = s.xdrTimeout;
                                }

                                xdr.open( s.type, s.url, true );
                                xdr.send( ( s.hasContent && s.data ) || null );
                            },
                            abort: function() {
                                if ( xdr ) {
                                    xdr.onerror = jQuery.noop();
                                    xdr.abort();
                                }
                            }
                        };
                    }
                });
            }
        })( jQuery );

        $(function(){
            $.ajax({
                url:"http://qianjl.com/cross/data",
                xhrFields: {
                    withCredentials: true
                },
                dataType:"json",
                type:"POST",
                crossDomain: true
            }).success(function(response){
                $("#message").html(response);
            }).error(function(XMLHttpRequest, textStatus, errorThrown){
                console.log(errorThrown);
            });
        });
    </script>
</head>
<body>

<div id="message"></div>

</body>
</html>
{% endhighlight %}

服务端【StringMVC】
{% highlight xml %}
    <mvc:cors>
        <mvc:mapping path="/cross/*"/>
    </mvc:cors>

    <mvc:annotation-driven>
        <mvc:message-converters>
            <bean class="com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter4">
                <property name="defaultCharset" value="UTF-8"></property>
                <property name="supportedMediaTypes" value="application/json"></property>
            </bean>
        </mvc:message-converters>
    </mvc:annotation-driven>
{% endhighlight %}

默认会添加两个响应头：Access-Control-Allow-Credentials:true  Access-Control-Allow-Origin:http://luantt.com

可以对服务端详细配置：
{% highlight xml %}
<mvc:mapping path="/api/**"
		allowed-origins="http://domain1.com, http://domain2.com"
		allowed-methods="GET, PUT"
		allowed-headers="header1, header2, header3"
		exposed-headers="header1, header2" allow-credentials="false"
		max-age="123" />
{% endhighlight %}

服务端【Nginx】
{% highlight shell %}
    add_header Access-Control-Allow-Origin $http_origin;
	add_header Access-Control-Allow-Credentials true;
	add_header Access-Control-Allow-Methods GET,POST;
{% endhighlight %}

注：Access-Control-Allow-Credentials true 不能和Access-Control-Allow-Origin * 共同使用

### 2.2使用postMessage跨域

在HTML5中新增了postMessage方法，postMessage可以实现跨文档消息传输（Cross Document Messaging），Internet Explorer 8, Firefox 3, Opera 9, Chrome 3和 Safari 4都支持postMessage。该方法可以通过绑定window的message事件来监听发送跨文档消息传输内容。

需要跨域的页面
{% highlight html %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>message</title>
    <script src="http://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"></script>
    <script>
        $(function(){
            var proxy_origin = "http://qianjl.com";
            var message_iframe = document.createElement("iframe");
            message_iframe.src = "http://qianjl.com/proxy.html";
            message_iframe.id = "proxy_iframe";
            $("body").after(message_iframe);
            //监听
            window.addEventListener('message',function(event){
                //监听消息
                var origin = event.origin || event.originalEvent.origin; // For Chrome, the origin property is in the event.originalEvent object.
                if (origin !== proxy_origin)
                    return;
                //获取消息，请求资源，并将请求的资源发送到被代理页面
                $("#message").html(event.data);

            },false);

            $("#request").click(function(){
                //发送消息
                var message = {
                    url : "http://qianjl.com/cross/data",
                    type:"get",
                    data:{date:new Date().getTime()},
                    dataType:"json"
                };
                message_iframe.contentWindow.postMessage(JSON.stringify(message),"http://qianjl.com");
            });
        });
    </script>
</head>
<body>

<table>
    <tr>
        <td>发送请求</td>
        <td><button id="request">发送</button></td>
    </tr>
    <tr>
        <td>响应内容</td>
        <td><div id="message"></div></td>
    </tr>
</table>

</body>
</html>
{% endhighlight %}

被跨域访问的服务端代理页面
{% highlight html %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>代理页面</title>
    <script src="http://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"></script>
    <script>
        var proxy_origin = "http://luantt.com";
        //监听被代理页面发送的请求
        window.addEventListener('message',function(event){
            //监听消息
            var origin = event.origin || event.originalEvent.origin; // For Chrome, the origin property is in the event.originalEvent object.
            if (origin !== proxy_origin)
                return;
            //获取消息，请求资源，并将请求的资源发送到被代理页面
            var proxy_data = JSON.parse(event.data);
            $.ajax({
                url:proxy_data.url,
                type:proxy_data.type?proxy_data.type:"get",
                data:proxy_data.data?proxy_data.data:{},
                dataType:proxy_data.dataType?proxy_data.dataType:"json",
                success:function(response){
                    event.source.postMessage(JSON.stringify(response),"http://luantt.com");
                },
                error:function(XMLHttpRequest, textStatus, errorThrown){
                    console.log(errorThrown);
                }
            });

        },false);
    </script>
</head>
<body>

</body>
</html>
{% endhighlight %}

上面只是简单的实现过程，对IE8不支持，在github有功能相似，但功能更强的js脚本：https://github.com/jpillora/xdomain 很简单就可以实现跨域


### 2.2使用webSocket跨域

需要跨域的页面：
{% highlight html %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>webSocket</title>
    <script src="http://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"></script>
    <script>
        $(function(){

            var ws = new WebSocket("ws://qianjl.com/socket");

            ws.onopen = function(even){
                console.log("通信打开");
            }
            ws.onmessage = function(even){
                console.log("获取消息：" + even.data);
                $("#webSocket_message").html(even.data);
            }
            ws.onerror = function(msg, url, lineNo, columnNo, error){
                console.log("通信错误：");
                var string = msg.toLowerCase();
                var substring = "script error";
                if (string.indexOf(substring) > -1){
                    alert('Script Error: See Browser Console for Detail');
                } else {
                    var message = [
                        'Message: ' + msg,
                        'URL: ' + url,
                        'Line: ' + lineNo,
                        'Column: ' + columnNo,
                        'Error object: ' + JSON.stringify(error)
                    ].join(' - ');

                    alert(message);
                }

                return false;
            }
            ws.onclose = function(even){
                console.log("通信关闭");
            }

            $("#webSocket_request").click(function(){
                ws.send("webSocket 通信。。。");
            });
        });
    </script>
</head>
<body>

<table>
    <tr>
        <td>webSocket发送请求</td>
        <td><button id="webSocket_request">发送</button></td>
    </tr>
    <tr>
        <td>webSocket响应内容</td>
        <td><div id="webSocket_message"></div></td>
    </tr>
</table>

</body>
</html>
{% endhighlight %}

服务端代码：
{% highlight html %}
@RestController
public class MyTextWebSocketHandler extends TextWebSocketHandler
{
    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception
    {
        HttpHeaders handshakeHeaders = session.getHandshakeHeaders();
        for (Map.Entry<String, List<String>> handshakeHeader : handshakeHeaders.entrySet())
        {
            String key = handshakeHeader.getKey();
            List<String> value = handshakeHeader.getValue();
            for (String string : value)
            {
                System.out.println(key + " : " + string);
            }
        }
        Map<String, Object> attributes = session.getAttributes();
        for (Map.Entry<String, Object> aStringObjectEntry : attributes.entrySet())
        {
            System.out.println(aStringObjectEntry.getKey() + " : " + aStringObjectEntry.getValue());
        }
        System.out.println(session.getId());
        WebSocketMessage webSocketMessage = new TextMessage("接收到消息。。。");
        session.sendMessage(webSocketMessage);
        System.out.println(message.getPayload());
    }
}
{% endhighlight %}

{% highlight xml %}
    <!--WebSocket引擎配置-->
    <bean id="servletServerContainerFactoryBean" class="org.springframework.web.socket.server.standard.ServletServerContainerFactoryBean">
        <!--异步发送超时时间 毫秒-->
        <property name="asyncSendTimeout" value="60000"></property>
        <!--会话超时时间 毫秒-->
        <property name="maxSessionIdleTimeout" value="6000000"></property>
        <!--字符串最大缓存大小-->
        <property name="maxTextMessageBufferSize" value="65535"></property>
        <!--二进制流最大缓存大小-->
        <property name="maxBinaryMessageBufferSize" value="65535"></property>
    </bean>

    <websocket:handlers allowed-origins="http://luantt.com">
        <websocket:mapping path="/socket" handler="myTextWebSocketHandler"/>
        <websocket:handshake-interceptors>
            <bean class="org.springframework.web.socket.server.support.HttpSessionHandshakeInterceptor">
                <property name="copyAllAttributes" value="true"></property>
                <property name="createSession" value="true"></property>
            </bean>
        </websocket:handshake-interceptors>
    </websocket:handlers>
{% endhighlight %}

使用nginx时，需要增加：
{% highlight xml %}
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $http_connection;
{% endhighlight %}

[github][github]

[github]: https://github.com/jlqian