# 包结构

- common
  - 通用工具类，Stringuti或者JDK的增强类FastThreadLocal
- buffer netty自己实现的ByteBuffer缓冲区
- transport 核心包 定义了传输层的接口和类
  - 实现了不同的协议 sctp,http,ftp
- handler 内置的通道处理器 提供IP过滤，SSL等处理类
- codec 协议编码的抽象和实现，包括JSON，base64
- netty-example 提供使用demo

# ByteBuf

- 池化/非池化
- Heap/Direct
- Safe/Unsafe

# 对应关系

- 一个channel对应一个ChannerlPipeline
- ChannerlPipeline包含双向的ChannelHandlerContext链
- 一个ChannelHandlerContext包含一个ChannelHandler
- 一个Channel会绑定到一个EventLooping上
- 一个NioEventLoop维护一个Selector
- 一个NIOEventLoop相当于一个线程

