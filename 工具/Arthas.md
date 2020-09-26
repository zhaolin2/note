# Arthas

## 官方文档

官方文档  https://alibaba.github.io/arthas/advanced-use.html 

## 运行

```shell
as.bat <pid>
```

## dashboard

## thread  < pid >

这个id一般是dashboard中显示的

显示出来堆栈

##  当前最忙的前N个线程并打印堆栈 

```shell
thread -n 3
```

## 查看加载的类的信息

- SC  search class



```shell
# 模糊搜索
$ sc demo.*

# 打印详细信息
$ sc -d demo.MathGame

# 打印出类的Field信息
$ sc -d -f demo.MathGame
```

##  查看已加载类的方法信息 

- SM search method

```shell
# 查看所有方法信息
$ sm java.lang.String

# 查看单个方法
$ sm -d java.lang.String toString
```

## 反编译

```shell
# 查看反编译信息 (CLassLoader+Location+Source)
$ jad java.lang.String

# 只显示源码
$ jad --source-only demo.MathGame

# 反编译指定函数
$ jad demo.MathGame main
```

## Trace

```shell
# 指定方法
$ trace demo.MathGame run

# 如果方法调用的次数很多，那么可以用-n参数指定捕捉结果的次数。比如下面的例子里，捕捉到一次调用就退出命令。
$ trace demo.MathGame run -n 1

# 可以看是否跳过jdk的函数
--skipJDKMethod <value>
$ trace --skipJDKMethod true demo.MathGame run

# 耗时查询  只会展示耗时大于10ms的调用路径
$ trace demo.MathGame run '#cost > 10'

# 跟踪多个的路径
$ trace -E com.test.ClassA|org.test.ClassB method1|method2|method3
```

## 观察调用情况

-  `返回值`、`抛出异常`、`入参` 

```shell
# 在方法执行前监控：-b，遍历深度：-x 2
$ watch cn.javastack.springbootbestpractice.web.JsonTest getUserInfo '{params, returnObj}' -x 2 -b
```

## Stack

```shell
$ stack demo.MathGame primeFactors

# 据条件表达式来过滤
$ stack demo.MathGame primeFactors 'params[0]<0' -n 2

# 据执行时间来过滤
$ stack demo.MathGame primeFactors '#cost>5'
```

