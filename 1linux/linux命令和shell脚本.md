# linux结构

![wps9308.tmp](picture/wps9308.tmp.jpg)

内核：是系统的心脏，是运行程序和管理像磁盘和打印机等硬件设备的核心程序。

Shell：是系统的用户界面，提供了用户和内核进行交互操作的一种接口。它接收用户输入的命令并把它送入内核去执行，是一个命令解释器。但它不仅是命令解释器，而且还是高级编程语言，shell编程。

FILE SYSTEMS(文件系统)：文件系统是文件存放在磁盘等存储设备上的组织方法，Linux支持多种文件系统，如ext3,ext2,NFS,SMB,iso9660等

应用程序：标准的Linux操作系统都会有一套应用程序例如X-Window,Open Office等



| 目录        |                                                              |
| ----------- | ------------------------------------------------------------ |
| /bin        | 存放二进制可执行文件(ls,cat,mkdir等)，常用命令一般都在这里。 |
| /etc        | 存放系统管理和配置文件                                       |
| /home       | 存放所有用户文件的根目录，是用户主目录的基点，比如用户user的主目录就是/home/user，可以用~user表示 |
| /usr        | 用于存放系统应用程序，比较重要的目录/usr/local 本地系统管理员软件安装目录（安装系统级的应用）。这是最庞大的目录，要用到的应用程序和文件几乎都在这个目录。 /usr/x11r6 存放x window的目录 /usr/bin 众多的应用程序  /usr/sbin 超级用户的一些管理程序  /usr/doc linux文档  /usr/include linux下开发和编译应用程序所需要的头文件  /usr/lib 常用的动态链接库和软件包的配置文件  /usr/man 帮助文档  /usr/src 源代码，linux内核的源代码就放在/usr/src/linux里  /usr/local/bin 本地增加的命令  /usr/local/lib 本地增加的库 |
| /opt        | 额外安装的可选应用程序包所放置的位置。一般情况下，我们可以把tomcat等都安装到这里。 |
| /proc       | 虚拟文件系统目录，是系统内存的映射。可直接访问这个目录来获取系统信息。 |
| /root       | 超级用户（系统管理员）的主目录（特权阶级^o^）                |
| /sbin       | 存放二进制可执行文件，只有root才能访问。这里存放的是系统管理员使用的系统级别的管理命令和程序。如ifconfig等。 |
| /dev        | 用于存放设备文件。                                           |
| /mnt        | 系统管理员安装临时文件系统的安装点，系统提供这个目录是让用户临时挂载其他的文件系统。 |
| /boot       | 存放用于系统引导时使用的各种文件                             |
| /lib        | 存放跟文件系统中的程序运行所需要的共享库及内核模块。共享库又叫动态链接共享库，作用类似windows里的.dll文件，存放了根文件系统程序运行所需的共享文件。 |
| /tmp        | 用于存放各种临时文件，是公用的临时文件存储点。               |
| /var        | 用于存放运行时需要改变数据的文件，也是某些大文件的溢出区，比方说各种服务的日志文件（系统启动日志等。）等。 |
| /lost+found | 这个目录平时是空的，系统非正常关机而留下“无家可归”的文件（windows下叫什么.chk）就在这里 |



# linux命令

Linux中的命令严格区分大小写的！



### 区别

```
linux命令 和 shell 区别：

shell相当于经过装饰的命令行，和命令行一样，都能操作linux。

但是shell是面向过程的，相当于有了一定的逻辑和过程，而命令行只是单一的操作。
```



### 格式

```
命令名称 [命令参数] [命令对象]


命令名称：有很多，我们会学习其中重要的一些

命令参数：可选，有两种格式：
		长格式：--help
		短格式：-h

命令对象：可选，就是命令作用的目标，可以是文件、目录、URL等等
```



### 基本命令

```sh
cd /bin							切换目录

pwd								显示当前工作目录
ls								列出目录内容
ll								列出目录详细内容
ls -a							列出目录隐藏文件

mkdir aaa						创建目录
mkdir -p aaa/bbb/ccc			创建多级目录
touch 1.txt						创建文件
rm -rf 2.txt 					强制递归删除目录或文件

cp 1.txt aaa/					复制文件
cp -r aaa/ bbb/					复制目录
mv 1.txt ccc/					剪切文件或目录
mv 2.txt 3.txt					重命名

tar -cvf ddd.tar ddd			打包
tar -zcvf ddd.tar.gz ddd		压缩
tar -xvf ddd.tar -C aaa			解包到aaa
tar -zxvf ddd.tar.gz			解压缩在当前路径

echo hello						打印
cat 3.txt						查看文件全部内容
more 3.txt						查看文件内容，可翻页
tail -10f						动态显示最后10行
grep 关键字 文件名 --color 		查找关键字并高亮显示	

vim 3.txt						编辑文件
按i							   编辑模式
:q								退出
:wq								保存退出
:q!								强制退出
/文本							   查找，按n下一个

cat 2.txt >> 3.txt && cat 3.txt	追加写入
cat 2.txt > 3.txt && cat 3.txt	覆盖写入
管道 “|”						   将一个命令的输入变成另一个命令的输入
```



