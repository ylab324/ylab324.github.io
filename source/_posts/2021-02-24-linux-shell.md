---
title: 'Linux Shell'
date: 2021-02-24
categories: Coding
tags: [shell,script,linux,bash]
---

:heavy_dollar_sign:  

<!-- more -->


## 启动环境  

* session  

	当我们登录系统后，bash程序启动，并且会读取一系列称为启动文件的配置脚本，这些文件定义了默认的可供所有用户共享的shell环境。然后是读取更多位于我们自己家目录中的启动文件，这些启动文件定义了用户个人的shell环境。确切的启动顺序依赖于要运行的shell会话类型。有两种shell会话类型：一个是登录shell会话，另一个是非登录shell会话。    

	登录shell会话的启动文件：  
	
	| 文件             | 内容                                                                                                                           |
	| :--------------- | :----------------------------------------------------------------------------------------------------------------------------- |
	| /etc/profile	   | 应用于所有用户的全局配置脚本												                                                    |
	| ~/.bash_profile  | 用户个人的启动文件。可以用来扩展或重写全局配置脚本中的设置									                                    |
	| ~/.bash_login	   | 如果文件 ~/.bash_profile 没有找到，bash会尝试读取这个脚本									                                    |
	| ~/.profile	   | 如果文件 ~/.bash_profile 或文件 ~/.bash_login 都没有找到，bash会试图读取这个文件。这是基于Debian发行版的默认配置，比方说Ubuntu |

	非登录shell会话的启动文件：  
	
	| 文件             | 内容						                            |
	| :--------------- | :----------------------------------------------------- |
	| /etc/bash.bashrc | 应用于所有用户的全局配置文件				            |
	| ~/.bashrc	   | 用户个人的启动文件。可以用来扩展或重写全局配置脚本中的设置 |

	在普通用户看来，文件 ~/.bashrc 可能是最重要的启动文件，因为它几乎总是被读取。非登录 shell 默认 会读取它，并且大多数登录 shell 的启动文件会以能读取 ~/.bashrc 文件的方式来书写。  

* 启动选项  

	为了方便debug，有时在启动Bash的时候，可以加上启动参数。  
	`-n` - 不运行脚本，只检查是否有语法错误  
	`-v` - 输出每一行语句运行结果前，会先输出该行语句  
	`-x` - 每一个命令处理之前，先输出该命令，再执行该命令  
	```bash
	$ bash -n scriptname  
	$ bash -v scriptname  
	$ bash -x scriptname  
	```


## 变量  

* 变量分类  

	环境变量：`env`或`printenv`命令，可以显示所有环境变量；`set`命令可显示所有变量（包括环境变量和自定义变量），以及所有的Bash函数。  
	自定义变量：局部变量、全局变量。  
	变量命名规则：  
	- 变量名可由字母数字字符（字母和数字）和下划线字符组成  
	- 变量名的第一个字符必须是一个字母或一个下划线  
	- 变量名中不允许出现空格和标点符号  

* 读取变量  

	读取变量的时候，直接在变量名前加上`$`就可以了。读取变量的时候，变量名也可以使用花括号`{}`包围，比如 $a 也可以写成 ${a} 。这种写法可以用于变量名与其他字符连用的情况。  
	如果变量值包含连续空格（或制表符和换行符），最好放在双引号里面读取。  

* 删除变量  

	`unset`命令用来删除一个变量：  
	```bash  
	unset NAME  
	```
	这个命令不是很有用。因为不存在的Bash变量一律等于空字符串，所以即使`unset`命令删除了变量，还是可以读取这个变量，值为空字符串。  
	所以，删除一个变量，也可以将这个变量设成空字符串:  
	```bash  
	$ foo=''  
	$ foo=  
	```
	上面两种写法，都是删除了变量foo。  
  
* 输出变量：`export`命令  

	```bash
	export NAME=value
	```
	上面命令执行后，当前shell及随后新建的子shell，都可以读取变量`$NAME`。子shell如果修改继承的变量，不会影响父shell。  

* 特殊变量（包含位置参数）  

	- `$?` - 上一个命令的退出码，用来判断上一个命令是否执行成功。返回值是0，表示上一个命令执行成功；如果是非零，上一个命令执行失败  
	- `$$` - 当前shell的进程ID  
	- `$_` - 上一个命令的最后一个参数  
	- `$!` - 最近一个后台执行的异步命令的进程ID  
	- `$0` - 当前shell的名称，脚本文件名  
	- `$1`到`$9` - 对应脚本的第一个参数到第九个参数。通过参数展开方式可以访问的参数个数多于9个。只要指定一个大于9的数字，用花括号括起来就可以，例如 ${10}、${55}、${211} 等  
	- `$-` - 当前shell的启动参数  
	- `$#` - 参数的总和  
	- `$@` - 全部的参数。当它被用双引号引起来的时候， 它把每一个位置参数展开成一个由双引号引起来的分开的字符串  
	- `$*` - 全部的参数。当它被用双引号引起来的时候，展开成一个由双引号引起来的字符串，包含了所有的位置参数，每个位置参数由shell变量`$IFS`值的第一个字符分隔开，默认为空格，但是可以自定义  


* 变量的默认值  
	
	- `${varname:-word}`：如果变量varname存在且不为空，则返回它的值，否则返回word。它的目的是返回一个默认值，比如${count:-0}表示变量count不存在时返回0。  
	- `${varname:=word}`：上面语法的含义是，如果变量varname存在且不为空，则返回它的值，否则将它设为word，并且返回word。它的目的是设置变量的默认值，比如${count:=0}表示变量count不存在时返回0，且将count设为0。  
	-  `${varname:+word}`：上面语法的含义是，如果变量名存在且不为空，则返回word，否则返回空值。它的目的是测试变量是否存在，比如${count:+1}表示变量count存在时返回1（表示true），否则返回空值。  
	-  `${varname:?message}`：上面语法的含义是，如果变量varname存在且不为空，则返回它的值，否则打印出varname: message，并中断脚本的执行。如果省略了message，则输出默认的信息“parameter null or not set.”。它的目的是防止变量未定义，比如${count:?"undefined!"}表示变量count未定义时就中断执行，抛出错误，返回给定的报错信息undefined!。  
			
	
	上面四种语法如果用在脚本中，变量名的部分可以用数字1到9，表示脚本的参数。  
	```bash
	filename=${1:?"filename missing."}
	```
	上面代码出现在脚本中，1表示脚本的第一个参数。如果该参数不存在，就退出脚本并报错。  
	
* 其他  

	- `declare` - 声明一些特殊类型的变量，为变量设置一些限制。  
	- `readonly` - 等同于`declare -r`，用来声明只读变量，不能改变变量值，也不能`unset`变量。  
	- `let` - 声明变量时，可以直接执行算术表达式：  
		```bash
		$ let foo=1+2
		$ echo $foo
		3
		```


## 函数  

* Bash 函数定义的语法有两种。  
	```bash
	# 第一种
	fn() {
	# codes
	}
	
	# 第二种
	function fn() {
	# codes
	}
	```
* 如果函数与脚本、别名同名，优先级：别名>函数>脚本。  
* 参数变量：与脚本的参数变量是一致的。  
* 调用时，就直接写函数名，参数跟在函数名后面。  
* Bash 函数体内直接声明的变量，属于全局变量，整个脚本都可以读取。  
* 函数里面可以用`local`命令声明局部变量。  



## 数组  

* 创建数组  

	逐个赋值：ARRAY[INDEX]=value  
	一次性赋值：ARRAY=(value1 value2 ... valueN)  
	
	定义数组时，可以使用通配符：  
	```bash
	$ mp3s=( *.mp3 )
	```
	上面例子中，将当前目录的所有MP3文件，放进一个数组。  
	
	还可以通过指定下标，把值赋给数组中的特定元素：  
	```bash
	$ days=([0]=Sun [1]=Mon [2]=Tue [3]=Wed [4]=Thu [5]=Fri [6]=Sat)  
	```
	
	先用`declare -a`命令声明一个数组，也是可以的：  
	```bash
	$ declare -a ARRAYNAME
	```
	
	`read -a`命令则是将用户的命令行输入，读入一个数组：  
	```bash
	$ read -a dice
	```
	上面命令将用户的命令行输入，读入数组dice。  
	
* 读取数组  

	读取单个元素：  
	```bash
	$ echo ${array[i]}   # i 是索引
	```
	
	读取所有成员：  
	```bash
	$ foo=(a b c d e f)
	$ echo ${foo[@]}
	a b c d e f
	```
	`@`和`*`是数组的特殊索引，表示返回数组的所有成员。  
	
	遍历数组:  
	```bash
	for i in "${names[@]}"; do
		echo $i
	done
	```
	`@`和`*`放不放在双引号之中，是有差别的。  
	
	拷贝一个数组的最方便方法，就是写成下面这样：  
	```bash
	$ hobbies=( "${activities[@]}" )
	```
	
	这种写法也可以用来为新数组添加成员：  
	```bash
	$ hobbies=( "${activities[@]" diving )
	```
	
* 默认位置  

	如果读取数组成员时，没有读取指定哪一个位置的成员，默认使用0号位置。  
	
* 数组的长度  

	要想知道数组的长度（即一共包含多少成员），可以使用下面两种语法：  
	```bash
	${#array[*]}
	${#array[@]}
	```
	
* 提取数组序号  

	`${!array[@]}`或`${!array[*]}`，可以返回数组的成员序号，即哪些位置是有值的。  
	
