---
title: PointCNN论文阅读
date: 2020-12-08 20:00:23
index_img: /img/Pic/cnn_img.jpeg
banner_img: /img/Pic/PointCloud.jpg
category: [Research]
tags: PointCNN
math: true
---

# PointCNN

回顾之前读过的 PointNet++ 是受到 CNN 启发，通过局部的特征提取器和hierarchy 结构解决了 PointNet 在 local 上的劣势。但同时这种基于 PointNet 的顺序不依赖性使用的 symmetry function 最终总会导致一部分的信息丢失，PointCNN 直接使用 CNN 架构和一种自定义的 $\chi -Conv$ 操作，使常规的卷积也能处理点云。

---

## 关键问题 - PointCloud 上的卷积

我们都知道普通的 2d 卷积的实现方式，因为在图片上像素都是紧密相连的，很好实现，但对于 3d 的 Points 点的空间位置，距离，顺序都会对卷积造成挑战和影响，下图就展示了普通的 2d卷积 和 3d PointCloud 卷积的区别。

![](https://s3.ax1x.com/2020/12/08/r9iVTU.png)


显然 PointCloud 上的卷积和形状(即点的相对位置) 和输入顺序有关。如果直接在上面做卷积就会发生灾难。

![](https://s3.ax1x.com/2020/12/08/r9infJ.png)

上图我们看出，直接做卷积的话 $f_{ii}$ 和 $f_{iii}$ 不同的形状信息被丢失了（这正是我们希望保留的），而 $f_{iii}$ 与 $f_{iv}$ 之间仅因为顺序不同而造成了结果不同，即顺序有关，这是我们不希望看到的。

对于这个的解决方式，PointCNN 的作者使用了一个 $\chi$ - transformer 矩阵，先对 sample point 作用 $\chi$ 变换矩阵，再进行 Convolution 从而做到顺序无关的同时，不像 symmetry function 会丢失信息。

![有点懒得翻译了，直接放原文](https://s3.ax1x.com/2020/12/08/r9ilOx.png)

可以看出这个 $\chi$ - transformation 是通过训练一个多层感知机获得的。同时产生输入点集的权重和排序，进而直接作用一个经典的卷积。

![](https://s3.ax1x.com/2020/12/08/r9imY4.png)

简单来说就是我们希望对于 $\chi_{iii}$ 、 $\chi_{iv}$ 对于 $f_{iii}$ 、 $f_{iv}$ 的作用能够得到相同的feature，一种简单的实现方式就是找到一个变换矩阵 $\Pi$ 使得 $f_{iii}^{T} = \Pi \times f_{iv}^T$ 这样  $\chi_{iii} = \chi_{iv} \times \Pi$ 就可以达到目标。

值得注意的是，多层感知机得出的 $\Pi$ 并非一定是理想化的0101矩阵，而是有如0.1，0.9这样接近01的值，也可以将其理解为 feature 的权重这样得到的结果并不是完美的 即 $f_{iii}^{T} ≈ \Pi \times f_{iv}^T$

---

## 点云的平移不变性

对于平移不变性的处理 PointCNN 很简单的使用了局部坐标系的转换，对于分割问题，采用最远点采样方式，然后对于采样点取 k - nearnest neighbours 将坐标系转换到采样点为中心的局部坐标系(即减去采样点坐标)

![](https://s3.ax1x.com/2020/12/08/r9iekF.png)

对于卷积层，具体算法如下

![](https://s3.ax1x.com/2020/12/08/r9i3m6.png)

- 先做feature的选点和聚合，先转换到采样点局部坐标系
- 利用MLP将每个点变换到高维空间$C_{\delta}$ （这里用的是一维卷积）
- Concat 其他特征信息
- **对每个局部区域中的点使用MLP，得到变换矩阵X**
- 将得到的 $\chi$ 作用在 $F_*$ 上生成顺序不依赖的 $F_{\chi}$
- 对顺序不依赖的 $F_{\chi}$ 做 Convolution (这里也是一维卷积)

## 可视化

![](https://s3.ax1x.com/2020/12/08/r9i80K.png)

对于 $\chi$ - transformer 的效果有上图的 visualization 可以看到 $F_*$ 中不同的 class 还是有overlap的因为对于 $F_*$ 其仅将特征提取到了高维，但其结果还是依赖于输入的顺序，所以不能够很好的划分featrue，而经过了 $\chi$ - transformer 不同的feature 被划分开来且更为密集，结果较好。

## 模型效果

![](https://s3.ax1x.com/2020/12/08/r9iMlR.png)

可以看到准确率保持在一个比较高的水平。

![](https://s3.ax1x.com/2020/12/08/r9iQ61.png)

同时 PointCNN 具有更少的参数 (0.6M) 以及极快的速度 (0.012s)
PointCNN 极具前景可以将 CNN 现有的工作进行应用。后续会看 PointCNN++ 等论文。

## 总结

PointCNN 直接将点云和CNN结合将点云的工作引入到了一个开阔的领域，但近来也有一些工作在质疑 CNN 的必要性，google 的论文 Attention is all you need 就指出在 NLP 领域 transformer attention based 方法巨大的前景，也为 3D instance segmentation 提供了新思路。
关于 PointCNN 这篇经典读的还不是很透，关于 sample 选点的顺序问题和一些训练细节还没有弄的很懂，之后找时间细看。