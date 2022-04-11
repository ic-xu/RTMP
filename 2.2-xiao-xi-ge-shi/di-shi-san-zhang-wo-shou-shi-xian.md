# 第十三章：握手实现

### 流程：

rtmp 连接从握手开始。它包含三个固定大小的块。客户端发送的三个块命名为 C0,C1,C2；服务端发送的三个块命名为 S0,S1,S2。 握手序列：

* 客户端通过发送 C0 和 C1 消息来启动握手过程。客户端必须接收到 S1 消息，然后发送 C2 消息。客户端必须接收到S2 消息，然后发送其他数据。
* 服务端必须接收到 C0 或者 C1 消息，然后发送 S0 和 S1 消息。服务端必须接收到 C2 消息，然后发送其他数据。至此，握手过程完成。 详细流程示意图如下：

```java
    +-------------+                           +-------------+
    |    Client   |       TCP/IP Network      |    Server   |
    +-------------+            |              +-------------+
          |                    |                     |
    Uninitialized              |               Uninitialized
          |          C0        |                     |
          |------------------->|         C0          |
          |                    |-------------------->|
          |          C1        |                     |
          |------------------->|         S0          |
          |                    |<--------------------|
          |                    |         S1          |
     Version sent              |<--------------------|
          |          S0        |                     |
          |<-------------------|                     |
          |          S1        |                     |
          |<-------------------|                Version sent
          |                    |         C1          |
          |                    |-------------------->|
          |          C2        |                     |
          |------------------->|         S2          |
          |                    |<--------------------|
       Ack sent                |                  Ack Sent
          |          S2        |                     |
          |<-------------------|                     |
          |                    |         C2          |
          |                    |-------------------->|
     Handshake Done            |               Handshake Done
          |                    |                     |
              Pictorial Representation of Handshake
                         [译]握手示意图
```

### 流程解析：

客户端可以 C0和C1 一起发送，协议没有规定C0和C1的先后顺序，只规定了客户端必须要收到S1才能发送C2,收到S2才能发送其他数据。服务端收到C2才能发送其他数据，那么流程可以如下：

* 客户端先发送C0和C1
* 服务端收到C0和C1之后，发送S0、S1、S2.
* 客户端收到S2之后，发送C2(因为是基于TCP ,所以收到C2，那么C0和C1肯定收到了)

### 握手数据包格式

```

                 0 1 2 3 4 5 6 7
                +-+-+-+-+-+-+-+-+
                |   version     |
                +-+-+-+-+-+-+-+-+
                 C0 and S0 bits



              0                   1                   2                   3
              0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
             +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
             |                        time (4 bytes)                         |
             +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
             |                     version (4 bytes)                         |
             +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
             |                         key (764 bytes)                       |
             +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
             |                      digest (764 bytes)                       |
             +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                                      C1 and S1 bits
```

握手部分代码如下：

```
     // read c0 and c1
            //如果不够C0和C1 就暂时不读数据
            if (in.readableBytes() < VERSION_LENGTH + HANDSHAKE_LENGTH) {
                return;
            }

            //读取 C0 数据包
            byte clientRtmpVersion = in.readByte();
            log.info("client RTMP Version is {}", clientRtmpVersion);

            // 读取C1数据包
            in.readBytes(clientHandshake);

            //发送S0、S1、S2
            writeS0S1S2(ctx);
```

组装S0、S1、S2数据包。

```

    /**
     * write s0  s1 and S2
     *
     * @param ctx ChannelHandlerContext
     */
    private void writeS0S1S2(ChannelHandlerContext ctx) {
        // S0+S1+S2
        ByteBuf responseBuf = Unpooled.buffer(VERSION_LENGTH + HANDSHAKE_LENGTH + HANDSHAKE_LENGTH);
        // version = 3 
        responseBuf.writeByte(S0);
        // 写入S1 time 4bytes
        responseBuf.writeInt(0);
        // 写入S1 zero 4bytes
        responseBuf.writeInt(0);
        // s1 random bytes
        responseBuf.writeBytes(RandomGeneratorTools.generateRandomData(HANDSHAKE_LENGTH - 8));
        // 写入S2 time 4bytes
        responseBuf.writeInt(0);
        // 写入S2 zero 4bytes
        responseBuf.writeInt(0);
        // 写入S2 随机数。1526 bytes(764+764)
        responseBuf.writeBytes(RandomGeneratorTools.generateRandomData(HANDSHAKE_LENGTH - 8));

        ctx.writeAndFlush(responseBuf);
    }
```

### 完整代码和注释如下:

```
@Slf4j
public class HandShakeHandler extends ByteToMessageDecoder {


    /**
     * 4+4+764+764 = 1536
     **/
    private static final int HANDSHAKE_LENGTH = 1536;
    private static final int VERSION_LENGTH = 1;
    /**
     * server rtmp version=3
     **/
    static byte S0 = 3;
    boolean c0c1done;

    /**
     * store handshake message
     */
    byte[] clientHandshake = new byte[HANDSHAKE_LENGTH];
    boolean handshakeDone;

    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) {

        if (handshakeDone) {
            ctx.fireChannelRead(in);
            ctx.channel().pipeline().remove(this);
            return;
        }

        if (!c0c1done) {
            // read c0 and c1
            //如果不够C0和C1 就暂时不读数据
            if (in.readableBytes() < VERSION_LENGTH + HANDSHAKE_LENGTH) {
                return;
            }

            //读取 C0 数据包
            byte clientRtmpVersion = in.readByte();
            log.info("client RTMP Version is {}", clientRtmpVersion);

            // 读取C1数据包
            in.readBytes(clientHandshake);

            //发送S0、S1、S2
            writeS0S1S2(ctx);
            c0c1done = true;

        } else {
            // read c2
            if (in.readableBytes() < HANDSHAKE_LENGTH) {
                return;
            }
            // read s2
            in.readBytes(clientHandshake);

            // handshake done
            clientHandshake = null;
            handshakeDone = true;
            ctx.channel().pipeline().remove(this);
        }

    }

    /**
     * write s0  s1 and S2
     *
     * @param ctx ChannelHandlerContext
     */
    private void writeS0S1S2(ChannelHandlerContext ctx) {
        // S0+S1+S2
        ByteBuf responseBuf = Unpooled.buffer(VERSION_LENGTH + HANDSHAKE_LENGTH + HANDSHAKE_LENGTH);
        // version = 3
        responseBuf.writeByte(S0);
        // 写入S1 time 4bytes
        responseBuf.writeInt(0);
        // 写入S1 zero 4bytes
        responseBuf.writeInt(0);
        // s1 random bytes
        responseBuf.writeBytes(RandomGeneratorTools.generateRandomData(HANDSHAKE_LENGTH - 8));
        // 写入S2 time 4bytes
        responseBuf.writeInt(0);
        // 写入S2 zero 4bytes
        responseBuf.writeInt(0);
        // 写入S2 随机数。1526 bytes(764+764)
        responseBuf.writeBytes(RandomGeneratorTools.generateRandomData(HANDSHAKE_LENGTH - 8));

        ctx.writeAndFlush(responseBuf);
    }

}
```
