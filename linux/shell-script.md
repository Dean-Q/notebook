---
description: Linux可执行脚本.sh
---

# Shell Script

## File Structure

```sh
#! /bin/bash
# the first line start with #! 
#    specify the enviroment in which the script will be run

# some annotations

# logic code
```

## Variable assignment and use

```bash
#! /bin/bash

# create a variable and assign it a value
str="monday"

# use the variable
echo "today is : $str"
echo ${str}  #recommend
```

### Debug shell script

```sh
sh -x shell_file.sh: it will execute each step of the script and print and output it
    to the screen

result as below:
# debug shell script: successful execute
[root@localhost shell]# sh -x s1.sh 
+ str=monday
+ echo 'today is: monday'
today is: monday
+ echo monday
monday
```

### Variable Substitution

```sh
# to use "\" to escape
#! /bin/bash
str="100"
echo "RMB: ￥${str}"
echo "Dollar: \$ ${str}"
```

### Pass parameter to script

```sh
# in script, it can accept parameter as the format $1...$n
#! /bin/bash

# $0: script's name
echo \$0: $0
# $1: the first parameter
echo \$1: $1
# $#: the number of script parameters
echo \$#: $#
# $*: all parameters form a string
echo \$*: $*
# $@: each parameter is a string
echo \$@: $@
# $?: the return value of the previous command, 
#    0:execute successful, non 0:execute fail
echo \$?: $?
# $$: the pid of current progress
echo \$$: $$
```

### Quotation mark rules

```sh
#! /bin/bash

name=haha
# single qutoes: all character in single qutoes are common character
echo '$name'   # will print $name

# double qutoes: keep variable properties and replace it with value
echo "$name"  # will print haha

# inverted quotation marks: when a command is enclosed in inverted qutotation marks,
# the command will be execute and results will be the value of the expression
echo `date`  # will print Mon Dec 15 10:19:23 CST 2023
```

## Arrays

```bash
# constant list: elements are separated by spaces
arr1=("a1" "a2" "a3")

# define arrays by declare and initializes the element
declare -a arr2=("b1" "b2" "b3")

# define first and then assign the value to the element
declare -a arr3
arr3[0]="c1"
arr3[1]="c2"

# view the length of the array
echo ${#arr3[*]}

# read an array element
echo ${arr1[0]}

# delete an array element
unset arr1[0]
```

## Operator

```bash
-eq # equal
-ne # unequal
-lt # less than
-gt # greater than
-le # less than or equal to
-ge # greater than or equal to

==  # compare two strings to see if they are equal
!=  # compare two strings to see if they are unequal

!   # inverse operation
-o  # or operation
-a  # and operation
&&  # short circuit and operation
||  # short circut or operation

-b file # check whether the file is a block device file
-c file # check whether the file is the character device file
-d file # check whether the file is a dirctory
-f file # check whether the file is a normal file
        # (neither a directory nor a device file)
-r file # check whether the file is readable
-w file # check whether the file is writable
-x file # check whether the file is executable
-s file # check whether the file is empty, if it isn't empty, return true
-e file # check whether the file(includes directory) exist
```

## Conditional statement

```bash
# if else statement
#! /bin/bash
cmd=$1
if [ $cmd == "start" ]; then
        echo "start operation"
elif [ $cmd == "stop" ]; then
        echo "stop operation"
elif [ $cmd == "restart" ]; then
        echo "restart operation"
else
        echo "other operation"
fi

# pattern matching
#! /bin/bash
cmd=$1
case $cmd in
        "start") echo "start operation"
                ;;
        "stop") echo "stop operation"
                ;;
        "restart") echo "restart operation"
                ;;
        *) echo "other operation"
                ;;
esac
```

## loop Statement

```bash
# for in
#! /bin/bash
# get parameter
base_path=$*
arr=(`ls $base_path`)
# traverse the array
for f in ${arr[*]}
do
    if [ -f $f ]; then
        echo "$f is a file"
    fi
done


# for control variable
#! /bin/bash
sum=0
count=0
for((i=1;i<=10;i++))
do
    sum=$((sum+i))
    if [ $((i%2)) -eq 0 ]; then
        ((count++))
    fi
done
echo sum:${sum}, count:${count}
```

## Function Definition

```bash
# define a function without return value
#! /bin/bash
fun(){
    echo "there's a function"
    }
echo "function starts ..."
fun
echo "function compelete"



# define a function withe return value
num1=$1
num2=$2
funandReturn(){
    return $(($num1+$num2))
    }
funandReturn
echo "the sum of two digits is $? "
```
