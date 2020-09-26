# 根据端口杀线程

```powershell
## 查看特定端口
netstat -ano | findstr "8081"

## 查看特定线程的详细信息(启动信息)
tasklist | findstr 14396

# 杀掉特定pid
taskkill /pid 14396  -t  -f
```

# 输出特定环境变量

```powershell
# 需要进入命令行才可以查看  powershell是无法输出的
window+r 输入cmd

echo %PATH%
```

# Jdk

```powershell
JAVA_HOME
D:\JavaSourceComplier\jdk1.7.0_80

CLASSPATH
.;%JAVA_HOME%/lib/dt.jar;%JAVA_HOME%/lib/tools.jar

PATH
%JAVA_HOME%\bin;
%JAVA_HOME%\jre\bin;
```

# 命令

## nc

监听特定端口

```shell
nc -l -p 9900
```

## telnet

发起tcp连接

```shell
telnet localhost 8080

# ctrl+]回显内容
```



# 小技巧

## 设置每个窗口独立输入法

设置搜索 

输入法 

会有高级键盘设置 允许我为每个应用窗口试用不同的输入法的选项

## 资源管理器设置为打开此电脑

文件管理器

- 查看
- 选项  更改文件夹和搜索选项
- 更改即可

## 添加此电脑 回收站 控制面板

- 设置
- .。。。。个性化
- 主题
- 桌面图标设置

## 窗口颜色

主要是为了更快的分辨前台页面

- 设置
- 个性化
- 。颜色
- 标题栏和窗口颜色

## 搜索栏

- 搜索框 
- 右键
- 搜索  
  - 显示搜索图标

## 输入法可以设置哪种语言优先

- 设置
- 时间和语言
- 语言
- 设置首选语言