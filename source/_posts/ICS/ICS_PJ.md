---
title: ICS_PJ Y86 Extended CPU
date: 2021-01-06 10:23:45
index_img: /img/ICS_Lab1/top.jpg
category: [ICS]
tags: [CPU]
---

*项目地址： https://github.com/fdu2019xzy/ICS_Y86*

---

## 一、项目基本信息

### 1. 项目简介

Y86-Extended 是由谢子飏和王少文组队合作完成的项目，作为复旦大学 ICS (上) 的期末PJ，
本项目在实现了本项目在基础流水线处理器架构之上添加了**更高级的分支预测**、**硬件栈**、 **y86 指令集扩展**，并编写了一个**汇编器** yyas (支持宏定义)，以及其他特殊功能。结合精美的前端，根据上传文件性质不同 (yo/ys) 可以在普通模式和编译模式下运行程序, 具有较高的鲁棒性, 能够直观的看到基本信息、所有的寄存器信息 (包括流水线寄存器)、运行进度、当前指令和其他信息。

支持任意**前进回溯**、**设置断点**、**终端输出** (包含**彩色矩阵**)等扩展功能

### 2. 项目分工

王少文：后端 CPU 主体设计及汇编器
谢子飏： (中) 前端及测试

### 3. 项目架构及设计思路

我们的项目使用**前后端分离**开发策略，考虑到运行速度后端由 C++ 编写，中前端使用 Python, Vue.js, Js 开发。

- 项目架构图

<img src="https://s3.ax1x.com/2021/01/06/sAvtMV.png" style="width:500px"/>

- 交互逻辑和运行过程

整个交互逻辑和运行过程分为，前端上传 ys/yo 文件 post 到中端的 lauch.py 的flask服务器，flask驱动编译器编译并将编译好的yo文件传到 CPU 中。CPU 按设计跑完输出 data.json 文件，前端 fetch 后端 CPU 生成的 data.json 在前端按操作逻辑进行展示。

由于后端 C++ 跑的非常快，我们前端在上传后几乎能够即时获得 data, 而这样前后端分离的好处在于，我们前端可以非常自由的展现数据，所有的前进回溯甚至跳跃，都可以在 O(1) 内实现，不会出现很大的限制。

---

## 二、使用方式及功能详解

### 1. 具体使用方法
(见项目目录下 README.md)

### 2. 功能详解

- 主体功能介绍图

