# 第十二章：启动代码

初始化Netty的boss线程池和work 线程池组，如下所示

```
/** * init */public void initArgPost() {    //判断是否使用 epoll    if (LiveConfig.useEpoll) {        log.info("Netty is using Epoll");        workerGroup = new NioEventLoopGroup(new DefaultThreadFactory("work"));        bossGroup = new NioEventLoopGroup(new DefaultThreadFactory("boss"));        channelClass = NioServerSocketChannel.class;    } else {        log.info("Netty is using NIO");        workerGroup = new NioEventLoopGroup(new DefaultThreadFactory("work"));        bossGroup = new NioEventLoopGroup(new DefaultThreadFactory("boss"));        channelClass = EpollServerSocketChannel.class;    }    executor = new DefaultEventExecutorGroup(handlerThreadPoolSize);}
```

&#x20;``&#x20;
