# 2.9.6、play:播放

客户端发送此命令来通知服务器开始播放流。多次使用此命令可以创建一个播放列表。如果想要创建一个动态播放列表来在不同的直播或点播流之间切换，可以通过多次调用播放命令，同时将 Reset 字段设置为 false。相反，如果想要立即播放指定的流，先清理掉之前的播放队列，再调用播放命令，同时将 Reset 字段设置为 true。

从客户端发送给服务器的命令结构如下：

```java
    +--------------+----------+-----------------------------------------+
    | Field Name   |   Type   |             Description                 |
    +--------------+----------+-----------------------------------------+
    | Command Name |  String  | Name of the command. Set to "play".     |
    +--------------+----------+-----------------------------------------+
    | Transaction  |  Number  | Transaction ID set to 0.                |
    | ID           |          |                                         |
    +--------------+----------+-----------------------------------------+
    | Command      |   Null   | Command information does not exist.     |
    | Object       |          | Set to null type.                       |
    +--------------+----------+-----------------------------------------+
    | Stream Name  |  String  | Name of the stream to play.             |
    |              |          | To play video (FLV) files, specify the  |
    |              |          | name of the stream without a file       |
    |              |          | extension (for example, "sample"). To   |
    |              |          | play back MP3 or ID3 tags, you must     |
    |              |          | precede the stream name with mp3:       |
    |              |          | (for example, "mp3:sample". To play     |
    |              |          | H.264/AAC files, you must precede the   |
    |              |          | stream name with mp4: and specify the   |
    |              |          | file extension. For example, to play the|
    |              |          | file sample.m4v,specify "mp4:sample.m4v"|
    |              |          |                                         |
    +--------------+----------+-----------------------------------------+
    | Start        |  Number  | An optional parameter that specifies    |
    |              |          | the start time in seconds. The default  |
    |              |          | value is -2, which means the subscriber |
    |              |          | first tries to play the live stream     |
    |              |          | specified in the Stream Name field. If a|
    |              |          | live stream of that name is not found,it|
    |              |          | plays the recorded stream of the same   |
    |              |          | name. If there is no recorded stream    |
    |              |          | with that name, the subscriber waits for|
    |              |          | a new live stream with that name and    |
    |              |          | plays it when available. If you pass -1 |
    |              |          | in the Start field, only the live stream|
    |              |          | specified in the Stream Name field is   |
    |              |          | played. If you pass 0 or a positive     |
    |              |          | number in the Start field, a recorded   |
    |              |          | stream specified in the Stream Name     |
    |              |          | field is played beginning from the time |
    |              |          | specified in the Start field. If no     |
    |              |          | recorded stream is found, the next item |
    |              |          | in the playlist is played.              |
    +--------------+----------+-----------------------------------------+
    | Duration     |  Number  | An optional parameter that specifies the|
    |              |          | duration of playback in seconds. The    |
    |              |          | default value is -1. The -1 value means |
    |              |          | a live stream is played until it is no  |
    |              |          | longer available or a recorded stream is|
    |              |          | played until it ends. If you pass 0, it |
    |              |          | plays the single frame since the time   |
    |              |          | specified in the Start field from the   |
    |              |          | beginning of a recorded stream. It is   |
    |              |          | assumed that the value specified in     |
    |              |          | the Start field is equal to or greater  |
    |              |          | than 0. If you pass a positive number,  |
    |              |          | it plays a live stream for              |
    |              |          | the time period specified in the        |
    |              |          | Duration field. After that it becomes   |
    |              |          | available or plays a recorded stream    |
    |              |          | for the time specified in the Duration  |
    |              |          | field. (If a stream ends before the     |
    |              |          | time specified in the Duration field,   |
    |              |          | playback ends when the stream ends.)    |
    |              |          | If you pass a negative number other     |
    |              |          | than -1 in the Duration field, it       |
    |              |          | interprets the value as if it were -1.  |
    +--------------+----------+-----------------------------------------+
    | Reset        | Boolean  | An optional Boolean value or number     |
    |              |          | that specifies whether to flush any     |
    |              |          | previous playlist.                      |
    +--------------+----------+-----------------------------------------+
```

play 播放命令执行流程：

```java
         +-------------+                            +----------+
         | Play Client |             |              |   Server |
         +------+------+             |              +-----+----+
                |        Handshaking and Application       |
                |             connect done                 |
                |                    |                     |
                |                    |                     |
                |                    |                     |
                |                    |                     |
       ---+---- |------Command Message(createStream) ----->|
    Create|     |                                          |
    Stream|     |                                          |
       ---+---- |<---------- Command Message --------------|
                |     (_result- createStream response)     |
                |                                          |
       ---+---- |------ Command Message (play) ----------->|
          |     |                                          |
          |     |<------------  SetChunkSize --------------|
          |     |                                          |
          |     |<---- User Control (StreamIsRecorded) ----|
     Play |     |                                          |
          |     |<---- User Control (StreamBegin) ---------|
          |     |                                          |
          |     |<--Command Message(onStatus-play reset) --|
          |     |                                          |
          |     |<--Command Message(onStatus-play start) --|
          |     |                                          |
          |     |<-------------Audio Message---------------|
          |     |                                          |
          |     |<-------------Video Message---------------|
          |     |                    |                     |
                                     |
             Keep receiving audio and video stream till finishes
                 Message flow in the play command
```

命令执行过程中的消息流如下：

* 当客户端接收到服务器返回的 createStream 成功的消息时，开始发送播放命令。
* 服务器接收到播放命令后，发送设置块大小的消息。
* 服务器发送一条用户控制消息，消息内包含了 StreamlsRecorded 事件和流ID。事件类型位于消息的前 2 个字节，流 ID 位于消息的最后 4 个字节。
* 服务器发送一条用户控制消息，消息内包含了 StreamBegin 事件，用于通知客户端开始播放流。
* 如果客户端已经成功发送了播放命令，那么服务器发送两条 onStatus 命令给客户端，命令的内容为 NetStream.Play.Start 和 NetStream.Play.Reset。服务器只有在客户端发送了设置有重置标签的播放命令后，才能发送 NetStream.Play.Reset 命令。如果服务器找不到客户端请求播放的流，那么发送 NetStream.Play.StreamNotFound 命令给客户端。
* 之后，服务器发送音频和视频数据给客户端。
