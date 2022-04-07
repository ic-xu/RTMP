# 2.9.11、publish: 发布

客户端发送此消息，用来发布一个有名字的流到服务器。其他客户端可以使用此流名来播放流，接收发布的音频，视频，以及其他数据消息。

客户端发送给服务器的命令结构如下：

```java
    +--------------+----------+----------------------------------------+
    | Field Name   |   Type   |             Description                |
    +--------------+----------+----------------------------------------+
    | Command Name |  String  | Name of the command, set to "publish". |
    +--------------+----------+----------------------------------------+
    | Transaction  |  Number  | Transaction ID set to 0.               |
    | ID           |          |                                        |
    +--------------+----------+----------------------------------------+
    | Command      |  Null    | Command information object does not    |
    | Object       |          | exist. Set to null type.               |
    +--------------+----------+----------------------------------------+
    | Publishing   |  String  | Name with which the stream is          |
    | Name         |          | published.                             |
    +--------------+----------+----------------------------------------+
    | Publishing   |  String  | Type of publishing. Set to "live",     |
    | Type         |          | "record", or "append".                 |
    |              |          | record: The stream is published and the|
    |              |          | data is recorded to a new file.The file|
    |              |          | is stored on the server in a           |
    |              |          | subdirectory within the directory that |
    |              |          | contains the server application. If the|
    |              |          | file already exists, it is overwritten.|
    |              |          | append: The stream is published and the|
    |              |          | data is appended to a file. If no file |
    |              |          | is found, it is created.               |
    |              |          | live: Live data is published without   |
    |              |          | recording it in a file.                |
    +--------------+----------+----------------------------------------+
```

服务器接收到此消息后，回复 onStatus 命令来标记发布的开始。

示例1：发布录制的视频 此示例阐述了发布者如果发布视频流到服务器。其他客户端可以订阅并播放此视频流。

```java
         +--------------------+                     +-----------+
         |  Publisher Client  |        |            |    Server |
         +----------+---------+        |            +-----+-----+
                    |           Handshaking Done          |
                    |                  |                  |
                    |                  |                  |
           ---+---- |----- Command Message(connect) ----->|
              |     |                                     |
              |     |<----- Window Acknowledge Size ------|
      Connect |     |                                     |
              |     |<-------Set Peer BandWidth ----------|
              |     |                                     |
              |     |------ Window Acknowledge Size ----->|
              |     |                                     |
              |     |<------User Control(StreamBegin)-----|
              |     |                                     |
           ---+---- |<---------Command Message -----------|
                    |   (_result- connect response)       |
                    |                                     |
           ---+---- |--- Command Message(createStream)--->|
       Create |     |                                     |
       Stream |     |                                     |
           ---+---- |<------- Command Message ------------|
                    | (_result- createStream response)    |
                    |                                     |
           ---+---- |---- Command Message(publish) ------>|
              |     |                                     |
              |     |<------User Control(StreamBegin)-----|
              |     |                                     |
              |     |-----Data Message (Metadata)-------->|
              |     |                                     |
    Publishing|     |------------ Audio Data ------------>|
      Content |     |                                     |
              |     |------------ SetChunkSize ---------->|
              |     |                                     |
              |     |<----------Command Message ----------|
              |     |      (_result- publish result)      |
              |     |                                     |
              |     |------------- Video Data ----------->|
              |     |                  |                  |
              |     |                  |                  |
                    |    Until the stream is complete     |
                    |                  |                  |
              Message flow in publishing a video stream
```

示例2：从录制的流发布元数据 本示例展示了发布元数据的消息交换过程。

```java
      +------------------+                       +---------+
      | Publisher Client |         |             |   FMS   |
      +---------+--------+         |             +----+----+
                |     Handshaking and Application     |
                |            connect done             |
                |                  |                  |
                |                  |                  |
        ---+--- |---Command Messsage(createStream) -->|
    Create |    |                                     |
    Stream |    |                                     |
        ---+--- |<---------Command Message------------|
                |   (_result - command response)      |
                |                                     |
        ---+--- |---- Command Message(publish) ------>|
Publishing |    |                                     |
  metadata |    |<------ UserControl(StreamBegin)-----|
 from file |    |                                     |
           |    |-----Data Message (Metadata) ------->|
                |                                     |
                    Publishing metadata
```
