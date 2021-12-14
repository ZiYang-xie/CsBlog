---
title: 「论文阅读」The Unix Time-Sharing System
date: 2021-11-15 15:29:48
index_img: /img/OS/banner.png
category: [OS]
tags: [OS]
math: true
---

# The Unix Time-Sharing System
*DENNIS M. RITCHIE AND KEN THOMPSON*

## 1. 基本介绍

### 1.1 Unix 的历史进程

​	Multics，名称来自于多任务信息与计算系统（MULTiplexed Information and Computing System）的缩写，它是一套分时多任务，是1964年由贝尔实验室、MIT及美国通用电气公司所共同参与研发，其在GE 645上开发和部署。

​	1969 贝尔实验室退出 Multics 的开发，DENNIS M. RITCHIE & KEN THOMPSON 一同开发了 Unix，

​	一开始Unix是在 *Digital PDP-7*  使用汇编语言进行编写，*1971*年，*Ken Thompson* 申请到了一台 *PDP-11/24*的 机器，于是*Unix*第二版出来了。*1973* 用C重写了Unix，形成了Unix Version 4，第四版运行在更新的PDP-ll/40, /45上

​	直到*1974*年*7*月 DENNIS M. RITCHIE 和 KEN THOMPSON 在  *the Communications of the ACM* 发表了 *The Unix Time-Sharing System* 让Unix系统第一次广泛的为学界所了解。

<img src="https://n3x0.com/wp-content/uploads/2019/10/unix-co-founder-ken-thompsons-bsd-password-has-finally-been-cracked.jpg" style="zoom:50%;" />

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gwen89cbzej30pd0jft9z.jpg" style="zoom:100%;" />



### 1.2 Unix 的特点

​	UNIX 是一个**多用途 (general-purpose)，多用户 (multi-user), 交互式(interactive)** 的操作系统

- **层级**的文件系统
- **兼容**的文件，设备和内部处理I/O
- 初始化异步进程的能力
- 对每个用户都可选的系统命令行语言
- 用多种语言实现的超过100个子系统



**[讨论问题1] 如何理解此处的 “初始化异步进程的能力” （the ability to initiate asynchronous processes）的含义**

<p hidden>
	分时操作系统 (Time Sharing)
  在上世纪50年代末60年代早期，很多操作系统还处于批处理时期（batch processing）
  单一进程长时间独享CPU
</p>

### 1.3 关于Time Sharing 

