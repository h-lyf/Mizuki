---
title: ImageMagick
published: 2025-11-19
pinned: false
description: ImageMagick 最常用、最实用的命令汇总。
image: "images/imagemagick.png"
tags: [ImageMagick, 软件, 开源]
category: 开源软件
licenseName: "Unlicensed"
author: 御剑乘风
# sourceLink: "https://github.com/h-lyf/Mizuki/blob/master/src/content/posts/imagemagick.md"
draft: false
date: 2025-11-19
pubDate: 2025-11-19
---

[ImageMagick®](https://imagemagick.org/) 是一款免费且[开源](https://imagemagick.org/script/license.php)的软件套件，用于编辑和处理数字图像。它可用于创建、编辑、合成或转换位图图像，并支持广泛的文件[格式](https://imagemagick.org/script/formats.php)，包括 JPEG、PNG、GIF、TIFF 和 PDF。

# 介绍

ImageMagick 广泛应用于网页开发、图形设计、视频编辑等行业，同时也用于科学研究、医学成像和天文学。其多功能性和可定制性，加上强大的图像处理能力，使其成为各种图像相关任务的流行选择。

ImageMagick 包含命令行界面，用于执行复杂的图像处理任务，以及 API，用于将其功能集成到软件应用程序中。它使用 C 语言编写，可在多种操作系统上使用，包括 Linux、Windows 和 macOS。

ImageMagick 的官网位于 [https://imagemagick.org](https://imagemagick.org/)，该软件的源代码可以通过[仓库](https://github.com/ImageMagick/ImageMagick)获取。

# 常用命令

1. **基本信息查看**
    - 查看图片详细信息
    ```bash
    magick identify image.jpg
    ```
    - 批量查看文件名、分辨率、文件大小
    ```bash
    magick identify -format "%f %wx%h %b" *.jpg
    ```

2. **格式转换**
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

3. **调整大小**
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

4. **批量处理**
    - 批量把文件夹所有图片缩放到最大宽1920，保持比例，不放大
    ```bash
    magick mogrify -path resized -resize 1920x -quality 90 *.jpg
    ```
    - 批量转格式 + 压缩
    ```bash
    magick mogrify -format webp -quality 80 *.jpg
    ```
    - 给所有图片加水印（右下角）
    ```bash
    magick mogrify -gravity southeast -draw "image over 20,20 0,0 'watermark.png'" *.jpg
    ```

5. **裁剪**
    - 从(100,200)位置裁剪800x600
    ```bash
    magick input.jpg -crop 800x600+100+200 output.jpg
    ```
    - 按4:3比例居中裁剪
    ```bash
    magick input.jpg -gravity center -crop 4:3 +repage output.jpg
    ```

6. **添加文字水印**
    - 清晰水印
    ```bash
    magick input.jpg -fill white -stroke black -strokewidth 2 -pointsize 48 -font Helvetica -gravity southeast -annotate +30+30 "© 2025 张三" output.jpg
    ```
    - 半透明水印
    ```bash
    magick input.jpg -fill "rgba(255,255,255,0.5)" -pointsize 60 -gravity center -annotate +0+0 "DRAFT" output.jpg
    ```

7. **添加图片水印**
    - 清晰水印
    ```bash
    magick input.jpg watermark.png -gravity southeast -geometry +30+30 -composite output.jpg
    ```
    - 半透明水印
    ```bash
    magick watermark.png -channel A -evaluate Multiply 0.5 +channel watermark_fade.png
    magick input.jpg watermark_fade.png -gravity southeast -composite output.jpg
    ```

8. **压缩优化**
    - 通用压缩，去除元数据
    ```bash
    magick input.jpg -strip -quality 85 output.jpg
    ```
    - JPG 最佳压缩
    ```bash
    magick input.jpg -sampling-factor 4:2:0 -strip -interlace JPEG -colorspace sRGB optimized.jpg
    ```
    - PNG 最大压缩
    ```bash
    magick input.png -strip -define png:compression-level=9 output.png
    ```

9. **拼接图片**
    - 水平拼接
    ```bash
    magick *.jpg +append result.jpg
    ```
    - 垂直拼接
    ```bash
    magick *.jpg -append result.jpg
    ```
    - 2x2 网格拼图
    ```bash
    magick image1.jpg image2.jpg image3.jpg image4.jpg -gravity center -background skyblue +append -append mosaic.jpg
    ```

10. **常用特效**
    - 高斯模糊
    ```bash
    magick input.jpg -blur 0x8 output.jpg
    ```
    - 素描效果
    ```bash
    magick input.jpg -charcoal 2 output.jpg
    ```
    - 复古褐色
    ```bash
    magick input.jpg -sepia-tone 80% output.jpg
    ```
    - 增加饱和度50%
    ```bash
    magick input.jpg -modulate 100,150,100 output.jpg
    ```
    - 染色效果
    ```bash
    magick input.jpg -colorize 30,60,90 red output.jpg
    ```

11. **去背景**
    ```bash
    magick input.jpg -remove background output.png
    ```

**更多用法可到 [ImageMagick 官网](https://imagemagick.org/)查看！**
