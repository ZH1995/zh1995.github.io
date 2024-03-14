---
title: "Golang里map的底层实现原理"
date: 2024-01-05T21:09:05+08:00
slug: 2024-01-05-20240105_golang_map
type: posts
draft: false
categories:
  - 技术
tags:
  - golang
---
## 底层实现

### hmap结构体

map底层是用哈希表实现的，对应hmap结构体。

每个hmap结构体包含元素个数、bucket数组、oldbucket数组、bucket数组长度的对数等多个字段。

#### bmap结构体

每个bucket是由bmap结构体实现的，它包含tophash数组、k-v数组、overflow指针三个字段。

tophash数组存储哈希值的高八位。k-v数组最多可以存储8个键值对，超出8个时，会新建bucket，并挂在到overflow指针上，形成链表。

## 扩容条件

当bucket过少或过多时，都会触发map的扩容操作。

## 扩容方式

### 增量扩容

先将hmap的原bucket数组挂在到oldbucket，然后新建2倍大小的新bucket。每次搬迁2个k-v对，直到所有k-v对都搬迁完为止。

扩容过程中查询时，先查询oldbucket。若没查到，再查新bucket。

扩容过程中新增时，直接写入新bucket。

### 等量扩容

和增量扩容类似，只不过是缩减挂在overflow链表上的bucket数量，使分散的key更紧凑的分布在bucket上。

## 注意事项

并发读写map会触发panic，可以使用Sync.map。