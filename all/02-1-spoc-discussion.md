# lec 3 SPOC Discussion

## **提前准备**
（请在上课前完成）


 - 完成lec3的视频学习和提交对应的在线练习
 - git pull ucore_os_lab, v9_cpu, os_course_spoc_exercises  　in github repos。这样可以在本机上完成课堂练习。
 - 仔细观察自己使用的计算机的启动过程和linux/ucore操作系统运行后的情况。搜索“80386　开机　启动”
 - 了解控制流，异常控制流，函数调用,中断，异常(故障)，系统调用（陷阱）,切换，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在linux, ucore, v9-cpu中的os*.c中是如何具体体现的。
 - 思考为什么操作系统需要处理中断，异常，系统调用。这些是必须要有的吗？有哪些好处？有哪些不好的地方？
 - 了解在PC机上有啥中断和异常。搜索“80386　中断　异常”
 - 安装好ucore实验环境，能够编译运行lab8的answer
 - 了解Linux和ucore有哪些系统调用。搜索“linux 系统调用", 搜索lab8中的syscall关键字相关内容。在linux下执行命令: ```man syscalls```
 - 会使用linux中的命令:objdump，nm，file, strace，man, 了解这些命令的用途。
 - 了解如何OS是如何实现中断，异常，或系统调用的。会使用v9-cpu的dis,xc, xem命令（包括启动参数），分析v9-cpu中的os0.c, os2.c，了解与异常，中断，系统调用相关的os设计实现。阅读v9-cpu中的cpu.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
 - 在piazza上就lec3学习中不理解问题进行提问。

## 第三讲 启动、中断、异常和系统调用-思考题

## 3.1 BIOS
-  x86中BIOS从磁盘读入的第一个扇区是是什么内容？为什么没有直接读入操作系统内核映像？

BIOS读取硬盘的0磁头、0柱面、1扇区的内容，然后把控制权交给这里面的MBR(Main Boot Record)。
因为文件系统并没有建立，BIOS也不知道硬盘里存放的是什么，所以BIOS是无法直接启动操作系统。另外一个硬盘可以有多个分区，每个分区都有可能包括一个不同的操作系统，BIOS也无从判断应该从哪个分区启动。

- 比较UEFI和BIOS的区别。

UEFI启动对比BIOS启动的优势有三点：
(1) 安全性更强：UEFI启动需要一个独立的分区，它将系统启动文件和操作系统本身隔离，可以更好的保护系统的启动。
(2) 启动配置更灵活：EFI启动和GRUB启动类似，在启动的时候可以调用EFIShell，在此可以加载指定硬件驱动，选择启动文件。比如默认启动失败，在EFIShell加载U盘上的启动文件继续启动系统。
(3) 支持容量更大:传统的BIOS启动由于MBR的限制，默认是无法引导超过2TB以上的硬盘的。随着硬盘价格的不断走低，2TB以上的硬盘会逐渐普及，因此UEFI启动也是今后主流的启动方式。

- 理解rcore中的Berkeley BootLoader (BBL)的功能。

## 3.2 系统启动流程

- x86中分区引导扇区的结束标志是什么？

0x55AA

- x86中在UEFI中的可信启动有什么作用？

通过启动前的数字签名检查来保证启动介质的安全性。

- RV中BBL的启动过程大致包括哪些内容？



## 3.3 中断、异常和系统调用比较
- 什么是中断、异常和系统调用？

中断：外部意外的响应。
异常：指令执行意外的响应。
系统调用：系统调用指令的响应。

- 中断、异常和系统调用的处理流程有什么异同？

源头：中断来源于外部，异常是內存程序执行所引起的，系统调用是操作系统提供给应用程序的接口。
响应方式：中断和异常由硬件响应，系统调用则由操作系统响应。
处理机制：各自具有处理机制。

- 以ucore/rcore lab8的answer为例，ucore的系统调用有哪些？大致的功能分类有哪些？

## 3.4 linux系统调用分析
- 通过分析[lab1_ex0](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex0.md)了解Linux应用的系统调用编写和含义。(仅实践，不用回答)
- 通过调试[lab1_ex1](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex1.md)了解Linux应用的系统调用执行过程。(仅实践，不用回答)


## 3.5 ucore/rcore系统调用分析 （扩展练习，可选）
- 基于实验八的代码分析ucore的系统调用实现，说明指定系统调用的参数和返回值的传递方式和存放位置信息，以及内核中的系统调用功能实现函数。

在 ucore 中，执行系统调用前，需要将系统调用的参数出存在寄存器中。eax 表示系统调用类型，其余参数依次存在 edx, ecx, ebx, edi, esi 中。
    ...
    int num = tf->tf_regs.reg_eax;
    ... 
    arg[0] = tf->tf_regs.reg_edx; 
    arg[1] = tf->tf_regs.reg_ecx;
    arg[2] = tf->tf_regs.reg_ebx;
    arg[3] = tf->tf_regs.reg_edi;
    arg[4] = tf->tf_regs.reg_esi;
    ...