### 系统命令

```
date							显示当前系统时间
df –h							显示磁盘信息
free –m							显示内存状态
uname –a						显示系统信息
who								显示登入的用户。

top								任务管理器
ps –ef | grep ssh				查找进程
kill -9 2868					强制杀死进程 

halt -p							关机
reboot 							重启

ln -s 源文件地址 联接地址		  创建软连接(快捷方式)
```



### 网络命令

```bash
ifconfig						显示网络
ping 192.168.56.1				查看网络是否通畅
netstat -an | grep 3306			查看端口占用情况
service network restart			重启网络服务
```



### 权限管理

![1534510730127](picture/1534510730127.png)

```
chmod 777 1.txt					更改文件权限
```



### 用户管理

root用户

```
useradd lisi					新增用户
passwd lisi						设置密码

vim /etc/profile				配置普通用户的权限
```

普通用户

```
visudo							打开普通用户sudo权限文件
输入lisi  ALL=(ALL)   ALL		   lisi即可暂时拥有root权限

有sudo权限后，即可用sudo+命令暂时有root权限
sudo ls 
```

切换用户

```
su root
su lisi 需要输入lisi的密码
```



### 环境变量

显示环境变量

```
`export`命令显示当前系统定义的所有环境变量
`echo $PATH`命令输出当前的`PATH`环境变量的值
```



当前终端暂时有效，当前用户

```
export PATH=/home/uusama/mysql/bin:$PATH
```



所有终端永久有效，当前用户

```
vim ~/.bashrc

export PATH=$PATH:/home/uusama/mysql/bin

source ~/.bashrc
```



所有终端永久有效，当前用户

```
vim ~/.bash_profile

export PATH=$PATH:/home/uusama/mysql/bin

source ~/.bash_profile
```



所有终端永久有效，所有用户

```
vim /etc/bashrc

export PATH=$PATH:/home/uusama/mysql/bin

source /etc/bashrc
```



所有终端永久有效，所有用户

```
vim /etc/profile

export PATH=$PATH:/home/uusama/mysql/bin

source /etc/profile
```



所有终端永久有效，所有用户

```
vim /etc/profile

export PATH=$PATH:/home/uusama/mysql/bin

source /etc/environment
```



环境变量分类

```
用户级别环境变量定义文件：`~/.bashrc`、`~/.profile`（部分系统为：`~/.bash_profile`）

系统级别环境变量定义文件：`/etc/bashrc`、`/etc/profile`(部分系统为：`/etc/bash_profile`）、`/etc/environment`
```



Linux加载环境变量的顺序如下：

1. `/etc/environment`
2. `/etc/profile`
3. `/etc/bash.bashrc`
4. `/etc/profile.d/test.sh`
5. `~/.profile`
6. `~/.bashrc`



# shell脚本

Bash 也是大多数Linux 系统默认的 Shell。



vim aaa.sh

```bash
#!/bin/bash
echo "hello linux"

# 注释
```

sh ./aaa.sh



### 变量

```bash
#定义
#不能有空格
#首字符为字母
your_name="itcast.cn"

#使用
echo $abc

#只读变量
readonly variable_name

#删除变量
unset variable_name
```



### 参数传递

```bash
#!/bin/bash

echo "脚本名称:"$0

echo $1

echo $2

echo $3

echo "参数个数为： $#";

echo "传递的参数作为一个字符串显示： $*";
```

sh ./aaa.sh  aaa  bbb  ccc



### 运算符

算术表达式

```bash
#字符串
echo 'expr $a + $b'
#运算表达式
echo `expr $a + $b`
echo `expr $a - $b`
echo `expr $a \* $b`
echo `expr $a / $b`
#$[]也可以四则运算
d=$[$a+10]
```

条件表达式

```bash
EQ 就是 EQUAL等于

NQ 就是 NOT EQUAL不等于 

GT 就是 GREATER THAN大于　 

LT 就是 LESS THAN小于 

GE 就是 GREATER THAN OR EQUAL 大于等于 

LE 就是 LESS THAN OR EQUAL 小于等于
```



### 引号

字符串拼接不用+，直接挨着写一起即可。

单引号里面的东西都是字符串，双引号里面用$可以引用变量，嵌套引号最外层起作用

```shell
vim test.sh 

#!/bin/bash
do_date=$1
echo '$do_date'
echo "$do_date"
echo "'$do_date'"
echo '"$do_date"'
echo `date`

test.sh 2020-06-14
```



### 流程控制

选择

```bash
#!/bin/bash
a=10
b=20

if [ $a -eq $b ]
then
  echo "a = b"
else
  echo "a != b" 
fi
```

循环

```bash
#!/bin/bash
for m in `ps -ef`
do
  echo $m
done

while true
do
  sleep 1
done
```



### 函数

```bash
#!/bin/bash

function func(){
  echo "hello world"
  echo "第一个参数: $1"
  echo "第二个参数: $2"
  return  $1
}

func $1 $2
```

































