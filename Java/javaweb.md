## servlet

​	在java Web程序中，Servlet是用来接受`HttpServletRequest`请求的,通过service()来进行相应的处理，最后会返回一个HttpServletResponse给用户。

​	**一个Servlet可以设置多个UrI访问，Servlet不是线程安全的。**

### Servlet和CGI

#### CGI

1.给每个请求启动一个CGI程序的系统进程，要是请求的多，开销过大

2.需要重复的编写网络协议的代码和编码 耗时

#### Servlet

1.只需要启动一个操作系统进程以及加载一个jvm，降低了系统开销

2.如果要做相同的请求时。只需要加载一个类

3.能直接和web服务器进行交互，还能在各个程序之间共享数据

## Servlet生命周期

servlet方法：

```java
- `void init(ServletConfig config) throws ServletException`
- `void service(ServletRequest req, ServletResponse resp) throws - ServletException, java.io.IOException`
- `void destroy()`
- `java.lang.String getServletInfo()`
- `ServletConfig getServletConfig()`
```

**生命周期：** **Web容器加载Servlet并将其实例化后，Servlet生命周期开始**，

容器运行其**init()方法**进行Servlet的初始化；请求到达时调用Servlet的**service()方法**，

service()方法会根据需要调用与请求对应的**doGet或doPost**等方法；

当服务器关闭或项目被卸载时服务器会将Servlet实例销毁，此时会调用Servlet的**destroy()方法**。**init方法和destroy方法只会执行一次，service方法客户端每次请求Servlet都会执行**。

Servlet中有时会用到一些需要初始化与销毁的资源，因此可以把初始化资源的代码放入init方法中，销毁资源的代码放入destroy方法中，这样就不需要每次处理客户端的请求都要初始化与销毁资源。

## get和post的区别

 GET请求会被浏览器主动cache，而POST不会 

 GET请求只能进行url编码，而POST支持多种编码方式。 

 GET请求在URL中传送的参数是有长度限制的，而POST么有。 

 GET比POST更不安全，因为参数直接暴露在URL上，所以不能用来传递敏感信息。
GET参数通过URL传递，POST放在Request body中。

tcp来说：

 GET产生一个TCP数据包；POST产生两个TCP数据包。 

- 对于GET方式的请求，浏览器会把http header和data一并发送出去，服务器响应200（返回数据）；

- 而对于POST，浏览器先发送header，服务器响应100 continue，浏览器再发送data，服务器响应200 ok（返回数据）。

## 转发和重定向

**转发是服务器行为，重定向是客户端行为。**

- **转发**（Forward）通过RequestDispatcher对象的forward（HttpServletRequest request,HttpServletResponse response）方法实现的。

-  **重定向（Redirect）** 是利用服务器返回的状态码来实现的。客户端浏览器请求服务器的时候，服务器会返回一个状态码。服务器通过 `HttpServletResponse` 的 `setStatus(int status)` 方法设置状态码。如果服务器返回301或者302，则浏览器会到新的网址重新请求该资源。 

forward:一般用于用户登陆的时候,根据角色转发到相应的模块. redirect:一般用于用户注销登陆时返回主页面和跳转到其它的网站等

## 自动刷新

自动刷新不仅可以实现一段时间之后自动跳转到另一个页面，还可以实现一段时间之后自动刷新本页面。Servlet中通过HttpServletResponse对象设置Header属性实现自动刷新例如：

```java
Response.setHeader("Refresh","5;URL=http://localhost:8080/servlet/example.htm");
```

其中5为时间，单位为秒。URL指定就是要跳转的页面（如果设置自己的路径，就会实现每过5秒自动刷新本页面一次）

## Jsp和Servlet

​		Servlet是一个特殊的Java程序，它运行于服务器的JVM中，能够依靠服务器的支持向浏览器提供显示内容。JSP本质上是Servlet的一种简易形式，JSP会被服务器处理成一个类似于Servlet的Java程序，可以简化页面内容的生成。Servlet和JSP最主要的不同点在于，Servlet的应用逻辑是在Java文件中，并且完全从表示层中的HTML分离开来。而JSP的情况是Java和HTML可以组合成一个扩展名为.jsp的文件。有人说，Servlet就是在Java中写HTML，而JSP就是在HTML中写Java代码，当然这个说法是很片面且不够准确的。JSP侧重于视图，Servlet更侧重于控制逻辑，在MVC架构模式中，JSP适合充当视图（view）而Servlet适合充当控制器（controller）。

## Jsp的工作原理

jsp是先编译后部署，客户端第一个访问jsp页面的时候，会生成class文件，其中生成的类继承HttpJspBase

，通过这个class来提供服务.

如果jsp页面发生改动，tomcat会在下次请求的时候，重新编译jsp文件

## jsp内置对象

JSP有9个内置对象：

- request：封装客户端的请求，其中包含来自GET或POST请求的参数；
- response：封装服务器对客户端的响应；
- pageContext：通过该对象可以获取其他对象；
- session：封装用户会话的对象；
- application：封装服务器运行环境的对象；
- out：输出服务器响应的输出流对象；
- config：Web应用的配置对象；
- page：JSP页面本身（相当于Java程序中的this）；
- exception：封装页面抛出异常的对象。

## request.getAttribute()和 request.getParameter()有何区别

`getParameter()`是获取 POST/GET 传递的参数值；

`getAttribute()`是获取对象容器中的数据值；

 1）request.getParameter()取得是通过容器的实现来取得通过类似post，get等方式传入的数据，request.setAttribute()和getAttribute()只是在web容器内部流转，仅仅是请求处理阶段。
（2）request.getParameter()方法传递的数据，会从Web客户端传到Web服务器端，代表HTTP请求数据。request.getParameter()方法返回String类型的数据。
request.setAttribute()和getAttribute()方法传递的数据只会存在于Web容器内部 

## 讲解JSP中的四种作用域

JSP中的四种作用域包括page、request、session和application，具体来说：

- **page**代表与一个页面相关的对象和属性。
- **request**代表与Web客户机发出的一个请求相关的对象和属性。一个请求可能跨越多个页面，涉及多个Web组件；需要在页面显示的临时数据可以置于此作用域。
- **session**代表与某个用户与服务器建立的一次会话相关的对象和属性。跟某个用户相关的数据应该放在用户自己的session中。
- **application**代表与整个Web应用程序相关的对象和属性，它实质上是跨越整个Web应用程序，包括多个页面、请求和会话的一个全局作用域。