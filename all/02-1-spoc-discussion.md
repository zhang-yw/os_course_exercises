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
- x86中BIOS从磁盘读入的第一个扇区是是什么内容？为什么没有直接读入操作系统内核映像？

  ​	对待硬盘，所有的BIOS都是读取硬盘的0磁头、0柱面、1扇区的内容，然后把控制权交给这里面的MBR (Main Boot Record）。

  ​	加载操作系统是个很复杂的过程：就比如uCore，我们需要完成对ELF文件格式的解析和文件本身的读入。BIOS工作在实模式，本身访存范围只有1MB（能使用的数据只有0 ~ 0x7c00的范围），而且代码长度被限制在64KB。为了将OS读入到高地址的内存中，需要BIOS进行模式的切换。但是，如果BIOS进行了实模式到保护模式的切换，就不能实现向后兼容了。而且不同的OS的文件格式和处理方法也有差异，这会导致BIOS十分复杂。因此，让OS提供自己的启动程序是最好的选择。

- 比较UEFI和BIOS的区别。

  ​	统一可扩展固件接口 (Unified Extensible Firmware Interface, UEFI) 是一种个人电脑系统规格，用来定义操作系统与系统固件之间的软件界面，作为BIOS的替代方案。

  ​	UEFI启动对比BIOS启动的优势有三点：

  1. **安全性更强**：UEFI启动需要一个独立的分区，它将系统启动文件和操作系统本身隔离，可以更好的保护系统的启动；
  2. **启动配置更灵活**：EFI启动和GRUB启动类似，在启动的时候可以调用EFIShell，在此可以加载指定硬件驱动，选择启动文件。比如默认启动失败，在EFIShell加载U盘上的启动文件继续启动系统；
  3. **支持容量更大**：传统的BIOS启动由于MBR的限制，默认是无法引导超过2TB以上的硬盘的。随着硬盘价格的不断走低，2TB以上的硬盘会逐渐普及，因此UEFI启动也是今后主流的启动方式。

- 理解rcore中的Berkeley BootLoader (BBL)的功能。

## 3.2 系统启动流程

- x86中分区引导扇区的结束标志是什么？

  0x55AA

- x86中在UEFI中的可信启动有什么作用？

  通过启动前的数字签名检查来保证启动介质的安全性。

- RV中BBL的启动过程大致包括哪些内容？

## 3.3 中断、异常和系统调用比较
- 什么是中断、异常和系统调用？
  - 中断：外部意外的响应；
  - 异常：指令执行意外的响应；
  - 系统调用：系统调用指令的响应。
- 中断、异常和系统调用的处理流程有什么异同？
  - 相同点：都会进入异常服务例程，切换为内核态。
  - 不同点：
    - 源头不同，中断源是外部设备，异常和系统调用源是应用程序；
    - 响应方式不同，中断是异步的，异常是同步的，系统调用异步和同步都可以。
    - 处理机制不同，中断对用户程序是透明的，异常会重新执行用户指令或杀死用户进程，系统调用一般是用户程序调用的。
- 以ucore/rcore lab8的answer为例，ucore的系统调用有哪些？大致的功能分类有哪些？
  - 进程管理：包括 fork/exit/wait/exec/yield/kill/getpid/sleep
  - 文件操作：包括 open/close/read/write/seek/fstat/fsync/getcwd/getdirentry/dup
  - 内存管理：pgdir命令
  - 外设输出：putc命令

## 3.4 linux系统调用分析
- 通过分析[lab1_ex0](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex0.md)了解Linux应用的系统调用编写和含义。(仅实践，不用回答)
- 通过调试[lab1_ex1](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex1.md)了解Linux应用的系统调用执行过程。(仅实践，不用回答)


## 3.5 ucore/rcore系统调用分析 （扩展练习，可选）
- 基于实验八的代码分析ucore的系统调用实现，说明指定系统调用的参数和返回值的传递方式和存放位置信息，以及内核中的系统调用功能实现函数。

  -  在 ucore 中，执行系统调用前，需要将系统调用的参数出存在寄存器中。 eax表示系统调用类型，其余参数依次存在 edx, ecx, ebx, edi, esi 中。

  ```c
  ...
  int num = tf->tf_regs.reg_eax;
  ...
  arg[0] = tf->tf_regs.reg_edx;
  arg[1] = tf->tf_regs.reg_ecx;
  arg[2] = tf->tf_regs.reg_ebx;
  arg[3] = tf->tf_regs.reg_edi;
  arg[4] = tf->tf_regs.reg_esi;
  ...
  ```

- 以ucore/rcore lab8的answer为例，分析ucore 应用的系统调用编写和含义。

  ```c
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
  ```

  这段代码是用户态的syscall函数，其中num参数为系统调用号，该函数将参数准备好后，通过 `SYSCALL` 汇编指令进行系统调用，进入内核态，返回值放在 `eax` 寄存器，传入参数通过 `eax` ~ `esi` 依次传递进去。
  在内核态中，首先进入 `trap()` 函数，然后调用 `trap_dispatch()`进入中断分发，当系统得知该中断为系统调用后，OS调用如下的 `syscall` 函数：

  ```c
  void
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
  ```

  该函数得到系统调用号 `num = tf->tf_regs.reg_eax;`，通过计算快速跳转到相应的 `sys_` 开头的函数，最终在内核态中，完成系统调用所需要的功能。

- 以ucore/rcore lab8的answer为例，尝试修改并运行ucore OS kernel代码，使其具有类似Linux应用工具`strace`的功能，即能够显示出应用程序发出的系统调用，从而可以分析ucore应用的系统调用执行过程。

  利用 `trap.c` 的 `trap_in_kernel()` 函数判断是否是用户态的系统调用，调用 `syscall()` 时传入此参数

  ```c
      case T_SYSCALL:
          syscall(trap_in_kernel(tf));
          break;
  ```

  更改 `syscall()` 的函数原型为

  ```c
      void syscall(bool);
  ```

  之后在 `syscall(bool)` 中加入输出即可

  ```c
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
  ```

  下面是qemu运行的输出结果片段，可以看出在用户程序输出前调用了 `SYS_open`，输出 `sh is running` 的过程中调用了 `SYS_write`

  ```
  Iter 1, No.0 philosopher_sema is thinking
  kernel_execve: pid = 2, name = "sh".
  SYSCALL: 100
  SYSCALL: 100
  SYSCALL: 103
  ....
  ```


## 3.6 请分析函数调用和系统调用的区别
- 系统调用与函数调用的区别是什么？
  * 汇编指令的区别
     * 系统调用：使用INT和IRET指令
     * 函数调用：使用CALL和RET指令
  * 安全性的区别
     * 系统调用有堆栈和特权级的转换过程，函数调用没有这样的过程，系统调用相对更为安全
  * 性能的区别
     * 时间角度：系统调用比函数调用要做更多和特权级切换的工作，所以需要更多的时间开销
     * 空间角度：在一些情况下，如果函数调用采用静态编译，往往需要大量的空间开销，此时系统调用更具有优势
- 通过分析x86中函数调用规范以及`int`、`iret`、`call`和`ret`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？
- 通过分析RV中函数调用规范以及`ecall`、`eret`、`jal`和`jalr`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？


## 课堂实践 （在课堂上根据老师安排完成，课后不用做）
### 练习一
通过静态代码分析，举例描述ucore/rcore键盘输入中断的响应过程。

### 练习二
通过静态代码分析，举例描述ucore/rcore系统调用过程，及调用参数和返回值的传递方法。
