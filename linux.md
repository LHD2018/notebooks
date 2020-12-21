## 1. 内核操作

+ 常用命令
~~~bash
uname -r 	# 查看当前内核

dpkg -l | grep linux-image		# 列出当前所有内核

sudo apt-get purge --remove linux-image-x.x.0-xx-generic # 删除内核

~~~

### 1.1 安装内核

+ 去[ubuntu仓库](https://kernel.ubuntu.com/~kernel-ppa/mainline/)下载合适的内核，包括4个.deb文件，如图所示：
![](https://i.loli.net/2020/08/30/kz7MUWolSegmvFH.png)

~~~bash
//进入下载目录执行下面命令
sudo dpkg -i *.deb
~~~


## 2. shell

### 2.1 入门
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
</br>

### 2.2 变量
~~~bash


name=value	# 等号两边都不能有空格，如需要则进行转义

export name 	# 将变量设为环境变量

${name} 	# 引用变量

$(command) 	# 在变量中使用命令，但需要在双引号中，单引号就是字面文字

unset name		# 取消变量

declare [-aixr] name 	# 声明变量，a：数组，i：整形，x：环境变量，r：只读




~~~

## 3. 进程

### 3.1 进程常用函数
+ `getpid()` 获取当前进程id
+ `fork()`函数用来产生一个新的进程，在父进程中返回子线程id，在子线程中返回0

### 3.2 进程常用命令
+ `ps -ef` 列出所有进程
+ `ps -aL` 查看当前运行的轻量级进程
+ 命令后加`&`变成守护进程，使其后台运行
+ `killall <name>` 杀死进程
+ `kill <pid>` 杀死进程，pid可由`ps -ef | grep <name>`获取
</br>
### 3.3 进程通信
+ 管道（pipe）：无名管道用于父子进程、命名管道无限制
+ 消息队列（message）：进程向队列中添加消息，其他进程读取消息
+ 信号（signal）：通知其他进程有事件发生
+ 共享内存（shared memory）：多个进程共用一块内存
+ 信号量（semaphore）：进程之间对共享资源加锁
+ 套接字（socket）：可用于不同计算机进程

#### 3.3.1 信号
+ `signal(int signum, sighandler_t handler);`
+ + signum:信号编号 
+ + handler:处理方式，可以是自定义信号处理函数，或忽略`SIG_IGN`, 或默认`SIG_DFL`
</br>
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
</br>
+ 发送信号：`int kill(pid_t pid, int sig)`
+ + pid:要发送的进程id（>0：指定进程id，=0：父子进程发送，二者都收到，=-1：广播给系统所有进程
+ + sig：：准备发送的信号代码，假如其值为零则没有任何信号送出
+ + 返回值：成功返回0， 失败返回-1
</br>

#### 3.3.2共享内存
+ 共享内存（Shared Memory）就是允许多个进程访问同一个内存空间，是在多个进程之间共享和传递数据最高效的方式。

+ `int shmget(key_t key, size_t size, int shmflg);`获取或创建共享内存
+ + key:共享内存的编号，唯一且通常用16进制表示
+ + size：共享内存大小
+ + shmflg：访问权限，如0666|IPC_CREAT（所有用户可以访问）
+ + 返回值：返回共享内存标识符
</br>
+ `void *shmat(int shm_id, const void *shm_addr, int shmflg)`;将共享内存连接到当前进程
+ + shm_id:内存的标识符
+ + shm_addr:指定共享内存连接到当前进程中的地址位置，通常为0
+ + shmflg:通常为0
+ + 返回值：一个指向共享内存第一个字节的指针
</br>
+ `int shmdt(const void *shmaddr);` 将共享内存从当前进程分离
+ + shmaddr: 共享内存第一个字节的指针
+ + 返回值：成功返回0， 失败返回-1
</br>
+ `int shmctl(int shm_id, int command, struct shmid_ds *buf);` 删除共享内存
+ + shm_id: 共享内存标识符
+ + command：通常为IPC_RMID
+ + buf：通常为0
</br>
+ 常用命令
~~~bash
# 查看共享内存
ipcs -m
# 删除共享内存
ipcrm -m <共享内存编号>
~~~
</br>
+ 代码示例
~~~c ++
#include <unistd.h>
#include <sys/ipc.h>
#include <sys/shm.h>

using namespace std;


int main(){

    int shmid; // 共享内存标识符
 
    // 创建共享内存，键值为0x5005，共1024字节。
    shmid = shmget((key_t)0x5005, 1024, 0640|IPC_CREAT);
    
    char *ptext=0;   // 用于指向共享内存的指针
    
    // 将共享内存连接到当前进程的地址空间，由ptext指针指向它
    ptext = (char *)shmat(shmid, 0, 0);
    
    // 把共享内存从当前进程中分离
    shmdt(ptext);
    
    // 删除共享内存
    shmctl(shmid, IPC_RMID, 0);

    return 0;
}
~~~
</br>

#### 3.3.3 信号量

+ 信号量（信号灯）本质上是一个计数器，用于协调多个进程（包括但不限于父子进程）对共享数据对象的读/写。

+ 代码示例
~~~c ++
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <errno.h>
#include <sys/ipc.h>
#include <sys/sem.h>
 
class CSEM
{
private:
  union semun  // 用于信号灯操作的共同体。
  {
    int val;
    struct semid_ds *buf;
    unsigned short *arry;
  };
 
  int  sem_id;  // 信号灯描述符。
public:
  bool init(key_t key); // 如果信号灯已存在，获取信号灯；如果信号灯不存在，则创建信号灯并初始化。
  bool wait();          // 等待信号灯挂出。
  bool post();          // 挂出信号灯。
  bool destroy();       // 销毁信号灯。
};
 
int main(int argc, char *argv[])
{
   CSEM sem;
 
   // 初始信号灯。
   if (sem.init(0x5000)==false) { printf("sem.init failed.\n"); return -1; }
   printf("sem.init ok\n");
  
   // 等待信信号挂出，等待成功后，将持有锁。
   if (sem.wait()==false) { printf("sem.wait failed.\n"); return -1; }
   printf("sem.wait ok\n");
 
   sleep(50);  // 在sleep的过程中，运行其它的book259程序将等待锁。
  
   // 挂出信号灯，释放锁。
   if (sem.post()==false) { printf("sem.post failed.\n"); return -1; }
   printf("sem.post ok\n");
  
   // 销毁信号灯。
   if (sem.destroy()==false) { printf("sem.destroy failed.\n"); return -1; }
   printf("sem.destroy ok\n");
}
 
bool CSEM::init(key_t key)
{
  // 获取信号灯。
  if ( (sem_id=semget(key,1,0640)) == -1){
    // 如果信号灯不存在，创建它。
    if (errno==2){
      if ( (sem_id=semget(key,1,0640|IPC_CREAT)) == -1) return false;
 
      // 信号灯创建成功后，还需要把它初始化成可用的状态。
      union semun sem_union;
      sem_union.val = 1;
      if (semctl(sem_id,0,SETVAL,sem_union) <  0) return false;
    }else return false;
  }
 
  return true;
}
 
bool CSEM::destroy()
{
  if (semctl(sem_id,0,IPC_RMID) == -1) return false;
 
  return true;
}
 
bool CSEM::wait()
{
  struct sembuf sem_b;
  sem_b.sem_num = 0;
  sem_b.sem_op = -1;
  sem_b.sem_flg = SEM_UNDO;
  if (semop(sem_id, &sem_b, 1) == -1) return false;
   
  return true;
}
 
bool CSEM::post()
{
  struct sembuf sem_b;
  sem_b.sem_num = 0;
  sem_b.sem_op = 1;  
  sem_b.sem_flg = SEM_UNDO;
  if (semop(sem_id, &sem_b, 1) == -1) return false;
 
  return true;
}
~~~


## 4. 线程
+ 在同一个进程中，可以运行多个线程，运行于同一个进程中的多个线程，它们彼此之间使用相同的地址空间，共享全局变量和对象，启动一个线程所消耗的资源比启动一个进程所消耗的资源要少。

~~~bash
# 查看线程
top -H
ps -xH
# 查看主线程和子线程关系
pstree -p <主线程id>
~~~


### 4.1 线程创建
+ `int pthread_create(pthread_t *thread, const pthread_attr_t *attr,void *(*start_routine) (void *), void *arg);` 创建线程
+ + thread：指向线程标识符的地址
+ + attr：线程属性，一般为NULL
+ + start_routine：线程执行函数名
+ + arg：线程函数的参数

</br>
### 4.2 结束线程
+ 函数执行完毕，自动消亡
+ 调用`pthread_exit(0)`结束
+ 被主线程或其他线程停止

### 4.3 线程资源回收
+ 线程有joinable和unjoinable两种状态，创建线程默认joinable，线程函数执行完后不会释放系统资源，这种线程称为“僵尸线程”。
+ 设置线程属性
+ 在开启线程后，在主函数中调用`pthread_detach(pthid);`
+ 在线程函数中调用`pthread_detach(pthread_self());`

### 4.4 线程同步
+ 线程的锁的种类有互斥锁、读写锁、条件变量、自旋锁、信号灯。

#### 4.4.1 互斥锁
+ `int pthread_mutex_init(pthread_mutex_t *mutex,const pthread_mutex_attr_t *mutexattr);` // 初始化锁
+ + mutex:锁标识符
+ + mutexattr：锁属性，null为缺省值，**一般就填0**
+ + + PTHREAD_MUTEX_TIMED_NP：普通锁，缺省值。
+ + + PTHREAD_MUTEX_RECURSIVE_NP，嵌套锁，允许同一个线程对同一个锁成功获得多次，并通过多次unlock解锁。
+ + + PTHREAD_MUTEX_ERRORCHECK_NP，检错锁，如果同一个线程请求同一个锁，则返回EDEADLK，否则与PTHREAD_MUTEX_TIMED_NP类型动作相同。
+ + + PTHREAD_MUTEX_ADAPTIVE_NP，适应锁，动作最简单的锁类型，等待解锁后重新竞争。

+ `int pthread_mutex_lock(pthread_mutex *mutex);` // 阻塞加锁，如果是锁是空闲状态，本线程将获得这个锁；如果锁已经被占据，本线程将排队等待，直到成功的获取锁。
+ `int pthread_mutex_trylock( pthread_mutex_t *mutex);` // 非阻塞加锁，该函数语义与 pthread_mutex_lock() 类似，不同的是在锁已经被占据时立即返回 EBUSY，不是挂起等待。

+ `int pthread_mutex_unlock(pthread_mutex *mutex);` // 释放锁

+ `int pthread_mutex_destroy(pthread_mutex *mutex);` // 销毁锁

## 5. 调用可执行程序
### 5.1 excl函数族
+ `int execl(const char *path, const char *arg, ...);`
+ + path：可执行程序完整路径
+ + arg：命令参数
+ + ...：具体内容，最后要有0
+ + 返回值：成功，不会返回，失败返回-1
+ + **注意：如果执行成功则函数不会返回，当在主程序中成功调用execl后，被调用的程序将取代调用者程序，也就是说，execl函数之后的代码都不会被执行。**
+ 如：`int iret=execl("/bin/ls","/bin/ls","-l","/usr/include/stdio.h",0);`

### 5.2 system函数
+ `int system(const char * string);`
+ + string：命令
+ + 返回值：成功返回shell的返回值，失败返回-1
+ + **注意它的实现其实是fork了一个子进程去执行excl，所以后面的代码还会执行**


## 6. 库文件

### 6.1 静态库
+ 静态库在编译的时候，主程序文件与静态库一起编译，把主程序与主程序中用到的库函数一起整合进了目标文件。这样做优点是在编译后的可执行程序可以独立运行，并且执行速度快。缺点是，如果所使用的静态库发生更新改变，我们的程序必须重新编译，并且内存开销大。

~~~bash
 g++ -c -o libpublic.a public.cpp # 编译库
 g++ -o main main.cpp -L<path> -lpublic  # 使用
~~~

+ 静态库文件名的命名方式是“libxxx.a”,库名前加”lib”，后缀用”.a”，“xxx”为静态库名。
</br>

### 6.2 动态库
+ 动态库在编译时并不会被连接到目标代码中，而是在程序运行时才被载入，因此在程序运行时还需要指定动态库的目录。优点：内存占用小，修改库，程序不用重新编译。缺点：不同平台移植复杂，程序执行时间开销大。

+ 动态库的命名方式与静态库类似，前缀相同，为“lib”，后缀变为“.so” “xxx”为动态库名。

~~~bash
g++ -fPIC -shared -o libpublic.so public.cpp # 编译
g++ -o main main.cpp -L<path> -lpublic  # 使用
# Linux系统中采用LD_LIBRARY_PATH环境变量指定动态库文件的目录。
export LD_LIBRARY_PATH=./lib 	# 如果有多个目录用`：`隔开
~~~

***使用时优先选用动态库***



## 7. gdb调试

+ gcc编译要加`-g`参数，如果用CMake，`CMAKE_BUILD_TYPE`使用debug


|命令|命令简写|说明|
|---|:---:|--|
|set args||设置程序参数|
|break|b|打断点|
|run|r|开始运行程序, 到断点停下来，如果没有遇到断点，程序一直运行下去。|
|next|n|执行当前行，如果当前是函数调用，不会进入函数内部|
|step|s|执行当前行，如果当前行是自定义函数，则进入|
|print|p|打印变量值|
|continue|c|继续运行直到遇到断点或执行完|
|set var name=value||设置变量值|
|quit|q|退出gdb|

</br>
### 7.1 gdb多进程调试
~~~bash
# 调试父进程(default)
set follow-fork-mode parent
# 调试子进程
set follow-fork-mode child

# 设置调试模式(default [on]),off其他进程将会被挂起
set detach-on-fork [on|off]

# 查看调试的进程
info inferiors
# 切换调试进程
inferior [进程id]
~~~
</br>

### 7.2 gdb多线程调试
~~~bash
# 查看线程
info threads
# 切换线程
thread <线程id>

# 只运行[当前/全部]线程
set scheduler-locking [on/off]

# 指定某线程执行某gdb命令
thread apply <线程id> [cmd]
# 全部线程执行某命令
thread apply all [cmd]
~~~


## others
+ `tail -f <filename>` 实时查看文件
+ `grep <name>` 搜索过滤

### `init`
+ 是由内核启动的第一个用户级进程
+ 0：停机或关机
+ 1：单用户模式，只用root维护
+ 2：多用户模式，不能使用NFS（Net File System）
+ 3:完全多用户模式：标准运行状态
+ 4：安全模式
+ 5：图形界面
+ 6：重启

