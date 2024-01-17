---
cover: /assets/image/BE027.jpg
poster:
  headline: Spring-WebSocket实现长连接
  color: white
title: Spring-WebSocket实现长连接
tags:
  - Java
categories:
  - 后端
abbrlink: ba95dc11
date: 2024-01-07 17:17:21
---
#### 前言
项目要求前端去实时的访问接口拿取服务器时间，于是就想通过{% mark websocket %}去做一个长连接，websocket用Tomcat与Spring都可以做，这里只演示Spring的方式

#### 环境准备
```xml
	<!-- webSocket的依赖 -->
	<dependency>
	   <groupId>org.springframework.boot</groupId>
	   <artifactId>spring-boot-starter-websocket</artifactId>
	   <version>2.7.12</version>
	</dependency>

```
#### SpringWebSocket代码
##### 1. 添加拦截器
SpringWebsocket需要使用拦截器获取请求路径中的参数。

```java
  
import org.springframework.http.server.ServerHttpRequest;  
import org.springframework.http.server.ServerHttpResponse;  
import org.springframework.http.server.ServletServerHttpRequest;  
import org.springframework.stereotype.Component;  
import org.springframework.web.socket.WebSocketHandler;  
import org.springframework.web.socket.server.support.HttpSessionHandshakeInterceptor;  
  
import javax.servlet.http.HttpServletRequest;  
import java.util.Map;  
  
/**  
 * 因为 WebSocketSession 无法获得 ws 地址上的请求参数，所以只好通过该拦截器，获得 accessToken 请求参数，设置到 attributes 中。  
 * @author: Ban  
 * @date: 2024/1/7 17:13  
 * @param:  
 * @return:  
 */  
@Component  
public class WebSocketShakeInterceptor extends HttpSessionHandshakeInterceptor {  

    @Override  
    public boolean beforeHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler, Map<String, Object> attributes) throws Exception {  
        // 从请求路径中获得 accessToken        
        if (request instanceof ServletServerHttpRequest) {  
            ServletServerHttpRequest serverRequest = (ServletServerHttpRequest) request;  
            HttpServletRequest servletRequest = serverRequest.getServletRequest();  
            attributes.put("accessToken", servletRequest.getParameter("accessToken"));  
        }  
        // 调用父方法，继续执行逻辑  
        return super.beforeHandshake(request, response, wsHandler, attributes);  
    }  
  
}
```
##### 2. 添加消息处理器
添加对事件的处理逻辑，可按照自己的需求来
```java

import org.slf4j.Logger;  
import org.slf4j.LoggerFactory;  
import org.springframework.stereotype.Component;  
import org.springframework.web.socket.CloseStatus;  
import org.springframework.web.socket.TextMessage;  
import org.springframework.web.socket.WebSocketSession;  
import org.springframework.web.socket.handler.TextWebSocketHandler;  
  
import java.text.SimpleDateFormat;  
import java.util.Date;  
  
/**  
 * 处理连接和消息  
 * @author: Ban  
 * @date: 2024/1/7 17:10  
 */@Component  
public class DemoWebSocketHandler extends TextWebSocketHandler {  

    private Logger logger = LoggerFactory.getLogger(getClass());
  
    /**  
     * 对应 open 事件  
     * @author: Ban  
     * @date: 2024/1/7 17:11  
     * @param: [session]  
     * @return: void  
     */
    @Override  
    public void afterConnectionEstablished(WebSocketSession session) throws Exception {  
        logger.info("[afterConnectionEstablished][session({}) 接入]", session);  
        // 解析 accessToken 
        String accessToken = (String) session.getAttributes().get("accessToken");  
        System.out.println("连接建立，token：" + accessToken);  
    }  
  
    /**  
     * 对应 message 事件  
     * @author: Ban  
     * @date: 2024/1/7 17:11  
     * @param: [session, textMessage]  
     * @return: void  
     */
    @Override  
    public void handleTextMessage(WebSocketSession session, TextMessage textMessage) throws Exception {  
        logger.info("[handleMessage][session({}) 接收到一条消息({})]", session, textMessage);  
        // 业务需求：前端实时访问接口获取服务器时间  
        Date date = new Date();  
        SimpleDateFormat dateFormat= new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");  
        session.sendMessage(new TextMessage(dateFormat.format(date)));  
    }  
  
    /**  
     * 对应 close 事件  
     * @author: Ban  
     * @date: 2024/1/7 17:11  
     * @param: [session, status]  
     * @return: void  
     */
    @Override  
    public void afterConnectionClosed(WebSocketSession session, CloseStatus status) throws Exception {  
        logger.info("[afterConnectionClosed][session({}) 连接关闭。关闭原因是({})}]", session, status);  
        System.out.println(session.getAttributes().get("accessToken").toString() + "的连接已关闭。");  
    }  
  
    /**  
     * 对应 error 事件  
     * @author: Ban  
     * @date: 2024/1/7 17:11  
     * @param: [session, exception]  
     * @return: void  
     */
    @Override  
    public void handleTransportError(WebSocketSession session, Throwable exception) throws Exception {  
        logger.info("[handleTransportError][session({}) 发生异常]", session, exception);  
        System.out.println("连接异常！！");  
    }  
  
}
```

