# 2.9.7、play2：播放2

与播放命令的不同之处在于，播放 2 命令可以在不修改播放内容时间线的前提下切换到一个不同码率的流。服务器包含了多个不同码率的流文件用于支持客户端的播放 2 请求。

从客户端发送给服务器的命令结构如下：

```java
    The command structure from the client to the server is as follows:
    +--------------+----------+----------------------------------------+
    | Field Name   |   Type   |             Description                |
    +--------------+----------+----------------------------------------+
    | Command Name |  String  | Name of the command, set to "play2".   |
    +--------------+----------+----------------------------------------+
    | Transaction  |  Number  | Transaction ID set to 0.               |
    | ID           |          |                                        |
    +--------------+----------+----------------------------------------+
    | Command      |   Null   | Command information does not exist.    |
    | Object       |          | Set to null type.                      |
    +--------------+----------+----------------------------------------+
    | Parameters   |  Object  | An AMF encoded object whose properties |
    |              |          | are the public properties described    |
    |              |          | for the flash.net.NetStreamPlayOptions |
    |              |          | ActionScript object.                   |
    +--------------+----------+----------------------------------------+
```

有关 NetStreamPlayOptions 对象的公开属性的说明详见 AS3 语言的文档。 此命令的消息流如下所示：

```
        +--------------+                          +-------------+
        | Play2 Client |              |           |    Server   |
        +--------+-----+              |           +------+------+
                 |      Handshaking and Application      |
                 |               connect done            |
                 |                    |                  |
                 |                    |                  |
                 |                    |                  |
                 |                    |                  |
        ---+---- |---- Command Message(createStream) --->|
    Create |     |                                       |
    Stream |     |                                       |
        ---+---- |<---- Command Message (_result) -------|
                 |                                       |
        ---+---- |------ Command Message (play) -------->|
           |     |                                       |
           |     |<------------ SetChunkSize ------------|
           |     |                                       |
           |     |<--- UserControl (StreamIsRecorded)----|
      Play |     |                                       |
           |     |<------- UserControl (StreamBegin)-----|
           |     |                                       |
           |     |<--Command Message(onStatus-playstart)-|
           |     |                                       |
           |     |<---------- Audio Message -------------|
           |     |                                       |
           |     |<---------- Video Message -------------|
           |     |                                       |
                 |                                       |
        ---+---- |-------- Command Message(play2) ------>|
           |     |                                       |
           |     |<------- Audio Message (new rate) -----|
     Play2 |     |                                       |
           |     |<------- Video Message (new rate) -----|
           |     |                    |                  |
           |     |                    |                  |
           |  Keep receiving audio and video stream till finishes
                                      |
                  Message flow in the play2 command
```
