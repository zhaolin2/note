## 文件

### ll

 d rwxr-xr-x 

第一位

- -文件
- d 文件夹
- l 链接

后边分别表示  u g o 

root group others

### chmod

chmod 命令用来修改文件权限

\# chmod [ugoa] [+-=] [rwx] dirname/filename

用数字来设定权限

r : 4、w : 2、x : 1

4 = 100

2 = 010

1 = 001

实际上是按二进制取1的位来设置的权限

chmod 777 test.txt

7 = 111, 给test拥有者、所属群组、其他人所有权限



u：拥有者
g：所属群组
o：其他人
a：所有人
+：添加权限
-：移除权限
=：设定权限

为 test.txt 文件的所有用户添加读权限。
chmod a+r test.txt



显示操作系统相关信息

```
uname -a 显示系统的全部信息
-n  nodename 显示计算机1名
-r release 发行版本号
-s sysname 操作系统名称

uname
-a或--all 　显示全部的信息。
-m或--machine 　显示电脑类型。
-n或-nodename 　显示在网络上的主机名称。
-r或--release 　显示操作系统的发行编号。
-s或--sysname 　显示操作系统名称。
-v 　显示操作系统的版本。
--help 　显示帮助。
--version 　显示版本信息。

who 可以查询当前登录在系统上的登录用户的信息
who am i 等同于 who -m，只打印执行该命令的登录用户的信息
whoami 可以查询当前有效用户的名字
```

\>: 重定向符号，通常用于**输入输出**到文件

|: 管道符号，用于两个**程序**之间输入输出的连接

find:查找文件或目录

grep:在文件中查找字符串，语法:grep 字符串 文件名

tar

```
tar  -zxvf  filename.tar.gz -C /usr/local
tar zcvf /zzz -C /usr/local/zzz
```

**z ：表示 tar 包是被 gzip 压缩过的 (后缀是.tar.gz)，所以解压时需要用 gunzip 解压 (.tar不需要)**

**x ：表示 从 tar 包中把文件提取出来**

**v ：表示 显示打包过程详细信息**

**f ：指定被处理的文件是什么**

 **c ：表示创建一个新的打包文件** 

**- ：适用于参数分开使用的情况，连续无分隔参数不应该再使用（所以上面的命令不标准）**

-C 到指定文件夹

 三个查看文件 

 cat 一次性将文件内容全部输出 

more 可以分页查看 

less 使用光标向上或向下移动一行 

### chmod





D 答案：Java 进程异常消失，这个明显不对的。

 java采用的uincode编码，两个字节表示一个字符，因此 char型在java中占两个字节，而int型占四个字节，故总共占四个字节 

 静态方法属于静态绑定，编译器根据引用类型所属的静态类型为它绑定其对应的方法。此语句会翻译成invokestatic，该指令的调用中不会涉及this,所以不会依赖对象！ 还有引用类型=null，其实就是指该引用在堆中没有对应的对象，但是编译的时候还是能根据声明找到其所属的静态类型。 



 静态绑定
　　静态绑定（前期绑定）是指：在程序运行前就已经知道方法是属于那个类的，在编译的时候就可以连接到类的中，定位到这个方法。 

动态绑定（后期绑定）是指：在程序运行过程中，根据具体的实例对象才能具体确定是哪个方法。
　　动态绑定是多态性得以实现的重要因素，它通过方法表来实现：每个类被加载到虚拟机时，在方法区保存元数据，其中，包括一个叫做 方法表（method table）的东西，表中记录了这个类定义的方法的指针，每个表项指向一个具体的方法代码。如果这个类重写了父类中的某个方法，则对应表项指向新的代码实现处。从父类继承来的方法位于子类定义的方法的前面。



 passwd [选项] 用户名 

例如，我们使用 root 账户修改 lamp 普通用户的密码，可以使用如下命令：

[root@localhost ~]#passwd lamp

## 防火墙

```shell
systemctl stop firewalld.service      #停止firewall
systemctl disable firewalld.service    #禁止firewall开机启动 
>>>开启端口

firewall-cmd --zone=public --add-port=80/tcp --permanent
>>>重启防火墙

firewall-cmd --reload
```

## java调试

