# proj43-support-riscv-in-gvisor

在 2018 年由 Google 开源的 gVisor，是能为容器进程提供多层防御的用户态操作系统内核。
作为一个容器运行时，gVisor 提供了云原生安全（防止容器逃逸）、弹性资源的能力；作为一个应用态的操作系统内核，gVisor 又走在探索未来操作系统可能性的最前沿。
当面向未来的 gVisor 与新兴的处理器架构 RISC-V 相结合，我们就处在寒武纪大爆发的中心。

当前项目实现源码：

* https://github.com/google/gvisor

### 所属赛道

2025 全国大学生操作系统比赛的“OS 功能设计”赛道

### 参赛要求
- 如学生参加了多个项目，参赛学生选择一个自己参加的项目参与评奖
- 请遵循“2025 全国大学生操作系统比赛”的章程和技术方案要求

### 项目导师

陆斌
- github https://github.com/lubinszARM
- email lubin.lu@antgroup.com

苏航
- github https://github.com/DarcySail
- email darcy.sh@antgroup.com

### 难度

中等

### 特征

在 RISC-V 架构的芯片上，运行 ptrace 模式及 kvm 模式 的 gvisor。

### 文档

- [gVisor doc](https://gvisor.dev/)

### License

- [Apache-2.0](https://opensource.org/licenses/Apache-2.0)

## 预期目标

### 注意：下面的内容是建议内容，不要求必须全部完成。选择本项目的同学也可与导师联系，提出自己的新想法，如导师认可，可加入预期目标。

前置知识： linux 进程切换，异常处理，系统调用，信号处理，文件系统，内存管理，ABI，ptrace sysemu，golang 编程，bazel编译工具

### 1. 添加 linux 在 risc-v 芯片上对 ptrace 的支持
除了虚拟化技术之外，gvisor 还可以通过 ptrace-sysemu 对系统调用和异常（比如缺页中断，NMI，内存对齐等）进行捕获，提供了完整的用户态内存管理，文件系统和系统调用的实现。
而 linux 当前对 risc-v 芯片的 ptrace 功能支持尚不够完善，第一步我们需要调研 gvisor 中使用了 ptrace 的哪些功能，这些功能是否必须。如果不是必须，是否有绕过手段；如果必须，则需在 linux 中为risv-v提交 ptrace-sysemu的相关代码。

### 2. （可选） 添加 golang 对 risc-v 汇编级别的支持
在 golang 编译器中实现 gvisor 中会用到的汇编指令的支持。此步骤如果不实现的话，只能通过编写 risc-v 机器码来实现相关功能。
在 gvisor 项目的汇编文件中，以 WORD 关键词开头的代码行，就是由于当时 golang 不支持这些汇编指令，通过机器码（而不是汇编指令）的方式来实现所需的功能。
2.1 （可选）添加 bazel 对 risc-v 的支持。
bazel 为 gvisor 官方推荐的编译工具，但同样对 risc-v 这样一个新生指令集的支持不尚完善，需要我们做一些兼容性的操作。
这个子问题的解法可参考：
> https://github.com/bazelbuild/rules_go/pull/1574

### 3. 添加 gvisor 的 ptrace 平台对 risc-v 的支持
gvisor 中，ptrace platform 对指令集有感知的这一块代码主要在 arch/vdso/time/signal 等 package，此类以".s"结尾的汇编文件大约有10+个，需要我们针对risc-v平台进行重写。

### 4. 添加 gvisor 的 kvm 平台对 risc-v 的支持
有别于qemu+linux guest kernel这种系统级别的虚拟化方案。gVisor kvm平台提供了一个典型的进程虚拟化的实现方式。
在enable gVisor kvm平台之前，需要了解进程虚拟化的设计原理。
kvm平台主要开发任务有2部分：guest kernel 和虚拟化平台。
guest kernel部分，我们需要实现很薄的一层内核，此内核需要实现cpu初始化，页表mmu初始化、tlb，异常处理等。主要代码在ring0文件夹。
虚拟化部分的开发任务包括内存虚拟化和cpu虚拟化。主要代码在kvm文件夹。