* 提取数组成员  

	`${array[@]:position:length}`的语法可以提取数组成员。  
	如果省略长度参数length，则返回从指定位置开始的所有成员。  
	
* 追加数组成员  

	数组末尾追加成员，可以使用`+=`赋值运算符。它能够自动地把值追加到数组末尾。否则，就需要知道数组的最大序号，比较麻烦：  
	```bash
	$ foo=(a b c)
	$ echo ${foo[@]}
	a b c
	
	$ foo+=(d e f)
	$ echo ${foo[@]}
	a b c d e f
	```

* 删除数组  

	删除一个数组成员，使用`unset`命令：  
	```bash
	$ foo=(a b c d e f)
	$ echo ${foo[@]}
	a b c d e f
	
	$ unset foo[2]
	$ echo ${foo[@]}
	a b d e f
	```
	
	将某个成员设为空值，可以从返回值中“隐藏”这个成员。  
	直接将数组变量赋值为空字符串，相当于“隐藏”数组的第一个成员。  
	`unset ArrayName`可以清空整个数组。  
	
* 关联数组  

	关联数组使用字符串而不是整数作为数组索引。  
	`declare -A`可以声明关联数组。  
	```bash
	$ declare -A colors
	$ colors["red"]="#ff0000"
	$ colors["green"]="#00ff00"
	$ colors["blue"]="#0000ff"
	```
	整数索引的数组，可以直接使用变量名创建数组，关联数组则必须用带有`-A`选项的`declare`命令声明创建。  



## 字符串  

* 字符串的长度  

	获取字符串长度的语法如下：`${#varname}`。  

* 子字符串  

	字符串提取子串的语法如下：`${varname:offset:length}`。  
	这种语法不能直接操作字符串，只能通过变量来读取字符串，并且不会改变原始字符串。变量前面的美元符号可以省略：  
	```bash
	$ hello='abcdefg'
	$ echo ${hello:2:3}
	```
	如果省略length，则从位置offset开始，一直返回到字符串的结尾。  
	如果offset为负值，表示从字符串的末尾开始算起。注意，负数前面必须有一个空格，以防止与`${variable:-word}`的变量的设置默认值语法混淆。此时还可以指定length，length可以是正值，也可以是负值（负值不能超过offset的长度）。  

* 切割和替换  

	- 字符串头部的模式匹配  
	
		`${variable#pattern}` - 如果 pattern 匹配变量 variable 的开头，删除最短匹配（非贪婪匹配）的部分，返回剩余部分  
		`${variable##pattern}` - 如果 pattern 匹配变量 variable 的开头，删除最长匹配（贪婪匹配）的部分，返回剩余部分  
		
		```bash
		$ myPath=/home/cam/book/long.file.name
		$ echo ${myPath#/*/}
		cam/book/long.file.name
		$ echo ${myPath##/*/}
		long.file.name
		```

		`${variable/#pattern/string}` - 将头部匹配的部分，替换成其他内容  
		
		```bash
		$ foo=JPG.JPG
		$ echo ${foo/#JPG/jpg}
		jpg.JPG
		```

	- 字符串尾部的模式匹配  
	
		`${variable%pattern}` - 如果 pattern 匹配变量 variable 的结尾，删除最短匹配（非贪婪匹配）的部分，返回剩余部分  
		`${variable%%pattern}` - 如果 pattern 匹配变量 variable 的结尾，删除最长匹配（贪婪匹配）的部分，返回剩余部分  
		
		```bash
		$ path=/home/cam/book/long.file.name
		$ echo ${path%.*}
		/home/cam/book/long.file
		$ echo ${path%%.*}
		/home/cam/book/long
		```

		`${variable/%pattern/string}` - 将尾部匹配的部分，替换成其他内容  

		```bash
		$ foo=JPG.JPG
		$ echo ${foo/%JPG/jpg}
		JPG.jpg
		```

	- 任意位置的模式匹配  
	
		`${variable/pattern/string}` - 如果 pattern 匹配变量 variable 的一部分，最长匹配（贪婪匹配）的那部分被 string 替换，但仅替换第一个匹配  
		`${variable//pattern/string}` - 如果 pattern 匹配变量 variable 的一部分，最长匹配（贪婪匹配）的那部分被 string 替换，所有匹配都替换  

		```bash
		$ path=/home/cam/foo/foo.name
		$ echo ${path/foo/bar}
		/home/cam/bar/foo.name
		$ echo ${path//foo/bar}
		/home/cam/bar/bar.name
		```

		如果省略了string部分，那么就相当于匹配的部分替换成空字符串，即删除匹配的部分。  

		```bash
		$ path=/home/cam/foo/foo.name
		$ echo ${path/.*/}
		/home/cam/foo/foo
		```


* 改变大小写  

	| 格式		     | 结果								                            |
	| :------------- | :----------------------------------------------------------- |
	| ${parameter,,} | 把 parameter 的值全部展开成小写字母				            |
	| ${parameter,}	 | 仅仅把 parameter 的第一个字符展开成小写字母                  |
	| ${parameter^^} | 把 parameter 的值全部转换成大写字母                          |
	| ${parameter^}	 | 仅仅把 parameter 的第一个字符转换成大写字母（首字母大写）	|



## 算术运算  

* 算术表达式  

	`((...))`语法可以进行整数的算术运算：  
	```bash
	$ ((foo = 5 + 5))
	$ echo $foo
	10
	```
	如果要读取算术运算的结果，需要在`((...))`前面加上美元符号`$((...))`，使其变成算术表达式，返回算术运算的值。  
	`$((...))`的圆括号中，不需要在变量名之前加上`$`，不过加上也不报错。  
	如果在`$((...))`里面使用字符串，Bash 会认为那是一个变量名，如果不存在同名变量，Bash 就会将其作为空值，而`$((...))`会将空值当做0。  
	
	逗号`，`在`$((...))`内部是求值运算符，执行前后两个表达式，并返回后一个表达式的值：  
	```bash
	$ echo $((foo = 1 + 2, 3 * 4))
	12
	$ echo $foo
	3
	```

* 数值的进制  

	指定不同的数基  

	| 表示法	    | 描述							                            	|
	| :------------ | :------------------------------------------------------------ |
	| number	    | 默认情况下，没有任何表示法的数字被看做是十进制数（以10为底）	|
	| 0number       | 在算术表达式中，以零开头的数字被认为是八进制数				|
	| 0xnumber      | 十六进制表示法												|
	| base#number	| base进制的数													|

	其他进制转换为10进制：
	
	```bash 
	$ echo $((0xff))  
	255  
	$ echo $((2#11111111))  
	255
	$ echo $((8#11))
	9
	$ printf %d 0xFF
	$ ((num=8#123))
	$ echo ${num}
	83
	```
	
	10进制转换为其他进制：
	
	```bash
	$ echo "obase=16;65536" | bc # 利用bc计算器。echo "obase=进制;值" | bc
	10000
	$ echo "obase=8;65536" | bc
	200000
	$ printf %x 15
	```


* 位运算  

* 逻辑运算  

* 赋值运算  

* 求值运算  

* `expr`命令  

	`expr`命令支持算术运算，可以不使用`((...))`语法：  
	```bash
	$ expr 3 + 2
	5
	```
	`expr`命令支持变量替换：  
	```bash
	$ foo=3
	$ expr $foo + 2
	5
	```
	`expr`命令也不支持非整数参数。  

* `let`命令  

	`let`命令用于将算术运算的结果，赋予一个变量：  
	```bash
	$ let x=2+3
	$ echo $x
	5
	```
	注意，x=2+3这个式子里面不能有空格，如果包含空格，需要使用引号。  

	`let`可以同时对多个变量赋值，赋值表达式之间使用空格分隔：  
	```bash
	$ let "v1 = 1" "v2 = v1++"
	$ echo $v1,$v2
	2,1
	```


## 条件判断  

