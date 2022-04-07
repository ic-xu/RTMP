# 2.9.2、call 远程调用

### Call 调用

网络连接对象中包含的 call 方法，会在接收端执行远程过程调用（RPC）。被调用的 RPC 方法名作为 call 方法的参数传输。 从发送端到接收端的命令结构如下：

```java
    +--------------+----------+----------------------------------------+
    |Field Name    |   Type   |             Description                |
    +--------------+----------+----------------------------------------+
    | Procedure    |  String  | Name of the remote procedure that is   |
    | Name         |          | called.                                |
    +--------------+----------+----------------------------------------+
    | Transaction  |  Number  | If a response is expected we give a    |
    |              |          | transaction Id. Else we pass a value of|
    | ID           |          | 0                                      |
    +--------------+----------+----------------------------------------+
    | Command      |  Object  | If there exists any command info this  |
    | Object       |          | is set, else this is set to null type. |
    +--------------+----------+----------------------------------------+
    | Optional     |  Object  | Any optional arguments to be provided  |
    | Arguments    |          |                                        |
    +--------------+----------+----------------------------------------+
```

应答的命令结构如下:

```java
    +--------------+----------+----------------------------------------+
    | Field Name   |   Type   |             Description                |
    +--------------+----------+----------------------------------------+
    | Command Name |  String  | Name of the command.                   |
    |              |          |                                        |
    +--------------+----------+----------------------------------------+
    | Transaction  |  Number  | ID of the command, to which the        |
    | ID           |          | response belongs.
    +--------------+----------+----------------------------------------+
    | Command      |  Object  | If there exists any command info this  |
    | Object       |          | is set, else this is set to null type. |
    +--------------+----------+----------------------------------------+
    | Response     | Object   | Response from the method that was      |
    |              |          | called.                                |
    +------------------------------------------------------------------+
    
```
