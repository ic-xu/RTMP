# 2.3、协议控制消息

> 协议控制消息（`Protocol Control Messages`）用于客户端或服务端对协议相关属性（比如块大小、窗口大小和流带宽）进行配置，其消息类型为`1`、`2`、`3`、`5`和`6`。在两端进行音频、视频或信息数据传输前，两端都需要使用协议控制消息来**保证RTMP 块流协议所需的基本信息统一**。这些消息包含了必要的 RTMP 块流协议信息。这些协议控制消息必须使用 0 作为消息流ID（作为已知的控制流ID），同时使用 2 作为块流ID。协议控制消息接收立即生效；解析时，时间戳字段被忽略。