* `if`结构  

	- 语法：  
	
		```bash
		if commands; then
			commands
		[elif commands; then
			commands...]
		[else
			commands]
		fi
		```

	- `test`命令  
	
		`if`结构的判断条件，一般使用`test`命令，有三种形式：  
		1. `test expression`  
		2. `[ expression ]`  
		3. `[[ expression ]]`  

		上面三种形式是等价的，但是第三种形式还支持正则判断，前两种不支持。  

	- 判断表达式  
	
		1. 文件判断

		   文件系统相关测试：
		
		   | 表达式  | 如果下列条件为真则返回TRUE               |
		   | :------ | :--------------------------------------- |
		   | -f $var | 给定的变量包含正常的文件路径或文件名     |
		   | -x $var | 给定的变量包含的文件可执行               |
		   | -d $var | 给定的变量包含的是目录                   |
		   | -e $var | 给定的变量包含的文件存在                 |
		   | -c $var | 给定的变量包含的是一个字符设备文件的路径 |
		   | -b $var | 给定的变量包含的是一个块设备文件的路径   |
		   | -w $var | 给定的变量包含的文件可写                 |
		   | -r $var | 给定的变量包含的文件可读                 |
		   | -L $var | 给定的变量包含的是一个符号链接           |
		
		2. 字符串判断
		
		   字符串表达式：
		
		   | 表达式             | 如果下列条件为真则返回True                                 |
		   | :----------------- | :--------------------------------------------------------- |
		   | string             | string不为null                                             |
		   | -n string          | 字符串string的长度大于零                                   |
		   | -z string          | 字符串string的长度为零                                     |
		   | string1 = string2  | string1和string2相同。单或双等号都可以，不过双等号更受欢迎 |
		   | string1 == string2 | string1和string2相同。单或双等号都可以，不过双等号更受欢迎 |
		   | string1 != string2 | string1和string2不相同                                     |
		   | string1 > string2  | string1排列在string2之后                                   |
		   | string1 < string2  | string1排列在string2之前                                   |
		
		3. 整数判断
		
		   整形表达式：
		
		   | 表达式                | 如果为真...                |
		   | :-------------------- | :------------------------- |
		   | integer1 -eq integer2 | integer1等于integer2       |
		   | integer1 -ne integer2 | integer1不等于integer2     |
		   | integer1 -le integer2 | integer1小于或等于integer2 |
		   | integer1 -lt integer2 | integer1小于integer2       |
		   | integer1 -ge integer2 | integer1大于或等于integer2 |
		   | integer1 -gt integer2 | integer1大于integer2       |
		
		4. 综合表达式
		
		   逻辑操作符：
		
		   | 操作符 | 测试 | [[]] and (()) |
		   | :----- | :--- | :------------ |
		   | AND    | -a   | &&            |
		   | OR     | -o   | \|\|          |
		   | NOT    | !    | !             |
		
		
		5. 正则判断
		
		   `[[ string =~ regex ]]`  
		
		   上面的语法中，`regex`是一个正则表达式，`=~`是正则比较运算符。  
		
		   举个例子：  
		
		   ```bash
		   #!/bin/bash
		   
		   INT=-5
		   
		   if [[ "$INT" =~ ^-?[0-9]+$ ]]; then
		       echo "INT is an integer."
		       exit 0
		   else
		       echo "INT is not an integer." >&2
		       exit 1
		   fi
		   ```
		
		6. 算术判断
		
		   Bash还提供了`((...))`作为算术条件，进行算术运算的判断。
		
		   注意，算术判断不需要使用`test`命令，而是直接使用`((...))`结构。这个结构的返回值，决定了判断的真伪。
		
		   算术条件`((...))`也可以用于变量赋值：
		
		   ```bash
		   $ if (( foo = 5 ));then echo "foo is $foo"; fi
		   foo is 5
		   ```
		
		   上面例子中，(( foo = 5 ))完成了两件事情。首先把5赋值给变量foo，然后根据返回值5，判断条件为真。
		
		   注意，赋值语句返回等号右边的值，如果返回的是0，则判断为假。


* `case`结构  

	语法如下：  
	```bash
	case expression in
	  pattern )
		commands ;;
	  pattern )
		commands ;;
	  ...
	esac
	```

	`case`的匹配模式可以使用各种通配符，下面是一些例子：  

	| 模式		    | 描述				            |
	| :------------ | :---------------------------- |
	| a)		    | 若单词为"a"，则匹配	    	|
	| a\|b)		    | 若单词为"a"或"b"，则匹配	 	|
	| [[:alpha:]])	| 若单词是一个字母字符，则匹配	|
	| ???)		    | 若单词只有3个字符，则匹配	    |
	| *.txt)	    | 若单词以".txt"字符结尾，则匹配|
	| *) | 匹配任意单词。把这个模式作为case命令的最后一个模式，是一个很好的做法，可以捕获到任意一个与先前模式不匹配的数值；也就是说，捕获到任何可能的无效值 |

	执行多个动作：  
	Bash 4.0之后允许匹配多个条件，这时可以用`;;&`终止每个条件块。  
	条件语句结尾添加了`;;&`以后，在匹配一个条件之后，并没有退出`case`结构，而是继续判断下一个条件。  


## 循环  

* `while`循环  

	`while`循环有一个判断条件，只要符合条件，就不断循环执行指定的语句。  
	```bash
	while condition; do
	  commands
	done
	```

* `until`循环  

	`until`循环与`while`循环恰好相反，只要不符合判断条件（判断条件失败），就不断循环执行指定的语句。一旦符合判断条件，就退出循环。  
	```bash
	until condition; do
	  commands
	done
	```
	一般来说，`until`用得比较少，完全可以统一都是用`while`。  

* `for...in`循环  

	`for...in`循环用于遍历列表的每一项。  
	```bash
	for variable in list; do
	  commands
	done
	```

* `for`循环  

	`for`循环还支持 C 语言的循环语法。  
	```bash
	for (( expression1; expression2; expression3 )); do
	  commands
	done
	```
	注意，循环条件放在双重圆括号之中。另外，圆括号之中使用变量，不必加上美元符号`$`。  

* `break`, `continue`  

* `select`结构  

	`select`结构主要用来生成简单的菜单。它的语法与`for...in`循环基本一致。  
	```bash
	select name
	[in list]
	do
	  commands
	done
	```


## 终端打印  

`echo`是用于终端打印的基本命令。  

另一个可用于终端打印的命令是`printf`:  
```bash
$ printf "%-5s %-10s %-4.2f\n" 2 James 90.9989
```

