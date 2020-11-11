---
title: CS231n Loss Functions and Optimization 02
date: 2020-11-08 20:52:06
tags: [CV,Neural Network]
category: [CS231n]
index_img: /img/Cs231n/top.jpg
---

### Preview the Goal in this lecture
1. Define a loss function
2. Come up with a way of finding the paras that minimize the (1)
(optimization)

**The Remain Problem from last lecture**

- How to choose the W para ? 

![](https://s1.ax1x.com/2020/11/08/BTZxgK.png)

### Loss function

> A loss function tells how good our current classifier is.

$${(x_i,y_i)}_{i=1}^N$$

The $X_i$ is image and the $y_i$ is label (int)

The Total loss is defined as the func follows.

$$L = \frac{1}{N}\sum\limits_iL_i(f(x_i,W),y_i)$$
*Which is the sum of every single test's loss*

---

#### **Muticlass SVM loss**

Given an example $(x_i,y_i)$ where $x_i$ is the image and where $y_i$ is the (int) label, using the shorthand for the score vec $s = f(x_i,W)$

The SVM loss has the form:

![](https://s1.ax1x.com/2020/11/08/BTZ7B4.png)

> if the incorrect score is smaller than the right score (x margin), we set the loss to 0.
in this case the safe margin is set to one
**Margin choice depends on our need**


- Then we loop the class

![](https://s1.ax1x.com/2020/11/08/BTZqE9.png)

![](https://s1.ax1x.com/2020/11/08/BTZLNR.png)

- What if we use

$$L = \frac{1}{N}\sum\limits_iL_i(f(x_i,W),y_i)^2$$
> This is not a linear function and totally different, it's may be useful sometimes depends on the way you care about the errors.

#### Example Code

```python
def L_i_vectorized(x, y, W):
    scores = W.dot(x)
    margins = np.maximun(0, scores - scores[y] + margin)
    margins[y] = 0
    loss_i = np.sum(margins)
    return loss_i
    # pretty easy
```

![](https://s1.ax1x.com/2020/11/08/BTZO41.png)

> It just change the gap bettween scores

![](https://s1.ax1x.com/2020/11/08/BTZzjO.png)

![](https://s1.ax1x.com/2020/11/08/BTe9De.png)

> often use L2 regularization just Euclid norm.

![](https://s1.ax1x.com/2020/11/08/BTepuD.png)

> In this case the L1 and L2 reg is equal, but we can tell that L1 prefers the $w_1$ for it contains more zero, while the L2 prefers the $w_2$ for the weight is evenly spreaded through the test case.

> The Multiclass SVM loss just care about the gap bettween the right labels and the wrongs.

#### **Softmax Classifier**

![](https://s1.ax1x.com/2020/11/08/BTeiEd.png)

> We just want to make the true probability closer to 1 (closer the better, eq is the best), so the loss func can be chosed by using the -log on the $P$.

![](https://s1.ax1x.com/2020/11/08/BTeCHH.png)

> If we want to get the zero loss, the score may goes to inf! But Computer don't like that.

- Debugging Way
outcomes might be $logC$

---

![](https://s1.ax1x.com/2020/11/08/BTek4I.png)

![](https://s1.ax1x.com/2020/11/08/BTeECt.png)

---

### Optimization

#### Random Search - The Naive but Simplest way 
> Really Slow !!!

#### Gradient Descent
> We just get the Gradient of W and go down to the bottom (maybe local best?)

![](https://s1.ax1x.com/2020/11/08/BTeFUA.png)

**Code**

```python
# Vanilla Gradient Descent

while True:
    weight_grad = evaluate_gradient(loss_fun, data, weights)
    weights += -step_size * weight_grad
```

**Step size is called elearning rate which is important**

![](https://s1.ax1x.com/2020/11/08/BTeV8P.png)

> Since the N might be super large, we sample some sets called minibatch and use it to estimate the true gradient.

![](https://s1.ax1x.com/2020/11/08/BTeZgf.png)


---

![](https://s1.ax1x.com/2020/11/08/BTenKS.png)

![](https://s1.ax1x.com/2020/11/08/BTeuDg.png)

**Color Feature**
![](https://s1.ax1x.com/2020/11/08/BTeQEj.png)

**Gradient** *Extract the edge info*
![](https://s1.ax1x.com/2020/11/08/BTelUs.png)

**NLP?**
![](https://s1.ax1x.com/2020/11/08/BTeG80.png)

> clustering different image patches from images

![](https://s1.ax1x.com/2020/11/08/BTe15n.png)
- Differences
1. Extract the Feature at first and feed into the linear classificator
2. Convolutional Neutral Network would learn the feature automatically during the training process.