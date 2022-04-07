# 2.9.9、receiveAudio: 接收音频

网络流发送此消息通知服务器，是否要发送音频数据给客户端。

```java
    The command structure from the client to the server is as follows:
    +--------------+----------+----------------------------------------+
    | Field Name   |   Type   |             Description                |
    +--------------+----------+----------------------------------------+
    | Command Name |  String  | Name of the command, set to            |
    |              |          | "receiveAudio".                        |
    +--------------+----------+----------------------------------------+
    | Transaction  |  Number  | Transaction ID set to 0.               |
    | ID           |          |                                        |
    +--------------+----------+----------------------------------------+
    | Command      |  Null    | Command information object does not    |
    | Object       |          | exist. Set to null type.               |
    +--------------+----------+----------------------------------------+
    | Bool Flag    |  Boolean | true or false to indicate whether to   |
    |              |          | receive audio or not.                  |
    +--------------+----------+----------------------------------------+
```

如果服务器接收到带有 fasle 标签的消息后，不做任何回复。如果接收到带有 true 标签的消息，服务器回复带有\
NetStream.Seek.Notify 和 NetStream.Play.Start 的消息给客户端。
