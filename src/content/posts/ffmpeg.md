---
title: FFmpeg
published: 2025-11-20
pinned: false
description: A simple example of a Markdown blog post.
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

### 提取字幕
* 提取字幕为 ASS 格式
```bash
ffmpeg -i input.mp4 -map 0:s:0 subtitle.ass
```  
* 提取字幕为 SRT 格式
```bash
ffmpeg -i input.mp4 -map 0:s:0 subtitle.srt
```  
**-map 0:s:0 表示从第一个输入文件（input.mp4）提取第一个字幕流。如果你的 MKV 文件中有多个字幕流，可以调整数字（例如 0:s:1）来选择不同的字幕流。**
### 提取音频
* 提取音频为 MP3 格式
```bash
ffmpeg -i input.mp4 -vn -acodec libmp3lame output.mp3
```
* 提取音频为 WAV 格式
```bash
ffmpeg -i inpu.mp4 -vn output.wav
```
* 提取音频为原始音频格式（例如：AAC）
```bash
ffmpeg -i input.mp4 -vn -acodec copy output.aac
```