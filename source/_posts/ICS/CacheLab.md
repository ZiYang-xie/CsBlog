---
title: Cache Lab 理解高速缓存
date: 2021-04-13 16:13:55
index_img: /img/ICS_Lab1/top.jpg
category: [ICS]
tags: [Cache]
---

*注：本实验报告为 CMU CSAPP CacheLab 实验报告*

## 一、实验简介

​	Cache Lab实验主要在于帮助学生理解高速缓存的工作方式，以及如何针对Cache编写程序。实验主体总共分为两个部分 

### **PartA** 

编写一个Cache的模拟程序用以统计在L\S\M过程中Cache Hit\Miss\Eviction 的总数。

官方给出了 csim-ref 程序，我们需要编写代码实现其功能，输出 Hit/Miss/Eviction 的总数

```shell
Usage: ./csim-ref [-hv] -s <num> -E <num> -b <num> -t <file>
Options:
  -h         Print this help message.
  -v         Optional verbose flag.
  -s <num>   Number of set index bits.
  -E <num>   Number of lines per set.
  -b <num>   Number of block offset bits.
  -t <file>  Trace file.

Examples:
  linux>  ./csim-ref -s 4 -E 1 -b 4 -t traces/yi.trace
  linux>  ./csim-ref -v -s 8 -E 2 -b 4 -t traces/yi.trace
```

**测试方法：**

​	使用 test-csim.c 进行测试，测试器会输出你的编译器和标准结果，并进行比较计算分数。

```shell
xzy@ubuntu:~/Desktop/ICSLAB/CacheLab$ ./test-csim 
                        Your simulator     Reference simulator
Points (s,E,b)    Hits  Misses  Evicts    Hits  Misses  Evicts
     3 (1,1,1)       9       8       6       9       8       6  traces/yi2.trace
     3 (4,2,4)       4       5       2       4       5       2  traces/yi.trace
     3 (2,1,4)       2       3       1       2       3       1  traces/dave.trace
     3 (2,1,3)     167      71      67     167      71      67  traces/trans.trace
     3 (2,2,3)     201      37      29     201      37      29  traces/trans.trace
     3 (2,4,3)     212      26      10     212      26      10  traces/trans.trace
     3 (5,1,5)     231       7       0     231       7       0  traces/trans.trace
     6 (5,1,5)  265189   21775   21743  265189   21775   21743  traces/long.trace
    27

TEST_CSIM_RESULTS=27
```



### **PartB**

编写适应Cache的矩阵转置程序，使Cache的miss数尽量的小。

​	给出总大小1kb，每个 block 为 32byte 的直接映射Cache

​	共包含3个测试、分别为 32x32 \ 64x64 \ 61x67 大小的矩阵

**测试方法：**

​	使用 test-trans.c 进行测试

```shell
Usage: ./test-trans [-h] -M <rows> -N <cols>
Options:
  -h          Print this help message.
  -M <rows>   Number of matrix rows (max 256)
  -N <cols>   Number of  matrix columns (max 256)
Example: ./test-trans -M 8 -N 8
```



----



## 二、实验内容

### PartA

- **关于 Cache 的结构组成**

​	在 PartA 中我们需要在 csim.c 中编写模拟 Cache 运行的程序。学习过CSAPP第六章我们知道，Cache 的结构是由一行一个valid位，t个tag和一个长度为B的block组成的，总共分为多个Set，每个Set有E行。总共组成 Cache 大小 $C = B \times E \times S$ ( valid 和 tag 不计入 )