​	 其实Unix算是比较晚出现的分时操作系统，分时的概念最早在1954年被 [John Backus](https://en.wikipedia.org/wiki/John_Backus) 在 MIT提出

> The computers would handle a number of problems concurrently. Organizations would have input-output equipment installed on their own premises and would **buy time** on the computer much the same way that the average household buys power and water from utility companies.

​	 [John McCarthy](https://en.wikipedia.org/wiki/John_McCarthy_(computer_scientist)) at MIT in 1959 第一个实现了分时操作系统, 并一开始部署在 IBM704上，后面该版本的一个分支被认为是最早的一个分时系统即 CTSS。



## 2. 文件系统

> Everything is a file

​	UNIX中最重要的的工作是提供了一个文件系统。从用户的角度看，一共有三种类型的文件：普通磁盘文件、目录、特殊文件

### 2.1 普通文件

​	一个文件可以包含用户放置的任何类型的信息，例如符号或者二进制(对象)或程序。不需为系统指定特定的数据结构。

**文本文件**：包含简单的、由换行符分隔段落的字符串。

**二进制文件**：是当程序开始执行时，将会在出现在核心内存中的，按顺序存放的字的序列。

少数用户程序以更多样的方式组织文件结构，来实现更多的功能。例如汇编生成器和加载器需要特定格式的对象文件。

总之，普通文件的结构是由用户程序来管理的而非操作系统。



### 2.2 目录

> Directories provide the mapping between the names of files and the files themselves, and thus induce a structure on the file system as a whole.

#### 2.2.1 目录性质

​	每个用户都有一个自己文件的目录；他还可以方便地创建包含文件群组的子目录。目录的行为与普通文件完全相同，不同之处在于目录无法由非特权程序写入，只有系统可以控制目录的内容。但是，具有合适权限的任何人都可以像读取其他任何文件一样读取目录。

- 1. 层级架构

- 2. 用户隔离
- 3. 权限控制

#### 2.2.2 目录内容

​	系统维护几个目录供自己使用。其中之一是根目录 *(root)*。通过跟踪目录链中的路径直到找到所需的文件，可以找到系统中的所有文件。

​	另一个系统目录*（/bin）*包含所有通常需要使用的程序，即所有的**命令**。但是就如将看到的，程序不必驻留在此目录中即可执行。

#### 2.2.3 链接

​	同一个非目录文件可能会以不同的名称出现在多个目录中，就是我们熟知的链接。

​	UNIX与其他允许链接的系统不同，它与文件的所有链接都具有相同的状态。也就是说，文件在特定目录中不存在。文件的目录条目仅由其名称和指向实际描述文件的信息的指针组成，**因此，文件实际上独立于任何目录条目而存在，尽管实际上会使该文件与最后一个链接一起消失。**

**[讨论问题2] 如何理解文件实际上独立于任何目录条目而存在，尽管实际上会使该文件与最后一个链接一起消失**

其实就是文件inode的link count和文件描述符的reference count，当二者都掉为0的时候，文件就会被操作系统回收。

​	目录结构被限定为为有根树的形式。除特殊条目 `.` 和 `..` 外，每个目录必须作为另外一个目录（即其父目录）的条目出现，这是为了简化访问目录结构子树的程序的编写，更重要的是避免层次结构各部分的分离。如果允许链接到任意目录，则很难检测到从根到目录的最后一次连接何时断开。(即出现环)



### 2.3 特殊文件

> Special files constitute the most unusual feature of the UNiX file system.

**将外部设备抽象成一种文件**

​	UNIX支持的每个 I/O 设备都与至少一个这样的文件相关联。特殊文件的读取和写入与普通磁盘文件一样，但是请求读取或写入会导致关联设备的激活。每个特殊文件的条目都位于目录/dev中，尽管可以像普通文件一样链接到其中一个文件。因此，例如要打孔纸带，可以在文件/dev/ppt上写。每个通信线路，每个磁盘，每个磁带驱动器以及物理核心内存都存在特殊文件。当然，活动磁盘和核心专用文件受到保护，不会受到任意访问。

**[讨论问题3] 这样抽象有什么好处？**

- 文件和I/O设备尽可能相似
- 文件名和设备名具有相同的语法和含义，因此可以将以文件名作为参数的程序传递给设备名。
- 权限控制（设备作为特殊文件可以和普通文件一样有权限控制机制）
即程序不再需要关心他在和什么设备进行交互，只需要使用一个统一的文件接口即可。


#### 2.3.1 可移除的文件系统

​	尽管文件系统的根目录始终存储在同一设备上，但是不必将整个文件系统层次结构都驻留在该设备上。

​	mount用一个全新的子树（存储在可移动卷上的层次结构）替换层次结构树（普通文件）的叶子。挂载之后，可移动卷上的文件与永久文件系统上的文件之间几乎没有区别。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gweqa094umj30hd097mxw.jpg)

**[讨论问题5] 我们说当mount之后子树的文件和原来的文件系统上的文件几乎没有区别，那么区别在哪？**

我们不能跨越 mount point 创建 hard link，即不能创建从原来文件系统到挂载的文件树上的hard link


#### 2.3.2 文件保护

​	系统的每个用户都分配有一个唯一的用户标识号（UID）。当创建文件后，将会使用文件所有者的用户ID对其进行标记，还会为新文件提供一个七比特的保护位，其中六个指定文件所有者和所有其他用户的独立读取，写入和执行权限。

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gweq5c1xrhj30go0b4wf3.jpg" style="zoom:67%;" />

- **SetUid位**

  当文件或程序(统称为executable)被执行时, 操作系统会赋予文件**所有者的权限**

  这个位可以用来提供特权程序，在特定规则下访问修改其不能修改的文件。

**[讨论问题4] 能否举一个用到setuid的例子**

类Unix系统的密码一般是放置在“/etc/paswd”和“/etc/shadow”中
普通用户没有读写权限，用户如何更改密码？
实际上就是利用 passwd 程序的 setuid 位为on，在执行过程中，将用户身份变为ROOT
这样普通用户在执行“passwd”命令时，实际上以有效用户root的身份来执行的，并具有了相应的权限


#### 2.3.3 I/O请求 

​	系统的I/O调用旨在消除各种设备和访问方式之间的差异。 “随机”和“顺序”的I/O之间没有区别，系统也不施加任何逻辑记录大小。普通文件的大小由写入文件的最高字节决定，不需要也不可能预先确定文件的大小。

​	文件系统中没有用户可见的锁，对于可以打开文件进行读取或写入的用户数量也没有任何限制；尽管当两个用户同时写入文件时文件的内容可能会被打乱，但实际上不会出现困难。他们是不必要且不足够的。

**[讨论问题6] 为什么说给限制用户的读写加锁是不必要也不足够的？**

不必要是因为我们并不是维护一个单进程管理的单文件的大数据库，我们不需要保证文件内容的一致性。只需要保证整个文件系统的逻辑一致即可。
不足够是因为普通意义上的锁来说，即当一个用户在读的时候另一个用户不能写，但这样也无法完全避免混淆。
[for example, both users are editing a file with an editor which makes a copy of the file being edited.]

​	应该说，当两个用户同时从事诸如写同一文件，在同一目录中创建文件或删除彼此的打开文件之类的不便活动时，该系统具有足够的内部联锁来维持文件系统的逻辑一致性。



#### 2.3.4 文件系统的实现

![](https://tva1.sinaimg.cn/large/008i3skNgy1gwern4drsjj30cv08a3yw.jpg)

​	一个目录条目包含一个关联到文件的名字和一个指向自身的指针。这个指针是一个叫`i-number`的整型数。当文件被访问时，它的`i-number`被用作系统表(`i-list`)的索引

1. 所有者 
2. 保护位
3. 文件内容的物理磁盘或磁带的地址
4. 大小
5. 上次修改时间
6. 到文件的链接数，也就是文件在目录中出现的次数； 
7. 一个表示文件是否是目录的bit
8. 一个表示文件是否属于特殊文件的bit
9. 一个表示文件是大还是小的bit

所有固定或可移除磁盘的空间都被划分为512字节的块，地址空间从0到设备本身的容量上限

- **Inode**

​	每个文件的inode中都留有8个块的设备地址空间，一个小文件（即 $\le512*8 bytes$) 可以直接被储存，大文件则是8个indirect block每个最多指向256个块，这样最大可以存储下 $8\times256\times512=2^{20} bytes$ 的最大文件容量

- **Buffer Cache**

​	对于用户而言，文件的读写都是同步且没有缓存的。在read系统调用返回后，数据马上可用。而且很方便的是，在write系统调用之后，用户的工作空间可以重复使用。实际上系统维护了一个复杂的缓存机制，来大幅度降低访问文件所需要的I/O操作数。

​	Unix会查找缓冲区以确定受影响的磁盘块当前是否在主存中。如果不是，将从设备上读取。缓冲区中的对应的字节被替换，然后在待写入块列表中创建一个条目。write系统调用返回，尽管实际上I/O会晚一点才完成。相反的是，如果读取一个单独的字节，系统会确定该具有该字节是否已经在缓冲区中，如果是，该字节会被立刻返回。如果不是，这个块将被读取到缓冲区，然后取出该字节。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gwew5xspc1j30bg083weq.jpg)

