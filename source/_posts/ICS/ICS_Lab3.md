---
title: ICS-Lab4.2 各种链接姿势
date: 2020-11-15 17:37:43
index_img: /img/ICS_Lab1/top.jpg
category: [ICS]
tags: [Linker]
---

### Task0 - 简单链接

```cpp
#include "some.h"

int main(){
    testPrint();
    testPrint(5);
    notATest();
    return 0;
}
```

- **错误分析**

> 我们可以看到main函数中调用了三个函数，全是外部的，在链接时，符号表会进行搜索匹配。我们再来看some.h中定义了哪些函数.

```cpp
#include <cstdio>

void testPrint();
void testPrint(int num);
```

> 发现只有两个test函数,而没有notATest定义的函数，根据这个离谱的名字我们可以断定在cstdio中也没有同名函数。
所以最后符号表中没有匹配上，会引发链接错误。

- **解决方法**

> 直接将 notATest() 注释掉, 再在 makefile 中使用 
```
g++ main.cpp some.cpp -o main 
```

---

### Task1 - 链接与重复包含问题

- **解题过程**

通过阅读代码，我们可以发现改题的代码会根据宏DEBUG是否被定义而有不同的行为，决定是否打印更加详细的内容

```cpp
#include "function0.h"

void func0(){
    # ifdef DEBUG
    printDebug(); //打印debug信息 (详细操作在shared.cpp中)
    # else
    print(); //打印正常信息
    # endif
}
```

> 根据题意，我们需要在 Makefile 中,对于要求的 main1 需要做一个相等的条件判断

```makefile
main0:
    g++ main0.cpp function0.cpp function1.cpp shared.cpp -o main0
debug = False
main1:
ifeq ($(debug),True) 
	g++ main1.cpp -DDEBUG function0.cpp function1.cpp shared.cpp -o main1
else
	g++ main1.cpp function0.cpp function1.cpp shared.cpp -o main1
endif
```

*值得注意的是，makefile中 if 语句前不能有 tab 或者空格*

