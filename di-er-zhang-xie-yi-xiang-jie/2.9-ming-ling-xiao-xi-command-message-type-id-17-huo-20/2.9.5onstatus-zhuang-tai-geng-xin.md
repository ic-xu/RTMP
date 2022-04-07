# 2.9.5、onStatus 状态更新

服务器通过发送 onStatus 命令给客户端来通知网络流状态的更新。

```java
    +--------------+----------+----------------------------------------+
    | Field Name   |   Type   |             Description                |
    +--------------+----------+----------------------------------------+
    | Command Name |  String  | The command name "onStatus".           |
    +--------------+----------+----------------------------------------+
    | Transaction  |  Number  | Transaction ID set to 0.               |
    | ID           |          |                                        |
    +--------------+----------+----------------------------------------+
    | Command      |  Null    | There is no command object for         |
    | Object       |          | onStatus messages.                     |
    +--------------+----------+----------------------------------------+
    | Info Object  | Object   | An AMF object having at least the      |
    |              |          | following three properties: "level"    |
    |              |          | (String): the level for this message,  |
    |              |          | one of "warning", "status", or "error";|
    |              |          | "code" (String): the message code, for |
    |              |          | example "NetStream.Play.Start"; and    |
    |              |          | "description" (String): a human-       |
    |              |          | readable description of the message.   |
    |              |          | The Info object MAY contain other      |
    |              |          | properties as appropriate to the code. |
    +--------------+----------+----------------------------------------+
               Format of NetStream status message commands.
```
