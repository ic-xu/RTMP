# 2.9.8、deleteStream:删除流

如果需要销毁网络流对象，可以通过网络流发送删除流消息给服务器。 客户端发送给服务器的命令结构如下：

```java
    The command structure from the client to the server is as follows:
    +--------------+----------+----------------------------------------+
    | Field Name   |   Type   |             Description                |
    +--------------+----------+----------------------------------------+
    | Command Name |  String  | Name of the command, set to            |
    |              |          | "deleteStream".                        |
    +--------------+----------+----------------------------------------+
    | Transaction  |  Number  | Transaction ID set to 0.               |
    | ID           |          |                                        |
    +--------------+----------+----------------------------------------+
    | Command      |  Null    | Command information object does not    |
    | Object       |          | exist. Set to null type.               |
    +--------------+----------+----------------------------------------+
    | Stream ID    |  Number  | The ID of the stream that is destroyed |
    |              |          | on the server.                         |
    +--------------+----------+----------------------------------------+
```

服务器接收到此消息后，不做任何回复。
