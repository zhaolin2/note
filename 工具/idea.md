## 调试

### 计算表达式

在调试的计算表达式中

- 可以用来计算代码中的方法返回值或者校验数据

- 可以直接通过set来改变变量的值

### 断点条件设置

可以设置condition 为true 一般就是==或者equal

### 异常拦截

可以添加全局特定异常处理 自动定位到异常行

### 回退断点

Drop Frame 应该是回退一个栈帧

回退到上一个方法调用的地方，不过只能回退流程，不能回退对象和集合的更新

### 中断debug

中断debug可以强制返回 force Return

### 多线程调试

设置拦截级别为线程级别，以单个线程为级别进行拦截

