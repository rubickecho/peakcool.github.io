---
author: tangliangcheng
comments: true
date: 2017-07-31 23:13:00+00:00
layout: post
slug: learn node 05 day
title: 《深入浅出NodeJs》学习笔记 05 day
excerpt: Buffer是一个像Array的对象，但它主要用于操作字节
categories:
- 技术分享
---

## 理解Buffer

### Buffer对象

Buffer是一个像Array的对象，但它主要用于操作字节。
Buffer所占用的内存不是通过v8分配的，属于堆外内存。

在编码中，中文字在uft-8编码下占3各元素，字母和半角标点符号占用1个元素。

### Buffer内存分配

为了高效地使用申请来的内存，node采用了slab分配机制。slab是一种动态内存管理机制，最早诞生于SunOS操作系统中，目前在一些*nix操作系统中有广泛应用。slab就是申请一块申请好的固定大小的内存区域。

slab三种状态：

* full: 完全分配状态
* partial: 部分分配状态
* empty: 没有被分配状态

```js
new Buffer(size)
```

size 小于8kb即为小对象，大于为大对象。

#### 分配小Buffer对象

同一个slab可能分配给多个Buffer对象使用，只有这些小Buffer对象在作用域释放并都可以回收时，slab的8kb空间才会被回收。尽管创建了1个字节的Buffer对象，但是如果不释放它，实际可能是9kb的内存没有释放。

#### 分配大Buffer对象

如果需要超过8kb的Buffer对象，将会直接分配一个`SlowBuffer`对象作为slab单元，这个slab单元将会被这个大Buffer对象独占。

### Buffer的转换

Buffer对象可以与字符串之间相互转换，目前支持类型如下：

* ASCII
* UTF-8
* UTD-16LE/UCS-2
* Base64
* Binary
* Hex

```js
Buffer.isEncoding(encoding) //判断编码是否支持
```

#### 字符串转Buffer

```js
new Buffer(str, [encoding])
```

一个Buffer对象可以存储不同的编码类型的字符串转码的值，调用`write()`方法可以实现。

```js
buf.write(string, [offset], [length], [encoding])
```

**由于可以不断写入内容到Buffer对象中并且每次写入可以指定编码，所以Buffer对象中可以存在多种编码转化后的内容。值得注意的是，每种编码所用的字节长度不一样，将Buffer反转回字符串时需要小心处理。**

#### Buffer转字符串

```js
buf.toString([encoding], [start], [end]) //encoding默认为UTF-8
```

### Buffer的拼接

Buffer的使用场景通常是以一段一段的方式传输。

```js
data += chunk
```
这句代码隐藏了toString()操作，等价于:

```js
data = data.toString() + chunk.toString()
```

这样的方式在英文环境下不错有问题，但对于宽字节的中文就会出现乱字节问题。

### Buffer与性能

Buffer在文件I/O和网络I/O中运用广泛，尤其在网络传输中。在应用中，我们通常会操作字符串，但一旦在网络中传输，都需要转换为Buffer，以二进制数据传输。

在Node构建的Web应用中，可以选择将页面中的动态内容和静态内容分离，静态内容部分可以通过预先转换为Buffer的方式，使性能得到提升。由于文件自身是二进制数据，所以在不需要改变内容的场景下，尽量只读取Buffer，然后直接传输，不做额外的转换，避免损耗。
