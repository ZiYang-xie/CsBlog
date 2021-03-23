---
title: ICS-Lab2 二进制炸弹
index_img: /img/ICS_Lab1/top.jpg
date: 2020-11-06 15:44:39
category: [ICS]
tags: [Assembly]
---

# ICS-Lab2-Bomb

> 这个是CS:APP的第二个lab，主要着重于汇编代码的阅读

***

## 完成截图

<img src=https://codimd.s3.shivering-isles.com/demo/uploads/upload_244e1f55d2823d58f65eabab9478d7ce.png></img>

***

## Phase 1 - 入门
### 一、分析

> 练手入门题，用esi寄存器储存答案地址 (一个立即数)
```
mov    $0x402400,%esi
```
> 之后调用了一个 string_not_equal 函数比较输入和答案是否一致，一致就通过了。
```
callq  401338 <strings_not_equal>
```

### 二、gdb调试
> 看一下内存地址里面存了什么，获得flag

<img src=https://codimd.s3.shivering-isles.com/demo/uploads/upload_4801c7e177c56f6e7299c273d0120988.png></img>


- **答案**: Border relations with Canada have never been better.

***

## Phase 2 - 循环
### 分析

> 本题是一个do while Loop, 难度不大, 耐心读就行了

**关键位置**
- 信息1 ： 看到 read_six_number 知道输入6个数，再往下看

```
cmpl   $0x1,(%rsp) # 比较栈顶地址所存变量大小是否为1
je     400f30 <phase_2+0x34> # 如果为1 跳转至地址 400f30
callq  40143a <explode_bomb> # 如果不为1，直接炸了
jmp    400f30 <phase_2+0x34> # 跳转至地址 400f30
```

- 信息2 : 第一个数为1

下面进入Loop Body

<img src=https://codimd.s3.shivering-isles.com/demo/uploads/upload_27148224f8cf2be48266eaa52f50b2f8.png></img>

- 信息3 : 
可以看到这个循环把前一个数乘了2，跟后一个数比较, 如果相等就能够继续，不然就炸了。

> 综上也就是说这是一个首项为1，公比为2的等比数列，共6项。

所以答案就是 1 2 4 8 16 32

***

## Phase 3 - 分支
### 分析

> 第三题关键点在于用gdb查看一下jumptable

 我们先看一下输入，在输入了两个变量后，esi里放了内存中的一个可疑的东西，我们用gdb看一眼。
```
mov    $0x4025cf,%esi
```

```shell=
(gdb) p(char *) 0x4025cf
"%d %d"
```

 发现原来是输入两个整型，再往下看

```
cmpl   $0x7,0x8(%rsp) # 将 M(rsp + 8) 看作32位无符号数跟7比较
ja     400fad <phase_3+0x6a> # 如果大于就跳转至 0x400fad (炸弹炸了)
```

 发现如果输入的第一个数大于7就爆炸了，看来switch最多只有7个case

```
jmpq   *0x402470(,%rax,8) 
# 跳转至 (eax * 8 + 0x402470)处所存的地址 （jumptable）
```

> 最关键的是这一句，构造了一个 switch 的 jumptable，我们知道地址是 0x402470，按照 case * 8 + 0x402470 跳转到该地址里面的地址，所以我们用gdb看一下。

<img src=https://codimd.s3.shivering-isles.com/demo/uploads/upload_ae5e359c30ff5ccb9292a7472c39eb19.png></img>

- 我通关选了case 1（它比较特殊，处理它其他内存地址跳转都是按case从小到大顺序的，只有case 1 在最后一个，当然其他也都能过。）

- case 1 跳转到了 0x400fb9 地址

```
mov    $0x137,%eax 
# eax = 0x137 (311) (不用跳转了，下面就是 0x400fbe)
```

其将eax置为了0x137，要小心是16进制，所以对应十进制311

```
cmp    0xc(%rsp),%eax # 比较 M(rsp + 12) 和 eax
je     400fc9 <phase_3+0x86> # 如果相等就跳转至 0x400fc9 (过关了！)
```

最后是一个比较，如果eax和第二个输入值相同就过了。

- 本题答案（不唯一)

| case | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
| :----: | :----: | :----: | :----: | :----: | :----: | :----: | :----: | :----: |
| ans | 207 | 311 | 707 | 256 | 389 | 206 | 682 | 327 |


***

## Phase 4 - 递归
### 分析
- 这题是个递归，不过不用很深，很快就能看出答案。

先正常读两个数，放在rdx，rcx中，检查输入。

```
cmpl   $0xe,0x8(%rsp) 
# 比较 M(rsp + 8) (既 rdx) 与 0xe
jbe    40103a <phase_4+0x2e> 
# 如果 rdx <= 0xe (14) 跳转至 0x40103a, 不然就炸了 (作为无符号数)
```

