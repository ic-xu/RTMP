# 2.9.13、pause: 暂停

客户端发送此消息来通知服务器暂停或开始播放。

客户端发送给服务器的命令结构如下：

```java
    +--------------+----------+----------------------------------------+
    | Field Name   |   Type   |             Description                |
    +--------------+----------+----------------------------------------+
    | Command Name |  String  | Name of the command, set to "pause".   |
    +--------------+----------+----------------------------------------+
    | Transaction  |  Number  | There is no transaction ID for this    |
    | ID           |          | command. Set to 0.                     |
    +--------------+----------+----------------------------------------+
    | Command      |  Null    | Command information object does not    |
    | Object       |          | exist. Set to null type.               |
    +--------------+----------+----------------------------------------+
    |Pause/Unpause |  Boolean | true or false, to indicate pausing or  |
    | Flag         |          | resuming play                          |
    +--------------+----------+----------------------------------------+
    | milliSeconds |  Number  | Number of milliseconds at which the    |
    |              |          | the stream is paused or play resumed.  |
    |              |          | This is the current stream time at the |
    |              |          | Client when stream was paused. When the|
    |              |          | playback is resumed, the server will   |
    |              |          | only send messages with timestamps     |
    |              |          | greater than this value.               |
    +--------------+----------+----------------------------------------+
```

当流暂停成功，服务器发送 NetStream.Pause.Notify 状态消息给客户端，如果流未暂停，服务器发送NetStream.Unpause.Notify 状态消息给客户端。如果暂停失败，则发送 \_error 消息。
