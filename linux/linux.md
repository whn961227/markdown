## Linux

### linux 常用命令

| 命令                          | 命令解释         |
| ----------------------------- | ---------------- |
| top                           | 查看内存         |
| df -h                         | 查看磁盘存储     |
| netstat -tunlp \| grep 端口号 | 查看端口占用情况 |
| ps -aux                       | 查看进程         |
|                               |                  |
|                               |                  |
|                               |                  |

### awk

```shell
# log.txt 文件内容
2 this is a test
3 Are you like awk
This's a test
10 There are orange,apple,mongo
```

用法一：

```shell
awk '{[pattern] action}' {filenames} # 行匹配语句 awk '' 只能用单引号

# 每行按空格或 TAB 切割，输出文本中的 1,4 项
awk '{print $1,$4}' log.txt
-------------------------------------------
2 a
3 like
This's
10 orange,apple,mongo

# 格式化输出
awk '{printf "%-8s %-10s\n",$1,$4}' log.txt
-------------------------------------------
2        a
3        like
This's
10       orange,apple,mongo
```

用法二：

```shell
awk -F  #-F相当于内置变量FS, 指定分割字符

# 使用“，”分割
awk -F, '{print $1,$2}' log.txt
---------------------------------------------
2 this is a test
3 Are you like awk
This's a test
10 There are orange apple

# 或者使用内建变量
awk 'BEGIN{FS=","} {print $1,$2}' log.txt
---------------------------------------------
2 this is a test
3 Are you like awk
This's a test
10 There are orange apple

# 使用多个分隔符 先使用空格分割，然后对分割结果使用“，”分割
awk -F '[ ,]' '{print $1,$2,$5}' log.txt
---------------------------------------------
2 this test
3 Are awk
This's a
10 There apple
```

用法三：

```shell
awk -v  # 设置变量

awk -va=1 '{print $1,$1+a}' log.txt
---------------------------------------------
2 3
3 4
This's 1
10 11

awk -va=1 -vb=s '{print $1,$1+a,$1b}' log.txt
---------------------------------------------
2 3 2s
3 4 3s
This's 1 This'ss
10 11 10s
```

用法四：

```shell
awk -f {awk脚本} {文件名}

awk -f cal.awk log.txt
```

