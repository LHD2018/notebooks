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

## 进程

### 进程常用函数
+ `getpid()` 获取当前进程id
+ `fork()`函数用来产生一个新的进程，在父进程中返回子线程id，在子线程中返回0

### 进程常用命令
+ `pf -ef` 列出所有进程
+ 命令后加`&`变成守护进程，使其后台运行
+ `killall <name>` 杀死进程
+ `kill <pid>` 杀死进程，pid可由`ps -ef | grep <name>`获取

### 进程通信
+ 管道（pipe）：无名管道用于父子进程、命名管道无限制
+ 消息队列（message）：进程向队列中添加消息，其他进程读取消息
+ 信号（signal）：通知其他进程有事件发生
+ 共享内存（shared memory）：多个进程共用一块内存
+ 信号量（semaphore）：进程之间对共享资源加锁
+ 套接字（socket）：可用于不同计算机进程

#### 信号
+ `signal(int signum, sighandler_t handler);`
+ + signum:信号编号 
+ + handler:处理方式，可以是自定义信号处理函数，或忽略`SIG_IGN`, 或默认`SIG_DFL`

***`SIGKILL`(9) 信号无法捕获处理 ***

+代码示例
~~~c ++
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <signal.h>
 
void EXIT(int sig)
{
  printf("收到了信号%d，程序退出。\n",sig);
 
  // 在这里添加释放资源的代码
 
  exit(0);   // 程序退出。
}
 
int main()
{
  for (int ii=0;ii<100;ii++) signal(ii,SIG_IGN); // 屏蔽全部的信号
 
  signal(SIGINT,EXIT);  
  signal(SIGTERM,EXIT); // 设置SIGINT和SIGTERM的处理函数
 
  /* code */
~~~

+ 发送信号：`int kill(pid_t pid, int sig)`
+ + pid:要发送的进程id（>0：指定进程id，=0：父子进程发送，二者都收到，=-1：广播给系统所有进程
+ sig：：准备发送的信号代码，假如其值为零则没有任何信号送出
+ 返回值：成功返回0， 失败返回-1



## others
+ `tail -f <filename>` 实时查看文件
+ `grep <name>` 搜索过滤