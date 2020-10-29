## 变量
我们给变量a赋值直接用`a=10`，引用a的时候，用`$a`
```shell
a=10
echo $a
```
执行输出10

一些特殊变量

| 变量 | 含义                                                         |
| ---- | ------------------------------------------------------------ |
| $0   | 当前脚本的文件名                                             |
| $n   | 传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是\$1，第二个参数是$2 |
| $#   | 传递给脚本或函数的参数个数                                   |
| $*   | 传递给脚本或函数的所有参数                                   |
| $@   | 传递给脚本或函数的所有参数。被双引号(" ")包含时，与 $* 稍有不同 |
| $?   | 上个命令的退出状态，或函数的返回值                           |
| $$   | 当前Shell进程ID。对于 Shell 脚本，就是这些脚本所在的进程ID   |

> 注：$* 和 $@ 都表示传递给函数或脚本的所有参数，不被双引号(" ")包含时，都以"$1" "$2" … "$n" 的形式输出所有参数。
> 但是当它们被双引号(" ")包含时，"$*" 会将所有的参数作为一个整体，以"$1 $2 … $n"的形式输出所有参数；"$@" 会将各个参数分开，以"$1" "$2" … "$n" 的形式输出所有参数

```
# test.sh
echo "File Name: $0"
echo "First Parameter : $1"
echo "First Parameter : $2"
echo "Quoted Values: $@"
echo "Quoted Values: $*"
echo "Total Number of Parameters : $#"
echo "exit status : $?"
echo "shell pid: $$"
```
运行结果
```
File Name: ./test.sh
First Parameter : aa
First Parameter : bb
Quoted Values: aa bb
Quoted Values: aa bb
Total Number of Parameters : 2
exit status : 0
shell pid: 80397
```

