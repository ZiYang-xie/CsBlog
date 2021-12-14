---
title: OS Evolution Report
date: 2021-10-15 10:29:48
index_img: /img/OS/banner.png
category: [OS]
tags: [OS]
math: true
---

### 1. The Overall Summary of OS Evolution

​	The operating system is an indispensable part of the modern computer. It serves as the central component between the hardware device and user programs, providing mainly three basic functionality —— Multiplex, Isolation, and Interaction. It was not until 1971. OS officially became part of the computer science core curriculum by the promotion of the *SOSP* (symposium on operating systems principles) according to Denning's writing [^Denning2016] *Fifty Years of Operating Systems*.

​	Although everything looks obvious today, it's necessary for us to go back decades ago, thinking from the standpoint of the pioneers. The evolution of the operating system is full of hardship and lessons. It can also guide current OS development.

### 2. OS Evolution Stages

​	Batch systems, Interactive systems, Desktop systems, and Cloud-mobile systems four main stages are defined as the primary OS evolution Stages In Denning's writing from the user perspective.

​	Operating systems evolved with some features and came up with several OS principles. Denning classifies them as *Computation Law* and *Design Wisdom*, I prefer to call them engineering philosophy, and no matter how we define them, they should be *concise*, *timeless*, *effective*, and with *logical beauty*.

​    More specific OS evolution stages from an engineering perspective are concluded by Hansen [^Hansen2001]*The Evolution of Operating systems*.

​	As seven major phases are listed below.

- **Open Shop**

​	Back to Open Shop time, the 1950s, at that time, there was no concrete concept of Operating system. The computer is just a machine offer to the user as an open shop. People spend a significant amount of time setting up the devices, only a fraction of time for real computing, which is a considerable waste of computation resources. From another aspect, the bottleneck is the imbalance between super slow manual I/O operation and fast CPU computation speed. The Representative Computer at that time was IBM 701.

- **Batch Processing**

​    To address these problems, people came up with the idea that was using the machine itself to schedule its own workload (I/O procedure). And the Open Shop gradually turns to Closed Shop with only a few operators and satellite computers to deal with the I/O tasks. Users only need to hand in their punched cards and waiting for the results. It is the beginning of Batch Processing Era.

​	Inputting multiple jobs simultaneously is the advantage of Batch Processing, but the advantage also leads to its flaws. To reduce tape changing overhead, a large batch is needed, resulting in short jobs being bound with other jobs with FIFO output sequence to compute for a long time.

- **Multiprogramming**

​    OS evolution is not isolated. It is tightly connected to the rest parts of the computer to working as a system. In the 1960s, with hardware development, the large core memories, secondary storage with random access data channels, and hardware interrupt promoted OS's evolution, making multiprogramming practical.

​	Spooling technique tackles the inherent issue of Batch Processing, making the I/O devices sharing between jobs than exclusively owned by one specific job. A large random access buffer also makes the scheduling feasible. Using shortest-jobs-next to replace FIFO scheduling can deal with the short job waiting problem, however, It incurs some other problems, for example *Starvation* as well. 

- **Timesharing**

​    It is the timesharing idea that the first time making the computer interactive. One of the core problems it has addressed is the **Inbalance**. One is the imbalance between jobs' priority and the computation resources it is assigned, and another is the imbalance between people's slow operating speed and the fast CPU speed. Timesharing enables the system to do in-execution scheduling and interact with users with a time slice mechanism. 

​	Undeniably, Operating systems evolution is strongly connected with industry implementation. MIT CTSS and Multics verify the timesharing concept in engineering, and the business success of CTSS and other timesharing systems boost the evolution of OS in turn, one of the most outstanding contributions, *hierarchical file systems*. 

- **Concurrent Programming**

​    In terms of Concurrent programming, *Dijkstra* is the right person. He raised the THE multiprogramming system concept and made it himself. With semaphores for process synchronization and communication, people can address the underlying ***deadlock*** problems shown in the Multiprogramming and Timesharing Era. THE design philosophy (hierarchical structure) became the basic principle to design a system for complex problems nowadays *(e.g. OSI Network Structure)*. Dijkstra laid a solid foundation for concurrent programming and the following development of OS.

- **Personal Computing**

​     Personal computing making the operating system more flexible and creative because there are different users with different demands, and the operating systems are becoming more powerful and diversified. GUI (Graphic User Interface) is one of the most explicit examples of it.

- **Distributed Systems**

​    The advent of networks promotes the development of OS, making distributed systems practical. With network connection, the operating system can be distributed on different machines. This brought some new challenges, for instance, ensuring the integrity of file systems and memory systems becoming super tricky.

### 3. Conclusion 

​	The evolution of the Operating system is the result of the joint efforts of academia and industry field. From the beginning of Open Shop to the recent Distributed Systems, users' demand also plays a vital role in boosting the operating system design pattern, utilizing the hardware better and providing API to higher-level programs. Backing to the start point and looking at the OS development from a historical view, we can better understand the *5W* of OS. 

Learn the lessons, Start the future



### Reference

[^Denning2016]: Peter J. Denning. [Fifty Years of Operating Systems](./2016_CACM_Fifty Years of Operating Systems.pdf). *CACM* **59**(3): 30-32. 2016
[^Hansen2001]: Per Brinch Hansen. [The Evolution of Operating Systems](./2001_The Evolution of Operating Systems.pdf). *Classic Operating Systems: From Batch Processing To Distributed Systems* : 1-34. Springer, 2001