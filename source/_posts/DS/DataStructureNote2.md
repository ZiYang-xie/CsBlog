---
title: 素数筛法 (2020-12-7)
index_img: /img/Pic/DS.png
date: 2020-12-07 23:12:25
category: [DataStructure]
tags: [Number theory, Multiplication algorithm]
math: true
---

# 素数筛法

我们今天来讲讲筛法，今天上机出其不意的考了一手筛法，我居然天真的在写暴力素数判断。所以今天就来复习(重学)一下筛法。

如果我们想要判断一个数是否是素数有什么方法呢？ 暴力判断肯定是最容易想到的方法，按定义素数不能被1和其自身以外的数整除，我们自然会想到从 x - 1到2全判一遍的算法。但显然，我们做了很多重复的工作。

实际上我们可以发现，如果我们判断出了一个素数，那么它的倍数显然就不可能是素数，这样就可以直接将其开除“素籍“，踢出待选区域。这种筛选素数的方法就像一个筛子，每碰到一个素数就筛掉一批非素数，大大降低了复杂度。

![](https://images2015.cnblogs.com/blog/927750/201612/927750-20161229220529101-1487746442.png)

## 埃拉托斯特尼筛法

按上述思想写出代码，就是埃氏筛法。

- 代码实现

```cpp
int Eratosthenes(int n) {
    int p = 0;
    for (int i = 0; i <= n; ++i) 
        is_prime[i] = 1;
    is_prime[0] = is_prime[1] = 0;
    for (int i = 2; i <= n; ++i) 
    {
        if (is_prime[i]) // 直接从2开筛不会放进来一个非素数
        {
            prime[p++] = i; 
            if ((long long)i * i <= n)
                for (int j = i * i; j <= n; j += i)
                    // 因为从 2 到 i - 1 的倍数我们之前筛过了，这里直接从 i 的 i倍开始，提高了运行速度
                    is_prime[j] = 0;  // 是i的倍数的均不是素数
        }
    }
    return p;
}
```

### 复杂度计算

关于埃氏筛法的复杂度计算，我们来做一个推导，首先我们可以看出每次循环中，若当前素数是p，那么单次循环执行 n/p 次， 所以总的表达式就是 n 乘上所有素数的倒数和 $n\sum_{p} \frac{1}{p}$

关于所有素数的倒数和，在欧拉相关的论文中有明确的估计。

$$\sum_{p\le x} \frac{1}{p} = \ln\ln(x) + \gamma + \sum_{m = 2}^{\infty}{\mu (m){\frac{\zeta(m)}{m} + \delta}}$$
论文链接 [https://arxiv.org/pdf/math/0504289.pdf]

可知埃氏筛法的复杂度为 $O(nloglogn)$

---

## 欧拉筛法 （线性筛）

TODO