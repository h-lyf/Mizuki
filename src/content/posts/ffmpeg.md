---
title: FFmpeg
published: 2025-11-20
pinned: false
description: FFmpeg 最常用、最实用的命令汇总。
image: "images/ffmpeg.png"
tags: [FFmpeg]
category: 开源软件
licenseName: "Unlicensed"
author: 御剑乘风
sourceLink: "https://github.com/h-lyf/Mizuki/blob/master/src/content/posts/ffmpeg.md"
draft: false
date: 2025-11-20
pubDate: 2025-11-20
---

FFmpeg 是一套用于处理多媒体内容（例如音频、视频、字幕和相关元数据）的库和工具集合。

# 介绍
FFmpeg 是领先的多媒体框架，能够解码、编码、转码、复用、解复用、流式传输、过滤和播放几乎任何格式的视频。它支持最晦涩难懂的东西,从古老的格式到最前沿的格式。此外，FFmpeg 还具有高度可移植性,[FATE](http://fate.ffmpeg.org/) 可在 Linux、Mac OS X、Microsoft Windows、BSD、Solaris 等各种构建环境、机器架构和配置下运行。
> 它包含可供应用程序使用的 libavcodec、libavutil、libavformat、libavfilter、libavdevice、libswscale 和 libswresample 库，以及用于转码和播放的 ffmpeg、ffplay 和 ffprobe 库。

# 常用命令
1. 基本信息查看
    - 查看文件信息
    ```bash
    ffmpeg -i input.mp4
    ```
    - 更清晰的结构化信息
    ```bash
    ffprobe -v quiet -print_format json -show_format -show_streams input.mp4
    ```

2. 格式转换
    - 转码保持原质量
    ```bash
    ffmpeg -i input.mp4 output.mkv
    ```
    - 强制使用 H.265（HEVC），体积更小
    ```bash
    ffmpeg -i input.mp4 -c:v libx265 -crf 23 -preset medium output.mp4
    ```
    - 强制使用 AV1（体积最小）
    ```bash
    ffmpeg -i input.mp4 -c:v libsvtav1 -crf 30 -preset 4 output.mp4
    ```

3. 压缩视频
    - H.264 推荐压缩（兼容性最好
    ```bash
    ffmpeg -i input.mp4 -vcodec libx264 -crf 23 -preset veryfast output.mp4
    ```
    > crf 值解释：0（无损）- 51（最差），推荐 23-28
    > 23≈原画质，28≈体积减半，眼睛基本看不出差别
    - H.265 压缩（同画质体积减半）
    ```bash
    ffmpeg -i input.mp4 -c:v libx265 -crf 28 -preset fast output.mp4
    ```
    - 超强压缩（10倍压缩，画质仍有可接受度）
    ```bash
    ffmpeg -i input.mp4 -vf "scale=1280:720" -c:v libx265 -crf 32 -preset slow output.mp4
    ```

4. 提取音频
    ```bash
    ffmpeg -i input.mp4 -vn -c:a mp3 output.mp3
    ffmpeg -i input.mp4 -vn -c:a aac -b:a 192k output.m4a
    ```
    - 无损提取原音频
    ```bash
    ffmpeg -i input.mp4 -vn -c:a copy output.aac
    ```

5. 视频截取片段
    - 从 00:01:30 开始截取 30 秒
    ```bash
    ffmpeg -i input.mp4 -ss 00:01:30 -t 30 -c copy output.mp4
    ```
    > -c copy 零损耗快切（需关键帧对齐）
    ```bash
    ffmpeg -i input.mp4 -ss 00:01:30 -t 30 -c:v libx264 -crf 23 output.mp4
    ```
    > 精确截取（重新编码）
    - 从 00:01:30 到 00:02:30
    ```bash
    ffmpeg -i input.mp4 -ss 00:01:30 -to 00:02:30 -c copy fast_cut.mp4
    ```

6. 合并视频（同编码才支持无损合并）
    - 以文件列表方式合并
    ```bash
    # 先创建列表文件 list.txt
    # file '1.mp4'
    # file '2.mp4'
    # file '3.mp4'

    ffmpeg -f concat -safe 0 -i list.txt -c copy output.mp4
    ```

7. 提取字幕
    - 提取字幕为 SRT 格式
    ```bash
    ffmpeg -i input.mp4 -map 0:s:0 subtitle.srt
    ```  
    > 0:s:0 表示从第一个输入文件提取第一个字幕流。如果视频文件中有多个字幕流，可以调整数字（例如 0:s:1）来选择不同的字幕流。