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


## BASH

### 变量
~~~bash
name=value	//等号两边都不能有空格，如需要则进行转义

export name 	// 将变量设为环境变量

${name} 	//引用变量

$(command) 	// 在变量中使用命令，但需要在**双引号中**，单引号就是字面文字

unset name		// 取消变量

read [-pt] name		// 等待键盘输入，p：提示，t：设定等待时间

declare [-aixr] name 	//声明变量，a：数组，i：整形，x：环境变量，r：只读


~~~