- **I-list**

> The notion of the i-list is an unusual feature of UNIX

​	i-list架构独立于目录的层级关系，只需要对线性的i-list做一系列操作就可以很好的组织维护文件系统。

​	但同时i-list的设计引起了一些奇怪的问题，例如，谁应该负责文件所占用的空间。*（既然一个文件的所有目录条目都具有相同的状态，文件的所有者负责是不公平的）*

**[讨论问题5] 为什么说文件的所有者负责是不公平的？**

一个用户可能会创建文件，另一个用户可能会链接到这个文件，而第一个用户可能会删除文件。第一个用户仍然是文件的所有者，但是他应该对第二个用户负责，这是不合理的。



## 3. 进程和映像

### 3.1 进程映像

​	映像（image）其实就是我们熟知的进程上下文。它包括核心映像，通用寄存器的值，打开的文件，当前目录等内容。

<img src="https://media.geeksforgeeks.org/wp-content/uploads/20200514150923/process1.png" style="zoom:80%;" />

​	进程是一个映像的执行。当处理器代表一个进程去执行时，映像要驻留在核心内存中。在其他进程的执行过程中，它仍然驻留在核心内存中，除非一个高优先级的进程强制性的把它从内存中交换到固定磁头的磁盘驱动器上。

从高地址到低地址分为三段

