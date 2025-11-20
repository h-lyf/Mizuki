---
title: ImageMagick
published: 2025-11-19
pinned: false
description: A simple example of a Markdown blog post.
tags: [ImageMagick, 软件, 开源]
category: 软件
licenseName: "Unlicensed"
author: 御剑乘风
sourceLink: "https://github.com/h-lyf/Mizuki/blob/master/src/content/posts/imagemagick.md"
draft: false
date: 2025-11-19
pubDate: 2025-11-19
---

# 介绍

<p align="center">
<img align="center" src="https://camo.githubusercontent.com/a079acc61a9e8d5010858ed2b6fc1c8f9eb7b9b30b8c89c2be43f4fd6492de0a/68747470733a2f2f696d6167656d616769636b2e6f72672f696d6167652f77697a6172642e706e67" alt="ImageMagick logo" width="265" height="353"/>
</p>

[ImageMagick®](https://imagemagick.org/) 是一款免费且[开源](https://imagemagick.org/script/license.php)的软件套件，用于编辑和处理数字图像。它可用于创建、编辑、合成或转换位图图像，并支持广泛的文件[格式](https://imagemagick.org/script/formats.php)，包括 JPEG、PNG、GIF、TIFF 和 PDF。

## ImageMagick 是什么?

ImageMagick 广泛应用于网页开发、图形设计、视频编辑等行业，同时也用于科学研究、医学成像和天文学。其多功能性和可定制性，加上强大的图像处理能力，使其成为各种图像相关任务的流行选择。

ImageMagick 包含命令行界面，用于执行复杂的图像处理任务，以及 API，用于将其功能集成到软件应用程序中。它使用 C 语言编写，可在多种操作系统上使用，包括 Linux、Windows 和 macOS。

ImageMagick 的主网站位于 [https://imagemagick.org](https://imagemagick.org/)，该软件的源代码可以通过[仓库](https://github.com/ImageMagick/ImageMagick)获取。

# 用法

- [格式转换](https://imagemagick.org/)
```bash
magick input.jpg output.png
```

- 正方形最大程度裁剪成圆形
```bash
magick input.jpeg -alpha set ( +clone -threshold -1 -negate -fill white -draw "circle %[fx:w/2],%[fx:h/2] %[fx:w/2],0" ) -compose CopyOpacity -composite output.png
```