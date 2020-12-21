# CMake的使用

## 1. CMake介绍
   CMake 是一个跨平台编译（安装）工具，它能够根据平台的不同生成对应的makefile文件或者project文件，它在LInux上使用更加简单高效，我们再也不用编写复杂的makefile文件了，故现在的许多开源项目都采用cmake形式。

+ CMake 默认变量
|变量名|解释|
|---|---|
|`PROJECT_NAME`|工程名|
|`PROJECT_SOURCE_DIR`和`CMAKE_SOURCE_DIR`|工程的顶级目录|
|`CMAKE_BINARY_DIR`和`PROJECT_BINARY_DIR`|内部编译：工程顶级目录；外都编译：外部目录|
|`CMAKE_CURRENT_SOURCE_DIR`|当前处理的CMakeLists.txt文件所在目录|
|`LIBRARY_OUTPUT_DIR`|生成的库存放目录|
|`BINARY_OUTPUT_DIR`|生成的可执行文件存放目录|
|`CMAKE_MODULE_PATH`|引用其他CMake模块|
|`CMAKE_CXX_FLAGS`|设置c++编译器参数|
|`CMAKE_C_FLAGS`|设置c编译器参数|
|`CMAKE__BUILD_TYPE`|编译类型（debug或者release）|

</br>

## 2. CMake命令参数

+ `-G <generator_name>`：指定编译器名称，如`-G "MinGW Makefiles"`，对大小写敏感，而且编译器已设置到环境变量中。**一般不用**
+ `-D<var>:<type>=<value>`:添加变量及值到CMakeList.txt中。**注意-D后面不能有空格**，type为string时可省略。例如：cmake -DCMAKE_BUILD_TYPE~~:STRING~~=Debug

## 3. CMakeLists.txt编写

***CMakeLists.txt是CMake项目的核心***
~~~bash
project(<project_name>)	# 工程名

cmake_minimum_required(VERSION <version_number>）		# CMake的最低版本

aux_source_directory(<dir> <var>)		# 将dir目录的所有cpp文件保存到var中

add_executable(<exe_name> <src_name>) 	# 将所有cpp文件编译成exe_name的可执行文件

add_library(<lib_name> [STATIC|SHARED] <src_name>) # 将所有源文件编译生成lib_name的库文件(会自动加后缀名，静态库.a;动态库.so)，可选静态库，动态库，默认静态库

add_dependencies(<target_name> <depend_name>)	# 给指定目标添加需要依赖的目标，这里的前后目标必须是 add_executable、add_library生成的。

add_subdirectory(<source_dir>)		# 添加一个需要构建的子目录

set(<var> <value>）		# 将var的值设为value


message([STATUS|WARNING|SEND_ERROR] <content>) 	# 输出信息，可选信息类型，不过一般直接输出。

find_package(<package_name> REQUIRED) 	# 查找已安装包,后面头文件,库文件一般有固定格式,如${OpenCV_INCLUDE_DIRS},${OpenCV_LIBRARIES}

include_directories(<dir>) 		# 从dir目录中寻找头文件

find_path(<var> <name> <path>)	# 从path中查找包含文件name的路径，如果找到就将它的绝对路径存到var中

find_library(<var> <name> <path>)	# 从path中查找库文件name的路径，如果找到就将它的绝对路径存到var中

link_directories(<dir>)		# 从dir中搜索库文件

target_link_libraries(<exe_name> <lib_name>) # 链接库文件（不加后缀），当静态库和动态库同时存在时，优先链接动态库，也可以加后缀指定库类型

file(GLOB <var> <file_name>) # 将正则匹配file_name到的文件存到var中，GLOB不支持递归遍历子目录，如需要请用GLOB_RECURSE

~~~



