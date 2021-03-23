---
title: PointNet论文阅读
date: 2020-12-06 14:46:14
index_img: /img/Pic/cnn_img.jpeg
banner_img: /img/Pic/PointCloud.jpg
category: [Research]
tags: PointNet
math: true
---

### PointCloud

何谓点云，点云数据和普通的照片又有什么不同？实际上点云并非什么高深的东西，下图就是一张点云的可视化数据。
![点云街道](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1607249767189&di=e4964b8b875326b97e5ab06da6196601&imgtype=0&src=http%3A%2F%2F5b0988e595225.cdn.sohucs.com%2Fq_70%2Cc_zoom%2Cw_640%2Fimages%2F20180118%2F537a4b5eab09459881f9bc6ca18834f9.jpeg)

- **位置信息**
不同于原先2D的图像，每个像素点仅有RGB信息，点云自带了欧式空间中的位置信息，即坐标(x,y,z) 之后会附带一些RGB灰度值之类的普通图像像素就具有的东西。

了解了点云之后我们来解析一下PointNet这篇论文[1]

---

### 简介
不同于当时许多处理3D点云任务时，直接将3D点云转化成体素矩阵(Voxel grid)的方式，但由于3D空间不同于2D大量的空间里都是空的(zero grid)，将他们全部体素化会造成不必要的浪费。PointNet可以**直接输入点云**信息，来解决这一问题，同时避免一些转换中造成的其他问题。

### 点云数据的特点

**1. 无序性 （Unordered）**
我们都知道在欧式空间的度量中点和点之间是不存在像一维二维空间中的序关系的。(可能不严谨，数学不太好，暂时就这样理解)
对于点云数据信息，每个点**喂入的顺序不应当影响到结果**

**2. 局部相关性 (Interaction among points)**
我们知道在**空间中的点和其周边点之间的位置关系信息是有意义的**，它代表了物体的形状特征。这正是我们在2D CNN工作中卷积层所提取的东西，因而我们要保留这个特征信息。

**3. 不变性 （Invariance under transformation）**
对于点云数据应该满足一些**空间变换的不变性**，例如平移和旋转，这些都不会影响最终的结果

PointNet的工作就是尝试解决这三个问题。

---

### 网络结构

![](https://pic1.zhimg.com/80/v2-8dc76710bd09c25d5c8196d6aff56fec_1440w.jpg)

我们看一下 PointNet 的网络架构，首先喂入原始的 PointNet，是一个 $N\times 3$ 的矩阵(n个点,xyz)先通过一个 $T-Net$ 层进行对齐，然后用感知机 mlp 进行特征提取，装换到 $N\times 1024$ 1024维空间上再进行 Max Pooling 提取出 Global feature，然后就干自己该干的事去了，对分类任务，将全局特征通过mlp来预测最后的分类分数；对分割任务，将全局特征和之前学习到的各点云的局部特征进行串联，再通过mlp得到每个数据点的分类结果。
下文会展开更详细的讲解

### 解决无序性 (symmetry function)

对于无序性 PointCloud 数据的处理当时一般有三种方法，
- 1）对数据进行排序成 canonical order. 

对于这种方法，其实不存在一种高维空间的排序。由于CNN中数据抽象纬度很高，要保证高维中的点向一维的稳定映射无关数据顺序是十分困难的，所以其效果有限。

- 2) 用大量打乱数据序列训练一个 RNN，寄希望于这种方式能消除顺序依赖

这种方法看上去可行，然而其仍然无法做到完全顺序无关，在[2]中有详细证明。实验表示，这种方法对于小型数据具有一定鲁棒性，但对于大量point的场景仍然表现疲软。

**3) 通过一个简单的对称函数来处理每个点，获得每个点的 Global signature**

这是本文所使用的方法，很简单却很有效，我们知道一个symmetry function 的输出是无关顺序的, 例如我们熟悉的加法和乘法，这里选用了CNN中常用的 max 函数作为 symmetry function，通过 max pooling 消除数据顺序的影响。

![](https://s3.ax1x.com/2020/12/08/rpks4e.png)

$f$ 这个函数输出一个数据不依赖的值作为该组数据的 *Global signature* 每一个输入值 $x_{1\to n}$ 是PointCloud中的点，带有 (x,y,z)位置信息以及一些rgb信息，组成共 $NxN$ 的矩阵，$h$ 是一个变换方程，将 $1\times N$的向量转换为 $1\times K$，然后 $g$ 就是我们的 symmetry function 将 $N\times K$ 的矩阵映射到一个值 $R$ 就能够代表这组数据，因为 symmetry function 的顺序无关性，我们能够保证这个值对于相同数据的不同排列是相同的。

---

### 局部和全局数据聚合 (Local and Global Information Aggregation)

这边先讲一下我自己的想法，我很怀疑它到底有没有做局部信息的处理，既解决第二个问题，据PointNet这篇文章自己写的这一章，他说他做了。然后后面PointNet++ 说他没做。我这段看的也比较迷惑，但我个人倾向于原作者尝试去做但做的比较弱。

### 理论证明可行性 (Theory Analysis)
![拟合可行性](https://pic3.zhimg.com/80/v2-1bee125c29ac11faba0e0a095207a396_1440w.jpg)

这个定理说我们通过之前讲的 $h$ 和 Max pooling 操作能够拟合任意的连续集合函数 $f$
最坏的拟合就是将其转换成空间中的体素，但通常来说我们的神经网络在调校的过程中都会得到更好的 $h$ 来完成这一过程。

TODO: 详细证明过程在附录中我还没看

![扰动下的鲁棒性](https://pic1.zhimg.com/80/v2-43d9f406855cf5e4681cb3de08382b80_1440w.jpg)

定理二告诉了我们该模型在扰动之下具有鲁棒性，只要不影响关键点集 $C_S$，或者超出最密上限点集 $N_s$ 该模型所得出的 Global signature 都是相同的，(2) 指出关键点集的上限在于我们的转换函数 $h$ 转换的维数 $K$

对于定理二原论文中有直观的图解帮助大家构造 intuition
![](https://pic2.zhimg.com/80/v2-c1fc6e865ab685dcaefd18e8c063bef1_1440w.jpg)

最后的结果在当时看来是非常不错的。
![](https://s3.ax1x.com/2020/12/08/rpk2jI.png)

## 总结

PointNet 开创了3D点云的新纪元，日后的PointNet++，PointCNN等网络都是在其基础上发展起来的，作为点云的奠基其用十分简单的方式解决了3D点云数据的顺序 (symmetry function) 和空间不变性 (high demension). 但它没有很好的解决 Local 数据的关系，这一问题会在PointNet++中得到解决。

最后说点自己的话，感觉现在读论文并不是很静得下心来，读的也很笼统不深入，被太多乱七八糟的事情困扰心烦意乱。但事情总得做下去，状态是在做事的过程中一点点找回来的，干等等不来。注重积累一点点来，相信自己在不久的将来能够做出一点自己的东西。

### Reference
[1] PointNet论文链接：https://arxiv.org/abs/1612.00593
[2] O. Vinyals, S. Bengio, and M. Kudlur. Order matters: Sequence to sequence for sets. arXiv preprint arXiv:1511.06391, 2015.