![](https://tva1.sinaimg.cn/large/008eGmZEgy1gphwionzxcj30e30930t7.jpg)

- **关于 Evict 机制**

  同时由于采取 LRU 策略 （最近最少使用策略），我们需要维护一个 timestamp 来标记其访问时间的远近。可以一开始赋值 INF，然后逐渐减小，LRUstamp 最小的，就会被 Evict。

由此我们确定了 Cache 的组成形式，我们一个 Line 由 valid 位，tag位，LRUstamp，和一个B字节的 Block 组成。由于是模拟Cache，所以其实DataBlock可以不编写，但是为了尽量模拟真实Cache，还是分配了Block的内存

```c
typedef struct 
{
    int valid; // 0/1 : invalid/valid
    int tag;
    int LRUstamp; // 越小代表越早使用，可以evict
    int* Block; // B（2^b） 字节
} Line_t;
```

```c
// s个Set组成Cache
typedef struct {
    int S; // Set 个数 (2^s)
    int E; // 每个 Set 中 Line 的数量
    int B; // Block 大小 (2^b)
    Line_t*** cache_;
} Cache_t; 
```

我在Cache中设置了S, E, B以方便管理Cache大小，不用每次传参，并编写对应设置函数。

```c
void initCache(const int s, const int E, const int b) 
{
    Cache.S = (1 << s), Cache.E = E, Cache.B = (1 << b);
}
```



- **关于内存管理**

  之后便是编写函数来进行 Cache的内存分配和释放。这个没什么好说的，就是要养成分配完释放的好习惯，避免内存泄漏。

  ```c
  /* 分配 Cache */
  void mallocCache()
  {
      Cache.cache_ = (Line_t***)malloc(Cache.S* sizeof(Line_t **));
      for(int i = 0; i < Cache.S; ++i)
          Cache.cache_[i] = (Line_t **)malloc(Cache.E * sizeof(Line_t *));
      for(int i = 0; i < Cache.S; ++i)
          for(int j = 0; j < Cache.E; ++j)
              Cache.cache_[i][j] = (Line_t *)malloc(sizeof(Line_t));
      for(int i = 0; i < Cache.S; ++i)
          for(int j = 0; j < Cache.E; ++j) {
              Cache.cache_[i][j]->valid = 0;
              Cache.cache_[i][j]->tag = 0;
              Cache.cache_[i][j]->LRUstamp = 1e9;
              Cache.cache_[i][j]->Block = (int *)malloc(Cache.B * sizeof(int));
          }
  }
  ```

  ```c
  /* 释放 Cache */
  void freeCache()
  {
      for(int i = 0; i < Cache.S; ++i)
          for(int j = 0; j < Cache.E; ++j)
              free(Cache.cache_[i][j]->Block);
      for(int i = 0; i < Cache.S; ++i)
          for(int j = 0; j < Cache.E; ++j)
              free(Cache.cache_[i][j]);
      for(int i = 0; i < Cache.S; ++i)
          free(Cache.cache_[i]);
      free(Cache.cache_);
  }
  ```



- **关于 getter 函数**

  为了方便valid、tag等信息的查找，我编写了一系列getter函数。

  ```c
  /* 获得 Valid bit */
  int getValid(const int s, const int E) {return Cache.cache_[s][E]->valid;}
  
  /* 获得 Tag */
  int getTag(const int s, const int E) {return Cache.cache_[s][E]->tag;}
  
  /* 获得LRUstamp */
  int getLRU(const int s, const int E) {return Cache.cache_[s][E]->LRUstamp;}
  ```



- **对于Hit和Miss的判定**

  判定hit和miss，就是在Cache对应Set中对比valid和tag，如果valid为1标明有效，并且tag相同，则说明Hit，不然则Miss。

  ```c
  /* 查看是否命中 */
  int HitOrMiss(const int s, const int tag)
  {
      for(int i = 0; i < Cache.E; ++i)
          if(getValid(s, i) && getTag(s, i) == tag)
              return i; // 命中
      return -1; // Miss 
  }
  ```

  

- **关于是否Evict**

  根据上文，我们知道我们采取LRU的evict策略，那么在写的时候，如果valid位为1，而tag不一致发生写miss的情况，就要进行evict。evict的位置是LRUstamp最小的那一个。我们维护LRUstamp，在每次操作时，将其赋值为INF (1e9) 并将其余的LRUstamp减1，取最小的 LRUstamp 进行evict，用最新的 tag 和 数据 进行替换

```c
/* 更新LRU，越小越早使用 */
void lruUpdate(const int s, const int E)
{
    Cache.cache_[s][E]->LRUstamp = 1e9;
    for(int i = 0; i < Cache.E; ++i)
    {
        if(i != E)
            Cache.cache_[s][i]->LRUstamp--;
    }
}

/* 更新Cache */
void WriteCache(const int s, const int E, const int tag)
{
    Eviction(s, E, tag);
    Cache.cache_[s][E]->valid = 1;
    Cache.cache_[s][E]->tag = tag;
    lruUpdate(s, E);
}
```



- 关于模拟Cache

最后进行Cache的模拟，如果Hit了就值更新LRUstamp，Miss了就获取EvictionPose，并将其替换。

```c
/* 模拟Cache，获得 Hit 和 Miss 的次数 */
void getAns(const int s, const int tag)
{
    int hit_flag = HitOrMiss(s, tag);
    if(hit_flag != -1) { // Hit
        Hit++;
        if(verbose)
            printf("hit ");
        lruUpdate(s, hit_flag);
    }
    else  {
        Miss++;
        if(verbose)
            printf("miss ");
        WriteCache(s, getEvictPos(s), tag);
    }
}
```



最后完整代码，附于提交 ```csim.c``` 中



- 完成通过

```c
Your simulator     Reference simulator
Points (s,E,b)    Hits  Misses  Evicts    Hits  Misses  Evicts
     3 (1,1,1)       9       8       6       9       8       6  traces/yi2.trace
     3 (4,2,4)       4       5       2       4       5       2  traces/yi.trace
     3 (2,1,4)       2       3       1       2       3       1  traces/dave.trace
     3 (2,1,3)     167      71      67     167      71      67  traces/trans.trace
     3 (2,2,3)     201      37      29     201      37      29  traces/trans.trace
     3 (2,4,3)     212      26      10     212      26      10  traces/trans.trace
     3 (5,1,5)     231       7       0     231       7       0  traces/trans.trace
     6 (5,1,5)  265189   21775   21743  265189   21775   21743  traces/long.trace
    27
```



----



### PartB

​	在完成了Cache基本的组织架构之后，我们现在要开始编写Cache友好的程序，以矩阵转置为例。源代码中包含一个最naive的矩阵转置，也是我们平常经常写的。

```c
/* 
 * trans - A simple baseline transpose function, not optimized for the cache.
 */
char trans_desc[] = "Simple row-wise scan transpose";
void trans(int M, int N, int A[N][M], int B[M][N])
{
    int i, j, tmp;

    for (i = 0; i < N; i++) {
        for (j = 0; j < M; j++) {
            tmp = A[i][j];
            B[j][i] = tmp;
        }
    }    
}
```

测试之后会发现，对于4x4的矩阵，其miss次数有22次之多

```shell
func 1 (Simple row-wise scan transpose): hits:15, misses:22, evictions:19
```

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gphxvxlx02j312d0iw40v.jpg" style="zoom:50%;" />

我们可以发现，对于4x4的矩阵，由于Block大小是32bytes且为直接映射Cache，所以每8个int会在同一个Block/Set当中，4x4的矩阵被分在两个不同的 Block 中，对于A矩阵的访问是行优先的，而B矩阵的访问是列优先的。我们可以模拟这个过程

> 访问到 $A_{00}$  Cache cold miss，将 $B_{A_{1}}$ 加入 Cache 中
>
> 访问到 $B_{00}$  由于B和A有相同的组索引，所以导致相对位置相同的地方被映射到Cache的同一组中，造成tag不一致，产生miss将会把之前Load到Cache里的A进行驱逐，将 $B_{A_{1}}$ 驱逐 将 $B_{B_{1}}$ 加入 Cache 中
>
> 访问到 $A_{01}$  同理，将 $B_{B_{1}}$ 驱逐 将 $B_{A_{1}}$ 加入 Cache 中
>
> 之后 $B_{B_{2}}$ 和  $B_{A_{1}}$ 很幸运能够不产生thrash，有几次Cache hit，但是大部分情况下，还是会发生如同上面所描述的抖动。

因此，对于这种情况，我们应该如何修改代码，使其适应Cache，能够尽可能少的Miss呢？

- **暴力方法**

  有一种最暴力的方式，就是考虑到目前矩阵的大小比较小，所以我们可以采取定义8个局部变量的方法，一次性将一个Block全部Load到寄存器中，然后再进行转置，这样就不会发生如上面所描述的 Cache thrash。

  ```c
  void transpose_4x4(int M, int N, int A[N][M], int B[M][N])
  {
      int tmp0, tmp1, tmp2, tmp3, tmp4, tmp5, tmp6, tmp7;
  
  	for(int x = 0; x < 3; x += 2)
  	{
  		tmp0 = A[x][0], tmp1 = A[x][1], tmp2 = A[x][2], tmp3 = A[x][3];
  		tmp4 = A[x + 1][0], tmp5 = A[x + 1][1], tmp6 = A[x + 1][2], tmp7 = A[x + 1][3];
  
  		B[0][x] = tmp0, B[1][x] = tmp1, B[2][x] = tmp2, B[3][x] = tmp3; 
  		B[0][x + 1] = tmp4, B[1][x + 1] = tmp5, B[2][x + 1] = tmp6, B[3][x + 1] = tmp7; 
  	}
  }
  ```

  ```
  func 0 (Transpose submission): hits:29, misses:8, evictions:6
  ```

  可以看到miss降到了8，有极大的改进，我们用partA中写的csim进行分析。

  ```shell
  xzy@ubuntu:~/Desktop/ICSLAB/CacheLab$ ./csim -v -s 5 -E 1 -b 5 -t trace.f0
  S 18e08c,1 miss #系统miss
  L 18e0a0,8 miss #系统miss
  L 18e084,4 hit 
  L 18e080,4 hit 
  L 10e080,4 miss eviction 
  L 10e084,4 hit 
  L 10e088,4 hit 
  L 10e08c,4 hit 
  L 10e090,4 hit 
  L 10e094,4 hit 
  L 10e098,4 hit 
  L 10e09c,4 hit 
  S 14e080,4 miss eviction 
  S 14e090,4 hit 
  S 14e0a0,4 miss eviction # 多一次miss
  S 14e0b0,4 hit 
  S 14e084,4 hit 
  S 14e094,4 hit 
  S 14e0a4,4 hit 
  S 14e0b4,4 hit 
  L 10e0a0,4 miss eviction 
  L 10e0a4,4 hit 
  L 10e0a8,4 hit 
  L 10e0ac,4 hit 
  L 10e0b0,4 hit 
  L 10e0b4,4 hit 
  L 10e0b8,4 hit 
  L 10e0bc,4 hit 
  S 14e088,4 hit 
  S 14e098,4 hit 
  S 14e0a8,4 miss eviction 
  S 14e0b8,4 hit 
  S 14e08c,4 hit 
  S 14e09c,4 hit 
  S 14e0ac,4 hit 
  S 14e0bc,4 hit 
  S 18e08d,1 miss eviction #系统miss
  hits:29 misses:8 evictions:6
  ```

  可以看到除了前后3次系统固定的miss之外，距离理论下限4次还有1次，这一次发生在Store矩阵B的时候，由于B是列优先，所以在访问到$B_{B_{2}}$的时候会多发生一次miss。

	#### 32x32 矩阵转置

​		了解了之后我们来看32x32的矩阵转置，其正好是Cache 1kb的总大小，能够全部装进Cache。所以我们需要避免的就是连续在A,B

之间切换访问相对位置相同的区域（映射到同一个Block）从而导致的Cache thrash。

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gpi2azu5g2j31dl0qz42z.jpg" style="zoom:50%;" />

我们可以采取 8*8分块的方式，对于每一块中每一行属于同一个Block，经过8x8分块后我们发现，除了对角线的分块之外，其他分块的A、B互不影响，不会产生thrash。

```c
void transpose_32x32(int M, int N, int A[N][M], int B[M][N])
{
    int i, j, x, y;
		int tmp;
  
    for(i = 0; i < N; i += 8)
        for(j = 0; j < M; j += 8)
          for(x = i; x < i + 8; ++x)
              for(y = j; y < j + 8; ++y)
              {
                tmp = A[x][y], B[y][x] = tmp;
              }
  /*
    注意，这里应该学习示例程序中的 tmp = A[x][y], B[y][x] = tmp; 
    因为我们无法确定调用者是否会使A，B指向内存中的同一块区域从而造成意想不到的后果。
  */
}
```

```shell
func 0 (Transpose submission): hits:1735, misses:350, evictions:318 	# 8x8 分块
func 1 (Simple row-wise scan transpose): hits:870, misses:1183, evictions:1151	# Naive
```

可以看到相比于最初naive的版本1183miss，已经有了极大的改善，但是还是没有达到要求的小于300miss。



我们可以发现，除对角线之外的分块已经达到miss的下限，所以对角线是需要解决的地方。对于对角线上的分块，我们的处理和Naive方式没有什么区别，都会造成thrash。

有了前面4x4矩阵的铺垫，我们已经知道了应该如何处理这类情况，我们按照之前的解决方法类似解决。

```c
void transpose_32x32(int M, int N, int A[N][M], int B[M][N])
{
    int i, j, x, y;
    int tmp, tmp0, tmp1, tmp2, tmp3, tmp4, tmp5, tmp6, tmp7;

    for(i = 0; i < N; i += 8)
        for(j = 0; j < M; j += 8)
			for(x = i; x < i + 8; ++x)
			{
				if(i == j)
				{
					tmp0 = A[x][j], tmp1 = A[x][j + 1], tmp2 = A[x][j + 2], tmp3 = A[x][j + 3];
					tmp4 = A[x][j + 4], tmp5 = A[x][j + 5], tmp6 = A[x][j + 6], tmp7 = A[x][j + 7];

					B[j][x] = tmp0, B[j + 1][x] = tmp1, B[j + 2][x] = tmp2, B[j + 3][x] = tmp3; 
					B[j + 4][x] = tmp4, B[j + 5][x] = tmp5, B[j + 6][x] = tmp6, B[j + 7][x] = tmp7; 
				}
				else 
				{
					for(y = j; y < j + 8; ++y)
					{
						tmp = A[x][y], B[y][x] = tmp;
					}
				}
			}
}
```

```
func 0 (Transpose submission): hits:1766, misses:287, evictions:255
```

​	成功将miss降到了287。实际的理论下限是256，我们注意到这种操作方式A矩阵的miss已经达到了下限，而B矩阵的对角线还是会固定miss。这是因为在Load A之前，B矩阵的对角线行已经被前一个转置Load进去，所以会被evict掉，再次调用时就会产生一次Miss。所以我们可以对其进行展开，最后可以做到259次的miss（3次系统miss）由于代码可读性太低，这里不做展开。



---

#### 64x64矩阵转置

​	对于64x64矩阵转置，我们注意到其已经超过了Cache的大小，我们首先用8分块尝试一下。

```shell
func 0 (Transpose submission): hits:3586, misses:4611, evictions:4579 # 8分块
func 1 (Simple row-wise scan transpose): hits:3474, misses:4723, evictions:4691 # naive
```

发现没有什么改进，这是什么原因？我们从Cache大小的角度出发

![](https://tva1.sinaimg.cn/large/008eGmZEgy1gpi5dme0pcj31ok0tnteo.jpg)

我们进行4x4分块试验一下

```c
int i, j, x, y;
    int tmp, tmp0, tmp1, tmp2, tmp3;

    for(i = 0; i < N; i += 4)
        for(j = 0; j < M; j += 4)
			for(x = i; x < i + 4; ++x)
			{
				if(i == j)
				{
					tmp0 = A[x][j], tmp1 = A[x][j + 1], tmp2 = A[x][j + 2], tmp3 = A[x][j + 3];

					B[j][x] = tmp0, B[j + 1][x] = tmp1, B[j + 2][x] = tmp2, B[j + 3][x] = tmp3; 
				}
				else 
				{
					for(y = j; y < j + 4; ++y)
					{
						tmp = A[x][y], B[y][x] = tmp;
					}
				}
			}
```



```shell
func 0 (Transpose submission): hits:6402, misses:1795, evictions:1763
```

​	可以看到结果好了很多，但是还是没能达到1300的要求。其原因其实还是在最开始讲的4x4分块的例子中，由于B矩阵是按列访问，在4x4分块之后，访问对角两块的时候，会每一块都会各产生2次miss，有一个小的thrash。

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gpi5iw1wfdj31or0tj0vq.jpg" style="zoom:33%;" />

​	对自己电脑寄存器数量有自信的选手可能会选择开16个临时变量来一次性完成转置，但那样太过粗暴而不稳定。我们想是否有什么办法能够使4x4区域中的miss次数降低到1次？我们回去考虑8x8的分块情况。

![](https://tva1.sinaimg.cn/large/008eGmZEgy1gpi5pdjqnuj31tk0laq79.jpg)

- **对于A矩阵**

  我们的策略是一次load掉4行的内容，以防多次访问

- **对于B矩阵**

  我们发现，其实对于2、3两块分块，如果在2处出现miss，那么3其实也被加入了Cache不会发生miss，所以就有了如下策略。

  

  **解决方案：**

  

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gpi5wquog6j30vb0t4jtx.jpg" style="zoom:25%;" />

1. 我们可以先一次性Load A1、A2两块到B1、B2两块中，这样只会1次miss。

2. 我们按行将A3转置到B2中， 再将B2中原来放置的A2转置到A3中 (B2 Hit -> B3 miss ->下次B2 miss之后 B2、B3在Cache中便不会miss了) 共2次miss
3. 之后再将A4转置到B4中，这样会miss1次

总共miss4次，这样一来对于8x8 的 B矩阵，相当于每个4x4只miss了一次，完成了要求。

*< 代码太长见附件 >*

测试一下，效果拔群！

```
func 0 (Transpose submission): hits:9082, misses:1163, evictions:1131
```





---

#### 61x67矩阵

对于这类非方阵，我们采用最简单的分块策略进行测试。

|        | 4x4  | 8x8  | 16x16 | 17x17 |
| :----: | :--: | :--: | :---: | :---: |
| miss数 | 2425 | 2118 | 1992  | 1950  |

个人测试了 4x4 \ 8x8 \ 16x16 发现16x16可以满足miss数小于 2k 的要求。参考网上资料[1]后，发现 17x17是最优选择，miss数为1950

```c
void transpose_61x67(int M, int N, int A[N][M], int B[M][N])
{
    int i, j, x, y, tmp;

    for(i = 0; i < N; i += 17)
        for(j = 0; j < M; j += 17)
			for(x = i; x < N && x < i + 17; ++x)
				for(y = j; y < M && y < j + 17; ++y)
				{
					tmp = A[x][y];
					B[y][x] = tmp; 
				}		
}
```



----



## 三、实验结论

最后我们运行 ```drive.py``` 来测试我们的总分

```shell
Part A: Testing cache simulator
Running ./test-csim
                        Your simulator     Reference simulator
Points (s,E,b)    Hits  Misses  Evicts    Hits  Misses  Evicts
     3 (1,1,1)       9       8       6       9       8       6  traces/yi2.trace
     3 (4,2,4)       4       5       2       4       5       2  traces/yi.trace
     3 (2,1,4)       2       3       1       2       3       1  traces/dave.trace
     3 (2,1,3)     167      71      67     167      71      67  traces/trans.trace
     3 (2,2,3)     201      37      29     201      37      29  traces/trans.trace
     3 (2,4,3)     212      26      10     212      26      10  traces/trans.trace
     3 (5,1,5)     231       7       0     231       7       0  traces/trans.trace
     6 (5,1,5)  265189   21775   21743  265189   21775   21743  traces/long.trace
    27


Part B: Testing transpose function
Running ./test-trans -M 32 -N 32
Running ./test-trans -M 64 -N 64
Running ./test-trans -M 61 -N 67

Cache Lab summary:
                        Points   Max pts      Misses
Csim correctness          27.0        27
Trans perf 32x32           8.0         8         287
Trans perf 64x64           8.0         8        1163
Trans perf 61x67          10.0        10        1950
          Total points    53.0        53
```

满分 53.0/53



## 四、参考资料

[1] *Computer Systems: A Programmer's Perspective*, 3/E (CS:APP3e). Randal E. Bryant and David R. O'Hallaron, Carnegie Mellon University

[2] https://blog.csdn.net/xbb224007/article/details/81103995