![](https://s3.ax1x.com/2021/01/06/sAv8Gn.png)

![](https://s3.ax1x.com/2021/01/06/sAv3Ps.png)

- 当前指令模块

<img src="https://s3.ax1x.com/2021/01/06/sAvV2t.png" style="width:250px"/>

当前指令显示我们当前运行到的指令 (PC 所指)，并包含前后文2条指令的信息。

**隐藏断点设置**

为了前端的布局美观和统一性，断点设置被隐藏进了当前指令卡片中，点击当前指令卡片即可设置断点。

<img src="https://s3.ax1x.com/2021/01/06/sAvE8I.png" style="width:400px"/>

可以任意设置增删断点

- **终端字符及彩色矩阵输出**

根据后端的内存映射，我们可以让终端输出字符以及彩色矩阵。

<img src="https://s3.ax1x.com/2021/01/06/sAvJx0.png" style="width:300px"/>


- 附带我们编写的 YYAS 汇编器，能够实现 ys 到 yo 的编译


---


## 三、代码详解

### 1. 前端代码架构分析

- 前端代码架构图

![](https://s3.ax1x.com/2021/01/06/sAvl5j.png)

前端设计采用 Vue.js 和前端框架 iView 设计，由于 Vue 的**响应式渲染**性质，非常适合我们本次的前端要求，同时为了保持多平台使用的可能性，Vue并没有使用脚手架，所有的 Vue.js、jQuery 都采用了 **CDN 引入**模式。

其中 Index 是主页面入口，Index.js 包括 Vue app 的创建和 Vue 页面路由， Index.html 包含主页面的 DOM 树

Pages 里面包含了主要的界面，主页面是 main.js 包含 cpu 的主要功能。
所有的 Pages 作为 export 模块被引入到 Index.js 中进行路由。
main.js 里面包含了 main 页面所有**逻辑函数**，**声明周期钩子**，以及 main 页面的 DOM 树， 其他页面也是同样架构。

（由于是 CDN 引入 main 页面要插入路由的话， DOM 树应当是字符串形式传入 template 中）

CSS 里面包含了所有主要的样式表，对功能和作用区域的不同做了简单的分类。

Static 里面包含了需要读取的 Json data 信息，以及当前处理编译文件放置在 Source 中。

- **中台前后端链接代码**

前后端连接使用的是 python flask 服务器，通过接前端上传表单的 post，将获得的文件传入编译器编译之后喂入 CPU 中，获得 data.json 前端再 getData。

### 2. 后端代码架构分析

- 后端代码架构图

![](https://s3.ax1x.com/2021/01/06/sAvQaQ.png)

- 汇编器部分
  - 汇编器的可执行文件为yyas.py，这是包含了所有代码（不需要import）的最终文件，试试./yyas -v 和 ./yyas -h，有惊喜
  - instr.py为Y86所有指令的信息（不含Y86 Extended的扩展部分）
  - striper.py为Stage1时，将yo文件转化为yoraw的脚本，目前已经被弃用，仅作为存档提交。现在实现相同功能请使用./yyas -r -np
- CPU部分
  - Controller.cpp 为CPU的控制器，主要包含了流水线的控制逻辑
  - Device.cpp 为CPU流水线的核心，包含了FDEMW五个阶段的运行逻辑
  - Output.cpp 有一些用于输出的函数，用于Stage1输出至命令行或Stage2输出至json文件
  - Util.cpp包含了一些辅助的函数，例如In函数和Format函数（这里用了很多modern cpp的魔法）


---

<div style="page-break-after: always;"></div>

## 四、 实现细节

#### 前端细节

前端搭建了 anywhere 服务器 (用于解决跨域问题, 踩坑经历可以看第五节)，主要函数在 main.js 文件中， 通过各类声明周期函数和写的 handle 来处理上传，运行，停止，重置，以及断点设置等功能，此处不展开细讲，主要是后端。

（前端使用了 anywhere 服务器 + 中端 flask 服务器的双服务器架构，其主要是因为我们前端在 import 模块时不开本地服务器会产生跨域问题。对于跨域问题我们最开始遇到是在获取 data.json 的时候，踩坑过程中我们曾尝试通过jsonp 解决跨域问题，并且成功了，但在 import 模块时再次遇到了跨域问题，为了一劳永逸，我们直接使用了 anywhere 直接开了一个本地服。）

#### 后端细节

1. **关于流水线**：为了尽可能的靠近实际的硬件架构，我们的CPU核心实现分成了**wire**和**reg**两种struct，用来模拟verilog硬件语言中的wire和reg，reg是实际存在的寄存器，而wire只是用于逻辑中转。为了模拟CPU的并行化，我们采用了如下的逻辑

   1. 利用Reg中的数据计算F D E M W，写入wire变量（实际不存在）
   2. 利用现在的wire上的值，更新流水线及其他状态
   3. 将wire的值写入下一个Reg，例如f_wire写入D_Reg，当流水线状态为NORMAL时回到a

2. 每个阶段的具体实现逻辑， 与CSAPP上的电路实现差别不大，此处不再赘述

3. **关于扩展指令集**：我们的指令集扩展见**ISA.md**，具体到流水线上只需少许修改各个阶段的逻辑，与官方设计思路差不太多，依次修改FDEMW就行。

4. **关于分支预测策略**：我们的分支预测采用的是**2比特溢出预测器**，该预测器采用的逻辑如下图

   ![预测器](https://s3.ax1x.com/2021/01/06/sAvG2q.png)

   1. 当JXX的信号不是Jmp时，更新上述的状态机
   2. 若状态机信号为3或者2，则进行跳转
   3. 若状态机信号为1或者0，则不进行跳转

   除此之外，我们也需要相应的改变分支错判的逻辑，需要加一个IfJump信号，当E阶段输出的信号与IfJump信号不同，说明分支预测错误，需要清空流水线。

   这样的分支预测器，据统计可以达到90%的正确率（https://en.wikipedia.org/wiki/Branch_predictor），在我们的测试中，一般情况下可以提升10%-30%的性能

5. **关于复杂指令**：在我们增加的指令中，有一些指令如mulq、divq和remq都是很耗时的指令，如果强行让他们在一个周期内完成，我们的CPU时钟频率就会非常低。因此我们假定这部分指令所需要的时间为其他指令最长耗时的10倍，在E阶段设置一个counter，建立一个递减的状态机，使用下面的控制逻辑

   1. 如果指令属于上述复杂指令，将状态机置于10
   2. 如果状态机不为0，则使F D阶段进入STALL阶段，E阶段继续计算，但阻断E到M的输入，状态机减1
   3. 如果状态机为0，则流水线继续执行

   这样就可以实现维持较高CPU频率的同时，实现复杂指令。（即不会因为加了这个指令使其他指令的执行变慢）

6. **关于硬件栈**：为了避免每次调用ret指令时两个时钟周期的浪费，我们采用了CSAPP上的硬件栈。在硬件上，硬件栈可以由多路选择器和多个寄存器构成，在这里我们简单的使用Stack实现（大小为0x20）。采用如下的控制逻辑：

   1. 如果调用call指令，在栈没有满时，将地址载入硬件栈
   2. 如果调用ret指令，在栈非空时，弹栈，将值读出
   3. 在M阶段执行后，判断硬件栈的预测是否正确
      1. 若正确，则继续执行
      2. 若不正确，则清空 F D E的流水线

   使用硬件栈后，最差的情况即是与无硬件栈情况相同，最好情况在我们的测试中，可以增加30%+的性能。

7. 关于指令不可写和非指令不可读保护：我们的汇编器会计算yo文件中，最后一个是指令的位置，在CPU运行时，会进行动态的判断

   1. 在Fetch阶段读取不可读的位置，会产生INS错误
   2. 在其他阶段写入不可写的位置，会产生ADR错误

8. 关于宏定义：为了CPU的扩展性和兼容性，我们使用了宏定义来动态的开关某些特性，例如 ```OUTPUT_JSON``` 宏定义后即可输出 data_json 供前端使用。

#### 汇编器细节

我们的汇编器采用了依次处理的方式，如下(Code Explains itself)

```python
    lines = gen_list(lines) #解析关键词
    lines = remove_single_line_annot(lines) #去除单行注释
    lines = remove_multi_line_annot(lines) #去除多行注释
    lines = detach_label(lines) #找到对应的label
    lines_ref = copy.deepcopy(lines) #复制一份用于输出
    mem = get_memaddr(lines) #找到内存对应的地址
    labels = get_def_label(lines) #处理.define
    lines = replace_label(lines, mem, labels) #处理label
    byte_code = gen_byte_code(lines) #产生yo代码
```



#### 测试细节

我们采用Google Test 和 Python Unitest来辅助我们进行我们的程序测试。其在我们项目文件的 test 文件夹下，我们通过 shell 脚本和 python 脚本将模拟器的输出进行处理，转换成 gtest 代码以便于我们进行测试。

基本语句即为 ```EXPECT_EQ(x, y)```

- 测试代码结构

![](asset/测试代码架构.png)

使用 Google Test 我们能够清晰的看到测试点信息。

- 普通 yo_test

<img src="https://s3.ax1x.com/2021/01/06/sAvZxP.png" style="width:300px"/>

- Hornor 测试

<img src="https://s3.ax1x.com/2021/01/06/sAvmKf.png" style="width:300px"/>

- 附加测试

<img src="https://s3.ax1x.com/2021/01/06/sAvMVg.png" style="width:400px"/>

---

<div style="page-break-after: always;"></div>


### 五、特点和创新

1. **汇编器创新**：我们兼容现有的所有ys文件的基础上，还添加了一些伪指令和语法

   1. 不需要写逗号了：现在所有的ys指令操作数之间可以使用空格隔开
   2. mrmovq和rmmovq：现在不需要强制写括号，可以直接mov至常数地址（此时寄存器为RNONE，我们的CPU也做了相应的修改）如rmmovq %rax $800
   3. 伪指令
      1. .define 类似C语言的宏定义
      2. .string "mystring" 将后面的mystring转化为对应的ASCII码写入
      3. .byte .hword .word支持
   4. 能够生成yo文件和yoraw文件，生成的yo文件比官方的yo文件漂亮不少，完全对齐
   5. 有完整的命令行参数

   ```
   usage: yyas [-h] [-o OUTFILE] [-r] [-np] [-v] sourcefile
   
   yyas: Assemble .ys file to .yo file or/and .yo file to raw byte file
   
   positional arguments:
     sourcefile            The source file to be assembled
   
   optional arguments:
     -h, --help            show this help message and exit
     -o OUTFILE, --output OUTFILE
                           Assign the output file
     -r, --raw             Generate raw byte file
     -np, --noprefix       Do not generate prefix in raw output
     -v, --version         show version
   ```

2. **处理器创新**（见具体实现）

   1. 更好的分支预测器
   2. 硬件栈
   3. 几乎三倍的指令集支持（及其扩展含义）
   4. 复杂指令集的多周期支持

---

### 六、总结与回顾

#### PJ 心得总结及致谢

本次 PJ 让我们更加深入的理解了现代流水线处理器的原理，各种硬件栈，扩展指令集的设计也拓展了我们的技术边缘，非常不错的体验。最后非常感谢助教和金老师一学期的辛苦工作。在很多方面帮助了我们，助教的 Lab 非常精彩有趣，金老师的课十分幽默风趣，非常感谢各位的付出。