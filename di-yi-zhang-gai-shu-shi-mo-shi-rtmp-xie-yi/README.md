---
description: 描述1
cover: >-
  https://images.unsplash.com/photo-1647685101882-6bd1835e3263?crop=entropy&cs=srgb&fm=jpg&ixid=MnwxOTcwMjR8MHwxfHJhbmRvbXx8fHx8fHx8fDE2NDkzMTYwNjA&ixlib=rb-1.2.1&q=85
coverY: 0
---

# 第一章 概述(什么是RTMP协议)



_RTMP（Real Time Messaging Protocol）_ 是由 Adobe 公司基于 Flash Player 播放器对应的音视频 flv 封装格式提出的一种，基于TCP 的数据传输协议。本身具有稳定、兼容性强、高穿透的特点。常被应用于流媒体直播、点播等场景。常用于推推流方（主播）的稳定传输需求。

* `RTMP`工作在TCP之上，默认使用端口1935；
* `RTMPE`在RTMP的基础上增加了加密功能；
* `RTMPT`封装在[HTTP请求](https://link.jianshu.com/?t=https://baike.baidu.com/item/HTTP%E8%AF%B7%E6%B1%82)之上，可穿透[防火墙](https://link.jianshu.com/?t=https://baike.baidu.com/item/%E9%98%B2%E7%81%AB%E5%A2%99)；
* `RTMPS`类似RTMPT，增加了`TLS/SSL`的安全功能。

#### **消息主要分为三类:** `协议控制消息、数据消息、命令消息`。

\
协议控制消息

* Message Type ID = 1\~6，主要用于协议内的控制

数据消息

* Message Type ID = 8 9 18
  * 8: Audio 音频数据
  * 9: Video 视频数据
  * 18: Metadata 包括音视频编码、视频宽高等信息。

命令消息 Command Message (20, 17)

* 此类型消息主要有　NetConnection　和　NetStream　两个类，两个类分别有多个函数，该消息的调用，可理解为远程函数调用。