- **栈段**（由上之下增长）

- **数据段**

  可写非共享，可向上扩展

- **代码段**（写保护）

  单一拷贝被执行所有相同程序的进程共享。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gwetekcbeej309409mdg9.jpg)



### 3.2 进程

​	除了Unix引导它自身进入运行中（init 进程），只能通过使用fork系统调用创建一个新的进程。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gwetk06jscj30di049t8s.jpg)

​	当一个进程执行fork系统调用，它会被分离成两个独立的执行进程。这两个进程有各自独立的源印象的拷贝，并共享所有打开的文件。

### 3.3 管道

![](https://tva1.sinaimg.cn/large/008i3skNgy1gwetv8ny0tj30e606dt8v.jpg)

​	非常聪明的IPC机制，A只要一端如同写普通文件一样写入，B在另一端准备读取即可。A,B可以并发的执行，并且相比临时文件，在内存中的buffer不会有额外的磁盘I/O消耗。

## 4. Unix Shell

> For most users, communication with UNIX is carried on with the aid of a program called the Shell. (Interactive)

![](https://tva1.sinaimg.cn/large/008i3skNgy1gweu4951cfj309a09h3ys.jpg)

​	在最简单的情况下，一个命令行由命令名和跟随的参数组成，命令名和参数之间通过空格分隔。

```text
command arg1 arg2 ... argn
```

​	Shell将命令名和参数分割为独立的字符串，这样就能找到名为command的文件。command可能是一个包含"/"的路径名以指出系统中的任何文件。如果command被找到，它将被引入到核心内存中被执行。Shell收集到的参数对于command是可访问的。Shell重新回到自己的执行当中，并立刻准备接受用户输入的下一条命令。

### 4.1 标准I/O

​	设定 ```std_in = 0```、```std_out = 1```、```std_err = 2```，这样做有什么好处？

​	在Unix之前，很多程序没有建立输入输出的过程，而操作系统通常采用比较复杂的 [Job Control Language](https://en.wikipedia.org/wiki/Job_Control_Language) 脚本来建立输入输出的连接。

​	而Unix将输入输出默认连接到终端的键盘输入和终端的显示器上，以预定义的形式免去了很多不必要的工作。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gweunbmq9rj30gv090756.jpg)

## 5. 陷入

### 5.1 故障

​	PDP-11硬件检测到程序错误，例如对不存在的内存的引用。此类故障会导致CPU陷入内核。当发现非法行为时，除非做出其他安排，否则系统会终止该进程，并将image 写入当前目录中。(Core Dump)

### 5.2 中断 (interrupt)

​	程序产生了未预期的输出，可以通过键入“delete” 字符生成一个 `interrupt` 信号来停止程序，并且这个信号之后导致程序停止而不会 Core Dump。



## 6. Unix 设计哲学

### 6.1 简洁 (Simplicity)

> First, since we are programmers, we naturally designed the system to make it easy to write, test, and run programs.

- **User Friendly**

  ​	系统以使其易于编写，测试和运行程序。对编程便利性的渴望的最重要表达是该系统被安排用于交互 (interactive) 使用，即使原始版本仅支持一个用户。

  ​	我们认为，设计合理的交互式系统比“批处理”系统更具生产力和使用满意度。而且，这样的系统相当容易适应于非交互使用，而反之则不成立。

- **File Tree**

  层级的目录结构，让文件管理十分简洁清晰。

- **File Abstract**

  一切皆是文件，将外部设备和目录统一抽象为文件，统一管理，代码复用而高效。所谓 Less is More

  

### 6.2 易于部署 (Flexiblility)

> UNIX can run on hardware costing as little as $40,000, and less than two man-years were spent on the main system software.

​	用于安装UNIX系统的 PDP-11/45 是一个16位字长（8-bit byte），具有144K字节(byte)核心内存的计算机；UNIX占用了其中的42kb。然而这个系统包括了大量的设备驱动并且为I/O缓冲区和系统表分配了足够的空间；一个能够运行上述软件的最小系统，最小总共只需要 50kb 的内核。



### 6.3 可维护性 (Maintainable)

> Third, nearly from the start, the system was able to, and did, maintain itself.

​	由于实现的简洁性，系统可维护性很好，从1972~1973年DENNIS M. RITCHIE AND KEN THOMPSON 用B和C相继重写了Unix内核后，用高级语言开发的Unix有着很好的可移植和迭代能力。



**[讨论问题7] 你认为是什么让Unix如此成功？**
1. 设计哲学
  Unix 虽然不是最早的分时操作系统，但其提出的一系列颠覆性的概念可以说重塑了操作系统界很多实现的范式，包括层级的文件系统以及一切皆文件的思想，一直延续至今。在如今看来这些好似稀松平常的概念，在当时的视角下是绝不trivial的想法。
2. 时代背景
  在Multics的衬托下，Unix继承了Multics的功能，以其简单而有效的设计模式启发了大家，获得了商业上的巨大成功。
	可以说Unix的成功，是因为其出现在一个对的时间，并且做了对的事情。
  同样我们在回顾 UNIX 的设计发展历程的时候，对我们当今设计和进一步优化目前的操作系统有指导意义，可以说是以史为鉴，让我们能够洞见未来的发展。


### The Unix Family Tree

![](http://www.netneurotic.net/mac/unix/images/UNIXthumb.png)







## Reference

[1] [Ritchie, D.M.](https://en.wikipedia.org/wiki/Dennis_Ritchie); [Thompson, K.](https://en.wikipedia.org/wiki/Ken_Thompson) (July–August 1978). ["The UNIX Time-Sharing System"](https://web.archive.org/web/20101103053325/http://bstj.bell-labs.com/oldfiles/year.1978/BSTJ.1978.5706-2.html). *[Bell System Technical Journal](https://en.wikipedia.org/wiki/Bell_System_Technical_Journal)*. **57** (6). Archived from [the original](http://bstj.bell-labs.com/oldfiles/year.1978/BSTJ.1978.5706-2.html) on November 3, 2010.

[2] ["UNIX History"](http://www.levenez.com/unix/). *www.levenez.com*. Retrieved March 17, 2005.

[3] ["AIX, FreeBSD, HP-UX, Linux, Solaris, Tru64"](http://www.unixguide.net/). *UNIXguide.net*. Retrieved March 17, 2005.

[4] ["Linux Weekly News, February 21, 2002"](https://lwn.net/2002/0221/bigpage.php3). *lwn.net*. Retrieved April 7, 2006.

[5] [Lions, John](https://en.wikipedia.org/wiki/John_Lions): *Lions' ["Commentary on the Sixth Edition UNIX Operating System"](http://www.lemis.com/grog/Documentation/Lions/). with Source Code*, Peer-to-Peer Communications, 1996; [ISBN](https://en.wikipedia.org/wiki/ISBN_(identifier)) [1-57398-013-7](https://en.wikipedia.org/wiki/Special:BookSources/1-57398-013-7)





