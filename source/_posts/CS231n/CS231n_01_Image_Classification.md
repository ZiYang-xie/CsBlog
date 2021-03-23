---
title: CS231n Image Classification 01
date: 2020-11-07 22:11:53
tags: [CV,Neural Network]
category: [CS231n]
index_img: /img/Cs231n/top.jpg
math: true
---

**Preface:** This is the note of Stanford course CS231n, paving the way for my lab research.

# Image Classification
*A core task in Computer Vision*

---

### Computer' Work 
Input an image, and assign one of the label amoung the given labels.

- **The Problem:** 
1. Semantic Gap
2. Viewpoint variation
3. illumination 
4. Deformation
5. Occlusion
6. Intraclass variation

---

### An image classifier
> Coding might be difficult 

```python
def classify_image(image):
    # Do Some Magic
    return class_label
```

- Attmpts
  
![](https://s1.ax1x.com/2020/11/07/BIMSmD.png)

---

### Data-Driven Approach
1. Collect a dataset of images and labels
2. Use Machine Learning to train a classifier
3. Evaluate the classifier on new images

- First classifier: Nearest Neighbor

*Just Memorize all data and labels*
```python
def train(images, labels):
    # Machine Learning!
    return model
```

*Predict the label of the most similar training image*
```python
def predict(model, test_images):
    # Use model to predict labels
    return test_labels
```

**Example Dataset:** CIFAR10

![](https://s1.ax1x.com/2020/11/07/BIM9TH.png)

> **Issues:** Although pics may seem visually similar, but still give lots of errors.
> 
---

- Compare func used in it
### **K nearest Neighbors Method**

**L1 distance:** $d_1(I_1,I_2) = \sum\limits_{p} \mid I_1^p - I_2^p \mid$

![](https://s1.ax1x.com/2020/11/07/BIKXSx.png)

*Minimize the sum given the most similar pics*

#### **BackWards**

![](https://s1.ax1x.com/2020/11/07/BIKjl6.png)

#### **What it looks like**

![](https://s1.ax1x.com/2020/11/07/BIMp0e.png)

**Issues**

1. Isolated Yellow Point
2. Noisy of one single point (green into blue)

**Use K Nearest Neighbors to Optimize it**
![](https://s1.ax1x.com/2020/11/07/BIMitA.png)

---

*A Better Cmp Func*
**L2(Euclidean) distance:** $d_1(I_1,I_2) = \sqrt{\sum\limits_{p}{(I_1^p - I_2^p)}^2}$

![](https://s1.ax1x.com/2020/11/07/BIMFfI.png)

> The L1 Distance depends on the coordinate system, whenever there is a rotate, it would change the L1 Distance, while that won't happen in the L2 Distance case (simply because it's a circle)

---

#### **Hyperparameters**
- What's the best value of **k**
- What's the best **distance** to use? (L1,L2 or anything else)

*These things are preset rather than learn automatically from learning process*

This is **Very problem-dependent**, just try!, but How?

![](https://s1.ax1x.com/2020/11/07/BIME1P.png)

**Training & Validation process should not mixed with the test data**

- Cross Validation

![](https://s1.ax1x.com/2020/11/07/BIMApt.png)

- Validation process

![](https://s1.ax1x.com/2020/11/07/BIMV6f.png)

> using the validation data to choose the best hyperparameters.

![](https://s1.ax1x.com/2020/11/07/BIMu7Q.png)

> Cause we sum the offset, though the differences bettween pics and pics are various, they still got the same L2 distance, which is not so good.


---

### **Linear Classification**

- **Parametric Model**
![](https://s1.ax1x.com/2020/11/07/BIMZX8.png)

$$f(x,W) = Wx + b$$
> We need f(x,W) to be 10x1 and the x is actually 3072x1, so the W we input may be 10x3072, sometimes we add a bias to balance.

![](https://s1.ax1x.com/2020/11/07/BIMn0g.png)

![](https://s1.ax1x.com/2020/11/07/BIMMkj.png)

> It use a single line to separate the object based on its RGB info

But how can we tell the quality of W ?
(View the next lecture)

- **Problems**
![](https://s1.ax1x.com/2020/11/07/BIMQts.png)

> Since it's linear the Problems is obivious.
