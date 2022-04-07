# 2.9、命令消息 Command Message （type ID = 17 或 20）

  Command Message（命令消息，Message Type ID = 17 或 20）：RTMP 通过发送命令消息命令对端进行特定操作, 表示在客户端和服务器间传递的在对端执行某些操作的命令消息，connect 表示连接对端，对端如果同意连接的话就会记录发送端信息并返回连接成功消息，publish 表示开始向对方推流，接收端接收到命令后准备好接收对端发送的流信息。当信息使用 AMF0 编码时，Message Type ID = 20，AMF3 编码时 为 17。

>   服务器和客户端之间使用 AMF 编码的命令消息交互。 一些命令消息被用来发送操作指令，比如 connect，createStream,public，play，pause。另外一些命令消息被用来通知发送方请求命令的状态，比如 onstatus，result 等。一条命令消息包括命令对称、交互 ID、包含相关参数的命令对象。服务器和客户端通过在创建的流中远程调用的方式，使用命令消息来进行交互。

服务器发送给客户端的命令结构如下：

```java
    +--------------+----------+----------------------------------------+
    | Field Name   |   Type   |             Description                |
    +--------------+----------+----------------------------------------+
    | Command Name |  String  | _result or _error; indicates whether   |
    |              |          | the response is result or error.       |
    +--------------+----------+----------------------------------------+
    | Transaction  |  Number  | Transaction ID is 1 for connect        |
    | ID           |          | responses                              |
    |              |          |                                        |
    +--------------+----------+----------------------------------------+
    | Properties   |  Object  | Name-value pairs that describe the     |
    |              |          | properties(fmsver etc.) of the         |
    |              |          | connection.                            |
    +--------------+----------+----------------------------------------+
    | Information  |  Object  | Name-value pairs that describe the     |
    |              |          | response from|the server. ’code’,      |
    |              |          | ’level’, ’description’ are names of few|
    |              |          | among such information.                |
    +--------------+----------+----------------------------------------+
```

命令执行过程中的消息流如下:

客户端发送连接命令给服务器，获得与服务器连接的实例。 服务器在接收到连接命令后，发送应答窗口大小的消息给客户端。同时与连接命令中接到的应用建立连接。 服务器发送设置流带宽消息给客户端。 客户端在接收并处理了设置流带宽的消息后，发送应答窗口大小的消息给服务器。 服务器接着发送开始流的用户控制消息给客户端。 服务器发送 result 命令消息给客户端，通知连接状态是成功或失败。命令消息中包含了事务ID。消息中还包含了像 FMS\
版本之类的属性，以及级别，编码，描述，对象编码等信息。

```java
    +--------------+                              +-------------+
    |    Client    |             |                |    Server   |
    +------+-------+             |                +------+------+
           |              Handshaking done               |
           |                     |                       |
           |                     |                       |
           |                     |                       |
           |                     |                       |
           |----------- Command Message(connect) ------->|
           |                                             |
           |<------- Window Acknowledgement Size --------|
           |                                             |
           |<----------- Set Peer Bandwidth -------------|
           |                                             |
           |-------- Window Acknowledgement Size ------->|
           |                                             |
           |<------ User Control Message(StreamBegin) ---|
           |                                             |
           |<------------ Command Message ---------------|
           |       (_result- connect response)           |
           |                                             |
                 Message flow in the connect command
```



客户端和服务器通过 AMF 编码的数据交换命令。发送者发送包含命令名称，事务ID，包含相关参数的命令对象的消息。例如，通过连接命令中包含的 APP 参数来告诉服务器连接的对方是哪个客户端。接收方处理命令消息，并使用相同的事务ID应答。应答字符串为 \_result 或 \_error 或方法名，例如 verifyClient 或 contactExternalServer。事务 ID 标明了应答指向的命令。事务ID相当于 IMAP 协议或其他协议中的标签。命令字符串中的方法名，表明了发送端想要在接收端执行的方法。

下面的类对象被用来发送各种命令：

* NetConnection：服务器和客户端之间进行网络连接的一种高级表示形式。
* NetStream：代表了发送音频流，视频流，或其他相关数据的频道。当然还有一些像播放，暂停之类的命令，用来控制数据流。
