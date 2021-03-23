---
title: CS231n Training Neural Networks I 05
date: 2020-11-13 21:09:24
tags: [CV,Neural Network]
category: [CS231n]
index_img: /img/Cs231n/top.jpg
math: true
---

### OverView
1. One time setup
2. Training dynamics
3. Evaluation

### Part 1
- Activation Functions
- Data Preprocessing
- Weight initialization
- Batch Normalization
- Babysitting the Learning Process
- Hyperparameter Optimization

---

### Activation Functions

![](https://s3.ax1x.com/2020/11/13/D9uofs.png)

![](https://s3.ax1x.com/2020/11/13/D9uIYj.png)

#### Sigmoid 

$$\sigma(x) = 1/(1+e^{-x})$$

- Squashes numbers to range [0,1]
- Historically popular "firing rate" of a neuron

![](https://s3.ax1x.com/2020/11/13/D9uhTg.png)

**3 Problems**
1. Saturated the neural may kill the gradient.

![](https://s3.ax1x.com/2020/11/13/D9u5kQ.png)

> x is a very negative and very positive val, its gradient will be killed to zero

1. Sigmoid outputs are not zero-centered

![](https://s3.ax1x.com/2020/11/13/D9u7pn.png)

> For the sign of x and gradient is always the same, it gonna behaves like is above pic.

> thats why we need zero-mean data, to optimize the w just through the zig zag path.

1. exp() is a bit compute expensive

---

#### Tanh

- Squashes numbers to range [-1, 1]
- zero centered (nice)
- still kills gradients when saturated

![](https://s3.ax1x.com/2020/11/13/D9uHlq.png)

---

#### ReLU

$$f(x) = max(0, x)$$

![](https://s3.ax1x.com/2020/11/13/D9ub60.png)

- Does not saturate (in + region)
- Very computationally efficient
- Converges much faster than sigmoid/tanh in practice
- Actually more biologically plausible than sigmoid

**Problems**
1. Not zero-centered output
2. An annoyance:
3. when x <= 0 the gradient is slashed to zero (kill half the gradient)

![](https://s3.ax1x.com/2020/11/13/D9uX0U.png)

- Bad Init
- Learning rate too high

> people like to initialize ReLU neurons with slightly positive biases (eg 0.01), to increase the possibility that being activated.

---

#### Leaky ReLu

$$f(x) = max(0.01x,x)$$

![](https://s3.ax1x.com/2020/11/13/D9uqXV.png)

- Does not saturated
- Computationally efficient
- Converges much faster ...
- **will not die**

or **Para Rectifier ReLu**

$$f(x) = max(\alpha{x},x)$$

---

#### Exponential Linear Units (ELU)

![](https://s3.ax1x.com/2020/11/13/D9uOmT.png)

- All benefits of ReLU
- Closer to zero mean outputs
- Negative saturation regime adds some robustness to noise

*While it requires exp()*

---

#### Maxout "Neuron"

![](https://s3.ax1x.com/2020/11/13/D9uj7F.png)

---

#### In practice

- Use ReLU. (zbe careful with learning rates)
- Try Leaky *ReLU / Maxout / ELU*
- Don't use sigmoid

---

### Data Preprocessing

#### Step1: Preprocess the data

![](https://s3.ax1x.com/2020/11/13/D9uxk4.png)

- In CV we usually don't normalize the data.

![](https://s3.ax1x.com/2020/11/13/D9KSh9.png)

Practice above make the data zero-mean, but only the first layer, and that's why we need the activation func tobe zero-mean.

---

### Weight Initialization

#### First idea: Small random numbers

```python
W = 0.01* np.random.randn(D,H)
```

> Works Okay for small, but have problems in big one.

![](https://s3.ax1x.com/2020/11/13/D9KC11.png)

#### How about making Weight big?

![](https://s3.ax1x.com/2020/11/13/D9KP6x.png)

> it gonna saturated the regime to be either very possitive or very negative input of tanh, and comes out near zero gradients. The weight will not be updated.


#### Xavier initialization
*Woo my initialzation? haha*

![](https://s3.ax1x.com/2020/11/13/D9KiX6.png)

> ensure we are at the active region of tanh

![](https://s3.ax1x.com/2020/11/13/D9KA0O.png)

> Can be addressed by add an extra /2， to ensure the neural won't die in ReLU

---

### Batch Normalization

![](https://s3.ax1x.com/2020/11/13/D9KknK.png)

> To regulize the input tobe unit gaussian.

*I have no idea about it. What is unit gaussian?*

![](https://s3.ax1x.com/2020/11/13/D9KetH.png)

![](https://s3.ax1x.com/2020/11/13/D9KE7D.png)

![](https://s3.ax1x.com/2020/11/13/D9KZAe.png)

![](https://s3.ax1x.com/2020/11/13/D9Kmhd.png)

---

### Babysitting the Learning Process

- 1. Preprocess data
- 2. Choose the architecture:
- 3. Double check that the loss is reasonable

![](https://s3.ax1x.com/2020/11/13/D9KK1I.png)

#### The Learning Rate

- Very small learning rate 1e-6
> litter help

![](https://s3.ax1x.com/2020/11/13/D9K3B8.png)

- Very great learning rate 1e6
> go extreme

![](https://s3.ax1x.com/2020/11/13/D9KQjP.png)

- A Rough Range

$$1\times{10}^{-3} \to 1\times{10}^{-5} $$

---

### Hyperparameter Optimization

- Cross-validation strategy
*coarse -> fine*

- Random Sample

![](https://s3.ax1x.com/2020/11/13/D9Ku9A.png)

![](https://s3.ax1x.com/2020/11/13/D9KMct.png)

![](https://s3.ax1x.com/2020/11/13/D9K1nf.png)
