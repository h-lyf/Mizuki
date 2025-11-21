---
title: ImageMagick
published: 2025-11-19
pinned: false
description: A simple example of a Markdown blog post.
image: "images/imagemagick.png"
tags: [ImageMagick, 软件, 开源]
category: 软件
licenseName: "Unlicensed"
author: 御剑乘风
sourceLink: "https://github.com/h-lyf/Mizuki/blob/master/src/content/posts/imagemagick.md"
draft: false
date: 2025-11-19
pubDate: 2025-11-19
---

[ImageMagick®](https://imagemagick.org/) 是一款免费且[开源](https://imagemagick.org/script/license.php)的软件套件，用于编辑和处理数字图像。它可用于创建、编辑、合成或转换位图图像，并支持广泛的文件[格式](https://imagemagick.org/script/formats.php)，包括 JPEG、PNG、GIF、TIFF 和 PDF。

# 介绍

ImageMagick 广泛应用于网页开发、图形设计、视频编辑等行业，同时也用于科学研究、医学成像和天文学。其多功能性和可定制性，加上强大的图像处理能力，使其成为各种图像相关任务的流行选择。

ImageMagick 包含命令行界面，用于执行复杂的图像处理任务，以及 API，用于将其功能集成到软件应用程序中。它使用 C 语言编写，可在多种操作系统上使用，包括 Linux、Windows 和 macOS。

ImageMagick 的主网站位于 [https://imagemagick.org](https://imagemagick.org/)，该软件的源代码可以通过[仓库](https://github.com/ImageMagick/ImageMagick)获取。

# 常用用法

1. 基本信息查看
    - 查看图片详细信息
    ```bash
    magick identify image.jpg
    ```
    - 批量查看文件名、分辨率、文件大小
    ```bash
    magick identify -format "%f %wx%h %b" *.jpg
    ```

2. 格式转换
    - 任意格式转任意格式
    ```bash
    magick input.png output.jpg
    ```
    - RAW 转 JPG
    ```bash
    magick input.cr2 output.jpg
    ```
    - 多张图片合并成一个 PDF
    ```bash
    magick *.png output.pdf
    ```
    - 转 WebP 并控制质量
    ```bash
    magick input.jpg -quality 85 output.webp
    ```

3. 调整大小
    - 强制指定尺寸（会变形）
    ```bash
    magick input.jpg -resize 1920x1080 output.jpg
    ```
    - 强制拉伸
    ```bash
    magick input.jpg -resize 1920x1080\! output.jpg
    ```
    - 按百分比缩放
    ```bash
    magick input.jpg -resize 50% output.jpg
    ```
    - 缩放并居中裁剪正方形
    ```bash
    magick input.jpg -resize 800x800^ -gravity center -crop 800x800+0+0 output.jpg
    ```
    - 快速生成缩略图（去元数据，更小）
    ```bash
    magick input.jpg -thumbnail 200x200^ -gravity center -extent 200x200 thumb.jpg
    ```
    - 正方形最大程度裁剪成圆形
    ```bash
    magick input.jpeg -alpha set ( +clone -threshold -1 -negate -fill white -draw "circle %[fx:w/2],%[fx:h/2] %[fx:w/2],0" ) -compose CopyOpacity -composite output.png
    ```

4. 批量处理
    - 批量把文件夹所有图片缩放到最大宽1920，保持比例，不放大
    ```bash
    magick mogrify -path resized -resize 1920x -quality 90 *.jpg
    ```