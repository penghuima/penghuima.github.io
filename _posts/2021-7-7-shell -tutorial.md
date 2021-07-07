---
layout: post
title: "Shell教程"
categories: Linux
tags: linux shell
---

* content
{:toc}
### Shell 和 Shell 脚本的区别

Shell 是一个用 C 语言编写的程序，它是用户使用 Linux 的桥梁。Shell 是指一种应用程序，这个应用程序提供了一个界面，用户通过这个界面访问操作系统内核的服务。

Shell 脚本（shell script），是一种为 shell 编写的脚本程序。Shell 编程跟 JavaScript、php 编程一样，只要有一个能编写代码的文本编辑器和一个能解释执行的脚本解释器就可以了。业界所说的 shell 通常都是指 shell 脚本。

<!--more-->

### **Shell的版本**

Linux 的 Shell 种类众多，常见的有：

- Bourne Shell（/usr/bin/sh或/bin/sh）
- **Bourne Again Shell（/bin/bash）**
- C Shell（/usr/bin/csh）
- K Shell（/usr/bin/ksh）
- Shell for Root（/sbin/sh）
- ……

本教程关注的是 Bash，也就是 Bourne Again Shell，由于易用和免费，Bash 在日常工作中被广泛使用。同时，Bash 也是大多数Linux 系统默认的 Shell。

**sh 和 bash 的区别**

在一般情况下，人们并不区分 Bourne Shell 和 Bourne Again Shell，所以，像 **#!/bin/sh**，它同样也可以改为 **#!/bin/bash**。因为 **bash 是 sh 的增强版本**，在我们平常实地操作的时候如果sh这个命令不灵了我们应当使用bash。

**#!** 告诉系统其后路径所指定的程序即是解释此脚本文件的 Shell 程序。

### Hello Shell

创建一个 test.sh 脚本文件，内容如下

```shell
#!/bin/bash    
echo "Hello Shell !"
```

执行脚本文件，（在windows环境下使用 Git Bash窗口执行）

```shell
./test.sh        #方式1
/bin/sh test.sh  #方式2,这种方式直接运行解释器，在shell脚本里就不需要第一行指定注释器信息了
```

### Shell 变量

#### 变量

```shell
var_name="penghuima"
```

注意，变量名和等号之间不能有空格。

使用一个定义过的变量，只要在变量名前面加美元符号 $ 即可

```shell
echo ${var_name}  //输出 penghuima
```

变量名外面的花括号是可选的，加花括号是为了帮助解释器识别变量的边界。推荐给所有变量加上花括号，这是个好的编程习惯。

#### Shell 字符串

字符串是shell编程中最常用最有用的数据类型（除了数字和字符串，也没啥其它类型好用了），字符串可以用单引号，也可以用双引号。

单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的；

双引号里可以有变量，双引号里可以出现转义字符；

所以直接使用双引号就好了啊

```shell
var_name="penghuima"
str="Hello, I know you are "$your_name"! "
str1="hello, I know you are ${your_name} !"
echo $str $str1
#输出： 
# Hello, I know you are penghuima!
# hello, I know you are penghuima !
```

**获取字符串长度**

```shell
string="abcd"
echo ${#string} #输出 4
```

**提取子字符串**

```shell
string="penghuima"
echo ${string:1:4} # 输出 engh
```

#### Shell 数组

bash支持一维数组（不支持多维数组），并且没有限定数组的大小。

类似于 C 语言，数组元素的下标由 0 开始编号。获取数组中的元素要利用下标，下标可以是整数或算术表达式，其值应大于或等于 0

**定义数组**

在 Shell 中，用括号来表示数组，数组元素用"空格"符号分割开。定义数组的一般形式为：

```shell
数组名=(值1 值2 ... 值n)
array_name=(value0 value1 value2 value3) #定义数组
```

读取数组

```shell
${数组名[下标]}
echo ${array_name[1]}
echo ${array_name[@]}
```

使用 **@** 符号或 *****  符号可以获取数组中的所有元素.

获取数组长度

```shell
# 取得数组元素的个数
length=${#array_name[@]}
# 或者
length=${#array_name[*]}
```

#### Shell 注释

以 **#** 开头的行就是注释，会被解释器忽略。

### Shell 流程控制

#### if else

和 Java、PHP 等语言不一样，sh 的流程控制不可为空。如果else分支没有语句执行，就不要写

**if 语句语法格式：**  末尾的 fi 就是 if 倒过来拼写

```shell
if [ condition ]
then
    command1 
    ...
    commandN 
fi
```

可以写成一行的形式

```shell
if [ $(ps -ef | grep -c "ssh") -gt 1 ]; then echo "true"; fi
```

**注意：中括号 [] 与其中间的代码应该有空格隔开**

**if else 语法格式：**

```shell
if [ condition ]
then
    command1 
    ...
    commandN
else
    command
fi
```

**if else-if else 语法格式：**

```sh
if [ condition1 ]
then
    command1
elif [ condition2 ]   # elif
then 
    command2
else
    commandN
fi
```

例子如下：

```shell
a=10
b=20
if [ $a == $b ]    #注意：中括号 [] 与其中间的代码应该有空格隔开
then
   echo "a 等于 b"
elif [ $a -gt $b ]
then
   echo "a 大于 b"
elif [ $a -lt $b ]
then
   echo "a 小于 b"
else
   echo "没有符合的条件"
fi
```

#### For循环

for循环一般格式为：

```shell
for var in item1 item2 ... itemN
do
    command1
    ...
    commandN
done
```

写成一行：

```sh
for var in item1 item2 ... itemN; do command1; command2… done;
```

当变量值在列表里，for 循环即执行一次所有命令，使用变量名获取列表中的当前取值。命令可为任何有效的 shell 命令和语句。in 列表可以包含替换、字符串和文件名。

