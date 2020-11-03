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
</br>

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
</br>
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


#### 共享内存
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

#### 信号量

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


## others
+ `tail -f <filename>` 实时查看文件
+ `grep <name>` 搜索过滤