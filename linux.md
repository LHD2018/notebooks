## 内核操作

+ 常用命令
~~~bash
uname -r 	# 查看当前内核

dpkg -l | grep linux-image		# 列出当前所有内核

sudo apt-get purge --remove linux-image-x.x.0-xx-generic # 删除内核

~~~

### 安装内核

+ 去[ubuntu仓库](https://kernel.ubuntu.com/~kernel-ppa/mainline/)下载合适的内核，包括4个.deb文件，如图所示：
![](https://i.loli.net/2020/08/30/kz7MUWolSegmvFH.png)

~~~bash
//进入下载目录执行下面命令
sudo dpkg -i *.deb
~~~


## shell

### 入门
~~~bash

# shell文件后缀名为.sh, 且具有执行权限

#!/bin/bash		# 第一行，表示使用bash

read [-pt] name		# 等待键盘输入并赋给name，p：提示，t：设定等待时间

$(())		# 支持整型运算

test -erwxdf	#测试 e：文件存在为真 r:存在且可读 w：存在且可写 x：存在且可执行 d：存在为目录 f：存在为文件

command1 && command2		# command1执行完毕并正确，执行command2
command1 || command2		# command1执行完毕不正确，执行command2

$0 ~ $n		# shell的参数，$0为shell本身
$#			# 最后一个参数的标号

if 判断条件 ; then
	# do something
elif 判断条件 ; then
	# do something
else
	# do something
fi

case $变量 in
"case1")
	# do something
	;;
“case2”)
	# do something
	;;
esac


~~~

### 变量
~~~bash


name=value	# 等号两边都不能有空格，如需要则进行转义

export name 	# 将变量设为环境变量

${name} 	# 引用变量

$(command) 	# 在变量中使用命令，但需要在双引号中，单引号就是字面文字

unset name		# 取消变量

declare [-aixr] name 	# 声明变量，a：数组，i：整形，x：环境变量，r：只读




~~~

