ELF
========================================

ELF代表Executable and Linkable Format。它是一种对可执行文件、目标文件和
库使用的文件格式。它在Linux下成为标准格式已经很长时间，代替了早年的a.out格式。
ELF一个特别的优点在于，同一文件格式可以用于内核支持的几乎所有体系结构上。
这不仅简化了用户空间工具程序的创建，也简化了内核自身的程序设计。例如，
在必须为可执行文件生成装载例程时。但是文件格式相同并不意味着不同系统上的
程序之间存在二进制兼容性，例如，FreeBSD和Linux都使用ELF作为二进制格式。
尽管二者在文件中组织数据的方式相同，但在系统调用机制以及系统调用的语义方面，
仍然有差别。这也是在没有中间仿真层的情况下，FreeBSD程序不能在Linux下运行的
原因（反过来，同样如此）。有一点是可以理解的，二进制程序不能在不同体系结构
间交换（例如，为Al-pha CPU编译的Linux二进制程序不能在Sparc Linux上执行），
因为底层的体系结构是完全不同的。但由于ELF的存在，对所有体系结构而言，程序
本身的相关信息以及程序的各个部分在二进制文件中编码的方式都是相同的。Linux
不仅将ELF用于用户空间应用程序和库，还用于构建模块。内核本身也是ELF格式。

Layout
----------------------------------------

https://github.com/novelinux/linux-4.x.y/tree/master/fs/binfmt_elf.c/res/elf.jpg

### 链接视图

首先左边部分，它是以链接视图来看待elf文件的，从左边可以看出，
包含了一个ELF头部，它描绘了整个文件的组织结构。它还包括很多
节区（section）。这些节有的是系统定义好的，有些是用户在文件
在通过.section命令自定义的，链接器会将多个输入目标文件中的
相同的节合并。节区部分包含链接视图的大量信息：指令、数据、
符号表、重定位信息等等。除此之外，还包含程序头部表（可选）和
节区头部表，程序头部表，告诉系统如何创建进程映像。用来构造进程
映像的目标文件必须具有程序头部表，可重定位文件不需要这个表。
而节区头部表（Section Heade Table）包含了描述文件节区的信息，
每个节区在表中都有一项，每一项给出诸如节区名称、节区大小这类信息。
用于链接的目标文件必须包含节区头部表，其他目标文件可以有，也可以没有这个表。

**注意**: 尽管图中显示的各个组成部分是有顺序的，实际上除了ELF头部表以外，
其他节区和段都没有规定的顺序。

### 执行视图

右半图是以程序执行视图来看待的，与左边对应，多了一个段（segment）的概念，
编译器在生成目标文件时，通常使用从零开始的相对地址，而在链接过程中，链接器
从一个指定的地址开始，根据输入目标文件的顺序，以段（segment）为单位将它们拼
装起来。其中每个段可以包括很多个节（section）。

elf_format
----------------------------------------

https://github.com/novelinux/linux-4.x.y/tree/master/fs/binfmt_elf.c/elf_format.md

Data Structure
----------------------------------------

### Data Types

https://github.com/novelinux/linux-4.x.y/blob/master/include/uapi/linux/elf.h/elf_base_types.md

### ELF Header

对ELF格式中的各种头部结构，32位和64位系统需要分别定义数据结构。

#### struct elf32_hdr

https://github.com/novelinux/linux-4.x.y/blob/master/include/uapi/linux/elf.h/struct_elf32_hdr.md

#### struct elf64_hdr

https://github.com/novelinux/linux-4.x.y/blob/master/include/uapi/linux/elf.h/struct_elf64_hdr.md

### Program Header

#### struct elf32_phdr

https://github.com/novelinux/linux-4.x.y/blob/master/include/uapi/linux/elf.h/struct_elf32_phdr.md

#### struct elf64_phdr

https://github.com/novelinux/linux-4.x.y/blob/master/include/uapi/linux/elf.h/struct_elf64_phdr.md

### Section Header

节头表通过数组实现，每个数组项包含一节的信息。各个节构成了
程序头表中定义的各段的内容。下列数据结构表示一个节：

#### struct elf32_shdr

https://github.com/novelinux/linux-4.x.y/blob/master/include/uapi/linux/elf.h/struct_elf32_shdr.md

#### struct elf64_shdr

https://github.com/novelinux/linux-4.x.y/blob/master/include/uapi/linux/elf.h/struct_elf64_shdr.md

Samples
----------------------------------------

https://github.com/novelinux/linux-4.x.y/blob/master/samples/fs/test_elf/README.md
