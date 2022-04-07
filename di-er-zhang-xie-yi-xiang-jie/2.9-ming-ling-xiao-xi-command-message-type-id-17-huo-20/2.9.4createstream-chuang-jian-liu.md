# 2.9.4、createStream 创建流

### createStream: 创建流

```java
    +--------------+----------+----------------------------------------+
    | Field Name   |   Type   |             Description                |
    +--------------+----------+----------------------------------------+
    | Command Name |  String  | Name of the command. Set to            |
    |              |          | "createStream".                        |
    +--------------+----------+----------------------------------------+
    | Transaction  |  Number  | Transaction ID of the command.         |
    | ID           |          |                                        |
    +--------------+----------+----------------------------------------+
    | Command      |  Object  | If there exists any command info this  |
    | Object       |          | is set, else this is set to null type. |
    +--------------+----------+----------------------------------------+
```

从服务器发送给客户端的命令结构：

```java
    +--------------+----------+----------------------------------------+
    | Field Name   |   Type   |             Description                |
    +--------------+----------+----------------------------------------+
    | Command Name |  String  | _result or _error; indicates whether   |
    |              |          | the response is result or error.       |
    +--------------+----------+----------------------------------------+
    | Transaction  |  Number  | ID of the command that response belongs|
    | ID           |          | to.                                    |
    +--------------+----------+----------------------------------------+
    | Command      |  Object  | If there exists any command info this  |
    | Object       |          | is set, else this is set to null type. |
    +--------------+----------+----------------------------------------+
    | Stream       |  Number  | The return value is either a stream ID |
    | ID           |          | or an error information object.        |
    +--------------+----------+----------------------------------------+
```

```
![[1382048-20180502211951736-1852369930.png]]
```

!\[\[1382048-20180502221733808-557945243.png]]
