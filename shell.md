Title: Shell Basics  
Category: Script
Tags:Shell 
###Shell 相关
shell的语法真的和一般的语言不太一样，而且同一个功能有许多不同的语法，有的时候都被混淆了，这边记录一下日常要用到的一些命令。以后还是学点python吧。
###特殊变量
* $#   
提供到shell脚本或函数的参数总数。
* \$*, $@  
一次表示所有命令行参数，这两个参数可以用来把命令行参数传递给脚本或函数所执行的程序
* $*  
 将所有命令行参数当作单个字符串 
* $@  
 将所有命令行参数当作单独字符串
* $?  
  前一个命令的退出状态
* $$
  shell进程进程号
* $0  
  shell程序的名字
* $!   
  最近一个后台命令的进程编号
###单引号、双引号、反引号的区别
* 单引号中的内容就是一个字符串，特殊符号都回归本来面貌，没有特殊意义只是一个字符。
* 双引号中的内容也是一个字符串，但是特殊符号还有特殊作用，比如$1代表第一个参数。
* 反引号括起来的字符串被shell解释为命令行，在执行时，shell首先执行该命令行，并以它的标准输出结果取代整个反引号（包括两个反引号）部分。shell中还可以用\$(...)结构，$(...)格式受到POSIX标准支持，也利于嵌套。 
###函数 
* 讲函数，首先要明白shell的变量作用域范围，是类似与C的，即**函数的调用必须在定义之后**。 
* shell函数传参数，比较特别，没有括号，用空格分割
```
function foo(){
	echo $1;
	echo $2;
}
p1="abc dd"
p2="e"
foo "$p1" $p2 //$p1必须用双引号，因为用空格分割
```
* 函数的返回包括两部分，一个是函数执行的状态，一个是函数的返回值。  
函数在shell中就是一个command，使用return，是返回一个命令的退出状态[0-255].  
用echo可以使shell获取函数中的标准输出

```
$ cat test5b
#!/bin/bash
# using the echo to return a value
function dbl {
read -p "Enter a value: " value
echo $[ $value * 2 ]
}
result=`dbl`
echo "The new value is $result"
$ ./test5b
Enter a value: 200
The new value is 400
$ ./test5b
Enter a value: 1000
The new value is 2000
```

###变量
shell中的变量有全局变量和局部变量，默认声明的变量都是全局变量，在函数中可以使用局部变量

```
function addarray {
local sum=0
local newarray
newarray=(`echo "$@"`)
for value in ${newarray[*]}
do
sum=$[ $sum + $value ]
done
echo $sum
}
myarray=(1 2 3 4 5)
echo "The original array is: ${myarray[*]}"
arg1=`echo ${myarray[*]}`
result=`addarray $arg1`
echo "The result is $result"
$ ./test11
The original array is: 1 2 3 4 5
The result is 15
```

### Arithmetic Expression
"(( ))"[算术表达式][1]，可以用作其他语言的加减
```
n=$((n + 1))
```
还可以用作比较，好记又实用
```
 for ((i=0; i < 5; i++));do 
	 echo $i; 
 done 
```
###New Test
"[[ ]]"是新的test命令，可以支持类似于其他语言比较，还增加了正则功能。可以看下与Old 
Test命令的[区别][2]  

```
//是否包含某个字符串
string='My string';
if [[ $string == *My* ]]
then
  echo "It's there!";
fi
//比较
num=5;
while [[ $num > 0 ]];do
	num=$(($num - 1));
	echo $num;
done
```

###日常命令
- date 命令
 	- 获取今天时期：\`date +%Y%m%d\` 或 \`date +%F\` 或 $(date +%y%m%d) 
	- 获取昨天时期：\`date -d yesterday +%Y%m%d\` 
	- 获取前天日期：\`date -d -2day +%Y%m%d\` 

[1]:http://mywiki.wooledge.org/ArithmeticExpression#preview
[2]:http://mywiki.wooledge.org/BashFAQ/031
