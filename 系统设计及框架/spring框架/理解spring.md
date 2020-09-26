# 理解spring

## AOP

### 基本代理模式

#### jdk动态代理

**ProxyClassFactory**
ProxyClassFactory是Proxy的一个静态内部类。它的逻辑包括了下面三步：

包名的创建逻辑
		包名生成逻辑默认是com.sun.proxy，如果被代理类是non-public proxy interface（也就说实现的接口若不是public的，报名处理方式不太一样），则用和被代理类接口一样的包名，类名默认是$Proxy 加上一个自增的整数值
		调用ProxyGenerator. generateProxyClass生成代理类字节码
-Dsun.misc.ProxyGenerator.saveGeneratedFiles=true 这个参数就是在该方法起到作用，如果为true则保存字节码到磁盘。代理类中，所有的代理方法逻辑都一样都是调用invocationHander的invoke方法（上面源码我们也能看出来）
把代理类字节码加载到JVM。
		把字节码通过传入的类加载器加载到JVM中:defineClass0(loader, proxyName,proxyClassFile, 0, proxyClassFile.length);

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package com.sun.proxy;

import com.test.aop.Hello;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.UndeclaredThrowableException;

public final class $Proxy0 extends Proxy implements Hello {
    /*
    可以看到 equal tostring  hashcode 都会走代理类
    具体每个代理方法：逻辑都差不多就是 h.invoke，主要是调用我们定义好的invocatinoHandler逻辑,触发目标对象target上对应的方法;
    */
    private static Method m1;
    //只有这个是代理方法
    private static Method m3;
    private static Method m2;
    private static Method m0;
    
    
    static {
        try {
            m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
            m3 = Class.forName("com.test.aop.Hello").getMethod("sayHello");
            m2 = Class.forName("java.lang.Object").getMethod("toString");
            m0 = Class.forName("java.lang.Object").getMethod("hashCode");
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }

    public $Proxy0(InvocationHandler var1) throws  {
        super(var1);
    }

    public final boolean equals(Object var1) throws  {
        try {
            return (Boolean)super.h.invoke(this, m1, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    public final void sayHello() throws  {
        try {
            super.h.invoke(this, m3, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final String toString() throws  {
        try {
            return (String)super.h.invoke(this, m2, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final int hashCode() throws  {
        try {
            return (Integer)super.h.invoke(this, m0, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    
}

```



#### Cglib字节码代理