---
title: FFmpeg基础学习
tags:
---

## #1 Basic

### 1.1 编译FFmpeg

参考：https://www.jianshu.com/p/350f8e083e82

完全按这个文章来，可以顺利编译出FFmpeg的SO文件。



## #2 常用命令

ffmpeg -decoders：查看支持的解码器。

ffmpeg -encoders：查看支持的编码器。

ffmpeg -formats：查看支持的视频格式。

ffmpeg -filters：查看ffmpeg支持的滤镜。

ffmpeg -h：查看该类型的详细参数，如ffmpeg -h muxer=flv、ffmpeg -h encoder=h264