彩色输出：  
![shell颜色语法.png](https://i.loli.net/2020/06/08/DEpRWVLrby8qOxI.png)  
![shell涉及的颜色.png](https://i.loli.net/2020/06/08/Yu5AzgHGvN7VdpS.png)  
每种颜色都有对应的颜色码。比如：重置=0，黑色=30，红色=31，绿色=32，黄色=33，蓝色=34，洋红=35，青色=36，白色=37。  
要打印彩色文本，可输入如下命令：  
```bash
$ echo -e "\e[1;31m This is red text \e[0m"
```
`\e[1;31m`将颜色设为红色，`\e[0m`将颜色重新置回。  

要设置彩色背景，经常使用的颜色码是：重置=0，黑色=40，红色=41，绿色=42，黄色=43，蓝色=44，洋红=45，青色=46，白色=47。  
```bash
$ echo -e "\e[1;42m Green Background \e[0m"
```


## 重定向  

* 重定向  

	```bash
	$ ls -l /bin/usr 2> ls-error.txt #重定向标准错误
	$ ls -l /bin/usr &> ls-output.txt #重定向标准输出和错误到同一个文件
	$ ls -l /bin/usr > ls-output.txt 2>&1 #重定向标准输出和错误到同一个文件
	$ echo "hello world" > temp.txt #把echo命令的输出写入文件前，temp.txt中的内容首先会被清空
	$ echo "hello world" >> temp.txt #这种方法会将文本追加到目标文件中
	$ > ls-output.txt #如果我们需要清空一个文件内容（或者创建一个新的空文件），可以使用这样的技巧
	```

* `cat` 

	`cat`命令读取一个或多个文件，然后复制它们到标准输出。  
	如果`cat`没有给出任何参数，它会从标准输入读入数据，又因为标准输入默认情况下连接到键盘，它正在等待我们输入数据。下一步，输入`Ctrl-d`来告诉`cat`，在标准输入中，它已经到达文件末尾`EOF`。  
	```bash
	$ cat movie.mpeg.0* > movie.mpeg
	```

* 管道线  

	使用管道操作符`|`，一个命令的标准输出可以通过管道送至另一个命令的标准输入。  


* 过滤器  

	管道线经常用来对数据完成复杂的操作。有可能会把几个命令放在一起组成一个管道线。通常，以这种方式使用的命令被称为过滤器。过滤器接受输入，以某种方式改变它，然后输出它。  
	```bash
	$ ls /bin /usr/bin | sort | uniq | less
	$ ls /bin /usr/bin | sort | uniq | wc -l
	$ ls /bin /usr/bin | sort | uniq | grep zip
	$ echo `who | awk '{print $1}' | sort | uniq` | sed 's/ /, /g' > file.txt
	```

* `head`/`tail` - 打印文件开头部分/结尾部分  

	默认打印10行，通过`-n`参数指定打印的行数。  
	```bash
	$ ls /usr/bin | tail -n 5
	```
	`tail`有一个选项允许你实时地浏览文件。当观察日志文件的进展时，这很有用，因为它们同时被写入。  
	```bash
	$ tail -f /var/log/messages
	```
	通过`-f`参数，`tail`命令继续监测这个文件，当新的内容添加到文件后，它们会立即出现在屏幕上。这会一直继续下去直到你输入`Ctrl+c`。  

* `tee` - 从`stdin`读取数据，并同时输出到`stdout`和文件  
	为了和我们的管道隐喻保持一致，Linux提供了一个叫做`tee`的命令，这个命令制造了一个"tee"，安装到我们的管道上，`tee`程序从标准输入读入数据，并且同时复制数据到标准输出（允许数据继续随着管道线流动）和一个或多个文件。当在某个中间处理阶段来捕捉一个管道线的内容时，这很有帮助。这里，我们重复执行一个先前的例子，这次包含`tee`命令，在`grep`过滤管道线的内容之前，来捕捉整个目录列表到文件ls.txt。  
	```bash
	$ ls /usr/bin | tee ls.txt | grep zip
	```



## 引号和转义  

* 引号  

  - **单引号**：  

    无视所有特殊字符，原样输出，所有字符在它眼里都是普通字符，比如星号`*`、美元符号`$`、反斜杠`\`等；单引号字串中不能出现单引号（对单引号使用转义符后也不行）。

  - **双引号**: 

    无视文件通配符，但`$`、`\`和`` ` ``会起作用。

    双引号比单引号宽松，大部分特殊字符在双引号里面，都会失去特殊含义，变成普通符号。

    ```bash
    $ echo "*"
    *
    ```

    上面例子中，通配符`*`是一个特殊字符，放在双引号之中，就变成了普通字符，会原样输出。这一点需要特别留意，这意味着，双引号里面不会进行文件名扩展。

    但是三个特殊字符除外：美元符号`$`、反引号`` ` ``和反斜杠`\`。这三个字符在双引号之中，依然有特殊含义，会被Bash自动扩展。

    双引号还有一个作用，就是保存原始命令的输出格式：

    单行输出：

    ```bash
    $ echo $(cal)
    二月 2021 日 一 二 三 四 五 六 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28
    ```

    原始格式输出：

    ```bash
    $ echo "$(cal)"
    	二月 2021         
    日 一 二 三 四 五 六  
    	1  2  3  4  5  6  
    7  8  9 10 11 12 13  
    14 15 16 17 18 19 20  
    21 22 23 24 25 26 27  
    28                    
    
    ```

* 转义  

	某些字符在Bash里面有特殊含义（比如`$`、`&`、`*`）。  
	```bash
	$ echo $data
	
	$
	```
	上面例子中，输出$date不会有任何结果，因为`$`是一个特殊字符。  
	如果想要原样输出这些特殊字符，就必须在它们前面加上反斜杠，使其变成普通字符。这就叫做"转义"(escape)。  

	反斜杠除了用于转义，还可以表示一些不可打印的字符：  
	`\a` - 响铃  
	`\b` - 退格  
	`\n` - 换行  
	`\r` - 回车  
	`\t` - 制表符  
	如果想要在命令行使用这些不可打印的字符，可以把它们放在引号里面，然后使用`echo`命令的`-e`参数：  
	```bash
	$ echo a\tb
	atb
	$ echo -e "a\tb"
	a	b
	```
	上面例子中，命令行直接输出不可打印字符`\t`，Bash不能正确解析。必须把它们放在引号之中，然后使用`echo`命令的`-e`参数。  

	换行符是一个特殊字符，表示命令的结束，Bash收到这个字符以后，就会对输入的命令进行解释执行。换行符前面加上反斜杠转义，就使得换行符变成一个普通字符，Bash会将其当做空格处理，从而可以将一行命令写成多行。  
	```bash
	$ mv \
	/path/to/foo \
	/path/to/bar
	```
	等同于  
	```bash
	$ mv /path/to/foo /path/to/bar
	```


## 模式扩展  

Bash的模式扩展：  
	shell接收到用户输入的命令以后，会根据空格将用户的输入，拆分成一个个词元，然后，shell会扩展词元里面的特殊字符，扩展完后才会调用相应的命令。  

* 波浪线扩展  

	自动扩展成当前用户的主目录。  
	
* `?`字符扩展  

	代表文件路径里面的任意单个字符，不包括空字符。  
	
* `*`字符扩展  

	代表文件路径里面的任意数量的任意字符，包括零个字符。  

* 方括号扩展  

	1、匹配括号之中的任意一个字符。如果需要匹配连字号`-`，只能放在方括号内部的开头或结尾，比如[-aeiou]或[aeiou-]。  
	2、方括号扩展有一个简写形式`[start-end]`，表示匹配一个连续的范围。比如[a-c]等同于[abc]，[0-9]匹配[0123456789]。  
	
	举例：  
		[a-z]：所有小写字母；  
		[a-zA-Z]：所有小写字母与大写字母；  
		[a-zA-Z0-9]：所有小写字母、大写字母与数字；  
		[abc]*：所有以a、b、c字符之一开头的文件名；  
		program.[co]：文件program.c与文件program.o；  
		BACKUP.[0-9][0-9][0-9]：所有以BACKUP.开头，后面是三个数字的文件名。  
	
	这种简写形式有一个否定形式`[!start-end]`，表示匹配不属于这个范围的字符。比如[!a-zA-Z]表示匹配非英文字母的字符，[!1-3]表示排除1、2和3。  

* 大括号扩展  

	表示分别扩展成大括号`{...}`里面的所有值，各个值之间使用逗号分隔。比如{1,2,3}扩展成1 2 3。  
	```bash
	$ echo d{a,e,i,o,u}g
	dag deg dig dog dug
	```
	
	1、大括号内部的逗号前后不能有空格，否则，大括号扩展会失效。  
	
	2、逗号前面可以没有值，表示扩展的第一项为空。  
	```bash
	$ cp a.log{,.bak}
	```
	等同于  
	```bash
	$ cp a.log a.log.bak
	```
	
	3、大括号可以嵌套。  
	```bash
	$ echo {j{p,pe}g,png}
	jpg jpeg png
	```
	```bash
	$ echo a{A{1,2},B{3,4}}b
	aA1b aA2b aB3b aB4b
	```
	
	4、大括号也可以与其他模式联用，并且总是先于其他模式进行扩展。  
	```bash
	$ echo /bin/{cat,b*}
	/bin/cat /bin/b2sum /bin/base32 /bin/base64 ... ...
	```
	基本等同于  
	```bash
	$ echo /bin/cat;echo /bin/b*
	```
	上面例子中，会先进行大括号扩展，然后进行`*`扩展，等同于执行两条`echo`命令。  
	
	由于大括号扩展`{...}`不是文件名扩展，所以它总是会扩展的。这与方括号扩展`[...]`完全不同，如果匹配的文件不存在，方括号就不会扩展。这一点要注意区分。  
	
	不存在 a.txt 和 b.txt 时：  
	```bash
	$ echo [ab].txt
	[ab].txt
	$ echo {a,b}.txt
	a.txt b.txt
	```

	5、大括号扩展有一个简写形式`{start..end}`，表示扩展成一个连续序列。  
	```bash
	$ echo {a..c}
	a b c
	```
	这种简写形式支持逆序：  
	```bash
	$ echo {c..a}
	c b a
	```
	这种简写形式可以嵌套使用，形成复杂的扩展：  
	```bash
	$ echo .{mp{3,4},m4{a,b,p,v}}
	.mp3 .mp4 .m4a .m4b .m4p .m4v
	```

	6、大括号扩展的常见用途：  
	新建一系列目录：  
	```bash
	$ mkdir {2007..2009}-{01..12}
	```
	用于for循环：  
	```bash
	for i in {1..4}
	do
		echo $i
	done
	```
	如果整数前面有前导0，扩展输出的每一项都有前导0：  
	```bash
	$ echo {01..5}
	01 02 03 04 05
	$ echo {001..5}
	001 002 003 004 005
	```

	7、这种简写形式还可以使用第二个双点号`(start..end..step)`，用来指定扩展的步长：  
	```bash
	$ echo {0..8..2}
	0 2 4 6 8
	```
	多个简写形式连用，会有循环处理的效果：  
	```bash
	$ echo {a..c}{1..3}
	a1 a2 a3 b1 b2 b3 c1 c2 c3
	```

* 变量扩展  
	
	Bash将美元符号`$`开头的词元视为变量，将其扩展成变量值。变量名除了放在美元符号后面，也可以放在`${}`里面。  
	`${!string*}`或`${!string@}`返回所有匹配给定字符串string的变量名。  
	```bash
	$ echo ${!S*}
	SECONDS SHELL SHELLOPTS SHLVL SSH_AGENT_PID SSH_AUTH_SOCK
	```
	上面例子中，${!S*}扩展成所有以S开头的变量名。  

* 子命令扩展  
	
	`$(...)`可以扩展成另一个命令的运行结果，该命令的所有输出都会作为返回值。  
	```bash
	$ echo $(date)
	Tue Jan 12 10:56:55 CST 2021
	$ file $(ls /usr/bin/* | grep zip)
	```
	还有另一种较老的语法，子命令放在反引号之中，也可以扩展成命令的运行结果。  
	```bash
	$ echo `date`
	Tue Jan 12 10:58:51 CST 2021
	```
	
	`$(...)`可以嵌套，比如：  
	```bash
	$(ls $(pwd))
	```
	
	利用子shell生成一个独立的进程：  
	子shell本身就是独立的进程。可以使用`()`操作符来定义一个子shell：  
	```bash
	pwd;
	(cd /bin; ls);
	pwd;
	```
	当命令在子shell中执行时，不会对当前shell有任何影响；所有的改变仅限于子shell内。例如，当用cd命令改变子shell的当前目录时，这种变化不会反映到主shell环境中。  

* 算术扩展  

	`$((...))`可以扩展成整数运算的结果。  
	```bash
	$ echo $((2 + 2))
	4
	```

* 字符类  

	比如：`[[:digit:]]`匹配任意数字 0-9。  
	
	使用注意点：  
	1、通配符是先解释，再执行。  
	2、文件名扩展在不匹配时，会原样输出。  
	3、只适用于单层路径。所有文件名扩展只匹配单层路径，不能跨目录匹配，即无法匹配子目录里面的文件。或者说，`?`或`*`这样的通配符，不能匹配路径分隔符`/`。  
		如果要匹配子目录里面的文件，可以写成下面这样：  
	
	```bash
	$ ls */*.txt
	```
	
	4、文件名可以使用通配符。这时引用文件名，需要把文件名放在单引号里面。  
	```bash
	$ touch 'fo*'
	$ ls
	fo*
	```

* 量词语法  

	量词语法用来控制模式匹配的次数。它只有在Bash的`extglob`参数打开的情况下才能使用，不过一般是默认打开的。下面的命令可以查询：  
	```bash
	$ shopt extglob
	extglob on
	```
	
	量词语法有下面几个：  
	`?(pattern-list)` - 匹配零个或一个模式  
	`*(pattern-list)` - 匹配零个或多个模式  
	`+(pattern-list)` - 匹配一个或多个模式  
	`@(pattern-list)` - 只匹配一个模式  
	`!(pattern-list)` - 匹配零个或一个以上的模式，但不匹配单独一个的模式  
	
	`shopt`命令可以调整Bash的行为。它有好几个参数跟通配符扩展有关。  
	`shopt`命令的使用方法如下：  
	```bash
	$ shopt -s [optionname] #打开某个参数
	$ shopt -u [optionname] #关闭某个参数
	$ shopt [optionname] #查询某个参数关闭还是打开
	```

## 正则表达式  

shell本身不支持正则表达式，使用正则表达式的是shell命令和工具。  
正则表达式基本上是一种“表达式”， 只要工具程序支持这种表达式，那么该工具程序就可以用来作为正则表达式的字串处理之用。 例如 `vi`, `egrep`, `awk` , `sed`, `perl`, `less` 等等工具，因为她们有支持正则表达式， 所以，这些工具就可以使用正则表达式的特殊字符来进行字串的处理。但例如 `find`, `cp`, `ls`, `rm` 等系统命令一般并未支持正则表达式， 所以就只能使用 Bash 自己本身的通配符而已。

正则表达式元字符由以下字符组成：  
`^` `$` `.` `[ ]` `{ }` `-` `?` `*` `+` `( )` `|` `\`  

其他所有字符都被认为是原义字符。在个别情况下，反斜杠会被用来创建元序列，元字符也可以被转义为原义字符，而不是解释为元字符。  
注意：正如我们所见到的，当 shell 执行展开的时候，许多正则表达式元字符，也是对 shell 有特殊 含义的字符。当我们在命令行中传递包含元字符的正则表达式的时候，把元字符用引号引起来至关重要，这样可以阻止 shell 试图展开它们。  

1. 任何字符  

	圆点字符，用来匹配任意字符。如果我们在正则表达式中包含它，它将会匹配在此位置的任意一个字符。举个例子：  
	```bash
	$ grep -h '.zip' dirlist*.txt  
	bunzip2  
	bzip2  
	bzip2recover  
	gunzip  
	gzip  
	funzip  
	gpg-zip  
	preunzip  
	prezip  
	prezip-bin  
	unzip  
	unzipsfx
	```

2. 锚点  

	在正则表达式中，插入符号和美元符号被看作是锚点。这意味着正则表达式只有在文本行的开头或末尾被找到时，才算发生一次匹配。举个例子：  
	```bash
	$ grep -h '^zip' dirlist*.txt  
	zip  
	zipcloak  
	zipgrep  
	zipinfo  
	zipnote  
	zipspli  
	$ grep -h 'zip$' dirlist*.txt  
	gunzip  
	gzip  
	funzip  
	gpg-zip  
	preunzip  
	prezip  
	unzip  
	zip  
	$ grep -h '^zip$' dirlist*.txt  
	zip  
	```

	注意正则表达式`'ˆ$'`（行首和行尾之间没有字符）会匹配空行。  
	
3. 中括号表达式和字符类  

	通过使用中括号表达式，能够从一个指定的字符集合（包含在不加中括号的情况下会被解释为元字符的字符）中匹配单个字符。举个例子：  
	```bash
	$ grep -h '[bg]zip' dirlist*.txt  
	bzip2  
	bzip2recover  
	gzip  
	```

	一个字符集合可能包含任意多个字符，并且元字符被放置到中括号里面后会失去了它们的特殊含义。 然而，在两种情况下，会在中括号表达式中使用元字符，并且有着不同的含义。第一个元字符 是插入字符`^`，其被用来表示否定；第二个是连字符字符`-`，其被用来表示一个字符范围。  
	
4. 否定  

	如果在中括号表示式中的第一个字符是一个插入字符`^`，则剩余的字符被看作是不会在给定的字符位置出现的字符集合。举个例子：  
	```bash
	$ grep -h '[^bg]zip' dirlist*.txt  
	bunzip2  
	gunzip  
	funzip  
	gpg-zip  
	preunzip  
	prezip  
	prezip-bin  
	unzip  
	unzipsfx  
	```

	一个否定的字符集仍然在给定位置要求一个字符，但是这个字符必须不是否定字符集的成员。  
	插入字符如果是中括号表达式中的第一个字符的时候，才会唤醒否定功能；否则，它会失去它的特殊含义，变成字符集中的一个普通字符。  
	
5. 传统的字符区域  

6. POSIX字符集  

	POSIX 把正则表达式的实现分成了两类： 基本正则表达式（BRE）和扩展的正则表达式（ERE）。  

	BRE 和 ERE 之间有什么区别呢？这是关于元字符的问题。BRE 可以辨别以下元字符：  

	`^` `$` `.` `[ ]` `*`  
	
	其它的所有字符被认为是文本字符。ERE 添加了以下元字符（以及与其相关的功能）:  

	`( )` `{ }` `?` `+` `|`  
	
	```bash
	$ grep -Eh '^(bz|gz|zip)' dirlist*.txt  
	```

7. 限定符  

	扩展的正则表达式支持几种方法，来指定一个元素被匹配的次数。  
	* `?` - 匹配零个或一个元素  
	* `*` - 匹配零个或多个元素  
	* `+` - 匹配一个或多个元素  
	* `{}` - 匹配特定个数的元素  

	`{}`元字符都被用来表达要求匹配的最小和最大数目。它们可以通过四种方法来指定：  

	| 限定符 | 意思						                          |
	| :----- | :------------------------------------------------- |
	| n      | 匹配前面的元素，如果它确切地出现了 n 次            |
	| n,m	 | 匹配前面的元素，如果它至少出现了n次，但是不多于m次 |
	| n,	 | 匹配前面的元素，如果它出现了n次或多于n次           |
	| ,m	 | 匹配前面的元素，如果它出现的次数不多于m次          |

正则表达式的基本组成部分：  

| 正则表达式 | 描述                                       | 示例                                                                |
| :--------- | :----------------------------------------- | :------------------------------------------------------------------ |
| ^          | 行起始标记                                 | ^tux匹配以tux起始的行                                               |
| $          | 行尾标记                                   | tux$匹配以tux结尾的行                                               |
| .          | 匹配任意一个字符                           | Hack.匹配Hackl和Hacki，但是不匹配Hackl2和Hackil，它只能匹配单个字符 |
| []         | 匹配包含在[字符]之中的任意一个字符         | coo[kl]匹配cook或cool                                               |
| [^]        | 匹配除[^字符]之外的任意一个字符            | 9[^01]匹配92、93，但是不匹配91或90                                  |
| [-]        | 匹配[]中指定范围内的任意一个字符           | [1-5]匹配从1~5的任意一个数字                                        |
| ?          | 匹配之前的项1次或0次                       | colou?r匹配color或colour，但是不能匹配colouur                       |
| +          | 匹配之前的项1次或多次                      | Rollno-9+匹配Rollno-99、Rollno-9，但是不能匹配Rollno-               |
| *          | 匹配之前的项0次或多次                      | co*l匹配cl、col、coool等                                            |
| ()         | 创建一个用于匹配的子串                     | ma(tri)?x匹配max或matrix                                            |
| {n}        | 匹配之前的项n次                            | [0-9]{3}匹配任意一个三位数，[0-9]{3}可以扩展为[0-9][0-9][0-9]       |
| {n,}       | 之前的项至少需要匹配n次                    | [0-9]{2,}匹配任意一个两位或更多位的数字                             |
| {n,m}      | 指定之前的项所必须匹配的最小次数和最大次数 | [0-9]{2,5}匹配从两位数到五位数之前的任意一个数字                    |
| \|         | 交替一一匹配\|两边的任意一项               | Oct(1st\|2nd)匹配Oct 1st或Oct 2nd                                   |



## sed入门  

名字 `sed` 是 stream editor（流编辑器）的简称。  

`sed`地址表示法：  

| 地址			| 说明																																										|
| :------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| n				| 行号，n是一个正整数																																						|
| $				| 最后一行																																									|
| /regexp/		| 所有匹配一个POSIX基本正则表达式的文本行。注意正则表达式通过斜杠字符界定。选择性地，这个正则表达式可能由一个备用字符界定，通过\cregexpc来指定表达式，这里c就是一个备用字符 |
| addr1,addr2	| 从addr1到addr2范围内的文本行，包含地址addr2在内。地址可能是上述任意单独的地址形式																							|
| first~step	| 匹配由数字first代表的文本行，然后随后的每个在step间隔处的文本行。例如1~2是指每个位于偶数行号的文本行，5~5则指第五行和之后每五行位置的文本行								|
| addr1,+n		| 匹配地址addr1和随后的n个文本行																																			|
| addr!			| 匹配所有的文本行，除了addr之外，addr可能是上述任意的地址形式																												|

`sed`基本编辑命令：  

| 命令					| 说明																																																															|
| :-------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| =						| 输出当前的行号。																																																												|
| a						| 在当前行之后追加文本。																																																										|
| d						| 删除当前行。																																																													|
| i						| 在当前行之前插入文本。																																																										|
| p						| 打印当前行。默认情况下，sed程序打印每一行，并且只是编辑文件中匹配指定地址的文本行。通过指定-n选项，这个默认的行为能够被忽略。																																	|
| q						| 退出sed，不再处理更多的文本行。如果不指定-n选项，输出当前行。																																																	|
| Q						| 退出sed，不再处理更多的文本行。																																																								|
| s/regexp/replacement/	| 只要找到一个regexp匹配项，就替换replacement的内容。replacement可能包括特殊字符&，其等价于由regexp匹配的文本。另外，replacement可能包括序列\1到\9，其是regexp中相对应的子表达式的内容。在replacement末尾的斜杠之后，可以指定一个可选的标志，来修改s命令的行为。|
| y/set1/set2			| 执行字符转写操作，通过把set1中的字符转变为相对应的set2中的字符。注意不同于tr程序，sed要求两个字符集合具有相同的长度。（y命令不支持字符区域（例如，[a-z]），也不支持POSIX字符集。）																			|

**逆参照**：如果序列`\n`出现在`replacement`中，这里`n`是指从1到9的数字，则这个序列指的是在前面正则表达式中相对应的子表达式。为了创建这个子表达式，我们简单地把它们用圆括号括起来。  

`s`命令的另一个功能是使用**可选标志**，其跟随替代字符串。一个最重要的可选标志是`g`标志，其指示sed对某个文本行全范围地执行查找和替换操作，不仅仅是对第一个实例，这是默认行为。这里有个例子：  


例子：  

* 替换  
```bash
$ sed -i 's/text/replace/g' file
```
`-i`可以将替换结果应用于原文件。  
`/g`意味着`sed`会替换每一处匹配。  

* 移除空白行  
```bash
$ sed '/^$/d' file
```
空白行可以用正则表达式`^$`进行匹配。  
`/pattern/d`会移除匹配样式的行。  

* 已匹配字符串标记&  
在`sed`中，用`&`标记匹配样式的字符串，就能够在替换字符串时使用已匹配的内容。例如：  
```bash
$ echo this is an example | sed 's/\w\+/[&]/g'  
[this] [is] [an] [example]  
```
正则表达式`\w\+`匹配每一个单词，然后我们用`[&]`替换它。`&`对应于之前所匹配到的单词。  

* 子串匹配标记\1  
`&`代表匹配给定样式的字符串。但我们也可以匹配给定样式的其中一部分。  
```bash
$ echo this is digit 7 in a number | sed 's/digit \([0-9]\)/\1'  
this is 7 in a number  
```
这条命令将digit 7替换为7。样式中匹配到的子串是7。`\(pattern\)`用于匹配子串。模式被包括在使用斜线转义过的`()`中。对于匹配到的第一个子串，其对应的标记是`\1`，匹配到的第二个子串是`\2`，往后依次类推。  
下面的示例中包含了多个匹配：  
```bash
$ echo seven EIGHT | sed 's/\([a-z]\+\) \([A-Z]\+\)/\2 \1'
EIGHT seven
```

* 组合多个表达式  
利用管道组合多个`sed`命令可以用下面的方式代替：  
```bash
$ sed 'expression; expression' # sed 'expression' | sed 'expression'
```

* 引用  
`sed`表达式通常用单引号来引用。不过也可以使用双引号。双引号会通过对表达式求值来对其进行扩展。当我们想在sed表达式中使用一些变量字符串时，双引号就有用武之地了。  
```bash
$ text=hello
$ echo hello world | sed "s/$text/HELLO/"
HELLO world
```

* 其他例子：替换文本或文件中的字符串  
用另一个指定的数字替换文件中所有的3位数字：  
```bash
$ cat sed_data.txt
11 abc 111 this 9 file contains 111 11 88 numbers 0000
$
$ cat sed_data.txt | sed 's/\b[0-9]\{3\}\b/NUMBER/g'
11 abc NUMBER this 9 file contains NUMBER 11 88 numbers 0000
```
上面的单行命令只替换3位数字。\b[0-9]\{3\}\b是一个用于匹配3位数字的正则表达式。[0-9]表示从0到9的数字范围。{3}用来匹配3次之前的数字。\{3\}中的`\`用于赋予`{`和`}`特殊的含义。`\b`是单词边界标记。  


## awk入门  

`awk`脚本的结构基本如下所示：  
```bash
$ awk 'BEGIN{ commands } pattern{ commands } END{ commands }' file  
```
一个`awk`脚本通常由3部分组成：`BEGIN`语句块、`END`语句块和能够使用模式匹配的通用语句块。这3个部分是可选的，它们中任何一个部分都可以不出现在脚本中。脚本通常会被包含在单引号或双引号中。  

`awk`命令的工作方式如下所示：  
执行`BEGIN{ commands }`语句块中的语句。  
从文件或`stdin`中读取一行，然后执行`pattern{ commands }`。重复这个过程，直到文件全部被读取完毕。  
当读至输入流(input steam)末尾时，执行`END{ commands }`语句块。  

`BEGIN`语句块在`awk`开始从输入流中读取行之前被执行。这是一个可选的语句块，诸如变量的初始化、打印输出表格的表头等语句通常都可以写入`BEGIN`语句块中。  
`END`语句块和`BEGIN`语句块类似。`END`语句块在`awk`从输入流中读取完所有的行之后即被执行。像打印所有行的分析结果这类汇总信息，都是在`END`语句块中实现的常见任务（例如，在比较过所有的行之后，打印出最大数）。它也是一个可选的语句块。  
最需要的部分就是`pattern`语句块中的通用命令。这个语句块同样是可选的。如果不提供该语句块，则默认执行`{ print }`，即打印每一个读取到的行。`awk`对于读取的每一行，都会执行这个语句块。  

这就像一个用来读取行的`while`循环，在循环体中提供了相应的语句。  
每读取一行时，它就会检查该行和提供的样式是否匹配。样式本身可以是正则表达式、条件以及行匹配范围等。如果当前行匹配该样式，则执行`{}`中的语句。  
样式是可选的。如果没有提供样式，那么它就会默认所有的行都是匹配的，并执行`{}`中的语句。  

关于`print`：  
当`print`的参数是以逗号进行分割时，参数打印时则以空格作为定界符；  
在`awk`的`print`语句中，双引号是被当作拼接操作符使用的。  
```bash
$ echo | awk '{ var1="v1"; var2="v2"; var3="v3"; \
print var1"-"var2"-"var3; }'  
v1-v2-v3  
```

* 特殊变量  
以下是可用于`awk`的一些特殊变量：  
	- `NR` - 表示记录数量，在执行过程中对应于当前行号  
	- `NF` - 表示字段数量，在执行过程中对应于当前行的字段数  
	- `$0` - 这个变量包含执行过程中当前行的文本内容  
	- `$1` - 这个变量包含第一个字段的文本内容  
	- `$2` - 这个变量包含第二个字段的文本内容  
例如：  
```bash
$ echo -e "line1 f2 f3\nline2 f4 f5\nline3 f6 f7" | \
awk '{print "Line no:"NR", No of fields:"NF", $0="$0", $1="$1", $2="$2", $3="$3""}'
Line no:1, No of fields:3, $0=line1 f2 f3, $1=line1, $2=f2, $3=f3
Line no:2, No of fields:3, $0=line2 f4 f5, $1=line2, $2=f4, $3=f5
Line no:3, No of fields:3, $0=line3 f6 f7, $1=line3, $2=f6, $3=f7
```
可以用`print $NF`打印一行中最后一个字段，用`$(NF-1)`打印倒数第二个字段，其他字段依次类推即可。  
`awk`的`printf()`函数的语法和C语言中的同名函数一样。可以用这个函数来代替`print`。  

* 将外部变量值传递给`awk`  
借助选项`-v`，可以将外部值（并非来自`stdin`）传递给`awk`：  
```bash
$ VAR=10000
$ echo | awk -v VARIABLE=$VAR '{ print VARIABLE }'
10000
```
还有另一种灵活的方法可以将多个外部变量传递给`awk`，例如：  
```bash
$ var1="Variable1"; var2="Variable2"
$ echo | awk '{ print v1,v2 }' v1=$var1 v2=$var2
Variable1 Variable2
```
当输入来自于文件而非标准输入时，使用：  
```bash
$ awk '{ print v1 v2 }' v1=$var1 v2=$var2 filename
```
在上面的方法中，变量之间用空格分隔，以键-值对的形式（v1=$var1 v2=$var2）作为`awk`的命令行参数紧随在`BEGIN`、`{}`和`END`语句块之后。  

* 用`getline`读取行  
* 用样式对`awk`处理的行进行过滤  
可以为需要处理的行指定一些条件，例如：  
```bash
$ awk 'NR < 5'      # 行号小于5的行
$ awk 'NR==1,NR==4' # 行号在1到4之间的行
$ awk '/linux/'     # 包含样式linux的行（可以用正则表达式来指定样式）
$ awk '!/linux/'    # 不包含样式linux的行
```
要打印处于start_pattern与end_pattern之间的文本，使用下面的语法：  
```bash
$ awk '/start_pattern/, /end_pattern/' filename
```
例如：  
```bash
$ cat section.txt
line with pattern1
line with pattern2
line with pattern3
line end with pattern4
line with pattern5
$
$ awk '/pa.*3/, /end/' section.txt
line with pattern3
line end with pattern4
```

* 设定字段定界符  
默认的字段定界符是空格。我们也可以用 `-F "delimiter"` 明确指定一个定界符。  
```bash
$ awk -F: '{ print $1,$2,$3,$NF }' /etc/passwd
```
或者  
```bash
$ awk 'BEGIN { FS=":" } { print $1,$2,$3,$NF }' /etc/passwd
```

* 从`awk`中读取命令输出  
在下面的代码中，`echo`会生成一个空白行。变量cmdout包含命令grep root /etc/passwd的输出，然后打印包含root的行；  
将command的输出读入变量output的语法如下：  
"command" | getline output ;  
例如：  
```bash
$ echo | awk '{ "grep root /etc/passwd" | getline cmdout ; print cmdout }'  
root:x:0:0:root:/root:/bin/bash
```
通过使用`getline`，能够将外部shell命令的输出读入变量cmdout。



遇到的问题：

1. [`awk`中不能使用`shell`定义的变量，使用`-v`选项](https://blog.csdn.net/rj042/article/details/72860177)
2. [`kill`在`awk`中无所有](https://stackoverflow.com/questions/34236675/kill-command-doesnt-work-in-awk)



## 行操作  

`!n` - 执行历史文件里面行号为n的命令  
`!string` - 执行最近一个以指定字符串string开头的命令  
`!?string` - 执行最近一条包含字符串string的命令  
`Alt + .` - 插入上一个命令的最后一个词  
`Ctrl + k` - 剪切光标位置到行尾的文本  
`Ctrl + u` - 剪切光标位置到行首的文本  
`Ctrl + y` - 在光标位置粘贴文本  
`Ctrl + r` - 反向增量搜索。从当前命令行开始，向上增量搜索。意思是在字符输入的同时，bash会去搜索历史列表（直接出结果，并高亮匹配的第一个字），每多输入一个字符都会使搜索结果更接近目标。输入`Ctrl-r`来启动增量搜索，接着输入你要寻找的字。当你找到它以后，你可以敲入`Enter`来执行命令，或者输入`Ctrl-j`，从历史列表中复制这一行到当前命令行。再次输入`Ctrl-r`，来找到下一个匹配项（历史列表中向上移动）。输入`Ctrl-g`或者`Ctrl-c`，退出搜索。  


## Vim  

* 移动光标  

	`0` - (零按键) 移动到当前行的行首  
	`^` - 移动到当前行的第一个非空字符  
	`$` - 移动到当前行的末尾  

* 基本编辑  

	`u` - 撤销  
	`Ctrl + r` - 反撤销  

* 打开一行  

	`o` - 当前行的下方打开一行  
	`O` - 当前行的上方打开一行  

* 删除文本  

	`d$` - 从光标位置开始到当前行的行尾  
	`d0` - 从光标位置开始到当前行的行首  
	`d^` - 从光标位置开始到文本行的第一个非空字符  
	`dG` - 从当前行到文件的末尾  

* 复制文本  

	`y$` - 从当前光标位置到当前行的末尾  
	`y0` - 从当前光标位置到行首  
	`y^` - 从当前光标位置到文本行的第一个非空字符  
	`yG` - 从当前行到文件末尾  

* 连接行  

	`J`  


* 编辑多个文件  

	```bash
	$ vim file1 file2 file3 ...
	```
	
	文件之间的切换：  

	从这个文件切换到下一个文件`:n`；  
	回到先前的文件`:N`。  
		
	查看正在编辑的文件列表，使用`:buffers`命令。  
	
	要切换到另一个缓冲区（文件），输入`:buffer`, 紧跟着你想要编辑的缓冲器编号。  
	
	跨文件复制粘贴：正常复制粘贴。  


## 调试脚本  

* 内建功能  

	`set -x` - 在执行时显示参数和命令  
	`set +x` - 禁止调试  
	`set -v` - 当命令进行读取时显示输入  
	`set +v` - 禁止打印输入  

	```bash
	#!/bin/bash
	#文件名：debug.sh
	for i in {1..6}
	do
	set x
	echo $i
	set +x
	done
	echo "Script executed"
	```
	在上面的脚本中，仅在`-x`和`+x`所限制的区域，echo $i的调试信息才会被打印出来。  

	有一些环境变量常用于调试：  
	`LINENO` - 返回它在脚本里面的行号  
	`FUNCNAME` - 返回一个数组，内容是当前的函数调用堆栈。该数组的0号成员是当前调用的函数，1号成员是调用当前函数的函数，以此类推  
	`BASH_SOURCE` - 返回一个数组，内容是当前的脚本调用堆栈。该数组的0号成员是当前执行的脚本，1号成员是调用当前脚本的脚本，以此类推，跟变量`FUNCNAME`是一一对应关系  
	`BASH_LINENO` - 返回一个数组，内容是每一轮调用对应的行号  

	`${BASH_LINENO[$i]}`跟`${FUNCNAME[$i]}`是一一对应关系，表示`${FUNCNAME[$i]}`在调用它的脚本文件`${BASH_SOURCE[$i+1]}`里面的行号

* 自定义格式  

	```bash
	#!/bin/bash
	function DEBUG()
	{
	[ "$_DEBUG" == "on" ] && $@ || :
	}
	for i in {1..10}
	do
	DEBUG echo $i
	done
	```
	可以将调试功能置为"on"来运行上面的脚本：  
	```bash
	$ _DEBUG=on ./script.sh  
	```

* shebang  

	把`shebang`从`#!/bin/bash`改成`#!/bin/bash -xv`，这样一来，不用任何其他选项就可以启用调试功能了。 


* 注释  

	单行注释：`#`  
	多行注释：  
	```bash
	<<'!'
	commands
	...
	!
	```


## 常用命令  

### 进程  

```bash
ps -ef
ps -aux
```

用 `top` 命令动态查看进程。  

中断一个进程`Ctrl+c`。  

把一个进程放置到后台 (执行)：在程序命令之后，加上`&`字符：  
```bash
xlogo &
```

shell 的任务控制功能给出了一种列出从我们终端中启动了的任务的方法。执行 `jobs` 命令，可以看到输出列表：  
```bash
$ jobs
[1]+ Running xlogo &
```

进程返回到前台：  
一个在后台运行的进程对一切来自键盘的输入都免疫，也不能用 `Ctrl-c` 来中断它。为了让一个进程返回前台 (foreground)，这样使用 `fg` 命令：  
```bash
$ jobs
[1]+ Running xlogo &
$ fg %1
xlogo
```
使用 `fg` 命令，可以恢复程序到前台运行，或者用 `bg` 命令把程序移到后台。  

停止一个进程`Ctrl+z`：输入 `Ctrl-z`，可以停止一个前台进程。  

通过 `kill` 命令给进程发送信号。  
通过 `killall` 命令给多个进程发送信号。  

### 目录堆栈  

* `cd -`  

* `pushd`, `popd`  

	希望记忆多重目录，可以使用`pushd`命令和`popd`命令。它们用来操作目录堆栈。  
	`pushd`命令的用法类似`cd`命令，可以进入指定的目录：  
	```bash
	$ pushd dirname
	```
	上面命令会进入目录dirname，并将该目录放入堆栈。  
	`popd`命令不带有参数时，会移除堆栈的顶部记录，并进入新的堆栈顶部目录（即原来的第二条目录）。  
	
	- `-n`
		表示仅操作堆栈，不改变目录。  
		```bash
		$ popd -n
		```
		上面的命令仅删除堆栈顶部的记录，不改变目录，执行完成后还停留在当前目录。  

	- 整数参数  
	- 目录参数  

* `dirs`  

	`dirs`命令可以显示目录堆栈的内容，一般用来查看`pushd`和`popd`操作后的结果。  

	参数：  
		`-c`：清空目录栈。  
		`-l`：用户主目录不显示波浪号前缀，而打印完整的目录。  
		`-p`：每行一个条目打印目录栈，默认是打印在一行。  
		`-v`：每行一个条目，每个条目之前显示位置编号（从0开始）。  

### 校验和核实  

- `md5sum`  

	将输出的校验和重定向到一个文件，然后用这个MD5文件核实数据的完整性：  
	```bash
	$ md5sum filename > file_num.md5
	```
	```bash
	$ md5sum -c file_sum.md5
	```

- `sha1sum`  

	用法与md5sum类似：  
	```bash
	$ sha1sum filename > file_num.sha1
	```
	```bash
	$ sha1sum -c file_sum.sha1
	```

### read命令

`-t`参数 - 设置超时秒数  
`-p`参数 - 指定用户输入的提示信息  
`-a`参数 - 把用户的输入赋值给一个数组，从零号位置开始  
`-n`参数 - 指定只读取若干个字符作为变量值，而不是整行读取  
`-e`参数 - 允许用户输入的时候，使用ReadLine库提供的快捷键，比如自动补全  
其他参数 - 其他参数功能  

```bash
$ read -p "Enter one or more values > "
$ echo "REPLY = '$REPLY'"
```

```bash
$ read -p "What do you want? 
    1) AAA
    2) BBB
    3) CCC
    Type your choice: " MyChoice
$ echo $MyChoice
```

### 生成任意大小的文件  

```bash
$ dd if=/dev/zero of=junk.data bs=1M count=1
```
该命令会创建一个1MB大小的文件junk.data。  
`if`代表输入文件(Input File)，`of`代表输出文件(Output File)，`bs`代表以字节为单位的块大小(Block Size)，`count`代表需要被复制的块数。  


### 压缩归档文件  

选项`-a`指定从文件扩展名自动判断压缩格式。  
```bash
$ tar -cavf archive.tar.gz [FILES]
$ tar -cavf archive.tar.bz2 [FILES]
$ tar -cavf archive.tar.lzma [FILES]
```

```bash
$ tar -xvf archive.tar.gz
$ tar -xvf archive.tar.bz2
$ tar -xvf archive.tar.lzma
```

### mktemp命令  

`mktemp`命令就是为安全创建临时文件而存在的。虽然在创建临时文件之前，它不会检查临时文件是否存在，但是它支持唯一文件名和清除机制，因此可以减轻安全攻击的风险。  

Bash脚本使用`mktemp`命令的用法如下：  
```bash
#!/bin/bash

TMPFILE=$(mktemp)
echo "Our temp file is $TMPFILE"
```

为了确保临时文件创建成功，`mktemp`命令后面最好使用OR运算符`||`，保证创建失败时退出脚本。  
```bash
#!/bin/bash

TMPFILE=$(mktemp) || exit 1
echo "Our temp file is $TMPFILE"
```
为了保证脚本退出时临时文件被删除，可以使用`trap`命令指定退出时的清除操作。  
```bash
#!/bin/bash

trap 'rm -f "$TMPFILE"' EXIT

TMPFILE=$(mktemp) || exit 1
echo "Our temp file is $TMPFILE"
```

`mktemp`命令的参数：  
`-d`参数可以创建一个临时目录。  
`-p`参数可以指定临时文件所在的目录。默认是使用$TMPDIR环境变量指定的目录，如果这个变量没设置，那么使用/tmp目录。  
`-t`参数可以指定临时文件的文件名模板。  
```bash
$ mktemp -t mytemp.XXXXXXX -p test/
```


### trap命令  

`trap`命令用来在Bash脚本中响应系统信号。  
`trap`的命令格式如下：  
```bash
$ trap [动作] [信号1] [信号2] ...
```
上面代码中，"动作"是一个Bash命令，"信号"常用的有以下几个。  
HUP：编号1，脚本与所在的终端脱离联系。  
INT：编号2，用户按下 Ctrl + C，意图让脚本终止运行。  
QUIT：编号3，用户按下 Ctrl + 斜杠，意图退出脚本。  
KILL：编号9，该信号用于杀死进程。  
TERM：编号15，这是kill命令发出的默认信号。  
EXIT：编号0，这不是系统信号，而是 Bash 脚本特有的信号，不管什么情况，只要退出脚本就会产生。  

`trap`命令响应EXIT信号的写法如下：  

```bash
$ trap 'rm -f "$TMPFILE"' EXIT
```
上面命令中，脚本遇到EXIT信号时，就会执行rm -f "$TMPFILE"。  

trap命令的常见使用场景，就是在Bash脚本中指定退出时执行的清理命令。  
注意，trap命令必须放在脚本的开头。否则，它上方的任何命令导致脚本退出，都不会被它捕获。  

如果trap需要触发多条命令，可以封装一个Bash函数。  
```bash
function egress {
	command1
	command2
	command3
}
trap egress EXIT
```


### set命令,shopt命令  

**set -u** - 遇到不存在的变量就会报错，并停止执行  
**set -x** - 在运行结果之前，先输出执行的那一行命令  
脚本当中如果要关闭命令输出，可以使用`set +x`。  

Bash的错误处理  
如果脚本里面有运行失败的命令（返回值非0），Bash 默认会继续执行后面的命令。这种行为很不利于脚本安全和除错。  
实际开发中，如果某个命令失败，往往需要脚本停止执行，防止错误累积。这时，一般采用下面的写法。  
```bash
command || exit 1
```
如果停止执行之前需要完成多个操作，就要采用下面三种写法。  
写法一：  
```bash
command || { echo "command failed"; exit 1; }
```
写法二：  
```bash
if ! command; then echo "command failed"; exit 1; fi
```
写法三：  
```bash
command
if [ "$?" -ne 0 ]; then echo "command failed"; exit 1; fi
```

另外，除了停止执行，还有一种情况。如果两个命令有继承关系，只有第一个命令成功了，才能继续执行第二个命令，那么就要采用下面的写法。  
```bash
command1 && command2
```

**set -e**  
上面这些写法多少有些麻烦，容易疏忽。`set -e`从根本上解决了这个问题，它使得脚本只要发生错误，就终止执行。  
`set -e`根据返回值来判断，一个命令是否运行失败。但是，某些命令的非零返回值可能不表示失败，或者开发者希望在命令失败的情况下，脚本继续执行下去。这时可以暂时关闭`set -e`，该命令执行结束后，再重新打开`set -e`。  

还有一种方法是使用command || true，使得该命令即使执行失败，脚本也不会终止执行  
```bash
#!/bin/bash
set -e
foo || true
echo bar
```
上面代码中，true使得这一行语句总是会执行成功，后面的echo bar会执行。  

**set -o pipefail**  
`set -e`有一个例外情况，就是不适用于管道命令。  
所谓管道命令，就是多个子命令通过管道运算符`|`组合成为一个大的命令。Bash 会把最后一个子命令的返回值，作为整个命令的返回值。也就是说，只要最后一个子命令不失败，管道命令总是会执行成功，因此它后面命令依然会执行，`set -e`就失效了。  
`set -o pipefail`用来解决这种情况，只要一个子命令失败，整个管道命令就失败，脚本就会终止执行。  

**set -E**  
一旦设置了`-e`参数，会导致函数内的错误不会被`trap`命令捕获。  
`-E`参数可以纠正这个行为，使得函数也能继承`trap`命令。  

**其他参数**  

`set`命令总结：  
上面重点介绍的`set`命令的几个参数，一般都放在一起使用：  
```bash
set -Eeuxo pipefail
```
建议放在所有Bash脚本的头部。  


`shopt`命令  
`shopt`命令用来调整 Shell 的参数，跟`set`命令的作用很类似。  
`shopt`命令后面跟着参数名，可以查询该参数是否打开。   
`shopt`命令的使用方法如下：  
```bash
$ shopt -s [optionname] #打开某个参数
$ shopt -u [optionname] #关闭某个参数
$ shopt [optionname] #查询某个参数关闭还是打开
```

### xargs  

我们可以用管道将一个命令的`stdout`(标准输出)重定向到另一个命令的`stdin`(标准输入)。例如：  
```bash
cat foo.txt | grep "test"
```
但是，有些命令只能以命令行参数的形式，而无法通过`stdin`接受数据流。在这种情况下，我们没法用管道来提供那些只有通过命令行参数才能提供的数据。  

`xargs`能够处理`stdin`并将其转换为特定命令的命令行参数；`xargs`也可以将单行或多行文本输入转换成其他格式，例如单行变多行或是多行变单行。  

`-d`: 为输入指定一个定制的定界符。  
`-n`: 指定每行最大的参数数量。  
`-I`: 指定一个替换字符串，对于每一个参数，命令都会被执行一次。  

例子1：  
```bash
$ echo "splitXsplitXsplitXsplit" | xargs -d X  
split split split split  
$
$ echo "splitXsplitXsplitXsplit" | xargs -d X -n 2  
split split  
split split  
$
```

例子2：  
有一个cecho.sh脚本：  
```bash
#!/bin/bash
#文件名：cecho.sh

echo $*'#'
```

有一个args.txt文件：  
```bash
$ cat args.txt  
arg1  
arg2  
arg3  
```

试试如下用法：  
```bash
$ cat args.txt | xargs -I {} ./cecho.sh -p {} -l  
-p arg1 -l #  
-p arg2 -l #  
-p arg3 -l #  
```



### 其他命令  

`less` - 浏览文件内容  
`which` - 查找可执行文件的绝对路径  
`whereis` - 查找命令、二进制文件、man文件、源代码文件  
`wc` - 打印行数、字数和字节数   
`diff/patch`    

```bash
$ diff -Naur old_file new_file > diff_file  
$ patch < diff_file  
```

`history` - 显示历史列表内容  
`df` - 显示磁盘分区上可以使用的磁盘空间  
`du` - 显示每个文件和目录的磁盘使用空间 

```bash
$ du -sh #查看当前目录总计容量
232G	.
```

`free` - 显示内存  
`file` - 识别文件类型  
`sort` - 排序文本行  
`uniq` - 报道或省略重复行  
`cut` - 从每行中删除文本区域  
`paste` - 合并文件文本行  
`join` - 基于某个共享字段来联合两个文件的文本行  
`comm` - 逐行比较两个有序的文件  
`tr` - 翻译或删除字符  
`grep` - 打印匹配行   
`ln file link` - 创建硬链接  
`ln -s item link` - 创建链接符号，item可以是一个文件或是一个目录  
	当你删除一个符号链接时，只有这个链接被删除，而不是文件自身。如果先于符号链接删除文件，这个链接仍然存在，但是不指向任何东西。在这种情况下，这个链接被称为坏链接。在许多实现中， `ls`命令会以不同的颜色展示坏链接，比如说红色，来显示它们的存在。  
	当建立符号链接时，你既可以使用绝对路径名，也可用相对路径名。  
	不同目录，创建链接符号需要绝对地址，否则也是坏链接，会提示找不到文件。  
`rsync` - 备份/同步  
`time` - 计算命令执行时间   
`find` - 在一个目录层次结构中搜索文件  
`touch` - 更改文件时间  
`stat` - 显示文件或文件系统状态   
`alias` - 别名  

```bash
alias install='sudo apt-get install'
```

`date` - 获取、设置日期  
`sleep` - 延时  

## 问题

1. [[Linux资讯] Cshell 怎样实现变量名包含变量名？](https://bbs.csdn.net/topics/394186543)

## 参考  

1. [Linux命令大全](https://man.linuxde.net/)  
2. [中国程序员最容易读错的单词汇总（带正确发音示范）](https://www.linuxcool.com/pronunciation)  
3. [Bash Shell 中的 算术运算符、逻辑与或非(& | ！)运算符、整数关系运算符、字符串关系运算符、文件或目录测试运算符](https://blog.csdn.net/ljlfather/article/details/105106875)