> 这两行汇编告诉我们，rdx一定要小于 0xe (14) 且大于等于0, 不然炸了, 大幅度缩小了范围。

> 接下来就进入了函数递归调用，先做点预处理，把edx里面存一个立即数14，然后edi为第一个输入值，esi = 0 进入fun4

```
mov    $0xe,%edx # edx = 0xe (14)
mov    $0x0,%esi # esi = 0
mov    0x8(%rsp),%edi # edi = (第一个输入值)
callq  400fce <func4> # 调用func4
```
> 先不着急看fun4，先看看最后要怎么过关
```
test   %eax,%eax # eax & eax
jne    401058 <phase_4+0x4c> 
# 如果ZF == 0 就跳转（既eax != 0)，跳转至 0x401058 炸了
cmpl   $0x0,0xc(%rsp) # 比较 M(rsp + 12) 和 0
je     40105d <phase_4+0x51> # 如果相等就跳转到 0x40105d, 不然就炸了
```
- test 实际上就是一个与操作，所以我们知道需要 eax == 0 且 M(rsp + 12) == 0，到这我们发现，第二个条件只要我们一开始输入的第二个参数为0，就能够保证，那么下面我们就要看进入fun4之后如何让返回值 eax == 0

> 再回来看fun4，其分为两部分，一个是递归的主体，一个是判断是否继续递归。一开始先对eax 和 ecx 进行一些操作。
- 我们发现 eax 和 ecx 的值在第一层递归都被置为14，(esi 为 0)按其操作得到 eax 除2, ecx 逻辑右移 31 位为0, 接着其实就是比较 edi 和 rax, **相当于就是比较第一个参数和常数 7**

```
jle    400ff2 <func4+0x24> # 若ecx <= 就跳转至 0x400ff2
```
```
mov    $0x0,%eax # eax = 0;
cmp    %edi,%ecx # 比较 ecx 和 edi 
jge    401007 <func4+0x39> 
# 若 edi >= ecx 跳转至 0x401007 返回
```

- 接着是一个跳转, 如果满足我们就跳转至 0x400ff2, 我们发现这里已经满足了我们需要的 eax == 0，而想要结束就得使 edi >= ecx (7), 所以我们发现，对于上下两个跳转条件，只要 edi == ecx == 7 就能一直成立，从而直接达成条件，不用进入递归。

进而我们得到了本题答案：7 0

***

## Phase 5 - 指针
### 分析
- 这题我觉得是最好玩的一题，先直接分析如何通关。

```
mov    $0x40245e,%esi # esi = 0x40245e 
# 待比较的 string (flyers) 从 0x40245e 移动至 esi
```

- 我们在接近返回时看到了一个非常可疑的内存地址，直接给它打出来。

```
(gdb) p(char*) 0x40245e
$4 = 0x40245e "flyers"
```

> 发现是一个可疑字符串 flyers，阅读上下文汇编代码可知，最后是比较字符串是否和指定字符串 "flyers" 一致。

- 我们再往上看看要怎么输入

```
callq  40131b <string_length> # 比较字符长度是否为6
cmp    $0x6,%eax # 比较 eax 和 6
```

发现输入一定要是六个字符 *(于是试了试 flyers 果然不对)*

- 往下看，发现了一个 Loop 循环了6次

<img src=https://codimd.s3.shivering-isles.com/demo/uploads/upload_016df1440f7044a54fb4ced529595b58.png></img>

> 经过仔细阅读后，发现这个居然是遍历六个输入字符，将其 ascii 码低4位取出来作为偏移量 (offset),在一个基地址 （0x4024b0）后面取字符出来组成 flyers.

- 立刻开启 gdb 查看基地址附近的内存

**发现分别对应的偏移量是 9, 15, 14, 5, 6, 7**
> 直接查 ascii 码，发现对应 ionefg 、IONEFG 或者有一些不是字母的字符也行，只要低四位是正确的就可以。 

**本题答案:**  ionefg (答案不唯一)

***

## Phase 6 - Node结构体
### 分析
这题还是比较麻烦的，代码比较长也比较复杂，要耐心读。

- 这题的代码可以大致分为输入检测与处理和一个对结构体的顺序检测.
> 最开始上来先输入六个数之后有个双循环，外部保证输入的六个数要大于等于1，且小于等于6，内部保证互异。所以总体看来就是输入的六个数就是123456, 现在问题是输入的顺序。

<img src=https://codimd.s3.shivering-isles.com/demo/uploads/upload_7194e52688ff3d696a3b889e2b17d63f.png></img>

这段代码遍历了所有输入并用7减去了输入的每个数，所以最后做出答案要记得反一下。

