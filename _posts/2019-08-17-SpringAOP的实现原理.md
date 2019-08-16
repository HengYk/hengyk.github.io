---
layout: post
title: SpringAOP的实现原理
date: '2019-08-16 23:07'
description: "SpringAOP的实现原理"
tag: 源码系列文章（JAVA-CODE）
---

##### 博文参考

[Spring AOP实现原理](https://juejin.im/post/5af3bd6f518825673954bf22)

[动态代理之CGLIB](https://gupeng-ie.iteye.com/blog/1856608)

##### AOP(Aspect Orient Programming)

>         系统是由很多不同组件构成的，每个组件都负责特定的功能，除了自身核心功能外，还有诸如日志、缓存、安全、事务管理等额外的功能，它们称之为横切关注点，相同类型的横切关注点构成了一个切面。
>     
>         SpringAOP实现动态代理的两种方式：JDK动态代理和CGLIB动态代理。

##### 基于反射的JDK动态代理

```java
package cn.edu.xidian.ictt.yk.spring;

public interface Service {
    /**
     * add方法
     */
    void add();
}
```

```java
package cn.edu.xidian.ictt.yk.spring;

public class ServiceImpl implements Service {

    @Override
    public void add() {
        System.out.println("add.......");
    }
}
```

```java
package cn.edu.xidian.ictt.yk.spring;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class Test {

    public static void main(String[] args) {
        // 接口是JDK动态代理的最关键特征
        Service service = new ServiceImpl();
        // 代理类对象，核心是Proxy类和InvocationHandler接口
        Service serviceProxy = (Service) Proxy.newProxyInstance(
                service.getClass().getClassLoader(),
                service.getClass().getInterfaces(),
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        System.out.println("---before---");
                        method.invoke(service, args);
                        System.out.println("---after---");
                        return null;
                    }
                }
        );
        // 调用增强后的add方法
        serviceProxy.add();
    }
}
```

##### 基于继承的CGLIB(Code Generation Library)动态代理

```java
package cn.edu.xidian.ictt.yk.spring;

public class Base {

    public void add() {
        System.out.println("add........");
    }
}
```

```java
package cn.edu.xidian.ictt.yk.spring;

import org.springframework.cglib.proxy.Enhancer;
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

public class TestCGLIB {

    public static void main(String[] args) {

        // 创建enhancer对象
        Enhancer enhancer = new Enhancer();

        // 设置增强类（代理类）的父类对象
        enhancer.setSuperclass(Base.class);

        // 设置回调方法，参数为增强的代理类对象
        enhancer.setCallback(new MethodInterceptor() {
            @Override
            public Object intercept(Object proxy, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                System.out.println("---before---");
                methodProxy.invokeSuper(proxy, objects);
                System.out.println("---after---");
                return null;
            }
        });

        // 创建增强类（代理类）的实例
        Base base = (Base) enhancer.create();

        // 调用增强后add方法
        base.add();
    }
}
```

##### 总结

- **JDK动态代理是基于反射实现的，且只针对实现了接口的业务对象。**

- **CGLIB动态代理是基于继承实现的，可在运行时对目标对象进行子类化。**
