# 2.9.10、receiveVideo: 接收视频

网络流发送此消息通知服务器，是否要发送视频数据给客户端。 客户端发送给服务器的命令结构如下：

```java
    +--------------+----------+----------------------------------------+
    | Field Name   |   Type   |             Description                |
    +--------------+----------+----------------------------------------+
    | Command Name |  String  | Name of the command, set to            |
    |              |          | "receiveVideo".                        |
    +--------------+----------+----------------------------------------+
    | Transaction  |  Number  | Transaction ID set to 0.               |
    | ID           |          |                                        |
    +--------------+----------+----------------------------------------+
    | Command      |  Null    | Command information object does not    |
    | Object       |          | exist. Set to null type.               |
    +--------------+----------+----------------------------------------+
    | Bool Flag    |  Boolean | true or false to indicate whether to   |
    |              |          | receive video or not.                  |
    +--------------+----------+----------------------------------------+
```

如果服务器接收到带有 false 标签的消息后，不做任何回复。如果接收到带有 true 标签的消息，服务器回复带有 NetStream.Seek.Notify 和 NetStream.Play.Start 的消息给客户端。
