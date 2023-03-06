bc 命令是任意精度计算器语言，通常在linux下当计算器用。
它类似基本的计算器, 使用这个计算器可以做基本的数学运算。

**常用的运算：**
-   + 加法
-   - 减法
-   * 乘法
-   / 除法
-   ^ 指数
-   % 余数

# 语法
```bash
bc(选项)(参数)
```

**选项值**
-   -i：强制进入交互式模式；
-   -l：定义使用的标准数学库
-   ； -w：对POSIX bc的扩展给出警告信息；
-   -q：不打印正常的GNU bc环境信息；
-   -v：显示指令版本信息；
-   -h：显示指令的帮助信息。

**参数**
文件：指定包含计算任务的文件。

# 实例

交互式模式:
```
$ bc
bc 1.06.95
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'.
2+3
5
5-2
3
2+3*1
5
quit
```

通过管道符号:
```bash
$ echo "15+5" | bc
20
```

scale=2 设小数位，2 代表保留两位:
```bash
$ echo 'scale=2; (2.777 - 1.4744)/1' | bc
1.30
```

进制转换:
```

```