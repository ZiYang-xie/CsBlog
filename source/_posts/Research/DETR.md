---
title: DETR 论文阅读
date: 2021-01-22 13:47:44
index_img: /img/AI.png
category: [Research, Object Detection]
tags: [Transformer]
math: true
---

# End-to-End Object Detection with Transformers

*关键词： 目标检测， Transformer，End-to-End*



## 简介

​	Detr 这篇文章抛弃了传统 Fast-RCNN 基于 ROI 的目标检测方式，使用了Transformer以及其提出的 bipartite matching 做到位置无关和生成唯一的目标检测框，以此省去了 Anchor 和 NMS。以非常简单的架构达到媲美甚至超越 Fast-RCNN 的准确率，在能够做目标检测的同时，该模型还有较好的迁移能力，在原论文中通过 Transformer 的 Attention 机制实现了全景分割。




## 结构分析

![网络结构图](https://tva1.sinaimg.cn/large/008eGmZEgy1gmwgc8ubqoj310s07t435.jpg)

​	DETR 的网络结构很简单，分为三个部分，第一部分是一个传统 CNN 用于提取图片特征到更高维度，第二部分一个Transformer 的 Encoder 和 Decoder 来提取 Bounding Box，最后使用 Bipartite matching loss 来训练网络。



![结构细节](https://tva1.sinaimg.cn/large/008eGmZEgy1gmwghk322pj30m8069adc.jpg)

​		更加细致的划分可划分为 CNN backbone 部分，Transformer 中的 Encoder 和 Decoder 部分，预测前馈网络 (FFN) 部分。接下来会详细讲解。



### CNN 部分

​	DETR 的第一部分用了一个 CNN backbone 将 $x_{img} \in R^{3 \times H_0 \times W_0}$ (3 的 RGB 深度) 转换为 $f \in R^{C \times H \times W}$ 的特征层。论文中用了 $C = 2048, H = \frac{H_0}{32}, W = \frac{W_0}{32}$ 



### Transformer 部分

#### Encoder

​	先用 1x1 的卷积核将纬度从 C 降到 d，获得一个新的特征图 $R^{ d \times H \times W}$ 。由于 Encoder 需要一个序列，我们要将特征图拉平成为 $d \times HW$ 的向量，输入到 encoder 中，每一个 encoder 都是同样的结构，由一个多头注意力模块和一个前馈网络（FFN）组成。不像RNN，transformer 的输入是顺序无关的，于是我们也学 NLP 对每一个注意力层加一个位置编码 （position encodings）。将状态编码和之前拉平的特征图向量相加之后喂入 encoder 中。

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gmwioerjhbj30ea0ghgq4.jpg" style="zoom:80%;" />

#### Decoder

​	decoder 的输入有两个，一个是 Object Queries 另一个是刚刚 Encoder 的输出，结构和传统 Transformer 差不多。比较有意思的是这个输入的 Object Queries，由于 Transformer 是 fixed size，如果我们需要 N 个 Bounding Box 那么我们就需要 N 个输入，同时这个 Object Queries 顺便充当了 decoder 的 position encodings，是通过学习得来的。

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gmwi2hocbcj30eh0gb421.jpg" style="zoom:80%;" />

​	encoder 的输出直接喂到的是 Encoder-Decoder Attention 层，这 N 个位置嵌入要先通过自注意力层才获得 encoder 的信息。作者将这 N 个 序列最后生成的 Bounding Box 拿出来可视化，结果非常 Amazing 啊。

​	![](https://tva1.sinaimg.cn/large/008eGmZEgy1gmwinw5iwbj30wn075ale.jpg)

​	图中的不同颜色的点代表不同大小形态的 Bounding Box，绿色代表较小的 Bounding Box，紫色代较大的 Bounding Box，红色代表大的水平的 boxes，蓝色代表大的竖直的 boxes。 对于每一个输入序列，其都有所侧重，有的侧重与左侧的小 Bounding Box 有的侧重于中间大的Bounding Box。



### Bipartite matching loss

​	之前检测器往往通过anchor和groundtruth的IOU来确定正负样本，而DETR使用了bipartite matching loss 来确定正负样本。

​	![bipartite matching loss](https://tva1.sinaimg.cn/large/008eGmZEgy1gmzoqacu30j307d01waa0.jpg)

​	其中 $L_{match}$ 是 ground true 和预测 bounding boxes 之间的 pair-wise matching cost （一一配对 penalty）

​	![bipartite matching](https://tva1.sinaimg.cn/large/008eGmZEgy1gmzoq11i5jj303405hmxa.jpg) 

​	通过一一配对，就不需要再采取传统的 NMS 了，因为一个bounding box 只能和一个 ground true 进行匹配，必然会引入 loss。算法实现采用的是**匈牙利算法**，这个后期我会写在博客上。



### FFN

​	最后得出结果的网络，是一个三层的感知机，activation function 用的是 ReLU，以及一个线性映射层。输出的是每一个 bounding box 的中心坐标，以及它的宽高 $(x,y,w,h)$ 以及物体的分类。



## 实验

​	原文在 COCO 2017 数据集上做的实验，模型虽然简单但却消耗了大量的训练时间 （Training the baseline model for 300 epochs on 16 V100 GPUs takes 3 days, with 4 images per GPU），最后效果媲美甚至超过了经过良好调教的 Fast-RCNN 类的人工 head + anchor 的模式。

​	![实验结果](https://tva1.sinaimg.cn/large/008eGmZEgy1gmzpkgtz3xj30gj06n75k.jpg) 



## 总结

​	这篇文章还是相当惊艳的，一直以来不管是 anchor based 还是 anchor free 的目标检测方法都难以脱离人工定义 anchor 的过程，而本篇文章通过使用 Transformer + positional encoding 达到甚至超越了传统方法的性能，里面特别是 positional encodings 的 object queries 非常耐人寻味，这些 queries 学到的真的只是 positional encoding吗？这个 queries 是否有可能是 tasks 无关的？如果是能不能通过预训练的方式来提高训练速度，这些问题都是后期值得探索的。



## 参考

[1] Nicolas Carion, Francisco Massa, Gabriel Synnaeve, Nicolas Usunier, Alexander Kirillov, Sergey Zagoruyko: “End-to-End Object Detection with Transformers”, 2020; [arXiv:2005.12872](http://arxiv.org/abs/2005.12872)

[2] 如何评价FAIR的新论文DETR？ https://www.zhihu.com/question/397692959/answer/1258046044 

[3] 如何看待End-to-End Object Detection with Transformers? https://www.zhihu.com/question/397624847/answer/1250143331

[4] 详解Transformer （Attention Is All You Need）https://zhuanlan.zhihu.com/p/48508221

