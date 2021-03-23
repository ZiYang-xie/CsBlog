---
title: PointNet++论文阅读
date: 2020-12-08 14:36:58
index_img: /img/Pic/cnn_img.jpeg
banner_img: /img/Pic/PointCloud.jpg
category: [Research]
tags: PointNet
math: true
---
# PointNet++

## 综述

回顾一下 PointNet ，我们说PointNet解决了 PointCloud 输入的顺序无关问题，但其有一个缺点就是无法获取局部特征，即没有很好的处理 local information，使其在复杂场景下表现乏力。PointNet++正是针对PointNet这一弱点进行改进。

- 主要改进方式

1. 受到 CNN 神经元感受野不断扩大的启发，利用空间距离，使 PointNet对点集局部区域进行特征迭代提取。

2. 使用最远点提取方式，能够实现较为均匀的点采样

## 关键问题

一、如何做点集划分 (怎么划分空间)

二、如何利用特征提取器提取局部特征信息 (怎么提取特征)

这两个问题是相关的，如果和 CNN 做一个类比，在CNN中卷积核是基本的特征提取器，每个卷积核对应一个 n*n 的像素区域，在PointCloud中同样要找到结构相同的子区域和对应的特征提取器。

那么在 PointNet++ 中作者使用欧式空间中的邻接球作为子区域做点集的 Partition 使用 PointNet 作为特征提取器。

## 网络结构
![](https://pic1.zhimg.com/80/v2-f3f9a70d0052be1949a18c6e556572b8_1440w.jpg)

主要包括三部分
### Sample layer
主要对输入点进行采样，使用最远点采样法，选择$N$个点，能够更均匀的覆盖整个点集。
![](https://img-blog.csdnimg.cn/20200923132734170.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1FUVkxD,size_16,color_FFFFFF,t_70#pic_center)

### Grouping layer
确定点的局部划分，即确定邻接球的半径和内部的点的数量。
一种是 ball query 看半径，还有就是用 KNN 看内部点的数量

### PointNet layer
通过 PointNet 进行特征提取，假设每个 Group 中有 $n$ 个点，那么通过 PointNet 将 $n\times 3$ 的矩阵转换成 $n\times K$ 的矩阵。迭代进行操作，感受野逐渐扩大，维数逐渐提升。

## 不均匀点云数据

我们知道点云数据不像图片中的信息是紧密连接的，空间中的点有稀疏和稠密之分。这时候如果平均采样那么会在空间中点稀疏的地方造成浪费，在点密集的地方却不足以提取到足够的信息。为此 PointNet++ 作者提出了两种方式来保证更加优化的特征提取。

![](https://pic1.zhimg.com/80/v2-5389688194b56daf0311e926360f8e6c_1440w.jpg)

### 多尺度组合 (multi-scale grouping, MSG)

对于同一个点多次采用不同的Grouping，过PointNet之后将特征 concat，增加了很多计算量，耗时。

### 多分辨率组合（multi-resolution grouping, MRG）

相当于多层之间采用了不同的分辨率，先用小的grouping通过两层正常提取出的特征，和一次大的grouping提取出的特征进行一个concat，组合了两种分辨率的信息。

---

## 实验效果

![](https://s3.ax1x.com/2020/12/08/rpkt39.png)

从实验效果上我们可以看出其准确率明显好于单一的PointNet，并且在使用了 MSG 和 MRG 后对于大量点集的鲁棒性明显提升了。

