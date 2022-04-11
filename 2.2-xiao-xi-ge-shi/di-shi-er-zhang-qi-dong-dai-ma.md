# 第十二章：启动代码

### 初始化启动参数

Netty的boss线程池和work 线程池组,这里通过配置判断是否选择epoll ，epoll 针对不通平台提供了特定的优化。如下所示：

```
     /**
     * init
     */
    public void initArgPost() {
        //判断是否使用 epoll
        if (LiveConfig.useEpoll) {
            log.info("Netty is using Epoll");
            workerGroup = new NioEventLoopGroup(new DefaultThreadFactory("work"));
            bossGroup = new NioEventLoopGroup(new DefaultThreadFactory("boss"));
            channelClass = NioServerSocketChannel.class;
        } else {
            log.info("Netty is using NIO");
            workerGroup = new NioEventLoopGroup(new DefaultThreadFactory("work"));
            bossGroup = new NioEventLoopGroup(new DefaultThreadFactory("boss"));
            channelClass = EpollServerSocketChannel.class;
        }
        executor = new DefaultEventExecutorGroup(handlerThreadPoolSize);
    }
```

### 启动

启动这边主要是设置netty 的几个参数，这里主要 ChannelOption.SO\_REUSEADDR 设置为true，ChannelOption.SO\_BACKLOG 设置为128 ，

添加netty 的日志组件，方便调试.

```
addLast("logger", new LoggingHandler("Netty", LogLevel.INFO))
```



```
    /**
     * 运行启动方法。
     *
     * @throws Exception e
     */
    public void bootstrap() throws Exception {
        ServerBootstrap b = new ServerBootstrap();
        b.group(bossGroup, workerGroup)
                .channel(channelClass)
                //设置控制tcp 三次握手过程中全链接队列大小。
                .option(ChannelOption.SO_BACKLOG, 128)

                //设置地址可重用（作用是尽早的让地址可用）
                .option(ChannelOption.SO_REUSEADDR, true)

                .childHandler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    public void initChannel(SocketChannel ch) throws Exception {
                        ch.pipeline()
                                .addLast("logger", new LoggingHandler("Netty", LogLevel.INFO))
                                .addLast("handshake", new HandShakeHandler())
                                .addLast("decoder", new ChunkDecoder())
                                .addLast("encoder", new ChunkEncoder())
                                .addLast(executor, "", new RtmpMessageHandler());
                    }
                });

        channelFuture = b.bind(port).sync();

        log.info("RTMP Server start , listen at :{}", port);

    }

```

### 完整代码如下

```

/**
 * rtmp 服务器
 *
 * @author user
 */
@Slf4j
public class RTMPBootstrap {

    private final int handlerThreadPoolSize;
    private final int port;
    /**
     * 记录启动结果
     */
    private ChannelFuture channelFuture;
    /**
     * 工作线程池
     */
    private EventLoopGroup workerGroup;
    /**
     * 接受链接线程池
     */
    private EventLoopGroup bossGroup;

    /**
     * 指定channel 的创建工厂类型
     */
    private Class<? extends ServerSocketChannel> channelClass;

    /**
     * 处理线程池
     */
    private DefaultEventExecutorGroup executor;

    public RTMPBootstrap(int port, int threadPoolSize) {
        this.port = port;
        this.handlerThreadPoolSize = threadPoolSize;
        initArgPost();
    }


    /**
     * init
     */
    public void initArgPost() {
        //判断是否使用 epoll
        if (LiveConfig.useEpoll) {
            log.info("Netty is using Epoll");
            workerGroup = new NioEventLoopGroup(new DefaultThreadFactory("work"));
            bossGroup = new NioEventLoopGroup(new DefaultThreadFactory("boss"));
            channelClass = NioServerSocketChannel.class;
        } else {
            log.info("Netty is using NIO");
            workerGroup = new NioEventLoopGroup(new DefaultThreadFactory("work"));
            bossGroup = new NioEventLoopGroup(new DefaultThreadFactory("boss"));
            channelClass = EpollServerSocketChannel.class;
        }
        executor = new DefaultEventExecutorGroup(handlerThreadPoolSize);
    }

    /**
     * 运行启动方法。
     *
     * @throws Exception e
     */
    public void bootstrap() throws Exception {
        ServerBootstrap b = new ServerBootstrap();


        b.group(bossGroup, workerGroup)
                .channel(channelClass)
                //设置控制tcp 三次握手过程中全链接队列大小。
                .option(ChannelOption.SO_BACKLOG, 128)

                //设置地址可重用（作用是尽早的让地址可用）
                .option(ChannelOption.SO_REUSEADDR, true)

                .childHandler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    public void initChannel(SocketChannel ch) throws Exception {
                        ch.pipeline()
                                .addLast("logger", new LoggingHandler("Netty", LogLevel.INFO))
                                .addLast("handshake", new HandShakeHandler())
                                .addLast("decoder", new ChunkDecoder())
                                .addLast("encoder", new ChunkEncoder())
                                .addLast(executor, "", new RtmpMessageHandler());
                    }
                });

        channelFuture = b.bind(port).sync();

        log.info("RTMP Server start , listen at :{}", port);

    }

    public void close() {
        try {
            channelFuture.channel().closeFuture().sync();
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        } catch (Exception e) {
            log.error("close rtmp server failed", e);
        }
    }

}

```
