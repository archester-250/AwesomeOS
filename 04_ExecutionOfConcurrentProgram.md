# 理解并发程序执行

朴素的想法：死循环直至开锁，然后锁上自己的执行程序。

- 缺陷：还是没能避免两个进程同时满足条件，同时执行的情况。

## Peterson算法

两个进程的情况：

- 两个进程举:black_flag:
- 确认举好:black_flag:后将标签改为对方正在使用
- 看状态决定是否执行
  - 如果对方没举:black_flag:，就直接锁上执行
  - 如果对方和自己都举了:black_flag:，那就看标签是谁的谁就可以执行

## 检验某算法是否能避免缺陷发生：状态机

遍历所有可能会达到的状态，如果不会出现同时锁住并执行的情况则说明算法正确。（可编程自动化&可视化）
