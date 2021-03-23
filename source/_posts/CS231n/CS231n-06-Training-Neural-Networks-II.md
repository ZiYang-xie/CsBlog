---
title: CS231n Training Neural Networks II 06
date: 2020-11-13 21:11:23
tags: [CV,Neural Network]
category: [CS231n]
index_img: /img/Cs231n/top.jpg
math: true
---

### PreView
- Fancier optimization
- Regularization
- Transfer Learning

---

### Problem with SGD

![](https://s3.ax1x.com/2020/11/13/D9JOns.png)

> The zig-zag path reveal the drawbacks of SGD

![](https://s3.ax1x.com/2020/11/13/D9JbcQ.png)

> Stuck in the local minima.

- Saddle points much more common in high dimension.

**Add an Momentum term may solve these problems**

![](https://s3.ax1x.com/2020/11/13/D9JqXj.png)

![](https://s3.ax1x.com/2020/11/13/D9JXBn.png)

> Owing to the exsistance of momentum we can training more faster and overcome the problems mentioned before.


![](https://s3.ax1x.com/2020/11/13/D9JH1g.png)

- **Nesterov Momentum**

$$v_{t+1} = \rho{v_t}-\alpha{\nabla{f(x_t+\rho{v_t})}}
\\
x_{t+1} = x_t + v_{t+1}
$$

![](https://s3.ax1x.com/2020/11/13/D9Jj7q.png)

> some kind error correcting term of present v and the previous v

![](https://s3.ax1x.com/2020/11/13/D9JzNV.png)

---

### AdaGrad

```python
grad_squared = 0
while True:
    dx = compute_gradient(x)
    grad_squared += dx * dx
    x -= learning_rate * dx / (np.sqrt(grad_squared) + 1e-7)
```

> The basic idea about AdaGrad algorithm is that the step of dimention with smaller gradients will be divided by small vals and make it move faster, while greater one slower to avoid zig-zag behavior.

> while the step will become smaller and smaller while you get closer to the minima, but in turn with higher risks to stuck in the local minima.

---

### RMSProp

![](https://s3.ax1x.com/2020/11/13/D9JxA0.png)

> With a decay rate to make a smooth stop the reducing of steps.

![](https://s3.ax1x.com/2020/11/13/D9Y99U.png)

---

### Adam (almost)

```python
first_moment = 0
second_moment = 0
while True:
    dx = compute_gradient(x)
    first_moment = beta1 + first_moment + (1 - beta1) * dx
    # Momentum
    second_moment = beta2 * second_moment + (1 - beta2) * dx * dx
    # AdaGrad / RMSProp
    x -= learning_rate * first_moment / (np.sqrt(second_moment) + 1e-7)
```

> It combine the two methods, but with a little bug of the first step, which gonna be super large.

### Adam (full form)

```python
first_moment = 0
second_moment = 0
for t in range(num_iterations):
    dx = compute_gradient(x)
    first_moment = beta1 + first_moment + (1 - beta1) * dx
    # Momentum
    second_moment = beta2 * second_moment + (1 - beta2) * dx * dx
    # AdaGrad / RMSProp
    first_unbias = first_moment / (1 - beta1 ** t)
    second_unbias = second_moment / (1 - beta2 ** t)
    x -= learning_rate * first_unbias / (np.sqrt(second_unbias) + 1e-7)
```

> Bias correction for the fact that first and second moment estimates start at zero

- Great starting point
1. beta1 = 0.9
2. beta2 = 0.999
3. learning_rate = 1e-3 or 5e-4

---

### Decay the learning rate to make it finer

![](https://s3.ax1x.com/2020/11/13/D9YShT.png)

![](https://s3.ax1x.com/2020/11/13/D9YPc4.png)

---

### little bit Fancier Optimization 

![](https://s3.ax1x.com/2020/11/13/D9YC3F.png)

> First derivative optimization

![](https://s3.ax1x.com/2020/11/13/D9YijJ.png)

> Second derivative optimization, direct to the mini

![](https://s3.ax1x.com/2020/11/13/D9Yku9.png)

> Don't need learning rate, but impractical for Hessian has O(N^2) elements and Inverting takes O(N^3)

#### Quasi - Newton methods (BGFS)

![](https://s3.ax1x.com/2020/11/13/D9YABR.png)

![](https://s3.ax1x.com/2020/11/13/D9YEH1.png)

---

### In Practice:

- Using Adam
- If full batch updates can be afforded, try out **L-BFGS**

---

### Reduce the gap between train and unseen data

#### Model Ensembles

1. Train multiple independent models
2. At test time average their results

2% improvement maybe

![](https://s3.ax1x.com/2020/11/13/D9YeN6.png)

> Instead of using actual parameter vector, keep a moving average of the para vector and use that at test time

```python
while True:
    data_batch = dataset.sample_data_batch()
    loss = network.forward(data_batch)
    dx = network.backward()
    x += - learning_rate * dx
    x_test = 0.995*x_test + 0.005*x
```

#### Regularization to make single model performs better

- Dropout

![](https://s3.ax1x.com/2020/11/13/D9Ym4K.png)

![](https://s3.ax1x.com/2020/11/13/D9YK3D.png)

> Another interpretation is that you can percive each binary mask as a single model, so it just like dropout is training a large ensemble of models with shared paras.

![](https://s3.ax1x.com/2020/11/13/D9Yu9O.png)

![](https://s3.ax1x.com/2020/11/13/D9YMge.png)

#### Batch Normalization

> Which can achieve the same effect as the Dropout, for it includes some noises.
#### Data Augmentation

> To introduce noise to make it performs better on unseen data.

![](https://s3.ax1x.com/2020/11/13/D9YQjH.png)

#### Stochastic Depth

> Randomly drop layers during training.
> Use the full networks during testing.

---

### Transfer Learning

> There is no need for huge amount of data.

![](https://s3.ax1x.com/2020/11/13/D9Y1ud.png)
