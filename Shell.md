# 基本概念

**shell是用户使用linux的桥梁**，shell是指一种应用程序，这个应用程序提供了一个界面，用户通过这个界面访问操作系统的内核服务

shell是你（用户）和Linux（或者更准确的说，是你和Linux内核）之间的接口程序。你在提示符下输入的每个命令都由shell先解释然后传给Linux内核。

# 编写shell脚本

shell脚本后缀xxx.sh，运行可以使用git bash，直接输入./xxx.sh就可以运行![image-20210907172816073](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20210907172816073.png)

# shell变量

直接写xxx = xxx

```shell
#!/bin/bash

yourname='liaoyanyan'
echo "hello world"
echo ${yourname}
yourname='caoniam'
echo ${yourname}
```

## 只读变量

```shell
#!/bin/bash

readonly yourname='liaoyanyan'
echo "hello world"
echo ${yourname}
yourname='caoniam'
echo ${yourname}
```

![image-20210907173311207](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20210907173311207.png)

## 删除变量

使用unset xxx

```shell
yourname='liaoyanyan'
echo ${yourname}
unset yourname
echo ${yourname}
```

![image-20210907173559694](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20210907173559694.png)

## 变量类型

- 局部变量：局部变量只在本脚本中能够访问
- 环境变量：所有程序，shell脚本都能够访问的变量，如果需要的话，可以使用shell定义环境变量

# shell字符串

## 获取字符串长度

```shell
#!/bin/bash

yourname='liaoyanyan'
echo ${#yourname}
```

## 提取子字符串

```shell
#!/bin/bash

yourname='liaoyanyan'
echo ${yourname:1:4}
```

# shell数组

```shell
#!/bin/bash

yourname=(1 2 3)
echo ${yourname[0]}
echo ${yourname[1]}
```

@可以获取数组中所有的元素

```shell
yourname=(1 2 3)
echo ${yourname[0]}
echo ${yourname[1]}
echo ${yourname[@]}
```

![image-20210907174541214](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20210907174541214.png)

## 获取数组的长度

```shell
#!/bin/bash

yourname=(1 2 3)
length=${#yourname[@]}
echo $length
```

![image-20210907174643001](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20210907174643001.png)

# 向shell传递参数

类似于向main函数传递参数一样

shell通过$0, $1获取参数

```shell
args=$1
echo $args
```

![image-20210907175519232](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20210907175519232.png)

