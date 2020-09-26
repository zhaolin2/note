# String

## 不可变

### 原因

可以实现多个变量使用直接指针引用同一个实例(String Pool)

安全性 保证线程安全

性能 因为不可变性 可以计算一次hash值 存储在内部

​		假如是可变的 每次都要重新计算hash值

### 实现

两个字符串相加的时候，会使用StringBuilder来进行拼接

#### String Pool

```java
//会去常量池找对应的引用 找到就返回 找不到就创建
String test="abc";
String _yun=new String("abc").intern();
//true
System.out.println(test==_yun);
```



```java
字符串连接：
    //表达式只有常量时，编译期完成计算
    //表达式有变量时，运行期才计算，所以地址不一样
   字符串相加的时候  是使用StringBuilder来进行相加
    
intern：
    JDK1.6查找到常量池存在相同值的对象时会返回该对象的地址。
	JDK1.7后，intern方法还是会先去查询常量池中是否有已经存在，如果存在，则返回常量池中的引用，这一点与之前的没有什么区别，区别在于，如果常量池找不到对应的字符串，则不会将字符串拷贝到常量池，而只识在常量池中生成一个对原字符串的引用。
```

# StringBuilder

## append

因为是直接在char[] 里边进行操作，所以比String的更快

```java
abstract class AbstractStringBuilder implements Appendable, CharSequence {
    char[] value;
    int count;
    
    @Override
    public AbstractStringBuilder append(CharSequence s, int start, int end) {
        if (s == null)
            s = "null";
        if ((start < 0) || (start > end) || (end > s.length()))
            throw new IndexOutOfBoundsException(
                "start " + start + ", end " + end + ", s.length() "
                + s.length());
        int len = end - start;
        ensureCapacityInternal(count + len);
        //i表示的是s中的索引位置
        for (int i = start, j = count; i < end; i++, j++)
            value[j] = s.charAt(i);
        count += len;
        return this;
    }
    
}
```

