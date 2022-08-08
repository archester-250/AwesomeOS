# 可执行文件的加载

## 动态加载器

- 需要完成以下四个功能

```assembly
DL_HEAD

LOAD(libc.dl)# 加载动态链接库
IMPORT(putchar)# 引入动态链接库中的函数,加载外部符号
EXPORT(hello)# 为动态库导出符号


DL_CODE

hello:
	...
	call DSYM(putchar)# 动态链接符号
	...
DL_END
```