| $#   | 传递到脚本的参数个数                                         |
| ---- | ------------------------------------------------------------ |
| $*   | 以一个单字符串显示所有向脚本传递的参数。 如"$*"用「"」括起来的情况、以"$1 $2 … $n"的形式输出所有参数。 |
| $$   | 脚本运行的当前进程ID号                                       |
| $!   | 后台运行的最后一个进程的ID号                                 |
| $@   | 与$*相同，但是使用时加引号，并在引号中返回每个参数。 如"$@"用「"」括起来的情况、以"$1" "$2" … "$n" 的形式输出所有参数。 |
| $-   | 显示Shell使用的当前选项，与[set命令](https://www.runoob.com/linux/linux-comm-set.html)功能相同。 |
| $?   | 显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误。 |

# 基本运算符

- 算数运算符
- 关系运算符
- 布尔运算符
- 字符串运算符
- 文件测试运算符

```shell
val=`expr 2 + 2`
echo "两数之和为 : $val"
val=`expr 2 \* 2`
echo $val
val=`expr 2 - 2`
echo $val
```

## 关系运算符

| -eq  | 检测两个数是否相等，相等返回 true。                   | [ $a -eq $b ] 返回 false。 |
| ---- | ----------------------------------------------------- | -------------------------- |
| -ne  | 检测两个数是否不相等，不相等返回 true。               | [ $a -ne $b ] 返回 true。  |
| -gt  | 检测左边的数是否大于右边的，如果是，则返回 true。     | [ $a -gt $b ] 返回 false。 |
| -lt  | 检测左边的数是否小于右边的，如果是，则返回 true。     | [ $a -lt $b ] 返回 true。  |
| -ge  | 检测左边的数是否大于等于右边的，如果是，则返回 true。 | [ $a -ge $b ] 返回 false。 |
| -le  | 检测左边的数是否小于等于右边的，如果是，则返回 true。 |                            |

```shell
a=10
b=20

if [ $a -eq $b ]
then
  echo "$a -eq $b : a 等于 b"
else
  echo "$a -eq $b: a 不等于 b"
fi
```

## 文件测试运算符

它的作用就是测试文件的属性，例如是否可读，是否可写，是否可执行等等

| 操作符  | 说明                                                         | 举例                      |
| :------ | :----------------------------------------------------------- | :------------------------ |
| -b file | 检测文件是否是块设备文件，如果是，则返回 true。              | [ -b $file ] 返回 false。 |
| -c file | 检测文件是否是字符设备文件，如果是，则返回 true。            | [ -c $file ] 返回 false。 |
| -d file | 检测文件是否是目录，如果是，则返回 true。                    | [ -d $file ] 返回 false。 |
| -f file | 检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。 | [ -f $file ] 返回 true。  |
| -g file | 检测文件是否设置了 SGID 位，如果是，则返回 true。            | [ -g $file ] 返回 false。 |
| -k file | 检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。  | [ -k $file ] 返回 false。 |
| -p file | 检测文件是否是有名管道，如果是，则返回 true。                | [ -p $file ] 返回 false。 |
| -u file | 检测文件是否设置了 SUID 位，如果是，则返回 true。            | [ -u $file ] 返回 false。 |
| -r file | 检测文件是否可读，如果是，则返回 true。                      | [ -r $file ] 返回 true。  |
| -w file | 检测文件是否可写，如果是，则返回 true。                      | [ -w $file ] 返回 true。  |
| -x file | 检测文件是否可执行，如果是，则返回 true。                    | [ -x $file ] 返回 true。  |
| -s file | 检测文件是否为空（文件大小是否大于0），不为空返回 true。     | [ -s $file ] 返回 true。  |
| -e file | 检测文件（包括目录）是否存在，如果是，则返回 true。          | [ -e $file ] 返回 true。  |

```shell
file='/index.ts'

if [ -r $file ]
then
  echo '文件可读'
else
  echo "文件不可读"
fi
```

# echo命令

常见的功能：

- 打印
- 可以把显示结果定向到文件中
- 显示命令执行结果

## 结果定向到文件中

```shell
echo "caonima" > index.txt
```

txt文件中就有了caonima

## 显示命令执行结果

这个时候使用的是``而不是''

```shell
a=1

echo `date`
echo `echo $a`
```

![image-20210907181801414](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20210907181801414.png)

# test命令

test指令用于检查某个条件是否成立，它可以对数值，字符，文件三个方便的测试

感觉优点类似于if

```shell
if test $[a] -eq $[b]
then
  echo "相等"
else
  echo "不相等"
fi


if test $[c] -eq $[b]
then
  echo "相等"
else
  echo "不相等"
fi
```

另外，Shell 还提供了与( -a )、或( -o )、非( ! )三个逻辑操作符用于将测试条件连接起来

类似于那些与或非

# 流程控制

## if条件

有两种写法：

- 写成一行（适用于终端命令提示符）
- 不写成一行

```shell
if condition
then
    command1 
    command2
    ...
    commandN 
fi
```

```shell
if [ $(ps -ef | grep -c "ssh") -gt 1 ]; then echo "true"; fi
```

## for循环

```shell
for var in item1 item2 ... itemN
do
    command1
    command2
    ...
    commandN
done
```

遍历数组

```shell
array=(1 2 3 4 5)

for (( i = 0; i < ${#array[@]}; i++ )); do
    echo ${array[i]}
done
```

批量创建目录

```shell
array=(1 2 3 4 5)

for (( i = 0; i < ${#array[@]}; i++ )); do
    mkdir "bash${array[i]}"
done
```

按顺序输出字符串

```shell
for str in This is a string
do
    echo $str
done
```

for in语法

```shell
for loop in 1 2 3 4 5
do
    echo "The value is: $loop"
done
```

## while循环

while循环读取键盘信息

```shell
echo '按下 <CTRL-D> 退出'
echo -n '输入你最喜欢的网站名: '
while read FILM
do
    echo "是的！$FILM 是一个好网站"
done
```

# shell函数

```shell
func() {
  echo 'caonima'
}

func
```

# shell和bash的关系

shell是一个大的概念，可以使用不同的语法来编写shell脚本:

- Bourne Shell（/usr/bin/sh或/bin/sh）
- Bourne Again Shell（/bin/bash）
- C Shell（/usr/bin/csh）
- K Shell（/usr/bin/ksh）
- Shell for Root（/sbin/sh）

bash就是其中的一种

# .bat和.sh区别

