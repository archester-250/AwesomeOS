# xv6

- `initcode`是xv6运行的第一个进程

- `ecall`指令的执行过程

  - 关闭中断
  - 复制 $pc 到 $sepc
  - 设置 $sstatus 为 S-mode
  - 设置 $scause 为 trap 的原因 (ecall, 8)
  - 跳转到 $stvec ($pc = $stvec)

  在 xv6 中

  - Trampoline: $stvec = 0x3ffffff000 (只读)
    - 对`ecall`瞬间的状态做快照
    - 将trapframe填写完整
  - Trapframe (0x3fffffe000): 保存进程寄存器现场的内存
