---
layout: post
title: 从汇编上理解C中main函数argc argv参数
subtitle: C++
author: Kingtous
date: 2020-10-06 09:08:29
header-img: img/home-bg-geek.jpg
catalog: True
tags:
- C++
---

> 一直没探究一下argc和argv的原理，这次记录一下

使用macOS 10.15 SDK GCC 64位测试

### 来一段简单的C++程序

为了尽可能的调用argc、argv，设计下列代码

```c++
int main(int argc, char const *argv[])
{
	int total_params = argc + 1;
	for (int i = 0; i < argc; ++i)
	{
		const char * arr = argv[i];
	}
	return total_params;
}
```

### 反汇编

使用`g++ -O0 -fverbose-asm -S`进行反汇编

参数说明：

- `-O0`取消优化
- `-fverbose-asm`使用C++变量名为汇编变量，不重命名
- `-S`只输出汇编文件

> 以下以一段汇编，加执行完汇编后的栈情况进行演示

```assembly
pushq	%rbp
.cfi_def_cfa_offset 16
.cfi_offset %rbp, -16
movq	%rsp, %rbp
.cfi_def_cfa_register %rbp
```

| -24  | -20  | -16  | -12  | -8   | -4   | rbp              |
| ---- | ---- | ---- | ---- | ---- | ---- | ---------------- |
| -    | -    | -    | -    | -    | -    | 当前main数据地址 |

 此时将%rsp导入%rbp，原有%rbp导入栈

```c++
int total_params = argc + 1;
```

```assembly
movl	$0, -4(%rbp)
movl	%edi, -8(%rbp)
movq	%rsi, -16(%rbp)
movl	-8(%rbp), %edi
addl	$1, %edi
movl	%edi, -20(%rbp)
```

| -24  | -20                  | -16         | -12         | -8        | -4   | rbp              |
| ---- | -------------------- | ----------- | ----------- | --------- | ---- | ---------------- |
| -    | total_params(argc+1) | argv基地址1 | argv基地址2 | edi(argc) | 0    | 当前main数据地址 |

- -4(%rbp)为什么置0？（待解决）
- 电脑为64位，故基地址为8字节

```c++
for (int i = 0; i < argc; ++i)
{
  const char * arr = argv[i];
}
```

```assembly
LBB0_1: #i<argc                                ## =>This Inner Loop Header: Depth=1
movl	-24(%rbp), %eax
cmpl	-8(%rbp), %eax
jge	LBB0_4
## %bb.2: const char * arr = argv[i];   ##   in Loop: Header=BB0_1 Depth=1
movq	-16(%rbp), %rax
movslq	-24(%rbp), %rcx # rcx为i的值，即偏移量,一个双字符号扩展后送到一个四字地址
movq	(%rax,%rcx,8), %rax # 取出argv[i]的地址
movq	%rax, -32(%rbp)
## %bb.3: i++                           ##   in Loop: Header=BB0_1 Depth=1
movl	-24(%rbp), %eax
addl	$1, %eax
movl	%eax, -24(%rbp)
jmp	LBB0_1
LBB0_4:
movl	-20(%rbp), %eax
popq	%rbp
retq
.cfi_endproc
```

| -24         | -20                  | -16         | -12         | -8        | -4   | rbp              |
| ----------- | -------------------- | ----------- | ----------- | --------- | ---- | ---------------- |
| i(0~argc-1) | total_params(argc+1) | argv基地址1 | argv基地址2 | edi(argc) | 0    | 当前main数据地址 |

- `char*`为地址指针，64位系统下的`char*`指针为64位，8字节

### 总结

在macOS 64位下，

int argc的值保存在`%edi`上，之后转移到`%rbp-8`的地址上

char\*\* argv的值基地址保存在`%rsi`上，之后转移到`%rbp-16`的地址上，argv数组中的地址指针（char\*）以一个数组的形式保存，地址为`(%rsi,%rcx,8)`（8为64位系统下，32系统下位4）

### 附录