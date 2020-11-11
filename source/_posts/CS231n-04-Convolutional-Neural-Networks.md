---
title: CS231n Convolutional_Neural_Networks 04
date: 2020-11-11 00:57:54
tags: [CV,Neural Network]
category: [CS231n]
index_img: /img/Cs231n/top.jpg
---

### A bit of CNN History

![](https://s1.ax1x.com/2020/11/11/BOGJ5n.png)

![](https://s1.ax1x.com/2020/11/11/BOGGUs.png)

#### Fully Connected Layer

![](https://s1.ax1x.com/2020/11/11/BOGlDg.png)

#### Convolution Layer

![](https://s1.ax1x.com/2020/11/11/BOG1bQ.png)

> We just let the 5x5x3 filter $w$ to take a dot product between itself and a small 5x5z3 chunck of the image

$$W^Tx+b$$
**The $W$ and $x$ is streched into 1 dimention vec**

- Then the Outcome:

![](https://s1.ax1x.com/2020/11/11/BOG8Ej.png)

> We can use different layers on the top of it and get more activation maps stack them together to get a new image, just as the following pic depicted.

![](https://s1.ax1x.com/2020/11/11/BOGtCq.png)

> Then we can recursively do that work, make the front layer's output be the next layer's input

![](https://s1.ax1x.com/2020/11/11/BOGU2V.png)

**The Layers may look like..**
*Simple -> Complex*
 
![](https://s1.ax1x.com/2020/11/11/BOGN80.png)

![](https://s1.ax1x.com/2020/11/11/BOGrVJ.png)

- The Convolution of two signals:

$$f[x,y]*g[x,y] = \sum\limits_{n_1 = -\infin}^{\infin}\sum\limits_{n_2 = -\infin}^{\infin}f[n_1,n_2]·g[x-n_1,y-n_2]$$

- A little bit preview

![](https://s1.ax1x.com/2020/11/11/BOG0rF.png)

---

- Convolution Box

![](https://s1.ax1x.com/2020/11/11/BOGwKU.png)

> We can tell that the activation maps are becomming smaller after the filter, so we commonly use zero pad to deal with it.

![](https://s1.ax1x.com/2020/11/11/BOGBb4.png)

![](https://s1.ax1x.com/2020/11/11/BOGsa9.png)

> The Matrix shrinks from 32 —> 28 -> 24 (lose info)

---

### Summary
1. Accepts a volume of size $W_1 * H_1 * D_1$
2. Four Hyperparas
    - Number of filters $K$
    - spatial extent $F$
    - stride $S$
    - zero padding amount $P$
3. Produces a volume of size $W_2 * H_2 * D_2$
    - $W_2 = (W_1 - F + 2P)/S + 1$
    - $H_2 = (H_1 - F + 2P)/S + 1$
    - $D_2 = K$ *Depth keeps the same*

#### Common Settings

K = (powers of 2)
- F = 3, S = 1, P =1
- F = 5, S = 1, P =2
- F = 1, S = 1, P = 0 

---

### Conv details

- One by One CONV

![](https://s1.ax1x.com/2020/11/11/BOGy5R.png)

- EXAMPLE: CONV in pyTorch

![](https://s1.ax1x.com/2020/11/11/BOGRxK.png)

- The Brain/Neuron View of CONV

![](https://s1.ax1x.com/2020/11/11/BOG226.png)

- Pooling layer

![](https://s1.ax1x.com/2020/11/11/BOGg8x.png)

> Just spacially down sample the image to make it smaller.
Common Practice is Max Pooling

![](https://s1.ax1x.com/2020/11/11/BOGcP1.png)

> We may can just use stride to replace pooling?


