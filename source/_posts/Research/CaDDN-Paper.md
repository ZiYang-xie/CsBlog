---
title: CaDDN论文阅读
date: 2021-03-09 20:20:50
index_img: /img/AI.png
category: [Research, Object Detection]
tags: [Monocular OD]
math: true
---

# **Categorical Depth Distribution Network for Monocular 3D Object Detection**

**关键词： 单目3d检测、绝对深度分配网络**

**论文链接：**https://arxiv.org/pdf/2103.01100.pdf

![](https://tva1.sinaimg.cn/large/008eGmZEgy1godx97xntnj30bl08p0wn.jpg)

## 前言

​	单目3D检测的难点一直在于对深度的预测，实例从3D空间被映射到2D平面的图像上丢失了深度信息，对深度的处理一直是单目3D目标检测研究的重点方向，目前主流的方法主要分为三类。

**1、直接检测  (Direct Methods)**

​	直接检测方法没有显式的对深度进行学习，比较有代表性的是关键点检测方法，通过关键点结合几何特征来帮助3D-Bbox的检测，好处是简单直接且高效，但这类方法由于没有显式的学习深度信息，往往导致深度预测的结果不尽理想。

**2、基于深度  (Depth-Based Methods)**

​	基于深度的方法通常先会通过一个单目深度预测分支来得到一张深度图作为网络的输入从而辅助对深度的检测，由于有了深度信息其可被转换成点云来处理（可以用上3d检测的方法)。但由于其深度和目标检测分离训练的结构，导致其可能会丢失一些隐含的信息。

**3、基于网格  (Grid-Based Methods)**

​	基于网格的方法避免了对深度的直接预测，而是通过预测一个 BEV grid 的表达来作为3D检测网络的输入，OFT[1]提出了一种体素网格，通过把体素投影到图像平面上进而采样图像特征将其转换成BEV的形式。但这也会导致大量体素和特征的重叠从而降低检测的准确性。



​	**CaDDN** 网络对上面三种情况的优点进行结合，整体网络结构是同时训练了深度预测和3D检测（jointly）以期待其能够解决方法2中的问题，同时利用也将图像平面转换成了BEV的形式来提高检测的准确性。

![网络结构](https://tva1.sinaimg.cn/large/008eGmZEgy1godxz73f1jj30nm0dntcy.jpg)



## 网络结构

### Frustum Feature Network

- **Depth Distribution Network**

![](https://tva1.sinaimg.cn/large/008eGmZEgy1godzoe00t7j30b908dabe.jpg)

对于特征提取网络，其输出形似一个截角锥形，因而作者称其为Frustum feature Network，其输入是原始图像 $I \in R^{W_1\times H_1\times 3}$ 输出 $G \in R^{W_F\times H_F\times D \times C}$ 其中 W和H是特征的宽高，D 是深度桶，用以深度预测，C 是特征的维度。图像特征被用以在每个像素上预测绝对的深度分布 $D \in R^{W_F\times H_F\times D}$ 网络对每个像素预测其落入某一深度桶（深度离散化）的概率，总共D个。

（这一网络其是从 DeepLabV3[2] 上魔改过来的。）

- Image Channel Reduce

同时在分配深度桶的同时，网络另一分支用1x1卷积 + BN + ReLU 把特征的维数从256降到了64。



​	经过这两个分支后，将预测出来的深度桶和特征像素做外积得到了带有深度信息的特征图（$G(u,v) = D(u,v) \otimes F(u,v)$）且由于特征桶的结构，具有较高的容错性。称 G 为 frustum features



### Frustum to Voxel Trans

![](https://tva1.sinaimg.cn/large/008eGmZEgy1godzokeng2j30b907ujsr.jpg)



​	G 之后将被转换成体素的形式，这里的问题是在第分辨率的frustum features进行高分辨率的体素采样会导致过采样，导致出现大量相同的体素特征，浪费算力不说还降低预测准确性。所以这边作者直接把降采样前的特征层拉了过来，来保证不会出现上述问题。



 ### Voxel Collapse to BEV -> Detection

​	由于BEV在保证同等检测效果的情况下能够节省计算资源，作者将体素的 Z 轴和 channels 直接连接起来，继续用1x1卷积 + BN + ReLU 将体素块压缩成 BEV的形式。然后接着在 BEV 特征块上连接检测头进行 3D目标检测。



## 深度离散化

​	本文的主要亮点就是其对于深度的处理，其对每个像素位置分配了离散化的深度桶，预测其属于某一深度的概率。这里的深度离散化作者使用的是 LID (Linear-increasing discretization)[3] 因为其对不同深度之间提供了最为平衡的预测概率。对于检测任务的深度我们最需要的是检测目标的深度信息，而更少去在意背景点的深度。

​															$$d_c = d_min + \frac{d_{max}-d_{min}}{D(D+1)}\cdot{d_i}{(d_1+1)}$$

​	*（其中 $d_c$ 是连续深度值，[$d_{min},d_{max}$]是离散化的上下界，D是深度桶的数量，$d_i$ 是下标）*

![](https://tva1.sinaimg.cn/large/008eGmZEgy1goe03wlr9sj30dn0bd0tp.jpg)



## 实验效果

![KITTI实验效果](https://tva1.sinaimg.cn/large/008eGmZEgy1goe0ckxhf6j30r10fgn17.jpg)

​	可以看到其在车辆和新人检测上都打破了目前最好的检测方法，对骑行者的检测不如 MonoPSR 但也大大超过了其余检测模型。

![](https://tva1.sinaimg.cn/large/008eGmZEgy1goe0gh8vyhj30dw07jdgw.jpg)

这张表可以看出其将深度预测和特征融合所得到的 frustum features 确实有助于提升检测效果。



## 结语

​	个人觉得这篇paper的主要成功点在于其对于单目深度预测的创新解决方法，不同于传统的深度预测模型，提出了离散化深度将每个像素的深度概率离散化分配在不同的深度桶中，避免了网络过度依赖于深度的准确检测。但同时其也没有从根本上解决深度预测的问题，同时其中间对于特征图进行体素化投影再转为 BEV 的过程较为复杂，虽然其把降采样前的特征层拿来转成体素形式但是仍然还是无法避免会有损失。

​	个人觉得可以借鉴该模型的深度预测方法，应用在现有的模型上或者对该模型的网络结构进行简化，可能会带来更好的效果。



---

### Reference

[1]  Thomas Roddick, Alex Kendall, and Roberto Cipolla. Ortho graphic feature transform for monocular 3D object detection.*BMVC*, 2018

[2] Liang-Chieh Chen, George Papandreou, Florian Schroff, andHartwig Adam. Rethinking atrous convolution for semantic image segmentation. *arXiv preprint*, 2017

[3] Yunlei Tang, Sebastian Dorn, and Chiragkumar Savani. Center3d: Center-based monocular 3d object detection with joint depth understanding. *arXiv preprint*, 2020