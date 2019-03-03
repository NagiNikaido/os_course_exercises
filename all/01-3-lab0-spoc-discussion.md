# lec2：lab0 SPOC思考题

## **提前准备**
（请在上课前完成，option）

- 完成lec2的视频学习
- git pull ucore_os_lab, os_tutorial_lab, os_course_exercises  in github repos。这样可以在本机上完成课堂练习。
- 了解代码段，数据段，执行文件，执行文件格式，堆，栈，控制流，函数调用,函数参数传递，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在不同操作系统（如linux, ucore,etc.)与不同硬件（如 x86, riscv, v9-cpu,etc.)中是如何相互配合来体现的。
- 安装好ucore实验环境，能够编译运行ucore labs中的源码。
- 会使用linux中的shell命令:objdump，nm，file, strace，gdb等，了解这些命令的用途。
- 会编译，运行，使用v9-cpu的dis,xc, xem命令（包括启动参数），阅读v9-cpu中的v9\-computer.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
- 了解基于v9-cpu的执行文件的格式和内容，以及它是如何加载到v9-cpu的内存中的。
- 在piazza上就学习中不理解问题进行提问。

---

## 思考题

- 你理解的对于类似ucore这样需要进程/虚存/文件系统的操作系统，在硬件设计上至少需要有哪些直接的支持？至少应该提供哪些功能的特权指令？

直接支持：时钟中断、I/O、内存管理等。

特权指令：I/O设备控制、程序状态控制、对特殊寄存器操作的指令，还有对页表、TLB操作的指令等。

- 你理解的x86的实模式和保护模式有什么区别？物理地址、线性地址、逻辑地址的含义分别是什么？

实模式：最大寻址空间1MB，无保护机制。

保护模式：最大寻址空间4GB，有保护机制。

物理地址：访问物理设备所用的实际地址，一台计算机只有一个物理地址空间。

线性地址：逻辑地址向物理地址转化过程中的中间层，通过段机制形成的地址空间。

逻辑地址：访问指令使用的地址。

- 你理解的risc-v的特权模式有什么区别？不同 模式在地址访问方面有何特征？

RV的特权模式分为机器模式、监督模式、Hypervisor模式和用户模式。其中Hypervisor模式用于支持虚拟机监视器，其余三者的区别在于
地址访问的方式、系统安全性和能使用的特权指令。

- 理解ucore中list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）

- 对于如下的代码段，请说明":"后面的数字是什么含义
```
 /* Gate descriptors for interrupts and traps */
 struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
 };
```

变量占用的二进制位数。

- 对于如下的代码段，

```
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
}
```
如果在其他代码段中有如下语句，
```
unsigned intr;
intr=8;
SETGATE(intr, 1,2,3,0);
```
请问执行上述指令后， intr的值是多少？

0x20003

### 课堂实践练习

#### 练习一

1. 请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。

```
    /* libs/x86.h */
    
    static inline void *
    __memset(void *s, char c, size_t n) {
        int d0, d1;
        asm volatile (
                "rep; stosb;"
                : "=&c" (d0), "=&D" (d1)
                : "0" (n), "a" (c), "1" (s)
                : "memory");
        return s;
    }
```

这里是内嵌汇编，作用是把s开始的n个字节全设置成c。

2. (option)请在rcore中找一段你认为难度适当的RV汇编代码，尝试解释其含义。

#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore或rcore中宏定义的用途，并举例描述其含义。

```
    /* kern/mm/mmu.h */
    
    #define SEG(type, base, lim, dpl)                        \
         (struct segdesc){                                    \
             ((lim) >> 12) & 0xffff, (base) & 0xffff,        \
             ((base) >> 16) & 0xff, type, 1, dpl, 1,            \
             (unsigned)(lim) >> 28, 0, 0, 1, 1,                \
             (unsigned) (base) >> 24                            \
         }
```

这段宏定义是根据段类别、基址、粒度和特权等级生成对应段的descriptor。这是会反复用到的代码，并且
由于其中有很多magic number，打包成宏定义可以让代码更为明晰。



#### reference
 - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)
 - [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)
 - [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)
 - [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)
 - [IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)