##### 3. 添加配置文件
将前面定义的拦截器和消息处理器添加到配置中。
```java

import com.vt.common.security.handler.DemoWebSocketHandler;  
import com.vt.common.security.interceptor.WebSocketShakeInterceptor;  
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Configuration;  
import org.springframework.context.annotation.Primary;  
import org.springframework.scheduling.TaskScheduler;  
import org.springframework.scheduling.concurrent.ThreadPoolTaskScheduler;  
import org.springframework.web.socket.config.annotation.EnableWebSocket;  
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;  
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;  
  
/**  
 * 开启 springWebsocket  
 * @author: Ban  
 * @date: 2024/1/7 17:05  
 */
@Configuration  
@EnableWebSocket  
public class WebSocketConfiguration implements WebSocketConfigurer {  

	/**  
	 * addHandler：配置处理器  
	 * addInterceptors：配置拦截器  
	 * setAllowedOrigins: 设置跨域  
	 * @author: Ban  
	 * @date: 2024/1/7 17:09  
	 * @param: [webSocketHandlerRegistry]  
	 * @return: void  
	 */    
	@Override  
	public void registerWebSocketHandlers(WebSocketHandlerRegistry webSocketHandlerRegistry) {  
		webSocketHandlerRegistry  
				// 前端请求端口路径  
				.addHandler(new DemoWebSocketHandler(), "/springWebsocket")  
				.addInterceptors(new WebSocketShakeInterceptor())  
				.setAllowedOrigins("*");  
	}  
	
	/**  
	 * 注解: @Primary 是为了解决找不到bean的bug，指定优先级  
	 * @author: Ban  
	 * @date: 2024/1/7 17:09  
	 * @param: []  
	 * @return: org.springframework.scheduling.TaskScheduler  
	 */    
	@Bean  
	@Primary
	public TaskScheduler taskScheduler() {  
		ThreadPoolTaskScheduler taskScheduler = new ThreadPoolTaskScheduler();  
		taskScheduler.setPoolSize(10);  
		taskScheduler.initialize();  
		return taskScheduler;  
	}  
  
}
```

