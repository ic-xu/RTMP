# 第十四章：消息解码实现

### 消息格式预览

前面说到，RTMP 消息是按照 CHUNK 格式封装的，CHUNK 报文格式如下：

```java
    +--------------+----------------+--------------------+--------------+
    | Basic Header | Message Header | Extended Timestamp |  Chunk Data  |
    +--------------+----------------+--------------------+--------------+
    |                                                    |
    |<------------------- Chunk Header ----------------->|
                                Chunk Format
```

### 消息头数据结构

我们把除了数据以外的字段设置为消息头，则 RTMP 消息头数据结构对应的 java 代码如下：

```
@Data
public class RtmpHeader {

    //对应 Basic Header
    private BasicHeader basicHeader = new BasicHeader();

    // 对应 Message Header
    private MessageHeader messageHeader = new MessageHeader();

    //对应 Extended Timestamp
    private long extendedTimestamp;

    //获取一个时间差值
    private int timestampDelta;

    //外加一个字段记录消息体大小
    private int headerLength;
}
```
