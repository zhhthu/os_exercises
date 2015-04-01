# lab0 SPOC思考题

## 个人思考题

---

能否读懂ucore中的AT&T格式的X86-32汇编语言？请列出你不理解的汇编语言。
- 能够读懂

虽然学过计算机原理和x86汇编（根据THU-CS的课程设置），但对ucore中涉及的哪些硬件设计或功能细节不够了解？
- 与串口等外设进行的IO操作等


哪些困难（请分优先级）会阻碍你自主完成lab实验？
- 实验环境不兼容；实现的时候对部分代码的理解不够

>   

如何把一个在gdb中或执行过程中出现的物理/线性地址与你写的代码源码位置对应起来？
- 使用info line命令获取某地址对应的代码源码的位置


了解函数调用栈对lab实验有何帮助？
- 了解函数实现细节，更加方便的进行调试

你希望从lab中学到什么知识？
- 操作系统实现的具体细节以及实现框架等

>   

---

## 小组讨论题

---

搭建好实验环境，请描述碰到的困难和解决的过程。
- Mac OS上一开始搭建失败，改用Windows就成功了。

熟悉基本的git命令行操作命令，从github上的 http://www.github.com/chyyuu/ucore_lab 下载ucore lab实验
- 已完成

尝试用qemu+gdb（or ECLIPSE-CDT）调试lab1
- 已完成

对于如下的代码段，请说明”：“后面的数字是什么含义
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

- 每个域在结构中所占位数

对于如下的代码段，
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
SETGATE(intr, 0,1,2,3);
```
请问执行上述指令后， intr的值是多少？

- 65538

> https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab0/lab0_ex3.c

请分析 [list.h](https://github.com/chyyuu/ucore_lab/blob/master/labcodes/lab2/libs/list.h)内容中大致的含义，并能include这个文件，利用其结构和功能编写一个数据结构链表操作的小C程序
- 如下为小组实现的链表程序：
```
#include <list.h>
list_entry a,b,c,d,e;

int main()
{
    list_init(a);
    list_add_before(a, b);
    list_add_before(b, c);
    lit_del(a);
    list_add_after(b,d);
    return 0;
}
```

> 

---

## 开放思考题

---

是否愿意挑战大实验（大实验内容来源于你的想法或老师列好的题目，需要与老师协商确定，需完成基本lab，但可不参加闭卷考试），如果有，可直接给老师email或课后面谈。
- 不愿意，希望把更多的精力用在完成毕设上

>  

---

