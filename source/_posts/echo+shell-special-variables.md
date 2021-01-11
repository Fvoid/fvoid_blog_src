---
title: echo+shell 中的特殊变量
date: 2021-01-09 09:05:53
tags:

---

| variable | meanings                                                |
|:--------:|:------------------------------------------------------- |
| `$0`     | 当前脚本的文件名                                                |
| `$n`     | 传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是1，第二个参数是1，第二个参数是2。 |
| `$#`     | 传递给脚本或函数的参数个数。                                          |
| `$*`     | 传递给脚本或函数的所有参数。                                          |
| `$@`     | 传递给脚本或函数的所有参数。被双引号(" ")包含时，与 $* 稍有不同，下面将会讲到。            |
| `$?`     | 上个命令的退出状态，或函数的返回值。                                      |
| $$       | 当前Shell进程ID。对于 Shell 脚本，就是这些脚本所在的进程ID。                  |

## Example

test.sh

```shell
 1     #!/bin/bash
 2     echo "File Name: $0"
 3     echo "First Parameter : $1"
 4     echo "First Parameter : $2"
 5     echo "Quoted Values: $@"
 6     echo "Quoted Values: $*"
 7     echo "Total Number of Parameters : $#"
```

in the terminal

```shell
$ ./test.sh XC 666
File Name: ./test.sh
First Parameter : XC
First Parameter : 666
Quoted Values: XC 666
Quoted Values: XC 666
Total Number of Parameters : 2
```
