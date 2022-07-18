# 并发控制：互斥

## 自旋锁

- `xchg`：原子级别的交换

利用`xchg`实现自旋锁：

初始只有一把钥匙，多个线程同时执行`xchg`操作，谁最后手上有钥匙谁就拿到控制权。

```c
void lock() {while(xchg(&locked, 1));}
void unlock() {xchg(&locked, 0);}
```

通过一个原子的lock指令可以实现锁

在L1 cache要保证一致性，即在一个L1 cache中有了一个共享性内存变量，其余L1 cache就不能存在该变量。

- MESI
- M(Modified),脏值
- E(Exclusive),独占访问
- S(Shared),只读共享
- I(Invalid),不拥有cache line



- Compare-and-Swap的LR/SC实现
- 先读，如果锁处于解锁状态才会写入。

```c
int cas(int * addr, int cmp_val, int new_val)
{
    int old_val = *addr;
    if(old_val == cmp_val)
    {
        *addr = new_val;
        return 0;
    }
    return 1;
}
```

使用场景：

- 临界区几乎不拥堵
- 持有自旋锁时禁止执行流切换
- 操作系统内核的并发数据结构（短临界区）

## 互斥锁

- 自旋锁的缺陷
  - 触发处理器间的缓存同步，延迟增加
  - 得不到钥匙的线程会处于空转状态，利用率太低
  - 获得自旋锁的线程可能会被操作系统换到无锁的线程，造成浪费

实现过程：

- 先创建的线程可以获得标记，执行
- 后创建的线程进入等待队列，执行线程切换（线程阻塞，在等待队列中线程会休眠）
- 先创建的线程执行结束后，如果等待队列非空则取一个线程并允许其执行
- 等待队列为空则完成过程
- 要确保处理线程的过程是原子级别的

特性：

- 更快的slow path：上锁失败不再占用CPU自旋
- 更慢的fast path：即使上锁成功也需要进出内核（`syscall`）

## `Futex`

- Fast path：一条原子指令，上锁成功后立即返回
- Slow path：上锁失败则执行系统调用睡眠该进程

