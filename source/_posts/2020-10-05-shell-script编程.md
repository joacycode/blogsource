---
title: shell编程来实现一些自动化-基础篇
date: 2019-10-05 17:46:55
categories: shell
tags: shell
toc: true
---


`Shell` 是一个用 **C 语言编写的应用程序**，该应用程序提供了一个界面，用户通过这个界面**访问操作系统内核的服务**，如可以让用户使用 **Linux 命令**。

`Shell script`，是一种**为 shell 编写的脚本程序**。`shell` 和 `shell script` 是两个不同的概念，加以区分。后缀名一般是`.sh`。

常见的Linux的shell，以及默认的解释路径如下：

- Bourne Shell（/usr/bin/sh或/bin/sh）
- Bourne Again Shell（/bin/bash）
- C Shell（/usr/bin/csh）
- K Shell（/usr/bin/ksh）
- Shell for Root（/sbin/sh）

常用的shell为前两种，**bash为LinuxOS 默认的**，它是 Bourne Shell （sh）的扩展。 **与 Bourne Shell 完全兼容**，并且在 Bourne Shell 的基础上增加了很多特性。可以提供命令补全，命令编辑和命令历史等功能。它还包含了很多 C Shell 和 Korn Shell 中的优点，有灵活和强大的编辑接口，同时又很友好的用户界面。

最简单的shell脚本的文件如下：

```bash
#!/bin/bash
echo "Hello Aszero !"
```

其中第一行的`#!` 用来指定后面的shell脚本的解释程序路径，`/bin/bash`就是解释Shell 程序路径。



### 执行shell脚本

现有一个shell脚本文件`aszero.sh`

```bash
#!/bin/bash
echo "Hello Aszero !"
```

#### 如果在第一行声明了解释器的路径，linux shell中可以直接使用`sh`命令运行shell脚本

```shell
$ sh aszero.sh
```

#### 作为可执行程序

可以参考以前说过的`chmod`命令，可以改变文件的读写以及可执行权限，这里就可以把这个bash文件变成可执行文件。

```bash
$ chmod -x ./aszero.sh
$ ./aszero.sh
```

#### 直接将文件作为解释器的参数去运行解释器

```bash
# 这种情况下不需要在文件第一行声明解释器路径
$ /bin/bash ./aszero.sh
```



### 在shell脚本中写命令

用 **``**符号或者**$()**包裹起来

```bash
for file in `ls /etc`
#或
for file in $(ls /etc)
```



### Shell变量

#### 定义变量

shell变量和熟知的其他语言的变量略微不同，如变量名不需要声明类型、不需要加特殊标识符号、变量名和值之间的等号不能有空格。命名只能使用英文字母，数字和下划线组成，首字符不能是数字。

```bash
#合法
USER_NAME="aszero"
#非法
USER_NAME = "aszero"
100_USER_NAME="aszero"
$USER_NAME="aszero"
```

#### 使用变量

定义过的变量前加`$`符号，即可使用变量。

```bash
echo $USER_NAME
#或推荐加上花括号，识别变量边界，bian
echo ${USER_NAME}
```

#### 覆盖变量

变量定义之后，该变量非**只读变量**，就可以重新被定义覆盖原始值。只读变量不可被重新定义，否则报错。

```bash
USER_NAME="aszero"
echo ${USER_NAME}
USER_NAME="Joacycode"
echo ${USER_NAME}

#只读变量
readonly USER_NAME
```

#### 删除变量

删除变量后变量就如同没有定义一样，输出空，注意不能删除只读变量。

```bash
USER_NAME="aszero"
_READONLY_NAME="aszero"
unset USER_NAME
#只读变量
readonly _READONLY_NAME
#删除只读变量 报错 unset: _READONLY_NAME: cannot unset: readonly variable
unset _READONLY_NAME
```



### Shell传参与运算

#### 传参

脚本有些时候需要根据shell执行人的意志传入一些动态参数，满足不同场景下的执行方案。shell的传参和获取也非常便捷，入参就在调用的该shell文件后面增加，多个参数用空格分隔；获取参数时候用**$n**，**$0**表示文件名，其他数字按顺序表示入参。

```shell
$ sh test.sh fst sec
```

```bash
#test.sh
echo "文件名：$0"
echo "第一个参数: $1"
echo "第二个参数: $2"
#文件名：test.sh
#第一个参数: fst
#第二个参数: sec
```

| 参数 | 说明                   |
| :--: | :--------------------- |
| $[N] | 文件名以及入参信息     |
|  $@  | 分别显示所有入参       |
|  $*  | 一个字符串显示所有入参 |
|  $#  | 入参数量               |

```bash
echo "文件名：$#"
echo "所有参数: $*"
echo "所有参数: $@"
#文件名：2
#所有参数: fst sec
#所有参数: fst sec
```



#### 运算符

- 算数运算符
- 关系运算符
- 布尔运算符
- 字符串运算符
- 文件测试运算符

```bash
A=99
B=999
STRING_TEST="ImZero"
```

**算术运算符，**shell中支持算术运算需要借助 `expr 表达式`命令或者在**mac中**shell使用`$((表达式))`，才能完成算术运算。

```bash 
VAL=`expr $A + $B`
#或者
VAL=$(($A + $B))
```

<mark>注意</mark> 算术运算的乘法*****要使用转义字符，且运算符左右两边都要留有空格。

```bash
VAL=`expr $A \* $B`
```



**关系运算符**，主要有相等 `-eq`、不相等`-ne`、大于(等于)`-gt` `-ge`、小于(等于)`-lt` `-le`几种关系，其中等于和不等于条件表达式可以用算术运算符表示。条件表达式需要用 `[]`包裹。