- 接下来代码比较复杂，外面大循环循环了六次，内部有两个平行的小循环。作用是构造结构体，并在栈帧中将其存放位置按照输入的数的大小计算得出

<img src=https://codimd.s3.shivering-isles.com/demo/uploads/upload_9ea5cd30293a18357af1da93c35e0f59.png></img>


> 分析代码，我们先发现一种特殊情况就是当前计算的数为1时（输入为6）edx直接就是给定的地址 0x6032d0, 其余的都按照其大小，在第一个小循环中循环相应次数，给 rdx 在原地址上相应偏移16位。

> 接着下来将其存入栈帧中 rsp + 32 到 rsp + 80 的位置

- 使用 gdb 查看 node

<img src=https://codimd.s3.shivering-isles.com/demo/uploads/upload_f2ecdd05728cbefbba59b068a29fdfdc.png></img>


最后我们看如何通关

<img src=https://codimd.s3.shivering-isles.com/demo/uploads/upload_a18a67d1b4dbfa16a7fd8800e3ee304b.png></img>

> 发现通关条件是要求定序，前面node大于后面的节点，根据gdb node节点的值和要求我们得到了 3 4 5 6 1 2 的结果，最后不要忘记这是被7减过之后的结果，原来的输入要还原。所以答案就是 4 3 2 1 6 5

**本题答案:** 4 3 2 1 6 5

***

## Secret Phase- 递归
### 一、进入方法
- 输入上面六种答案之后，发现 secret phase 并没有出现，于是开始着手寻找入口。

> 根据最后结果出现的字符顺藤摸瓜找到了 phase_defuse 函数，一看发现其中有一个可疑的 <string_not_equal> 函数以及几个可疑的内存地址,统统用 gdb 打印。

```
(gdb) p(char*) 0x402619
$2 = 0x402619 "%d %d %s"
```

```
(gdb) p(char*) 0x402622
$3 = 0x402622 "DrEvil"
```

> 发现之前调用过的__isoc99_sscanf@plt 还有隐藏用法，在输入两个数后再输入一个字符串 "DrEvil" 就能成功开启secret phase.

- 所以我们在最后一次调用__isoc99_sscanf@plt的 phase 4 输入 7 0 DrEvil, 果然在 phase 6 之后进入了 secret phase。

### 分析
- 虽然说是隐藏关，但是复杂度和难度比 phase 6 低了不少，和 phase 4 一样是一个递归，但不同的是这次真的需要递归几次，但也不深。只要确定好路线还是比较容易的。

```
callq  40149e <read_line> # 读一行
...
callq  400bd0 <strtol@plt> # 调用 strtol@plt
```

> secret phase上来读了一整行然后调用了一个 strtol，经过阅读strtol的源码，发现它是以10为base将字符串转为一个整型，实际上就是剔除了最后答案中除了数字以外的字符。(所以写上答案数字然后乱输字母也能过 bushi)

```
lea    -0x1(%rax),%eax 
# eax = (rax) - 1
cmp    $0x3e8,%eax 
# eax == 0x3e8 ? (即判断返回值与0x3e9)
```

- 这段代码告诉我们输入的数要小于 1000

```
mov    $0x6030f0,%edi # edi = 0x6030f0 (36)
callq  401204 <fun7> # 调用fun7
```
- 将edi置为 0x6030f0 (里面存的是36) 接着开始调用fun7

> 我们先不着急看fun7, 老样子先看过关要求。

```
cmp    $0x2,%eax # 比较一下 eax 返回是否为2
```

发现非常简单，只要eax返回值为2就行

- 再来看fun7

<img src=https://codimd.s3.shivering-isles.com/demo/uploads/upload_64ed470376ec6b72dc43690bf9b4ea0e.png></img>

> 一看这个递归是逃不过了，但我们要让eax == 2, 路线其实非常明确，第一次先走 way3 将 eax 弄成1，再走 way1 让eax*2， 最后一层我们让eax == 0 最后返回我们得到的 eax 就等于2
*（eax = 0 -> 1 -> 2）*

> 关键是一个 edx 和 esi 的比较，edx == rdi, 然后每次改变rdi使其中储存地址中所储存的变量逐步接近 esi 完成递归操作。

根据所存地址(注意/d打出的是10进制)，可以很容易找出递归路径

<img src=https://codimd.s3.shivering-isles.com/demo/uploads/upload_dfddd1801cd10c701dd2753164434977.png style="width:65%"></img>

- 输入22可以正好满足需求
> **22 <= 36 因而 rdi = (rdi + 8) (存8)，22 > 8 因而 rdi = = (rdi + 16) (存22), 22==22 所以 eax 返回 0 ，返回 1， 返回2，最终过关**

**本题答案:** 22 (可以带非数字字符)




