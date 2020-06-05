---
title: "Effective Java 1"
date: 2020-06-04T20:53:58+08:00
draft: false
categories: ["Notes"]
tags: ["Java"]
---

"Effective Java"的第一部分(对象的创建与销毁)总结.

### 用static的方法(静态工厂方法)代替构造函数创建对象

**优点:**

- 可以有更合理的方法名;
- 可以重复的共用某些对象(节省开销, 不可变, 实例控制);
- 可以返回子类型(接口的实现类, 基于接口, 动态创建类), 更丰富的实现控制.

**缺点:**

- 类无法被继承;
- 是普通的static方法, 难以区分.

**解决方案:**

使用更有意义的名字, 并在文档中强调, 如`from`, `of`, `valueOf`, `instance`, `instanceOf`, `create`, `newInstance`, `get[Type]`, `new[Type]`.

### 当有许多构造参数时, 考虑使用*Builder*

```java
public class A {
  private int a;
  // ... more fields
  
  public static class Builder {
    private int a = 0;
    
    public Builder a(int val) {
      a = val;
      return this;
    }
    // ... more fields
    
    public A build() {
      return new A(this);
    }
  }
  
  private A(Builder builder) {
    a = builder.a;
    // ... more fields
  }
}
```



*Builder*是对Java没有可选参数的一种处理方式, 当一个类的构造参数非常多时(一般大于4个), 且有一些可选参数, 相比于为每个数量的参数定义构造函数或者[*JavaBeans*](https://en.wikipedia.org/wiki/JavaBeans)的定义模式, Builder能够有效结合两种方法的优点.

**优点:**

- 可读性好, 同时相较于*JavaBeans*, 不再限制对象必须是可变的;
- 灵活性高, 能够适配继承.

**缺点:**

- 额外的开销, 创建对象前必须创建*Builder*;
- 相比于简单的定义多构造函数, *Builder*更加复杂.

### 创建单例对象的3种方法

1. 用一个*final*的公有成员

   ```java
   public class A {
     public static final A INSTANCE = new A();
     
     private A() {...}
   }
   ```

   

2. 用一个*static*的工厂方法

   ```java
   public class A {
     private static final A INSTANCE = new A();
     private A() {...}
     
     public static A getInstance() {
       return INSTANCE;
     }
   }
   ```

   

3. 用`enum`

   ```java
   public enum A {
     INSTANCE;
     ...
   }
   ```

   

这边主要讨论第三种, 利用`Enum`来实现单例模式, 这种方式能够免费的提供序列化, 同时也是线程安全的, 单例对象在第一次被引用时由JVM创建. 通常情况下使用enum是最好的选择.

### 用私有的构造函数禁止实例化

一般出现在全是静态方法的类中.

```java
public class Util {
  private Util () {
    // throw new AssertionError();
  }
}
```



### 依赖注入而不是硬编码

### 避免不必要的对象创建

- String直接使用字面量创建;
- 对于频繁创建的对象可以提前创建, 如regex的Pattern对象;
- 避免频繁的装箱拆箱.

### 使用*try-with-resource*代替*try-finally*

### 总结

整体看下来并没有什么阻碍, 大多都是先已有的知识, 静态的工厂方法和*Builder*倒是给了我不少灵感, 另外还有一条"避免使用finalizers和cleaners"没有列处来, 因为没有必要.