---
title: Locality Principle
date: 2021-10-15 10:29:48
index_img: /img/OS/banner.png
category: [OS]
tags: [OS]
math: true
---

### 1.1 Intro

​	Locality Principle is a profound law of computing and has a wide application in many modern areas in computer science — virtual memory, cache, database, etc.  It is the combination of two locality aspects: 

- *(1) temporal locality*
- *(2) spatial locality*

​	The success of the locality principle attributes to the fact that locality works aline with human cognitive and coordinative behavior. *Divide and conquer* is the most common practice for humans when dealing with complex problems. We tend to break it into some small parts and work on them separately. Here is where the locality lies in. 

​	Understanding the locality principle can benefit a lot, including making better use of cache, predicting the phase boundaries, and extending to many different aspects in computer science, even our daily lives.

### 1.2 History

​	Looking backward, locality has emerged from the design of virtual memory systems back to 1959, the Atlas system. There were two main performance problems at that time. One is the address location problem, which is quickly solved by page table and TLB, while the another hard nut is the replacement problem of  the demand-paged virtual memory. Replacing the page that will not be used again for the longest time is a common belief, but obviously people can't estimate the next-use time for sure. 

​	1966 Belady's study pointed out that the performance under certain replacement strategy primarily depended on the way how compiler grouped the code blocks onto pages. Shortly after that, thrashing problem was discovered and swiftly became the nightmare for companies using virtual memory. IBM OS360 even excluded the virtual memory, for example.

​	By 1966,in Watson Lab, Belady and his partners had tested every replacement policy, and found the LRU (least recently used) replacement got the best performance, because of the reference clustering locality behavior. By using locality notion, MIT Project MAC defined the Intrinsic demand, the first kind of "*Working Set*" at the same year.

​	As it is explained in Denning's paper[^1]

> The working-set idea worked because the pages observed in the backward window were highly likely to be used again in the immediate future.

​	 The underlying mechanism of it is temporal and spatial locality.

### 1.3 Core 

​	Locality principle hit the key of general program behavior, which is strongly connected with people's coding style when solving a problem — *Divide and Conquer*. This coding style is shown as temporal and spatial clustering on data structures. 

​	For instance, when people doing a loop, the variables inside the  loop are often changed constantly, which is a kind of temporal clustering. In terms of spatial clustering, Array is an example when we put data into it and access it sequentially. 

​	The Locality principle can be applied through a distance function $D(x,t)$ to measure temporal of spatial locality, and we use $D(x, t) < T$ to describe the locality set of $x$ at time $t$, and the working set model can be expressed by $W(t, T)$ , $T$ is the time window from the current time $t$. It is a efficient way to track the phases of a program in a dynamic manner, instead of the traditional *static representation* following *Zipf Law*

​	The modern model of locality are more about context-awareness with four key ideas

- **An Observer:** Agent doing tasks with the help of software
- **Neighborhoods:** Group of obejcts 
- **Inference:** Method measures the content of neighborhoods
- **Optimal Actions:** actions performed by software 

### 1.4 Application

​	Locality principle has a wide-spread application in the computer field, for example, in the Search Engine area. Many search engine companies such as Google, using locality to arrange the outside data. Utilizing cache to speed up the searching process, sort the relative result and recent search log together, a typical example of the spatial and temporal locality.



---



## 2. Page Coloring

​	After wide application of virtual memory, people found that the Arbitrary Mapping between virtual page and physical page will cause the nonuniform use of Cache. Because of the locality principle, the data are mapping close to each other, resulting in the cache swap in and out constantly, causing thrashing.

​	By 1985, MIPS raised the page coloring technique [^2]   to solve that problem. Simply put, Page coloring is a technique that splitting the pages into many groups with different color and using the middle bit of virtual mem block to represent the group and map it to the corresponding physical page. This method separates the interrelated data into different parts of memory and cache block, solving the thrashing problems.


<img src="https://tva1.sinaimg.cn/large/008i3skNly1gvfslxmzuxj618v0u00xi02.jpg" style="zoom:40%;" />

​	Page coloring is easy to code, deploy and has a great improvement on system performance. After using page coloring the page non-assess correlation dropped significantly.[^3] today page coloring are being widely used in cache designing with different sample method.

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gvfsmp3635j60g50cw75c02.jpg" style="zoom:60%;" />



## Reference

[^1]: Denning, P. J. . Peter J. Denning. The Locality Principle. In Communication Networks and Computer Systems (J. Barria, Ed.). Imperial College Press (2006), 43-67. CHAPTER 4 The Locality Principle.
[^2]: TAYLOR, G., DAVIES, P., ANDFARAWAY,D, M. The TLB slice—A low-cost high-speed address translation mechanism. In ISCA-1990.
[^3]: Zhang, X., Dwarkadas, S., & Shen, K. (2009). Towards Practical Page Coloring-Based Multicore Cache Management. In *Proceedings of the 4th ACM European Conference on Computer Systems* (pp. 89–102). Association for Computing Machinery.

