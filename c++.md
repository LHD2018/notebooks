

#前言

+ ***之前学`c++`完全是学到哪算哪，从来没有系统性学过，现在重新开始。有些重点借此记录下来，希望能坚持下来，有所收获。***

![ASCII表镇楼](https://i.loli.net/2021/02/06/3QBLvHuJF4bYNzd.jpg)




## 1. 变量

### 1.1 常量——`#define`和`const`

+ `#define` (宏定义)：它在编译的预处理阶段起作用，只是简单的字符替换，没有类型检查，在哪使用就在哪被替换，内存中有若干个备份，且不能进行调试。
+ `const`：它在程序编译和运行阶段起作用，有对应的数据类型，会进行类型检查，在内存中只有一份备份，节省了内存空间，可以进行调试。
  
  
### 1.2 标识符

+ 标识符不能是关键字
+ 标识符只能由字母、数字、下划线构成
+ 第一个字符必须是字母或下划线
+ 标识符区分大小写
  
  
### 1.3 数据类型
+ 数据类型的意义：为变量分配合适的内存空间。
  
  
#### 1.3.1 整形
|数据类型|占用空间|表示范围|
|---|---|---|
|short(短整型)|2字节|-2^15 ~ 2^15-1|
|int(整型)|4字节|-2^31 ~ 2^31-1|
|long(长整型)|Windows 4字节, Linux为4字节（32位）,8字节（64位）|-2^31 ~ 2^31-1|
|long long(长长整型)|8字节|-2^63 ~ 2^63-1|



#### 1.3.2 浮点型（实型）
|数据类型|占用空间|表示范围|
|---|----|---|
|float(单精度)|4字节|7位有效数字|
|double(双精度)|8字节|15~16位有效数字|



#### 1.3.3 字符型
+ `char`字符型，占用一个字节，在内存中存储的是其对应的ASCIII码，常见的ASCII码有
   A—65，a—97。



#### 1.3.4 字符串
- `char[]`
- `string` : 需要包含头文件<string>



####  1.3.5 布尔类型
- true
- false



### 1.4 运算符
| 运算符 | 术语 | 示例 | 结果 |
| ------ | ---- | ---- | ---- |
|`%`|取模（取余）|10%3|1|
|`++`|前置++|a=2,b=++a|a=3,b=3|
|`++`|后置++|a=2,b=a++|a=3,b=2|
|`+=`|加等于|a=2,a+=3|a=5|
|`&`|逻辑‘与’|a=0&a++|false,a=1|
|`&&`|逻辑'与'（有短路）|a=0&&a++|a=0|
|`&`|位运算符|0x31&0x0f|0x01|
|`|`|逻辑‘或’|`a=1|a++`|a=2|
|`||`|逻辑'或'(有短路)|`a=1||a++`|a=1|
|`（?:）`|三目运算符|`a= (10<5?3:2)`|a=2|




## 2. 指针

### 2.1 指针大小
+ 32位操作系统占4字节
+ 64位操作系统占8字节



### 2.2 const修饰指针
|类型|示例|特点|
|---|---|---|
|常量指针|`const int* p = &a`|指针指向的值不能修改，指针的指向可以修改|
|指针常量|`int* const p = &a`|指针的指向不能修改，指向的值可以修改|
|——|`const int* const p = &a`|指针的指向不能修改，指向的值也不能修改|

### 2.3 智能指针

+ auto_ptr: c++11中已被抛弃

+ unique_ptr：实现独占式拥有或严格拥有概念，保证同一时间内只有一个智能指针可以指向该对象。不能通过赋值移交所有权，可以使用`move()`函数实现。
~~~c++
unique_ptr<string> pu1(new string ("hello world")); 
unique_ptr<string> pu2; 
pu2 = pu1;        // #1 不允许

unique_ptr<string> pu3; 
pu3 = unique_ptr<string>(new string ("You"));   // #2 允许

pu2 = move(pu1)		// 允许

~~~

+ shared_ptr实现共享式拥有概念。多个智能指针可以指向相同对象，该对象和其相关资源会在“最后一个引用被销毁”时候释放。从名字share就可以看出了资源可以被多个指针共享，它使用计数机制来表明资源被几个指针共享。可以通过成员函数use_count()来查看资源的所有者个数。除了可以通过new来构造，还可以通过传入auto_ptr, unique_ptr,weak_ptr来构造。当我们调用release()时，当前指针会释放资源所有权，计数减一。当计数等于0时，资源会被释放。

~~~c++
int main()
{
	string *s1 = new string("s1");

	shared_ptr<string> ps1(s1);
	shared_ptr<string> ps2;
	ps2 = ps1;

	cout << ps1.use_count()<<endl;	//2
	cout<<ps2.use_count()<<endl;	//2
	cout << ps1.unique()<<endl;	// 是否独占 0

	string *s3 = new string("s3");
	shared_ptr<string> ps3(s3);

	cout << (ps1.get()) << endl;	//033AEB48
	cout << ps3.get() << endl;	//033B2C50
	swap(ps1, ps3);	//交换所拥有的对象
	cout << (ps1.get())<<endl;	//033B2C50
	cout << ps3.get() << endl;	//033AEB48

	cout << ps1.use_count()<<endl;	//1
	cout << ps2.use_count() << endl;	//2
	ps2 = ps1;
	cout << ps1.use_count()<<endl;	//2
	cout << ps2.use_count() << endl;	//2
	ps1.reset();	//放弃ps1的拥有权，引用计数的减少
	cout << ps1.use_count()<<endl;	//0
	cout << ps2.use_count()<<endl;	//1
}

~~~

+ weak_ptr ：是一种不控制对象生命周期的智能指针, 它指向一个 shared_ptr 管理的对象。weak_ptr 设计的目的是为配合 shared_ptr 而引入的一种智能指针来协助 shared_ptr 工作, 它只可以从一个 shared_ptr 或另一个 weak_ptr 对象构造, 它的构造和析构不会引起引用记数的增加或减少。weak_ptr是用来解决shared_ptr相互引用时的死锁问题,如果说两个shared_ptr相互引用,那么这两个指针的引用计数永远不可能下降为0,资源永远不会释放。它是对对象的一种弱引用，不会增加对象的引用计数，和shared_ptr之间可以相互转化，shared_ptr可以直接赋值给它，它可以通过调用lock函数来获得shared_ptr。

## 3.结构体

### 3.1 C 中结构体定义
~~~c
typedef struct student_tag{
	string name;
	int age;
}Student;

// 两种使用方式
Student s1;
struct student_tag s2;
~~~

+ student_tag 是标识符,可以不写，即如下定义
~~~c
typedef struct {
	string name;
	int age;
}Student;

// 使用方式
Student s1;
~~~

### 3.2 C++ 中结构体定义

~~~c++
struct Student{
	string name;
	int age
};

// 使用
Student s1;

struct Student{
	string name;
	int age
}s2;	// s2 是一个变量
~~~

### 3.3 union联合体
+ 各成员共用一块内存空间，并且同时只有一个成员可以得到这块内存的使用权(对该内存的读写)，各变量共用一个内存首地址。一个union变量的总长度至少能容纳最大的成员变量，而且要满足是所有成员变量类型大小的整数倍

~~~c ++
// 联合体大小
union U2
{
    char a;   
    int b;
    short c;
    double d;
    int e[5];
}； // 大小为double（8字节）的整数倍，且要能装下e（20字节），故大小为24
~~~
~~~c ++
// 大小端问题
#include<stdio.h>
 
//联合体
union u3  
{
    char c[4];
    int i;   
}U4;
 
//主函数
int main(){ 
    U4.C[0]=0X04;
    U4.C[1]=0X03;
    U4.C[2]=0X02;
    U4.C[3]=0X11;
    printf("%x\n",U4.i);	// 输出为11020304 小端
    return 0;
}
~~~



## 4.内存分区模型
+ c++在程序**运行前**（编译完成后，生成可执行程序，但未运行）分为代码区和全局区
|类型|特点|回收机制|
|---|---|---|
|代码区|程序的二进制代码，共享只读|——|
|全局区|存放全局变量，静态变量，字符串常量和全局常量|由操作系统回收|

+ c++在程序**运行后**增加栈区和堆区
|类型|特点|回收机制|
|---|---|---|
|栈区|存放函数的参数值和局部变量（不要返回局部变量的地址）|由编译器自动分配和释放|
|堆区|利用new在堆区开辟内存|程序员分配释放，若程序员不释放（delete），结束后操作系统回收|



## 5. 引用
+ 引用的本质是指针，简化指针的操作




## 6. 类和对象
+ 面向对象的核心是：封装、继承、多态



### 6.1 封装
#### 6.1.1 封装的权限
|类型|特点|
|---|---|
|public|内部和外部都可以访问|
|protected|内部可以访问，外部不可以访问，其子类可以访问|
|private|内部可以访问，外部不可以访问，其子类也不能访问|
+ struct默认权限公开，而Class默认私有



#### 6.1.2 构造函数
+ 无参构造
+ 有参构造
+ 拷贝构造 `const`修饰的引用
|类型|特点|
|---|---|
|浅拷贝|编译器完成的简单赋值操作，可能造成堆区内存的重复释放|
|深拷贝|在堆区重新开辟一块空间|



#### 6.1.3 析构函数
+ 释放堆区的内存 


#### 6.1.4 初始化列表
`Student(int a , string b, int c) : id(a) : name(b) : age(c){}`



#### 6.1.5 静态成员变量和静态成员函数
+ 在编译阶段分配内存
+ 类内声明，类外初始化
+ 所有对象共享一份数据

+ 所有对象共用一个函数
+ 静态成员函数只能访问静态成员变量

+ 空类占一个字节
+ 非静态成员变量属于类



#### 6.1.6 this 指针
+ 本质是一个常量指针
+ 指向被调用的成员函数所属对象
+ 区分重名的成员变量和形参
+ 在非静态成员函数中返回对象本身` return *this`


#### 6.1.7 常函数和常对象

+ 常函数在函数后加const修饰
+ 常函数不能修改成员变量，如果要修改，成员变量需要用mutable修饰
+ 常对象在定义时用const修饰
+ 常对象只能调用常函数

~~~c++
Class Student{
	public:
	void goToSchool() const {
		cout << name << endl;
	}
	
	private:
	mutable string name;
}

const Student s;
~~~



#### 6.1.8 友元
+ 全局函数做友元，在类中声明函数并用`friend`修饰
+ 类做友元，在类中声明类并用`friend`修饰
+ 成员函数做友元，在类中声明成员函数并用`friend`修饰



#### 6.1.9 运算符重载
自定义数据类型的运算
+ 加号重载：`operator+()`，可以在成员函数也可以在全局函数
+ 左移重载：`operator<<()`, 在全局函数
+ 自增重载： `operator++()` ,在成员函数，用`int`占位符区分后置++；前置返回对象引用，后置返回新对象
+ 赋值重载：`operator=()`,在成员函数，深拷贝解决堆区内存重复释放
+ 函数调用重载：`operator()()`,又称防函数



### 6.2 继承

> class 子类 ： public/protected/private 父类
> 继承中先构造父类再构造子类，析构顺序相反
> 子类属性和父类属性重名，访问父类属性需要加作用域



#### 6.2.1 继承方式
+ public：父类私有属性不能访问，其他属性权限保持不变
+ protected：父类私有属性不能访问，其他属性权限全变成protected
+ private：父类私有属性不能访问，其他属性权限全变成private




### 6.3 多态
> 静态多态：编译时绑定地址
> 动态多态：运行时绑定地址

> 多态的条件：构成继承关系，子类重写父类虚函数`virtual`
> 多态使用：父类指针或引用指向子类对象
> 多态好处：
> + 代码可读性强
> + 扩展性强
> 

#### 6.3.1 纯虚函数与抽象类
+ `virtual void 函数名（参数） = 0` 
+ 有一个纯虚函数则构成抽象类，不能实例化

#### 6.3.2 虚析构和纯虚析构
+ 解决父类指针释放子类对象
+ 都需要具体的函数实现



## 7 文件操作

### 7.1 文本文件读写
+ 导入头文件`fstream`
+ 文件打开方式
|打开方式|解释|
|---|---|
|ios::in|读文件|
|ios::out|写文件|
|ios::ate|初始位置：文件尾|
|ios::app|追加方式写文件|
|ios::trunc|如果文件存在，先删除再创建|
|ios::binary|二进制方式|

### 7.2 二进制读写
+ 读：`read()`
+ 写：`write()`



## 8 泛型编程和STL

### 8.1 模板

#### 8.1.1 函数模板
~~~c++
template<typename T>
//函数体或函数声明
~~~
> 使用方式
> + 自动类型推导,不会发生隐式类型转换
> + 指定类型,会发生隐式类型转换
> 
> 函数模板与普通函数重载调用
> + 两者都可以实现,优先调用普通函数.
> + 通过空模板函数列表强制调用模板函数
> + 函数模板可以重载
> + 如果函数模板可以产生更好的匹配,那么调用函数模板
> 

#### 8.1.2 类模板
~~~c++
template<typename T1, typename T2>
class Person{
public:
	T1 name;
	T2 age;
}
~~~
+ 类模板只能指定类型,且可以有默认参数


### 8.2 STL
> + STL(standard template library) 六大组件:容器, 算法, 迭代器, 仿函数, 适配器, 空间配置器
> 
> > 容器
> > + 序列式容器:强调值的排序,每个元素都有固定的位置
> > + 关联式容器:二叉树结构,每个元素没有物理上的顺序关系
> > 算法
> > + 质变算法:改变了区间元素的内容,比如拷贝, 替换
> > + 非质变算法:不会改变区间元素的内容,比如查找,计数
> > 迭代器
> > + 双向迭代器:读写操作,并能向前向后操作
> > + 随机访问迭代器:读写操作,可以跳跃式访问任何数据
> > 


### 8.3 容器

#### 8.3.1 vector容器
~~~c++
vector<int> v;
v.push_back(elem);
v.pop_back();
vector<int>::iterator it = v.begin();
v.reserve(1000);	//预留空间
v.insert(v.begin(), 10);
~~~

#### 8.3.2 deque 容器
  支持前插和后插，内部通过中控器维护
~~~c++
deque<int> d;
d.push_front(elem);		//前插
d.pop_front();
d.push_back(elem);		//尾插
d.pop_back();
~~~

#### 8.3.4 stack 容器
  先进后出，不能进行遍历
~~~c++
stack<int> s;
s.empty();
s.size();
s.push(elem);
s.pop();
s.top();
~~~
#### 8.3.5 queue 容器

  先进先出，不能进行遍历

  ~~~C++
queue<int> q;
q.push(elem);
q.pop();
q.back();
q.front();
q.empty();
q.size();
  ~~~

#### 8.3.6 list 容器

+ 链表（list）由一系列结点构成，节点由存储数据的**数据域**，和存放下一个结点指针的**指针域**组成。
+ STL 中链表是一个双向循环链表。

+ 常用函数
~~~c++
// 构造函数与其他容器一样

list<int> data;

data.push_back(elem);		//尾插
data.pop_back();
data.push_front(elem);		//前插
data.pop_front();
data.insert(iterator, elem);

data.erase(iterator);
data.erase(beg_iterator, end_iterator);
data.remove(elem);

data.clear();

data.front();
data.back();

data.reverse();
data.sort();		//自定义数据类型排序需要指定规则

~~~


#### 8.3.7 set和multiset 容器

+ 常用函数
~~~c++
set<int> data;
data.insert(elem);

data.erase(iterator);
data.erase(beg_iterator, end_iterator);
data.erase(elem);
data.clear();

data.find(elem);	//查找，若存在返回元素迭代器
data.count(elem);	// 统计元素个数
~~~

+ set和multiset 的异同点
>   二者在插入时都会进行排序，所有函数相同
>   set不能插入重复元素，multiset可以插入重复元素

#### 8.3.8 map和multimap 容器

+ map中所有元素都是pair
+ pair由键值对组成
+ 所有元素会根据键自动排序
+ map不能插入重复元素，multimap 可以

+ 常用函数
~~~c++
map<int, int> data;

data.insert(make_pair(key, value));
data.insert(pair<int, int>(key, value);

data.erase(iterator)				//返回下一个元素的迭代器
data.erase(beg_iterator, end_iterator)	//返回下一个元素的迭代器
data.erase(key);
data.clear();

data.find(key);		//存在返回元素迭代器， 不存在返回data.end()
data.count(key);	//统计元素个数

~~~



### 8.4 仿函数（函数对象）

#### 8.4.1 仿函数概念

+ 仿函数重载的操作符`()`，调用的时候看起来像函数，但它的本质是一个类。

+ 特点
+ - 仿函数可以像普通函数一样调用，可以有参数和返回值
+ - 仿函数超出了普通函数的概念，可以有自己的状态
+ - 仿函数对象可以作为参数传递

#### 8.4.2 谓词

+ 谓词：仿函数返回值类型是bool
+ - 一元谓词：仿函数一个参数
+ - 二元谓词：仿函数两个参数

#### 8.4.3 内建函数对象

+ 需要包含`functional`头文件

+ 算数仿函数
+ - negate（取反）
+ - plus（加）

+ 关系仿函数
+ - equal_to（等于）
+ - greater（大于）
+ - less（小于）

+ 逻辑仿函数
+ - logical_not （逻辑非）


### 8.5 算法

***包含头文件`algorathm`***

+ 遍历算法
~~~c++
for_each(iterator_beg, iterator_end, _func);		// _func 可传函数或仿函数对象

transform(iterator_beg, iterator_end, dst_iterator_begin, _func); 	// 目标容器需要提前开辟空间_func 可传函数或仿函数对象
~~~

+ 查找算法
~~~c++
find(iterator_beg, iterator_end);		//存在返回元素迭代器， 不存在返回data.end()

find_if(iterator_beg, iterator_end, _pred);	 	//_pred是条件，是谓词

adjacent_find(iterator_beg, iterator_end);		// 查找相邻重复元素，返回第一个元素迭代器

binary_search(iterator_beg, iterator_end, value);	// 查找有序序列，返回bool

count(iterator_beg, iterator_end, value);		// 统计元素出现个数

count_if(iterator_beg, iterator_end, _pred);	// 按谓词统计个数
~~~

+ 排序算法
~~~c++
sort(iterator_beg, iterator_end, _pred);

random_shuffle(iterator_beg, iterator_end); 	// 乱序

merge(iterator1_beg, iterator1_end, iterator2_beg, iterator2_end, dst_iterator_beg)	// 合并两个容器到一个容器， 目标容器需要提前开辟空间

reverse(iterator_beg, iterator_end);	// 反序

~~~

+ 拷贝替换算法
~~~c++
copy(iterator_beg, iterator_end, dst_iterator_beg);	// 目标容器需要提前开辟空间

replace(iterator_beg, iterator_end, old_value, new_value);

replace(iterator_beg, iterator_end, _pred, new_value); // 将满足谓词条件的替换

swap(container_1, container_2);
~~~

***包含头文件`numeric`***

+ 数算生成算法
~~~c++
accumulate(iterator_beg, iterator_end, beg_value); 	// 将容器元素从起始值累加

fill(iterator_beg, iterator_end, value) 		// 向容器填充值

~~~

+ 集合算法
~~~c++
set_intersection(iterator1_beg, iterator1_end, iterator2_beg, iterator2_end, dst_iterator_beg); // 求两个容器的交集，两个容器必须是有序序列

set_union(iterator1_beg, iterator1_end, iterator2_beg, iterator2_end, dst_iterator_beg); 	// 求两个容器的并集，两个容器必须是有序序列

set_difference(iterator1_beg, iterator1_end, iterator2_beg, iterator2_end, dst_iterator_beg); 	// 求两个容器的差集，两个容器必须是有序序列

~~~


## 9 并发与多线程

+ 并发：指在同一时刻只能有一条指令执行，但多个进程指令被快速的轮换执行，使得在宏观上具有多个进程同时执行的效果，但在微观上并不是同时执行的，只是把时间分成若干段，使多个进程快速交替的执行。

+ 并行：指在同一时刻，有多条指令在多个处理器上同时执行。所以无论从微观还是从宏观来看，二者都是一起执行的

+ 分类：进程并发，线程并发
+ - 进程：是资源分配的最小单位，每启动一个可执行程序，称为创建一个进程
+ - 线程：是cpu调度的最小单位，在进程中，某一时刻某一个独立活动

### 9.1 线程使用

+ 主线程从main函数开始到结束，自定义线程也需要一个函数或仿函数(无参)

常用方法
~~~c++
#include <thread>	// 包含头文件

thread my_thread；

my_thread.join();	//阻塞主线程

my_thread.detach();	// 与主线程分离

my_thread.joinable();	// 线程是否可join或detach

~~~

</br>
#### 9.1.1 创建线程
+ 构造函数中传入函数名，如：`thread my_thread(pthFun);`

+ `CreateThread()`和`_beginthreade()`函数
~~~c ++
// 以`CreateThread()`为例
HANDLE WINAPI CreateThread(
  LPSECURITY_ATTRIBUTES  lpThreadAttributes,  	// 安全属性，一般传NULL
  SIZE_T                 dwStackSize,				// 线程栈空间大小，传0代表默认
  LPTHREAD_START_ROUTINE lpStartAddress,	// 线程函数地址（函数名）
  LPVOID                 lpParameter,				// 线程函数参数
  DWORD                  dwCreationFlags,			// 控制线程创建，一般传0
  LPDWORD                lpThreadId				// 返回线程id号，传NULL表示不需要返回
);

~~~
***推荐使用`_beginthreade()`，因为`CreateThread()`在使用标准c运行库时可能不安全***
</br>

### 9.2 互斥量（windows）

> 互斥量（mutex）：又称互斥锁，是一种用来保护临界区的特殊变量，有锁定（locked）和解锁（unlocked）两种状态。

#### 9.2.1 创建互斥量
~~~c ++
HANDLE CreateMutex(
  LPSECURITY_ATTRIBUTES lpMutexAttributes,	
  bool bInitialOwner,     			
  LPCTSTR lpName
);
~~~
+ 第一个参数：安全控制，一般是NULL
+ 第二个参数：确定互斥量的初始拥有者。如果传入TRUE表示互斥量对象内部会记录创建它的线程的线程ID号并将递归计数设置为1，由于该线程ID非零，所以互斥量处于未触发状态。如果传入FALSE，那么互斥量对象内部的线程ID号将设置为NULL，递归计数设置为0，这意味互斥量不为任何线程占用，处于触发状态。
+ 第三个参数：用来设置互斥量的名称，在多个进程中的线程就是通过名称来确保它们访问的是同一个互斥量。
+ 返回值：成功返回一个表示互斥量的句柄，失败返回NULL
</br>

#### 9.2.2 打开已经创建的互斥量(用于跨进程通信)
~~~c ++
// 某一个进程中的线程创建互斥量后，其它进程中的线程就可以通过这个函数来找到这个互斥量
HANDLE OpenMutex(
 DWORD dwDesiredAccess,
 bool bInheritHandle,
 LPCTSTR lpName    
);
~~~
+ 第一个参数：权限，一般传MUTEX_ALL_ACCESS
+ 第二个参数：互斥量句柄继承性，一般传入TRUE即可
+ 第三个参数：表示互斥量名称。
+ 返回值：成功返回一个表示互斥量的句柄，失败返回NULL

</br>
#### 9.2.3 等待互斥量触发
	DWORD WaitForSingleObject(HANDLE hHandle, DWORD dwMilliseconds);
+ 第一个参数：互斥量句柄
+ 第二个参数：时间间隔
+ 返回值：WAIT_FAILED（函数失败 ）WAIT_OBJECT_0 （指定的同步对象处于有信号的状态 ）WAIT_ABANDONED （拥有一个mutex的线程已经中断了，但未释放该MUTEX）WAIT_TIMEOUT （ 超时返回，并且同步对象无信号 ）

</br>
#### 9.2.4 解锁互斥量
	bool ReleaseMutex(HANDLE hMutex)

</br>
#### 9.2.5 释放互斥量（内核对象）
	CloseHandle(HANDLE hMutex) 