例子如下：

```shell
for loop in 1 2 3 4 5
do
    echo "The value is: $loop"
done
```

#### while 语句

```shell
#!/bin/bash
int=1
while(( $int<=5 )) #(( ))
do
    echo $int
    let "int++"  # bash let 命令，实现变量自加
done
```

#### case ... esac

**case ... esac** 为多选择语句，与其他语言中的 switch ... case 语句类似，是一种多分枝选择结构，每个 case 分支用右圆括号开始，用两个分号 **;;** 表示 break，即执行结束，跳出整个 case ... esac 语句，esac（就是 case 反过来）作为结束标记。

可以用 case 语句匹配一个值与一个模式，如果匹配成功，执行相匹配的命令。

**case ... esac** 语法格式如下：

```shell
case 值 in
模式1)
    command1
    ;;
模式2）
    command1
    ;;
esac
```

#### 跳出循环

在循环过程中，有时候需要在未达到循环结束条件时强制跳出循环，Shell使用两个命令来实现该功能：break和continue。

### Shell 基本运算符

Shell 和其他编程语言一样，支持多种运算符，包括：

- 算数运算符
- 关系运算符
- 布尔运算符
- 字符串运算符
- 文件测试运算符

原生bash不支持简单的数学运算，但是可以通过其他命令来实现，例如 awk 和 expr，expr 最常用

expr 是一款表达式计算工具，使用它能完成表达式的求值操作。

例如，两个数相加(注意使用的是反引号 \`（Esc下面） 而不是单引号 ' )：

```shell
val=`expr 2 + 2`
echo "两数之和为 : $val"
```

表达式和运算符之间要有空格，例如 2+2 是不对的，必须写成 2 + 2，这与我们熟悉的大多数编程语言不一样。完整的表达式要被  `` 包含

#### 算术运算符

`+`、`-`、`\*`、`/`、`%`、`=`、`==`、`!=`

**注意：**条件表达式要放在方括号之间，并且要有空格，例如: **[$a==$b]** 是错误的，必须写成 **[ $a == $b ]**。

#### 关系运算符

| 运算符 | 说明                                                | 举例                       |
| :----- | :-------------------------------------------------- | :------------------------- |
| -eq    | 检测两个数是否相等，相等返回 true                   | [ $a -eq $b ] 返回 false。 |
| -ne    | 检测两个数是否不相等，不相等返回 true               | [ $a -ne $b ] 返回 true。  |
| -gt    | 检测左边的数是否大于右边的，如果是，则返回 true     | [ $a -gt $b ] 返回 false。 |
| -lt    | 检测左边的数是否小于右边的，如果是，则返回 true     | [ $a -lt $b ] 返回 true。  |
| -ge    | 检测左边的数是否大于等于右边的，如果是，则返回 true | [ $a -ge $b ] 返回 false。 |
| -le    | 检测左边的数是否小于等于右边的，如果是，则返回 true | [ $a -le $b ] 返回 true。  |

#### 布尔运算符

下表列出了常用的布尔运算符，假定变量 a 为 10，变量 b 为 20：

| 运算符 | 说明                                              | 举例                                   |
| :----- | :------------------------------------------------ | :------------------------------------- |
| !      | 非运算，表达式为 true 则返回 false，否则返回 true | [ ! false ] 返回 true                  |
| -o     | 或运算，有一个表达式为 true 则返回 true           | [ $a -lt 20 -o $b -gt 100 ] 返回 true  |
| -a     | 与运算，两个表达式都为 true 才返回 true           | [ $a -lt 20 -a $b -gt 100 ] 返回 false |

#### 逻辑运算符

| 运算符 | 说明       | 举例                                       |
| :----- | :--------- | :----------------------------------------- |
| &&     | 逻辑的 AND | [[ $a -lt 100 && $b -gt 100 ]] 返回 false  |
| \|\|   | 逻辑的 OR  | [[ $a -lt 100 \|\| $b -gt 100 ]] 返回 true |

#### 字符串运算符

下表列出了常用的字符串运算符，假定变量 a 为 "abc"，变量 b 为 "efg"：

| 运算符 | 说明                                       | 举例                   |
| :----- | :----------------------------------------- | :--------------------- |
| =      | 检测两个字符串是否相等，相等返回 true      | [ $a = $b ] 返回 false |
| !=     | 检测两个字符串是否不相等，不相等返回 true  | [ $a != $b ] 返回 true |
| -z     | 检测字符串长度是否为0，为0返回 true        | [ -z $a ] 返回 false   |
| -n     | 检测字符串长度是否不为 0，不为 0 返回 true | [ -n "$a" ] 返回 true  |
| $      | 检测字符串是否为空，不为空返回 true        | [ $a ] 返回 true       |

### 输入/输出重定向

大多数 UNIX 系统命令从你的终端接受输入并将所产生的输出发送回到您的终端。一个命令通常从一个叫标准输入的地方读取输入，默认情况下，这恰好是你的终端。同样，一个命令通常将其输出写入到标准输出，默认情况下，这也是你的终端。

重定向命令列表如下：

| 命令            | 说明                                               |
| :-------------- | :------------------------------------------------- |
| command > file  | 将输出重定向到 file。                              |
| command < file  | 将输入重定向到 file。                              |
| command >> file | 将输出以追加的方式重定向到 file。                  |
| n > file        | 将文件描述符为 n 的文件重定向到 file。             |
| n >> file       | 将文件描述符为 n 的文件以追加的方式重定向到 file。 |
| n >& m          | 将输出文件 m 和 n 合并。                           |
| n <& m          | 将输入文件 m 和 n 合并。                           |
| << tag          | 将开始标记 tag 和结束标记 tag 之间的内容作为输入。 |