**Include 路径**
![](https://s3.ax1x.com/2020/11/15/DFeVyt.png)

- **问题**

1. 为什么两个function.h都引⽤了shared.h⽽没有出问题？本来有可能出什么问题。


> 因为 shared.h 中仅包含函数声明，而不包含函数的定义，因而不会有重定义问题。如果 #include shared.cpp 则会出现重定义问题。


2. 如果把shared.h中注释掉的变量定义取消注释会出什么问题？为什么？

> 会出现变量重定义问题，由于全局变量 string 被两次 include 到func0 和 func1 中，最后被同时引用至 main 中，导致重定义。在符号表中产生冲突，报错。

3. 通常使⽤shared.h中另外被注释掉的宏命令(#开头的那些⾏)来规避重复引⽤的⻛险，原理是什么？取消这些注释之后上⼀题的问题解除了吗？背后的原因是什么？

> 通过宏定义，在第一次 include 的时候定义宏为函数名，之后再次include 的时候由于已经 define 就不再次 include。这种方式的缺点是如果有已有宏名与函数名重复时，将会报错。使用 #pragama once 则是由c++编译器保证 include 一次，不会有宏名重复问题。

> 上题的问题并没有解决，因为FOO是全局变量，其赋了初值，被链接器标记为strong，被重复 include 到了 main 中，两个 strong 标记冲突报错。

---

### Task2 - 静态链接库

在这个 Task 中我们需要编译2个静态链接库，并链接3个静态链接库，完成编译。

**A Makefile**
```makefile
libA:	
	g++ -c A.cpp 
	ar -r libA.a A.o
```

> 先用g++ -c编译出A.o可重定位目标文件，再通过 ar 命令编译静态链接库。（libC 同理）

**Main Makefile**
```makefile
main : 
	cd A && make libA
	cd C && make libC
	g++ main.cpp B/libB.a A/libA.a C/libC.a -o main
```

> 再 CD 入每个目录进行 make 编译， 最后把静态链接库进行链接。 

- **静态库链接搜索路径顺序：**

1. ld会去找GCC命令中的参数-L
2. 再找gcc的环境变量LIBRARY_PATH
3. 再找内定目录 /lib /usr/lib /usr/local/lib 

- **问题**

1. 若有多个静态链接库需要链接，写命令时需要考虑静态链接库和源⽂件在命令中的顺序吗？是否需要考虑是由什么决定的？
> **需要考虑**，链接器在链接过程中按命令中输入的顺序进行符号表匹配，可以将这个匹配过程抽象的看作链接器在维护三个集合 E(待合并文件), U(被引用且尚未匹配), D（已匹配），根据顺序动态更新E, 和U,D, 最后如果 U 为空则正常整合 E 生成可执行文件，不然则报错有符号被引用了但未能匹配。所以如果我们的静态库都是相互独立的，那么顺序是没关系的。但如果互相依赖，那么我们必须保证在对某个符号的引用的库后，必然有一个库中存在对其的定义，不然则会报错。

1. 可以使⽤size main命令来查看可执⾏⽂件所占的空间，输出结果的每⼀项是什么意思？

| text  | data   |  bss  |  dec   | hex | filename |
| :---: | :---: | :---: | :---: | :---: | :---: |
|  22721  |  712  |  288 |  23721  | 5ca9 | main |

- **text:** 机器代码字节
- **data:** 包含静态变量和已经初始化的全局变量的数据段字节数大小
- **bss:** Block Started by Symbol (better save space) 存放程序中未初始化的全局变量的字节数大小，BBS段属于静态内存分配, 不占真实内存空间（仅占位符）
- **dec:** = test + data + bss
- **hex:** 16进制的dec
- **filename:** 顾名思义，文件名

---

### Task3 - 动态链接库

整体操作和上一个Task很像，只是链接的是动态链接库 .so

**A Makefile**
```makefile
libA:
	g++ -fPIC -c A.cpp
	g++ -shared -fPIC A.o -o libA.so
```

> 在 A 中先编译出 A.o 再用 -shared 编译出动态链接库 .so

**Main Makefile**
```makefile
main : 
	cd A && make libA
	cd C && make libC
	g++ main.cpp A/libA.so ./libB.so C/libC.so -o main
```

> 直接 cd 进去 make 出动态链接库后，再进行链接。

- **问题**

1. 动态链接库在运⾏时也需要查找库的位置，在Linux中，运⾏时动态链接库的查找顺序是怎样的？
> **动态链接时、执行时搜索路径顺序:**
> 1. 编译目标代码时指定的动态库搜索路径
> 2. 环境变量LD_LIBRARY_PATH指定的动态库搜索路径
> 3. 配置文件/etc/ld.so.conf中指定的动态库搜索路径
> 4. 默认的动态库搜索路径/lib
> 5. 默认的动态库搜索路径/usr/lib

2. 使⽤size main查看编译出的可执⾏⽂件占据的空间，与使⽤静态链接库相⽐占⽤空间有何变化？哪些部分的哪些代码（也要具体到本task）会导致编译出⽂件的占⽤空间发⽣这种变化？

| text  | data  |  bss  |  dec  | hex | filename |
| :---: | :---: | :---: | :---: | :---: | :---: |
|  3089 | 720  | 96 | 3905 | f41  |  main |

> **占用空间变小了**，因为不同于静态链接库将所有的静态库都整合入可执行文件中，动态链接库是在程序开始或正在运行时被链接加载的，所有可执行文件本身的空间占用会大幅缩小。

> main 中调用了 A, B, C 函数，所以其中的函数以及静态全局变量 A_name, B_name 会被被置于动态链接库.so中动态加载

1. 编译动态链接库时-fPIC的作⽤是什么，不加会有什么后果？
-fPIC 含义是 Generate position-independent code (PIC)，例如在汇编的 jmp 语句中通常使用的是固定的内部地址
```
100: COMPARE REG1, REG2
101: JUMP_IF_EQUAL 111
...
111: NOP
```
> 而通过 -fPIC 参数 jmp 语句所指向的是相对地址
使用的是代码段和数据段的OFFSET，从而实现位置无关，可以动态加载到内存中，不同进程可以共享。

```
100: COMPARE REG1, REG2
101: JUMP_IF_EQUAL CURRENT+10
...
111: NOP
```

> 如果不加，在某些系统下不会有很大的问题，但一般在 -shared 后面最好加上 -fPIC 来保证动态链接库是位置无关的，不然无法实现动态链接（由于位置相关即分配绝对内存地址，导致多个副本存在于内存中，无法实现动态链接）

**info in gcc manual**

- **-shared:**  Produce a shared object which can then be linked with other objects to forman executable. Not all systems support this option. For predictable results,
you must also specify the same set of options used for compilation (‘-fpic’,‘-fPIC’, or model suboptions) when you specify this linker option.*

4. 现在被⼴泛使⽤的公开的动态链接库如何进⾏版本替换或共存（以linux系统为例）？

> 通过动态链接，如果开发者需要维护程序的某一部分（某几个功能的函数），仅需要维护修改所在的动态链接库即可，然后将其发布。用户只需要替换动态链接库，在程序运行的时候自然会动态链接到新的链接库，**在接口保持不变**的情况下完成很自然流畅的版本更新。

---

### Task4 - ld手动链接

- **解题过程**
这题要求我们用ld链接器，进行手动链接，我们先试一下直接链接会发生什么

```makefile
#链接代码
ld -o main main.o some.o
```

```cpp
//报错信息
ld: warning: cannot find entry symbol _start; defaulting to 00000000004000b0
some.o: In function `notATest()':
some.cpp:(.text+0xc): undefined reference to `puts'
some.o: In function `testPrint()':
some.cpp:(.text+0x1f): undefined reference to `puts'
some.o: In function `testPrint(int)':
some.cpp:(.text+0x52): undefined reference to `printf'
Makefile:2: recipe for target 'main' failed
make: *** [main] Error 1
```

> 发现主要报错信息是 undefined reference to 'puts', 'printf', 说明标准库中的函数符号没有被成功匹配，我们需要把 stdc 加进去

```makefile
# 链接代码
ld -o main main.o some.o -lc
# 通过 -lc 命令直接添加标准库，也可以自行指定libc.so路径
```

> 可以发现main被成功编译出来，但运行时候发现bash报目录中无此文件
```bash
bash: ./main: No such file or directory
```

> 也就是我们并不能运行这个可执行文件，查找资料之后我们尝试指定使用的动态链接器再进行编译，就可以运行了

```makefile
ld -dynamic-linker /lib64/ld-linux-x86-64.so.2 -o main main.o some.o -lc
```

> 我查询了ld的手册试图查找原因，但并未发现为什么必须要使用--dynamic-linker指令

<div style="page-break-after: always;"></div>


- **--dynamic-linker=file**
>    Set the name of the dynamic linker.  This is only meaningful when generating dynamically linked ELF executables.  **The default dynamic linker is normally correct; don't use this unless you know what you are doing.**



> 接下去，我们的程序虽然能够运行起来了，但在 main 函数跑完之后会出现 *Segmentation fault (core dumped)* 提示，这也提示了我们对于 main 函数的初始化和结束可能并未正常执行。

- **GDB查看正常程序**

![](https://s3.ax1x.com/2020/11/15/DFeZOP.png)

> 通过 gdb 查看正常程序我们发现，正常的 main 函数执行栈中需要有两个函数为其保证环境 *__lib_csu_init* 和 *__libc_start_main*

- **__libc_start_main**
我们需要知道，在linux中，main函数的初始化环境和参数传递以及返回值处理工作是由 __libc_start_main 来保证的。一个正常的程序执行需要包含以下要素。

1. performing any necessary security checks if the effective user ID is not the same as the real user ID.
2. initialize the threading subsystem.
3. registering the *rtld_fini* to release resources when this dynamic shared object exits (or is unloaded).
4. registering the *fini* handler to run at program exit.
5. calling the initializer function *(*init)()*.
6. calling *main()* with appropriate arguments.
7. calling *exit()* with the return value from *main()*.

> 具体的初始化和结束调用路径异常复杂，在此不多赘述，放一张图有待进一步研究。

![](https://s3.ax1x.com/2020/11/15/DFemef.png)

*图片来源: https://luomuxiaoxiao.com/?p=516*

---

然后介绍完了主函数的初始化和返回，我们需要知道保证上述的两个函数在哪个动态链接库中，这就涉及到了 *crt1.o, crti.o, crtbegin.o, crtend.o, crtn.o* 这几个库
*参考： https://blog.csdn.net/farmwang/article/details/73195951*

> **crt是c runtime 的缩写**,用于执行进入main之前的初始化和退出main之后的扫尾工作。

| 目标文件 | crt1.o| crti.o| crtbegin.o| crtend.o | crtn.o |
| :---: | :---: | :---: | :---: | :---: | :---: |
| 作用 | 启动|初始化|构造|析构|结束 |

> 在标准的linux平台下,link的顺序是

	ld crt1.o crti.o [user_objects] [system_libraries] crtn.o

所以我们按照顺序进行链接有如下命令

```makefile
# 链接指令
ld -dynamic-linker /lib64/ld-linux-x86-64.so.2 -o main /usr/lib/x86_64-linux-gnu/crt1.o main.o some.o -lc /usr/lib/x86_64-linux-gnu/crti.o /usr/lib/x86_64-linux-gnu/crtn.o
```

> 成功的手动完成了程序的链接工作。

以上艰苦的工作告诉我们，不要轻易尝试手动链接，除非你知道你在干什么。平时好好用 gcc。

- 动态链接器⼀个操作系统中只需要⼀个吗？为什么？
> 一般来说只要有一个支持的动态链接器即可，完成程序的动态链接工作。 但linux中可能有两个 ld 的版本
1. ld.so针对a.out格式的二进制可执行文件
2. ld-linux.so针对ELF格式的二进制可执行文件

a.out是旧版类Unix系统中用于执行档、目的码和后来系统中的函数库的一种文件格式，该版本的链接器仍被保留用以向前支持。

---

### Task5 - 运行时打桩

Task5 是一个典型的运行时打桩的 task，给了我们编译好的login程序，让我们改变其行为，让它能够输出 login_success.

> 阅读源码，我们可以发现，login将我们输入的字符串做了hash，与已有某不明hash值，进行比较，如果相等则登陆成功。根据本 lab 的内容主要是讲 make 和链接的，让我们碰撞这个hash显然不是本意。那另一种方法就是通过运行时打桩，改变标准库的链接方式，让strcmp 链接到我们自己写的 strcmp 版本上去，从而我们可以控制其返回值让其返回0使得登陆成功。

*首先要注意 strcmp 并非只有判断密码时的一次唯一调用，在别处还有调用，所以我们要保证 strcmp 函数的基本功能正确*
```cpp
// mystrcmp.c
#define _GNU_SOURCE
#include <stdio.h>
#include <dlfcn.h>

int strcmp(const char *lhs, const char *rhs)
{
    int(*strcmpp)(const char *lhs, const char *rhs);
    strcmpp = dlsym(RTLD_NEXT, "strcmp");

    char tmp[] = "3983709877683599140";
    if(strcmpp(tmp, rhs) == 0)
        return 0;
    return strcmpp(lhs, rhs);
}
```

> 我们直接写一个自己的 strcmp 版本，用 dlsym 可以获取在运行时 strcmp 函数的指针 *（不这样做也可以，可以直接自己重写一遍strcmp程序）*，相当于可以直接使用 **真实的标准库中的 strcmp**，然后我们稍微改写一下，让它和我们的hash值比较的时候直接返回0，能够让我们不管输入什么密码都能够 login_success。

接下来我们先将我们写的strcmp编译成动态链接库

```shell
gcc -shared -fpic -o mystrcmp.so mystrcmp.c -ldl
```

> 然后使用 **LD_PRELOAD="./mystrcmp.so"** 指令，从而在 strcmp 动态链接到标准库之前让其优先匹配我们写的动态链接库中的 strcmp 符号。

这里需要注意由于是要直接运行 ./login, 所以我们需要把LD_PRELOAD的效果全局化，也既在前面加上 export 标记。

```shell
export LD_PRELOAD="./mystrcmp.so"
```

最后注意不要忘记卸载全局 preload， 不然之后所有程序都 preload 这个strcmp。

```shell
export LD_PRELOAD=NULL
```