---
title: CS231n Introduction to Convolutional neural network 03
date: 2020-11-10 00:57:26
tags: [CV,Neural Network]
category: [CS231n]
index_img: /img/Cs231n/top.jpg
math: true
---

### Computational graphs

![](https://s1.ax1x.com/2020/11/10/BbDAaD.png)

### BackPropagation - A method to compute the gradients of abitrarily complex function

- A recursive application of Chain rule

![](https://s1.ax1x.com/2020/11/10/BbDCKx.png)

> We get the gradient backprop from the front and comupte with the local gradient to prop to the back.

![](https://s1.ax1x.com/2020/11/10/BbDPr6.png)

> In some cases, some part of the graph can be represented by some func that we already know to simplify the computations. (trade off the math)

![](https://s1.ax1x.com/2020/11/10/BbDiqK.png)

- Patterns in backward flow

1. **add gate:** gradient distributor (local = 1)
2. **max gate:** gradient distributor (local = 1 & 0)
3. **mul gate:** gradient switcher (local = y & x )

*Gradients add at branches n->1*

- Then We Got the gradients in the form of Jacobian Matrix

![](https://s1.ax1x.com/2020/11/10/BbDkVO.png)

![](https://s1.ax1x.com/2020/11/10/BbDZPH.png)

*This place include some linear algebra*

### Modularized implementation: Forward / Backward API

```python
class ComputationalGraph(object):
    def forward(inputs):
        # 1. pass inputs to input gates
        # 2. forward the computational graph
        for gate in self.graph.nodes_topologically_sorted():
            gate.forward()
        return loss
    def backward():
        for gate in reversed(self.graph.nodes_topologically_sorted())
            gate.backward() # compute the gradients
        return inputs_gradients
```

#### EXAMPLE: MulGate

```python
class MultiplyGate(object):
    def forward(x, y):
        z = x*y
        self.x = x
        self.y = y
    def backward(dz):
        dx = self.y * z
        dy = self.x * z
        return [dx, dy]
```

> This practice is common.

![](https://s1.ax1x.com/2020/11/10/BbDeGd.png)

### Summary so Far

![](https://s1.ax1x.com/2020/11/10/BbDmRA.png)

### Neural networks

(Before) Linear score function: $f = Wx$
(Now) 2-layers Neural Network $f = W_2max(0, W_1x)$
....or more layers

![](https://s1.ax1x.com/2020/11/10/BbDEIe.png)

> The h is the scores W1 output, and we put one more linear layer W2 on the top of it to weighting the scores given by h 
