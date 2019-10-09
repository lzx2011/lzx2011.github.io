---
title: Nginx的 http_image_filter_module 模块使用说明
date: 2016-11-22 19:20:14
categories: Nginx
tags: Nginx
---
# Nginx的 http_image_filter_module 模块使用说明

# Nginx图片处理原理
这里需要用到 nginx的 http_image_filter_module 模块，这个模块可以很方便的实现图片缩放功能，只是默认的情况下并不会安装，需要自己编译安装才能行。编译的时候./configure 增加 –with-http_image_filter_module 编译安装即可

# Nginx图片处理的优缺点

## 优点

1. 操作简单。通过简单配置，省去了后端裁剪程序的复杂性。
2. 实时裁剪。可以实时访问在线裁剪图片。
3. 灵活性强。后端程序裁剪图片时需要知道裁剪图片的尺寸和质量，使用nginx裁剪可以实时裁剪任意尺寸的图片。
4. 不占用硬盘空间。
 
## 缺点

1. 消耗CPU和内存，访问量大的时候就会给服务器带来很大的负担。(可以通过使用Nginx缓存和缓存服务器来解决)

2. 功能不是很强大，支持的处理图片类型只包括JPEG, GIF, PNG, or WebP

# Nginx图片处理模块指令使用
## image_filter （重要）
设置要在图像上执行的转换类型

Syntax: | image_filter off;
-------- |-------------------
        | image_filter test;
          |image_filter size;
  			|image_filter rotate 90/180/270;
 			|image_filter resize width height;
 			|image_filter crop width height;
Default: | image_filter off;
Context: | location


### test

确保响应图片是JPEG、GIF，WEBP或PNG格式，否则返回415错误码。
 
### size

```java 
outputs information about images in a JSON format, e.g.:
 
{ "img" : { "width": 100, "height": 100, "type": "gif" } }
 
In case of an error, the output is as follows:
 
{}
```
以 json 格式返回原图的尺寸和类型

### rotate

逆时针旋转指定角度，只能指定这三个角度。参数值可以包含变量，这个模式可以单独使用也可以和resize、crop变换同时使用。

### resize width height

按比例对图像进行缩放，可以只指定一个尺寸，另一个尺寸用“-”。如果遇到错误，服务器返回415错误码。参数值可以包含变量。当与rotate参数一同使用时，旋转操作发生在缩放之后。图片会以长的一边为标准，然后等比缩放。

### crop width height
按比例裁剪图片，可以只指定一个尺寸，另一个尺寸用“-”。如果遇到错误，服务器返回415错误码。参数值可以包含变量。当与rotate参数一同使用时，旋转操作发生在裁剪之前。图片会以长的一边为标准，然后等比缩放，然后裁剪掉多余的部分。

### image_filter_buffer
 设置用于读取图像的缓冲区的最大大小  
 
Syntax:	|image_filter_buffer size;
----------|-----------------
Default:	|image_filter_buffer 1M;
Context:	|http, server, location

设置读取图片的最大缓冲区大小。当超过缓冲区大小时，返回 error 415 (Unsupported Media Type).

### image_filter_interlace
 如果启用，最终图像将隔行扫描
 
Syntax:	|image_filter_interlace on / off;
---------|-----------
Default:	|image_filter_interlace off;
Context:	|http, server, location

如果开启此功能，最终的图像是交错的。对于JPEG，最终图片是“渐进式JPEG”格式。图片一般是线性加载，设置后则变为交替加载图片。渐进式jpeg效果参见：http://www.zhangxinxu.com/wordpress/2013/01/progressive-jpeg-image-and-so-on/

### image_filter_jpeg_quality
设置转换JPEG图像的质量

Syntax:	|image_filter_jpeg_quality quality;
----------|---------------
Default:	|image_filter_jpeg_quality 75;
Context:	|http, server, location

设置转为JPEG图像的质量。接受的值从1到100。较小的值意味着低质的图片质量和更少的数据传输量。最大建议的值是95。参数可以包含变量。

### image_filter_sharpen 
通过设置锐化度，增加最终图像的清晰度。

Syntax:	|image_filter_sharpen percent;
--------|--------------
Default:	|image_filter_sharpen 0;
Context:	|http, server, location

增加最终图片的锐度。这个百分比可以超过100。0值禁用此功能。参数可以包含变量。

### image_filter_transparency
定义是否透明度时应保留转换GIF图像或PNG图像的调色板中指定的颜色。


Syntax:	|  image_filter_transparency on/off;
----------|------------------
Default:	|  image_filter_transparency on;
Context:	|  http, server, location

决定在转换GIF或PNG图片带有调色板定义的颜色时，透明是否会保留。丢失透明度可以是图片得到更好的质量。PNG的Alpha通道的透明总是会保留。

### image_filter_webp_quality
设置转化WebP图像所需的质量

Syntax:  |image_filter_webp_quality quality;
---------|------------------------   
Default: | image_filter_webp_quality 80;
Context: | http, server, location

设置转为webp图像的质量。

This directive appeared in version 1.11.6.

# 局限性
1. Nginx 的图片处理模块，暂时没有看到官方发布的能够给图片加水印功能的模块，在github上看到有人写了些这样的扩展功能，参见 `https://github.com/3078825/ngx_image_thumb`
 
2. Nginx 的实时性和访问的方便性上，GraphicsMagick 是无法比拟的，但是 GraphicsMagick 对图片的处理的功能要比nginx强大很多，比如nginx不能将图片旋转任意角度，不能在图片上加水印，处理图片类型有限等，相对nginx，GraphicsMagick 更适合对图片的异步处理。

# 参考文献
<a href="http://nginx.org/en/docs/http/ngx_http_image_filter_module.html" target="_blank">Module ngx_http_image_filter_module</a>
