---
title: 排序算法及编译器优化效果对比
date: 2021-03-08 22:11:52
index_img: /img/ICS_Lab1/top.jpg
category: [ICS]
tags: [Sort]
---



## *ICS (Normal) 第一周作业*



### 冒泡与快排用时对比



![冒泡排序](https://tva1.sinaimg.cn/large/008eGmZEgy1gocut0coevj31do0sggpl.jpg)



### 相同算法在 GCC 编译优化下用时对比



![冒泡排序](https://tva1.sinaimg.cn/large/008eGmZEgy1gocvbzmuy0j31cy0s842c.jpg)



![快速排序](https://tva1.sinaimg.cn/large/008eGmZEgy1gocut5qwjwj31a80smjvj.jpg)



![优化快速排序](https://tva1.sinaimg.cn/large/008eGmZEgy1gocutaoylej31b40sqtcm.jpg)





### 不同排序算法在不同GCC优化下的变化



![冒泡排序](https://tva1.sinaimg.cn/large/008eGmZEgy1gocutf9rb3j31cs0s8dih.jpg)



![快速排序](https://tva1.sinaimg.cn/large/008eGmZEgy1gocuthxygzj31b20t4jtt.jpg)



![优化快速排序](https://tva1.sinaimg.cn/large/008eGmZEgy1gocutkvifhj31b20swdi8.jpg)



---



### 改进快速排序



对于快速排序的改进策略一般有两种



- ***Pivot 三值取中***

快速排序最坏情况是枢纽元为最大或者最小数字，那么所有数都划分到一个序列去了时间复杂度为O(n^2)，为了保证最坏情况不出现，尽量使pivot能够二分序列，我们采用三值取中法，既开头、中间、结尾三个元素选取大小中等的元素与首元交换成为Pivot，避免最坏情况出现。



**效果：**在应用三值取中后可以发现优化快排的排序速度变得较为稳定，不像普通版本一样依赖于序列的形态。



- ***小规模插入排序***

即当快速排序所划分的子序列的长度小于某个定值k时，该子序列基本有序，可以采用插入排序的办法对子序列进行排序，从而使整体算法的时间复杂度的期望下降为 $O(nk+nlg(\frac{n}{k}))$



### 实验效果



![](https://tva1.sinaimg.cn/large/008eGmZEgy1gocuto6qwgj317w0t076c.jpg)



可以看到在同等数据量下，对于快速排序具有一定优化效果



![](https://tva1.sinaimg.cn/large/008eGmZEgy1gocutsrs9ij31au0sctcp.jpg)



![](https://tva1.sinaimg.cn/large/008eGmZEgy1gocutvjlj6j31bg0rkwig.jpg)



![](https://tva1.sinaimg.cn/large/008eGmZEgy1gocutym2uij31bm0sotct.jpg)



![](https://tva1.sinaimg.cn/large/008eGmZEgy1gocuu1tn5uj31b20swdk0.jpg)



![](https://tva1.sinaimg.cn/large/008eGmZEgy1gocuu4kk90j31b20s4n1a.jpg)



可以看到，随着优化等级的增加，优化的快排几乎成线性，更加稳定，而普通快排有所波动。