```bash
echo [ $A -eq $B ] # [$A == $B]
echo [ $A -ne $B ]  # [$A != $B]
```



**布尔运算符**，与或非，分别用`-a` `-o` `!`表示。

```bash
# 表示判断A小于100且要B大于1000是否成立
echo [ $A -lt 100 -a $B -gt 1000 ]
# 表示判断A小于100或B大于1000是否成立
echo [ $A -lt 100 -o $B -gt 1000 ]
```



**逻辑运算符**，与或，分别用`&&` `||`表示，条件逻辑运算要使用`[[条件逻辑运算]]`**双中括号**

```bash
# 表示判断A小于100且要B大于1000是否成立
echo [[ $A -lt 100 && $B -gt 1000 ]]
# 表示判断A小于100或B大于1000是否成立
echo [[ $A -lt 100 || $B -gt 1000 ]]
```

关于**单中括号**、**双中括号**其他说明：在 `[]` 表达式中，常见的 `>`, `<` 需要加转义字符，逻辑运算符 `||` 、`&&` ，它需要用 `-a[and]` `–o[or]` 表示。`[[]]` 运算符只是 `[]` 运算符的扩充，能够支持 >, < 符号运算不需要转义符，支持逻辑运算符`||` `&&` ，不再使用 `-a[and]` `–o[or]` 。



**字符串运算符**，检测当前字符串的语法糖符号，比如字符串为不为空，字符串长度是否为0，是否相等。

```bash
# 检测字符串是否为空
echo [$STRING_TEST] #不为空返回 true
# 检测字符串长度是否为0
echo [-z $STRING_TEST] #长度为0返回 true
```



**文件测试运算符**，文件测试运算符用于检测 Unix 文件的各种属性。**常用**的操作符如下：

| 操作符 | 说明                                                         |
| :----: | :----------------------------------------------------------- |
|   -d   | 检测文件是否是目录，如果是，则返回 true。                    |
|   -f   | 检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。 |
|   -r   | 检测文件是否可读，如果是，则返回 true                        |
|   -w   | 检测文件是否可写，如果是，则返回 true                        |
|   -x   | 检测文件是否可执行，如果是，则返回 true                      |
|   -s   | 检测文件是否为空（文件大小是否大于0），不为空返回 true。     |
|   -e   | 检测文件（包括目录）是否存在，如果是，则返回 true。          |



### Shell字符串

#### 字符串定义

shell中字符串可以用单引号也可以用双引号，两者的用法还是有区别的。单引号表示字符串，内部不能包含变量，所有字符都会原样输出；双引号表示字符串，内部可以包含变量，出现转义字符。

```bash
USER_NAME="aszero"
USER_JOB='does something interesting'
ASZERO_JOB="${USER_NAME} does something interesting"
```

#### 字符串方法
获取字符串长度，使用方法：`\${\#[String]}`。

```bash
USER_NAME="aszero"
echo ${#USER_NAME} #输出6
```
提取子字符串，使用方法：${[String]:Index:Length}，从什么地方开始截取多少字符。
```bash
USER_NAME="aszero"
echo ${USER_NAME:0:2} #as
```
查找子字符串位置，使用方法：`expr index "[String]" [String_child]`，执行命令用的\`\`或者$()都可以。
```bash
USER_NAME="aszero"
echo  `expr index "$USER_NAME" z` #输出3
```



### Shell数组

shell支持一维数组，用括号来表示数组，数组元素用"空格"符号分割开

```bash
# 数组名=(值1 值2 ... 值n)
STU_NAMES=("Ella" "Zrus" "Coco")
# 单独定义
STU_NAMES[0]="Ella"
STU_NAMES[1]="Zrus"
STU_NAMES[2]="Coco"
```

输出数组

```bash
# 输出全部
echo ${STU_NAMES[@]}
# 按下标输出 从0开始
echo ${STU_NAMES[n]
```

获取数组长度

```bash
echo ${#array_name[@]}
```



### Shell函数

#### 定义函数

```bash
[ function ] funname(){
    action;
    [return int;]
}
```

#### 注意

- return后面跟**数值0-255**，不能超过这个范围，如果没有return，将以最后一条命令运行结果，作为返回值。
- 所有函数在**使用前必须定义**。
- 函数**返回值**在调用该函数后通过 **$?** 来获得，**$?** 仅对其上一条指令负责。
- 函数入参直接连在函数名后面，多个参数空格隔开。

```bash
fn(){
  echo "第一个函数参数：$1"
  echo "全部函数参数个数：$#"
  return 249
}
fn 1 2 3
echo $?
#第一个函数参数：1
#全部函数参数个数：3
#249
```



### Shell流程控制

#### if-else语句

```bash
if condition
then
command1
elif # else-if
then
command2
else
command3
fi  # 结束标记
```



#### for循环

```bash
for var in item1 item2 ... itemN
do
    command1
    command2
    ...
    commandN
done
```
输出`/etc`下面所有的文件
```bash
for file in `ls /etc`
```

顺序输出当前列表中的数字

```bash
for loop in 1 2 3 4 5
do
    echo "The value is: $loop"
done
```



### Shell注释

```bash
# echo "单行注释"
```

```bash
:<<EOF
echo "多行注释"
echo "多行注释"
echo "多行注释"
EOF
```

```bash
fn(){
  echo "多行注释"
  echo "多行注释"
  echo "多行注释"
}
```



### shell中常用的linux命令

#### read

#### basename

#### dirname



>原创内容，欢迎交流转载请注明出处