以getpid为例，分析ucore的系统调用中返回结果的传递代码。
syscalls 是保存了各种系统调用的函数指针，进入 sys_getpid，直接返回当前进程的 pid。
    sys_getpid(uint32_t arg[]) {
        return current->pid;
    }
而各类系统调用返回结果统一存在eax中。
    tf->tf_regs.reg_eax = syscalls[num](arg);

- 以ucore/rcore lab8的answer为例，分析ucore 应用的系统调用编写和含义。

syscall(int num, ...) {
    va_list ap;
    va_start(ap, num);
    uint32_t a[MAX_ARGS];
    int i, ret;
    for (i = 0; i < MAX_ARGS; i ++) {
        a[i] = va_arg(ap, uint32_t);
    }
    va_end(ap);

    asm volatile (
        "int %1;"
        : "=a" (ret)
        : "i" (T_SYSCALL),
          "a" (num),
          "d" (a[0]),
          "c" (a[1]),
          "b" (a[2]),
          "D" (a[3]),
          "S" (a[4])
        : "cc", "memory");
    return ret;
}

这段代码是用户态的syscall函数，其中num参数为系统调用号，该函数将参数准备好后，通过 SYSCALL 汇编指令进行系统调用，进入内核态，返回值放在 eax 寄存器，传入参数通过 eax ~ esi 依次传递进去。在内核态中，首先进入 trap() 函数，然后调用 trap_dispatch() 进入中断分发，当系统得知该中断为系统调用后，OS调用如下的 syscall 函数

oid
syscall(void) {
    struct trapframe *tf = current->tf;
    uint32_t arg[5];
    int num = tf->tf_regs.reg_eax;
    if (num >= 0 && num < NUM_SYSCALLS) {
        if (syscalls[num] != NULL) {
            arg[0] = tf->tf_regs.reg_edx;
            arg[1] = tf->tf_regs.reg_ecx;
            arg[2] = tf->tf_regs.reg_ebx;
            arg[3] = tf->tf_regs.reg_edi;
            arg[4] = tf->tf_regs.reg_esi;
            tf->tf_regs.reg_eax = syscalls[num](arg);
            return ;
        }
    }
    print_trapframe(tf);
    panic("undefined syscall %d, pid = %d, name = %s.\n",
            num, current->pid, current->name);
}
该函数得到系统调用号 num = tf->tf_regs.reg_eax;，通过计算快速跳转到相应的 sys_ 开头的函数，最终在内核态中，完成系统调用所需要的功能。

- 以ucore/rcore lab8的answer为例，尝试修改并运行ucore OS kernel代码，使其具有类似Linux应用工具`strace`的功能，即能够显示出应用程序发出的系统调用，从而可以分析ucore应用的系统调用执行过程。

利用 trap.c 的 trap_in_kernel() 函数判断是否是用户态的系统调用，调用 syscall() 时传入此参数
    case T_SYSCALL:
        syscall(trap_in_kernel(tf));
        break;
        
更改 syscall() 的函数原型为
    void syscall(bool);
    
之后在 syscall(bool) 中加入输出即可
    int num = tf->tf_regs.reg_eax;
    if (num >= 0 && num < NUM_SYSCALLS) {
        if (syscalls[num] != NULL) {
            arg[0] = tf->tf_regs.reg_edx;
            arg[1] = tf->tf_regs.reg_ecx;
            arg[2] = tf->tf_regs.reg_ebx;
            arg[3] = tf->tf_regs.reg_edi;
            arg[4] = tf->tf_regs.reg_esi;

        if (!in_kernel) {
            cprintf("SYSCALL: %d\n", num);
        }
            tf->tf_regs.reg_eax = syscalls[num](arg);
            return ;
        }
    }
    
下面是qemu运行的输出结果片段，可以看出在用户程序输出前调用了 SYS_open，输出 sh is running 的过程中调用了 SYS_write
Iter 1, No.0 philosopher_sema is thinking
kernel_execve: pid = 2, name = "sh".
SYSCALL: 100
SYSCALL: 100
SYSCALL: 103
....

 
## 3.6 请分析函数调用和系统调用的区别
- 系统调用与函数调用的区别是什么？

指令不同，特权级不同，堆栈切换。

- 通过分析x86中函数调用规范以及`int`、`iret`、`call`和`ret`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？

SS:SP压栈

- 通过分析RV中函数调用规范以及`ecall`、`eret`、`jal`和`jalr`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？


## 课堂实践 （在课堂上根据老师安排完成，课后不用做）
### 练习一
通过静态代码分析，举例描述ucore/rcore键盘输入中断的响应过程。

### 练习二
通过静态代码分析，举例描述ucore/rcore系统调用过程，及调用参数和返回值的传递方法。
