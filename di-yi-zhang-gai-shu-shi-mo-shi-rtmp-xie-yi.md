---
description: 描述1
coverY: 0
---

# 第一章 概述(什么是RTMP协议)



_RTMP（Real Time Messaging Protocol）_ 是由 Adobe 公司基于 Flash Player 播放器对应的音视频 flv 封装格式提出的一种，基于TCP 的数据传输协议。本身具有稳定、兼容性强、高穿透的特点。常被应用于流媒体直播、点播等场景。常用于推推流方（主播）的稳定传输需求。

* `RTMP`工作在TCP之上，默认使用端口1935；
* `RTMPE`在RTMP的基础上增加了加密功能；
* `RTMPT`封装在[HTTP请求](https://link.jianshu.com/?t=https://baike.baidu.com/item/HTTP%E8%AF%B7%E6%B1%82)之上，可穿透[防火墙](https://link.jianshu.com/?t=https://baike.baidu.com/item/%E9%98%B2%E7%81%AB%E5%A2%99)；
* `RTMPS`类似RTMPT，增加了`TLS/SSL`的安全功能。
