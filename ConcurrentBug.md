# 并发bug及应对

## 防御性编程

assert：断言

## 死锁

### 必要条件

- 互斥：一个资源每次只能被一个进程使用
- 请求与保持：一个进程请求资源阻塞时，不释放已获得的资源
- 不剥夺：进程已获得的资源不能强行剥夺
- 循环等待：若干进程之间形成头尾相接的循环等待资源关系

### 避免死锁

#### AA-型死锁

```c
void os_run() {
  spin_lock(&list_lock);
  spin_lock(&xxx);
  spin_unlock(&xxx); // ---------+  此处发生了中断
}                          //    |
                           //    |
void on_interrupt() {      //    |
  spin_lock(&list_lock);   // <--+
  spin_unlock(&list_lock);
}
```

及早报告，及早修复

#### ABBA-型死锁

按照固定的顺序获得所有锁(lock-ordering)

## 数据竞争(Data race)

种类：读-写(写前写后读的数据不同)，写-写(写的顺序决定变量的值)

解决方法：上锁

大多数的并发bug

- 忘记上锁--原子性违反(Atomicity Violation,AV)

- 忘记同步--顺序违反(Order Violation,OV)

## 应对bug的方法

### Lockdep:运行时的死锁检查

记录所有观察到的上锁顺序，判断有无环出现

### ThreadSanitizer:运行时的数据竞争检查

对于发生在不同线程且至少有一个是写的x，y检查
$$
x\prec y\or y\prec x
$$

### 更多的检查：动态程序分析

运行时收集记录 

在事件发生时记录

- Lockdep::lock:/:unlock:
- ThreadSanitizer:内存访问+:lock:/:unlock:

解析记录检查问题

- Lockdep:有无环
- ThreadSanitizer：互相不连通

付出的代价：程序运行变慢但更容易找到bug

### 动态分析工具：Sanitizers

- [AddressSanitizer](https://clang.llvm.org/docs/AddressSanitizer.html) (asan); [(paper)](https://www.usenix.org/conference/atc12/technical-sessions/presentation/serebryany): 非法内存访问
  - Buffer (heap/stack/global) overflow, use-after-free, use-after-return, double-free, ...
  - Demo: [uaf.c](http://jyywiki.cn/pages/OS/2022/demos/uaf.c); [kasan](https://www.kernel.org/doc/html/latest/dev-tools/kasan.html)
- [ThreadSanitizer](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html) (tsan): 数据竞争
  - Demo: [fish.c](http://jyywiki.cn/pages/OS/2022/demos/fish.c), [sum.c](http://jyywiki.cn/pages/OS/2022/demos/sum.c), [peterson-barrier.c](http://jyywiki.cn/pages/OS/2022/demos/peterson-barrier.c); [ktsan](https://github.com/google/ktsan)
- [MemorySanitizer](https://clang.llvm.org/docs/MemorySanitizer.html) (msan): 未初始化的读取
- [UBSanitizer](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html) (ubsan): undefined behavior
  - Misaligned pointer, signed integer overflow, ...
  - Kernel 会带着 `-fwrapv` 编译

### 金丝雀：buffer overrun缓冲区溢出检查

### 编程中输出奇怪中文字符的原因

- 未初始化栈: `0xcccccccc`
- 未初始化堆: `0xcdcdcdcd`
- 对象头尾: `0xfdfdfdfd`
- 已回收内存: `0xdddddddd`

```
(b'\xcc' * 80).decode('gb2312')
```

结果：烫烫烫