#### 前端示例代码
```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="referrer" content="no-referrer">
    <title>websocket案例</title>
<style>
body{
    background: rgb(187,231,152);
    background: radial-gradient(circle, rgba(187,231,152,1) 38%, rgba(232,225,133,1) 100%);
    background-size: cover;
}
#sendMessage{
    height: 30px;
    color:brown;
}
#closeWebSocket{
    height: 30px;
    color: red;
}
#show{
    width: 500px;
    margin: auto;
    text-align: left;
    opacity: 0.7;
}
</style>
</head>
<body>
    <div style="width: 100%;text-align: center;color:black; background: none; opacity: 0.7;">
       <h1>websocket测试用例</h1>
       <button id="sendMessage">点击发送消息</button>
       <button id="closeWebSocket">点击关闭tcp连接</button>
       <div id="show">
            <h4 id="onopen">onopen:</h4>
            <h4 id="onmessage">onmessage:</h4>
            <h4 id="onerror">onerror:</h4>
            <h4 id="onclose">onclose:</h4>
       </div>
    </div>
    <script>
        //和websocket服务端建立tcp连接,刷新页面触发
        var ws = new WebSocket("ws://127.0.0.1:9999/exam/springWebsocket?accessToken=Bearer eea8bfa1-c7ac-428f-9ce0-749f8a1b4744"); //连接springWebsocket
        /* websocket客户端部分API如下 */

        // 1.连接建立时触发
        ws.onopen = function(){
            //send方法用于向服务器发送消息
            ws.send('这是客户端在tcp连接建立后发送的消息');
        let onopen = document.getElementById('onopen');
        onopen.innerText = 'onopen: ' + "webSocket客户端已和服务端创建tcp连接。";
        };
        // 2.接收到webSocket服务端消息就触发
        ws.onmessage = function(evt){
            let onmessage = document.getElementById('onmessage');
            onmessage.innerText = 'onmessage: ' + evt.data;
        };
        //3.出现异常时触发
        ws.onerror = function(evt){
            let onerror = document.getElementById('onerror');
            onerror.innerText = 'onerror: ' + "webSocket发生异常!";
        };
        // 4.连接关闭时触发
        ws.onclose = function(evt){
            let onclose = document.getElementById('onclose');
            onclose.innerText = 'onclose: ' + "webSocket的tcp连接已关闭。";
        };
        //点击发送消息按钮
        let sendMessageButton = document.getElementById('sendMessage');
        sendMessageButton.onclick = function () {
            ws.send('客户端发送了消息到服务器端');
        }
        //点击关闭tcp连接按钮
        let closeButton = document.getElementById('closeWebSocket');
        closeButton.onclick = function(){
            //关闭websocket的tcp连接
            ws.close();
        }
    </script>
</body>
</html>
```

#### Bug优化
然后在这其中也遇到了一些bug，在此做记录
##### 1. 缺少依赖
> {% checkbox symbol:times color:red checked:true Unable to start ServletWebServerApplicationContext due to missing ServletWebServerFactory bean %}

更新 Maven 依赖项以排除spring-webmvc
```xml
<!--web 模块-->  
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-web</artifactId>  
    <version>${spring-boot.version}</version>  
    <exclusions>  
        <!--排除tomcat依赖-->  
        <exclusion>  
            <artifactId>spring-boot-starter-tomcat</artifactId>  
            <groupId>org.springframework.boot</groupId>  
        </exclusion>  
    </exclusions>  
</dependency>
```
或者将 web 应用程序类型设置为`WebApplicationType.REACTIVE`，具体如下所示
```java
public class GatewayApp {
    public static void main(String[] args) {
        SpringApplication application = new SpringApplication(GatewayApp.class);
      	// 该设置方式 也可以解决我的前一个问题
        application.setWebApplicationType(WebApplicationType.REACTIVE);
        application.run(args);
    }
}

```

##### 2. 同时使用定时任务和websocket报错
> {% checkbox symbol:times color:red checked:true Bean named 'defaultSockJsTaskScheduler' is expected to be of type 'org.springframework.scheduling' %}

启动类Application中加入task的initialize。
> 注意：如果继续报错，报错信息如下。报错信息解读：发现两个定时任务的bean，不知道使用哪一个，springboot报错，这时候在 taskScheduler方法中加上@Primary注解，告诉springboot使用这个自定义的定时任务

```java
    @Primary
    @Bean
    public TaskScheduler taskScheduler(){
        ThreadPoolTaskScheduler taskScheduler = new ThreadPoolTaskScheduler();
        //只有池子里的任务有执行结束后，池子之外的任务才有机会被加入执行。
        // 更糟的情况是，当池子里的任务都在因为异常或业务要求（比如出错无限重试）而导致池子永远无法得到释放，将导致固定值之外的任务永远不会被执行！
        //taskScheduler.setPoolSize允许动态设置池子的大小,可动态设置-> todo 有隐患
        taskScheduler.setPoolSize(10);
        taskScheduler.initialize();
        return taskScheduler;
    }
```