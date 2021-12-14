---
title: 计网传输层 - 可靠传输协议实验
date: 2021-11-24 01:02:19
index_img: /img/tcp.jpeg
category: [Network]
tags: [RDT, GBN, SR]
math: true
---

# 计网传输层 - 可靠传输协议实验

*代码连接： [ZiYang-xie/RDT_protocol (github.com)](https://github.com/ZiYang-xie/RDT_protocol)*

## 1. 实验简介

​	本次实验要求完成 rdt3.0 基础停等协议的模拟，以及进阶的GBN和SR协议的实现。我们知道网络层的IP协议是不保证报文段按顺序正确交付，其只是“尽力而为”，因此在传输层中TCP需要实现可靠的传输协议为上层应用层提供可靠的传输支持。RDT3.0是最基础的可靠传输协议，其可以保证报文按序正确交付。而GBN和SR则在RDT3.0基础上添加了流水线架构，并且支持顺序or乱序ACK，从而进一步提升协议的速度。



## 2. 实验内容

​	实验的代码结构如下图所示，我们需要完成 A_output(), A_input(), A_timerinterrupt(), A_init(), B_input(), B_init()等函数的补充。

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gwo9yfv0m8j30oc0dgjsk.jpg" style="zoom:50%;" />

### 2.1 RDT3.0 (Alternating Bit)

![image-20211122220544951](https://tva1.sinaimg.cn/large/008i3skNly1gwoa2hbtkaj31i40fyact.jpg)

​	 其基本逻辑如上图所示，在实现代码的时候需要注意的是，通过一个标识位来标记当前状态来实现状态机。如果在ACK的等待状态接受到了上层的调用，则需要把报文先缓存下来，再到接受到ACK的时候检查缓存，按顺序发送。

```c
if (STATE == WAIT){
  	inform(__FUNCTION__, "Not yet acked, Buffer the Msg: %.20s", message.data);
  	cache_msg(&message);
  	return;
}
```

​	当 `checksum` 失败和收到重复的 `ACK` 的时候，需要重传上一次的分组。

​	报文的缓存使用循环队列模式，基础实现则是通过一个固定大小（足够大）的数组，加上循环的index实现。



### 2.2 Go Back N

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gwoaepdhylj31800aamyj.jpg" style="zoom:50%;" />

​	Go Back N 需要注意**累计确认**， 即 ACK(n): ACKs 所有n之前(包含n)的分组，同时定时器对应最老的那个没被ACK的分组。如果仅 ACK 窗口最左边的分组，会出现如下情况。

​	

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gwoahl3tgfj30zy0qkdi5.jpg" style="zoom:30%;" />

​	在传送分组的时候丢失了两个ACK，但因为没有 ACK0 整个系统就会陷入无限循环的卡死状态。GBN需要注意累计确认，将窗口滑到对应的位置，接收端的ACK发送正常接收的最后一个分组。



### 2.3 选择重传 (Selective Repeat)

​	

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gwoakrl3tnj31760owwhx.jpg" style="zoom:50%;" />

​	选择重传因为可以直接ACK乱序的分组，但是同时要保证交付到上层应用层的顺序，因此其在接收端也同样需要一个缓冲区，这个大小就为窗口大小，来暂存收到的报文段。当前面的报文全部被接受的时候就统一移动窗口，同样的对于发送端，当前面的报文全部被ACK之后可以移动窗口。

​	因此需要对缓冲区的内容进行标记，在实现中我是选用通过对传输报文的第一个字节标记 `\0` 来实现的。对于发送方，在缓冲区中如果被标记 `\0` 则说明该位置可写，即尚未存放分组或已经是离开窗口区域。

```c
void clean_pkt(char (*buffer)[20], uint32_t loc)
{
    buffer[loc][0] = '\0';
}
```





## 3. 性能测试

### 3.0 参数设置

```yaml
#define TIMEOUT 20
#define BUF_SZ 10000
#define WINDOW_SZ 10
```

### 3.1 基础情况 (无丢包和损坏)

- `参数: 10 0 0 10` 

> altBit: 139.041168ms
goBackN: 124.61444100000003ms
selectiveRepeat: 119.80553200000004ms

- ``` 参数: 100 0 0 10```

> altBit: 1172.531616ms
goBackN: 1058.7275695000003ms
selectiveRepeat: 1020.7928873333335ms

​	可以看到，在无丢包损坏的情况下，选择重传因为可以乱序ACK而占有优势，GBN因为流水线结构而速度较快，AltBit因为停等是速度最慢的。



### 3.2 丢包差错情况 

- 少量丢包和差错 `参数: 10 0.1 0.1 10` 

> altBit: 247.166718ms
goBackN: 219.95912200000004ms
selectiveRepeat: 227.78014633333336m

- 大量丢包和差错 `参数：10 0.5 0.5 10`

> altBit: 3346.8271479999994ms
goBackN: 2001.064117ms
selectiveRepeat: 2813.4893593333336ms

​	在丢包和损坏情况下，可以看到 GBN 和 SR 都起到了非常不错的作用，和 AltBit 拉开了差距。但我们同时可以看到 GBN 比 SR 的效果更好，这是因为在丢包的情况下，GBN可以直接重传全部，而性能损失较小。但SR因为要在接收端维护缓冲区，所以有一定的性能损失。



### 3.3 快速大量发包情况

我进而测试了较为极限情况下的大量快速发包的效果对比。可以发现选择重传在这种情况下比较占优。得益于其乱序 ACK 的设计

- 快速大量包 `参数: 1000 0 0 0.1`

> altBit: 11117.566406ms
goBackN: 8406.244140499999ms
selectiveRepeat: 7456.902832000002ms





## 4. 总结

​	本次实验中，通过代码模拟实现了可靠数据传输的基础版本 RDT3.0 以及其进阶GBN和SR协议的实现，更加深入的理解了可靠数据传输的工作原理。



## 附录

除了固定随机参数以外，测试代码中还提供了多次实验取均值的接口，以进一步测试。

```python
def multi_test(N):
    time_list = []
    for protocol in protocol_list:
        for i in range(N):
            time = test_time(protocol, args)
            time_list.append(float(time))
        avg_time = np.mean(time_list)
        print(f'[{protocol}]: {avg_time}ms')
```

- **部分实验截图**

![](https://tva1.sinaimg.cn/large/008i3skNly1gwoc6uggs1j30e00g7gno.jpg)
