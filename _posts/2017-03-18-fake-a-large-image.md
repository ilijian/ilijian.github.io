---
layout: post
title: "使用一个小图模拟一个大图"
description: "纯CSS代码，使用一个小图模拟一个大图"
tags: [memo]
---

在前端的日常编码中，我们经常与图片打交道，使用体积尽量小、质量尽量高的图片一直是我们的追求，为此我们绞尽脑汁，选取合适的图片格式，进行最大的压缩，使用代码进行各种各样的优化。本文所述的方法是采用一张尺寸小的图片来模拟一张尺寸较大的图，从而达到减小所需图片资源体积的目的。

# 使用小尺寸图片模拟大尺寸图片

## 前置条件

- 推荐使用compass编译，如果想要使用原生sass编译器编译，需要对代码做少许修改

## 特点

- 在宽度方向上模拟图片的行为，可以随着图片容器的宽度变化像目标大尺寸图片一样调整图片的缩放和相对位置的缩放

- 可以当做带背景的容器使用，便于添加内容文字

- 兼容双倍高清图

- 可以指定原图的缩放倍数

## demo

<div class="container">
  <div class="fake-image"></div>
  <img src="{{ site.url }}/images/fake-a-large-image/target.png" alt="目标尺寸图">
</div>

<style>
.fake-image {
  height: 0;
  width: 100%;
  padding-bottom: 50%;
  max-width: 1200px;
}

.fake-image:after {
  content: '';
  display: block;
  position: absolute;
  top: 0;
  left: 33.33333%;
  z-index: 0;
  padding-top: 8.33333%;
  width: 33.33333%;
  height: 0;
  padding-bottom: 33.33333%;
  background-image: url('/images/events/3t-black/xx.png?1489821175');
  background-size: contain;
  background-position: bottom;
  background-repeat: no-repeat;
  background-image: image-set(url('/images/events/3t-black/xx.png?1489821175') 1x);
  background-image: image-set(url('/images/events/3t-black/xx.png?1489821175') 1x, url('/images/events/3t-black/xx.png?1489821175') 2x);
  background-image: -webkit-image-set(url('/images/events/3t-black/xx.png?1489821175') 1x, url('/images/events/3t-black/xx.png?1489821175') 2x);
  background-image: -moz-image-set(url('/images/events/3t-black/xx.png?1489821175') 1x, url('/images/events/3t-black/xx.png?1489821175') 2x);
  background-image: -o-image-set(url('/images/events/3t-black/xx.png?1489821175') 1x, url('/images/events/3t-black/xx.png?1489821175') 2x);
  background-image: -ms-image-set(url('/images/events/3t-black/xx.png?1489821175') 1x, url('/images/events/3t-black/xx.png?1489821175') 2x);
}
</style>


## 代码

```css
// image-set mixin
@mixin image-set($image1x,$image2x) {
  background-image: image-url($image1x);
  background-image: image-set($image1x 1x,$image2x 2x);
  background-image: -webkit-image-set($image1x 1x,$image2x 2x);
  background-image: -moz-image-set($image1x 1x,$image2x 2x);
  background-image: -o-image-set($image1x 1x,$image2x 2x);
  background-image: -ms-image-set($image1x 1x,$image2x 2x);
}

// 使用一个小图来模拟大图
@mixin fake-image(
  $image-path,    //普通图的大小
  $image2x-path,  //双倍图的大小（如果没有，可以设置为与普通图相同路径）
  $target-width,  //模拟目标图的宽度
  $target-height, //模拟目标图的高度
  $left,          //普通图在模拟目标图中的x坐标，单位为px
  $top,           //普通图在模拟目标图中的y坐标，单位为px
  $scale: 1,      //普通图的缩放倍数
  $not-responsive: false //设置响应式，如果非响应式图片，那么图片将有最大限制，即模拟目标图的宽度
) {

  $image-width: image-width($image-path) * $scale;
  $image-height: image-height($image-path) * $scale;
  
  height: 0;
  width: 100%;
  padding-bottom: percentage($target-height / $target-width);

  @if($not-responsive) {
    max-width: $target-width;
  }

  &:after {
    content: '';
    display: block;
    position: absolute;
    top: 0;
    left: percentage($left / $target-width);
    z-index: 0;
    padding-top: percentage($top / $target-width);
    width: percentage($image-width / $target-width);
    height: 0;
    padding-bottom: percentage($image-height / $target-width);
    background-image: image-url($image-path);
    background-size: contain;
    background-position: bottom;
    background-repeat: no-repeat;
    
    @if($image2x-path) {
      @include image-set(image-url($image-path),image-url($image2x-path));
    }
  }

}
```