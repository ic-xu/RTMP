# 2.9.12、seek: 定位

客户端发送此消息来定位多媒体文件或播放列表的偏移（以毫秒为单位）。

客户端发送给服务器的命令结构如下：

```java
    +--------------+----------+----------------------------------------+
    | Field Name   |   Type   |             Description                |
    +--------------+----------+----------------------------------------+
    | Command Name |  String  | Name of the command, set to "seek".    |
    +--------------+----------+----------------------------------------+
    | Transaction  |  Number  | Transaction ID set to 0.               |
    | ID           |          |                                        |
    +--------------+----------+----------------------------------------+
    | Command      |  Null    | There is no command information object |
    | Object       |          | for this command. Set to null type.    |
    +--------------+----------+----------------------------------------+
    | milliSeconds |  Number  | Number of milliseconds to seek into    |
    |              |          | the playlist.                          |
    +--------------+----------+----------------------------------------+
```

当定位完成后，服务器回复 NetStream.Seek.Notify 状态消息给客户端。如果定位失败，将回复 \_error 消息。
