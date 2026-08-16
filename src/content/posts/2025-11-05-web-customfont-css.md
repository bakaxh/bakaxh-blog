---
title: 在Web开发中使用自定义CSS字体及提高加载速度
description: 探讨中文字体体积问题，使用unicode-range分割、WOFF2压缩、减少字符数、font-display、预加载和可变字体等优化策略。
author: 小狐
cover: https://bed.foxmoe.top/file/images/1785897174800_image.png
pubDate: 2025-11-05
updatedDate: 2025-11-05
tags: [CSS, 字体, 性能优化, 中文字体, Web开发]
categories: [前端技术]
---

## 中文字体体积

中文字体与英文字体在体积上存在显著差异，这主要源于两个方面：中文字体包含的字形数量远超英文字体（中文常用字就有3500-8000左右），而且中文字形的线条更为复杂，需要更多的数据来描述。

一个完整的中文字体文件往往在5-10MB之间，即便是经过压缩的.woff2格式，也常常超过1MB，让本就不富裕的家庭雪上加霜说是。

---

## 一、使用 unicode-range 编码范围

unicode-range 的本质是字符集分割技术，能够将一个完整的字体文件按字符编码范围拆分成多个逻辑部分。当页面中需要显示的文字恰好落在某个范围内时，浏览器只会加载对应的字体包，而不是整个字体文件。

通过css代码使用unicode-range:

```css
@font-face {
  font-family: 'MyFont';
  src: url('myfont.woff2') format('woff2');
  unicode-range: U+4E00-9FA5, U+0025-00F0;
  /* 格式为U+编码范围，多个范围以","分隔 */
}
```

中文字符的Unicode编码范围主要是：

- 基本汉字：U+4E00-9FA5（共20902个字符）
- 全角ASCII、全角标点：U+FF00-FFEF
- 半角片假名、半角平假名：U+FF61-FF9F

浏览器支持情况:
![浏览器支持情况](https://bed.foxmoe.top/file/images/1785897197813_image.png)

---

## 二、使用 Woff2/Woff 格式字体

与系统字体格式(TTF/OTF)不同，WOFF格式是专门为Web使用而设计的，包含了元数据和私有数据结构，允许浏览器优化字体加载。

WOFF2采用Brotli压缩算法，相比WOFF 1.0使用的zlib压缩，能够实现更高的压缩率。WOFF2平均比WOFF 1.0节省30%以上的文件大小，某些情况下可达50%以上。

下载FontCreator，将文件导出为woff/woff2就可以了。

![FontCreator](https://bed.foxmoe.top/file/images/1785897206714_image.png)

---

## 三、减少字体字符数量

由于整个字体的字符集可能非常大，例如本网站使用的全濑体(allseto)字符数量达到了42147，其中包含了不少平时用不到的字符集，可以直接删除。

通常情况下中文网站只需要CJK字符+基本字符就可以了。像是藏文、泰文、梵文等字符可以直接删除，或者设置导出网站字体时排除。

---

## 四、字体加载性能优化

### 1. font-display 属性控制字体加载期间的文本渲染行为

```css
@font-face {
  font-family: 'MyCustomFont';
  src: url('fonts/MyCustomFont.woff2') format('woff2');
  font-display: swap; /* 优先保证内容可读性 */
}
```

`font-display: swap` 确保在自定义字体加载期间，内容会立即使用回退字体显示，字体加载完成后再切换，避免了文本闪烁问题。

### 2. 预加载关键字体

对于首屏显示所需的关键字体，可以使用 preload 提示浏览器提前加载：

```html
<link rel="preload" href="fonts/MyFont.woff2" as="font" type="font/woff2" crossorigin>
```

### 3. 使用可变字体

可变字体将字体的多种变体（如不同字重、宽度）合并到一个文件中，可以减少HTTP请求。

浏览器默认字重 400，通常为 Regular。我们可以通过 `font-weight` 属性使用不同字重。

例如下述两个不同字重文件都使用 OPPO-Sans 名称，但在引入字体时设置了不同的字重和链接：

```css
@font-face {
  font-family: OPPO-Sans;
  font-weight: 400;
  font-display: swap;
  src: url(OPPOSansOS2-5000-Regular.woff2) format('woff2');
}
@font-face {
  font-family: OPPO-Sans;
  font-weight: 500;
  font-display: swap;
  src: url(OPPOSansOS2-5000-Medium.woff2) format('woff2');
}
```