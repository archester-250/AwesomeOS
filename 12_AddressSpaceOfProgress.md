# 进程的地址空间

`pmap` 进程号：查看该进程号对应的地址空间

- r：可读
- w：可写
- x：可执行

`vdso`：无需陷入内核的系统调用（支持获取时间戳等操作）

## 地址空间管理：`mmap`

```c
// 映射
void *mmap(void *addr, size_t length, int prot, int flags,
           int fd, off_t offset);
int munmap(void *addr, size_t length);

// 修改映射权限
int mprotect(void *addr, size_t length, int prot);
```



- 可以在状态上增加、删除、修改一段可以访问的内存
- 可以把文件映射到进程地址空间

- 地址空间可以实现进程隔离，一个进程的指针不能指向另外一个进程

## 代码注入 (Hooking)

我们可以改内存，也可以改代码！

The Light Side

- “软件热补丁” [`dsu.c`](http://jyywiki.cn/pages/OS/2022/demos/dsu.c) (mprotect)
- [Ksplice: Automatic rebootless Kernel updates](https://dl.acm.org/doi/10.1145/1519065.1519085) (EuroSys'09)