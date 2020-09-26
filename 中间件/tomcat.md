## tomcat

### tomcat所包含容器

Engine：表示整个Catalina servlet 引擎；
Host：表示包含有一个或多个 Context容器的虚拟主机；
Context：表示一个web 应用程序，一个Context 可以有多个 Wrapper；
Wrapper：表示一个独立的servlet；

以上4种容器都是 org.apache.catalina包下的接口：分别为Engine，Host， Context， Wrapper，他们都继承自Container接口。这4个接口的标准实现是 StandardEngine类，StandardHost类，StandardContext类，StandardWrapper类，他们都在 org.apache.catalina.core 包内；
Attention）



### io模型

**BIO**	阻塞式IO，采用传统的java IO进行操作，该模式下每个请求都会创建一个线程，适用于并发量小的场景

**NIO**	同步非阻塞，比传统BIO能更好的支持大并发，tomcat 8.0 后默认采用该模式
**APR**	tomcat 以JNI形式调用http服务器的核心动态链接库来处理文件读取或网络传输操作，需要编译安装APR库
**AIO**	异步非阻塞，tomcat8.0后支持

