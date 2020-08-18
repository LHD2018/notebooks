## ROS 介绍
  [ROS](https://www.ros.org/)是机器人操作系统（Robot Operating System）的英文缩写。ROS是用于编写机器人软件程序的一种具有高度灵活性的软件架构。ROS的原型源自斯坦福大学的STanford Artificial Intelligence Robot (STAIR) 和 Personal Robotics (PR)项目。

# 1. ROS基础

## 1.1 ROS 安装
~~~bash
# 添加源
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'

# 添加公钥
sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654

# 更新源
sudo apt update

# 安装桌面完整版
sudo apt-get install ros-kinetic-desktop-full

# 初始化 rosdep
sudo rosdep init
rosdep update

# 环境配置
echo "source /opt/ros/kinetic/setup.bash" >> ~/.bashrc
source ~/.bashrc

# 构建工厂依赖
sudo apt-get install python-rosinstall python-rosinstall-generator python-wstool build-essential

~~~

## 1.2 ROS 工作环境配置
~~~bash
# 创建catkin工作空间
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/
catkin_make

# 工作空间环境配置
echo "source ~/catkin_ws/devel/setup.bash" >> ~/.bashrc
source ~/.bashrc

~~~


## 1.3 ROS 常用命令
~~~bash
rospack find [package_name]		//查找包位置

rospack depends1 [package_name]  //查看包的一级依赖

rospack depends [package_name]	// 查看包的所有依赖

roscd [package_name]		//切换到包或包的子目录

rosls [package_name]		//列出包所含文件和目录
~~~

## 1.4 ROS程序包的创建与编译

~~~bash
# 程序包创建
cd ~/catkin_ws/src
catkin_create_pkg [package_name] [depend1] [depend2] [...]

# 程序包编译
cd ~/catkin_ws
catkin_make

catkin_make --pkg [package_name]	# 单独编译一个包
~~~

## 1.5 ROS节点、话题、服务

### 1.5.1 节点（Node）
  一个节点其实只不过是ROS程序包中的一个可执行文件。ROS节点可以使用ROS客户库与其他节点通信。节点可以发布或接收一个话题。节点也可以提供或使用某种服务。

+ 节点常用命令
~~~bash
rosnode list	# 查看当前节点

rosnode info [node_name]	# 查看节点详细信息

rosnode ping [node_name]	#测试节点连通性

rosrun [package_name] [node_name]	#运行包下某一节点

rosrun rqt_graph rqt_graph		# 图形显示节点通信
~~~

### 1.5.2 话题（Topic）
  节点之间通过**话题**进行通信

+ 话题常用命令
~~~bash
rostopic list	# 列出当前话题(可加-h查看帮助)
rostopic list -v 	# 显示出有关所发布和订阅的话题及其类型的详细信息

rostopic echo [topic_name] 	#查看话题发布的数据

rostopic type [topic_name]	#查看话题发布的数据类型

rosmsg show [msg_type_name] #查看消息的详细信息

rostopic pub [topic_name] [msg_type_name] [msg] # 向话题发布数据
~~~

### 1.5.3 服务（service）
  服务（services）是节点之间通讯的另一种方式。服务允许节点发送请求（request） 并获得一个响应（response）。

+ 服务常用命令
~~~bash
rosservice list		#查看当前服务

rosservice type [service_name]		# 查看服务类型

rossrv show [service_type_name]		#查看服务详细参数

rosservice call [service_name] 	 	# 调用服务

~~~


### 1.5.4 参数
  rosparam使得我们能够存储并操作ROS 参数服务器（Parameter Server）上的数据。参数服务器能够存储整型、浮点、布尔、字符串、字典和列表等数据类型。rosparam使用YAML标记语言的语法。一般而言，YAML的表述很自然：1 是整型, 1.0 是浮点型, one是字符串, true是布尔, [1, 2, 3]是整型列表, {a: b, c: d}是字典。

+ 参数常用命令
~~~bash
rosparam list 	# 列出所有参数

rosparam set [param_name] [data] 	# 设置参数

rosparam get [param_name] 		# 获取参数

rosparam dump [filename(.yaml)]		# 保存参数

rosparam load [filename(.yaml)]		# 加载参数

~~~


## 1.6 ROS消息和ROS服务

+ 消息（msg）
   msg文件就是一个描述ROS中所使用消息类型的简单文本，存放在package的msg目录下，每行声明一个数据类型和变量名。
   
+ 服务（srv）
  一个srv文件描述一项服务。它包含两个部分：请求和响应。它存放在package的srv目录下，请求和响应用`---`分割。
  

**具体使用看官网[wiki](http://wiki.ros.org/cn/ROS/Tutorials/CreatingMsgAndSrv)**

## 1.7 ROS话题（topic）和服务（service）的区别

|  |Topic|Service|
|---|---|---|
|**通信方式**|异步通信|同步通信|
|**通信模型**|Publisher--Subscriber(多对多)|Client--Server(多对一)|
|**通信数据格式**|msg|srv|
|**比喻**|A发朋友圈，B刷朋友圈的时候看到了，朋友圈就是话题|A打电话给B，B接通了并回复|
|**应用**|连续高频数据发送与接收，如激光雷达|偶尔调用或具体功能：拍照|


## 1.8 ROS launch文件的使用

+ launch文件通过XML文件实现多节点的配置和启动，根目录采用<launch>标签定义

~~~xml
 <launch>

    <!-- Turtlesim Node-->
    <node pkg="turtlesim" type="turtlesim_node" name="sim"/>
    <node pkg="turtlesim" type="turtle_teleop_key" name="teleop" output="screen"/>

    <node pkg="learning_tf" type="turtle_tf_broadcaster" args="/turtle1" 
    name="turtle1_tf_broadcaster" />
    <node pkg="learning_tf" type="turtle_tf_broadcaster" args="/turtle2" 
    name="turtle2_tf_broadcaster" />

    <node pkg="learning_tf" type="turtle_tf_listener" name="listener" />

  </launch>
~~~

+ launch文件语法

~~~xml
<!--启动节点-->
<node pkg="package_name" type="executable_name" name="node_name" />
<!--后面可再加 output, respawn, required, ns, args 参数-->

<!--设置参数，存储在参数服务器中-->
<param name="param_name" value="param_value" />

<!--加载参数文件-->
<rosparam file="params.yaml" command="load" ns="params" />

<!--launch文件中的局部变量-->
<arg name="arg-name" default="arg-value" />

<!--重映射资源命名-->
<remap from="old_name" to="new_name" />

<!--launch 文件嵌套-->
<include file="$(dirname)/other.launch" />

~~~


