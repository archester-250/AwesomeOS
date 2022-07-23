# 什么是可执行文件

ABI(Application Binary Interface)：应用程序二进制接口

状态机execve重置后的状态包括寄存器和内存(地址空间)两个部分

可执行文件是一个描述了状态机

- 初始状态
- 迁移

的数据结构

execve决定了一个文件能否正常执行。

- 在可执行文件中一行的开头加上#!，后面加上可执行文件的路径合成参数可以execve这段内容

## 调试与栈展开(stack unwinding)

## 重定向relocation

```assembly
0000000000000000 <main>:
   0:   f3 0f 1e fa         endbr64 
   4:   48 83 ec 08         sub    $0x8,%rsp
   8:   31 c0               xor    %eax,%eax
   a:   e8 00 00 00 00      callq  ????????# 调用hello函数，重定位前不知道其位置
   f:   31 c0               xor    %eax,%eax
  11:   48 83 c4 08         add    $0x8,%rsp
  15:   c3                  retq   
```

重定向前main.o的反汇编后的代码

```c
#include <stdio.h>
#include <stdint.h>
#incldue <assert.h>

int main();

void hello() {
    char * p = (char *)main + 0xa + 1;
    int32_t offset = *(int32_t *)p;
    assert((char *)main + 0xf + offset == (char *)hello)
}
```

(验证重定位后偏移量改变至正常位置的代码)

## DIY二进制文件格式

```c
struct executable {
  uint32_t entry;
  struct segment *segments;
  struct reloc *relocs;
  struct symbol *symbols;
};
struct segment { uint32_t flags, size; char data[0]; }
struct reloc   { uint32_t S, A, P; char name[32]; };
struct symbol  { uint32_t off; char name[32]; };
```

- segment
  - flags：指明文件的读写、执行权限
  - size：大小
  - data：数据
- reloc重定位
  - S,A,P
  - name：表明是哪个名字要重定位

### 缺陷

- 名字应该集中存储(32个容易出现空缺)-- 真实的ELF的设计