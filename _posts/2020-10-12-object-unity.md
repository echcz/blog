---
layout: post
title: "对象单一性的探索"
subtitle: "单例模式与单态模式"
date: 2020-10-12 10:12:00
author: "Echcz"
header-style: "text"
catalog: true
tags:
  - "设计模式"
---
{% raw %}
## 前言

一般来说，类与他们的实例通常是一对多的关系，即一个类有多个不同的实例。但有一些类，它们就应该只有一个实例，或者至少看起来他的实例们表现得像是同一个。这个实例往往应当在程序启动时被创建出来，并且只在程序结束时才被删除。它们通常是这个程序的基础对象，负责生成、管理和协调程序的其它对象。如果这个基础对象可以有多个副本，那么很可能导致严重的逻辑错误。例如每个副本只是管理了整个程序对象的子集，而不是全部，这样使用某个副本可能无法访问到本在程序中的某个对象。

不过强行对象单一性的机制似乎有些多余，我们完全可以在代码之外做一个约定，只创建某类的一个实例。事实上，这是有道理的，尤其是在没有急迫并且有意义的需要时。但"源代码就是设计"，如果强制对象的单一性的机制是轻量级，那么传达设计意图带来的收益就是超过实施这些机制的代价。

##　实现方法: JAVA描述

### 使用静态类?

关于如何保证对象单一性，我们可能会被诱导使用静态类，即只有静态成员和方法的类，毕竟类天然就是"单例"的。但它有很严重的局限，因为类不是对象，所以不能被传递与被持有。并且使用静态类，也不能通过继承达到对代码的复用和享用多态的好处。所以我们往往需要的是一个真正的对象，而不是静态类。

### 使用单例模式

#### 私有化构造器

单例模式说白了，就是如何控制构造器的使用。所以私有化构造器，然后只暴露一个访问实例的端点，这个端口只保证实例化一个实例，就可以达成目标了。

``` java
public class Singleton {
    private static Singleton instance = null;

    private Singleton() {}

    public static Singlention getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

这是一个很简单且轻量的实现，现在只能通过`getInstance()`静态方法才能获取到实例，这个方法会先检查，如果已经有实例了，就直接返回这个实例。不过这个实现有一个问题，那就是在并发访问时，可能导致生成多个实例，因为`getInstance()`不是一个原子操作。当然可以使用`synchronized`关键词修饰`getInstance()`方法，但这样太粗暴了，会让此方法在并发程序中成为一个低性能操作。仔细分析下，其实我们需要让其串行化的只有创建当还没有创建实例的时候，如果已经创建实例了，就直接返回该实例，没有共享数据的修改，不需要做串行化。我们可以使用一个叫`双重检查(double-check)`的技术来实现优化。

``` java
public class Singleton {
    private static volatile Singleton instance = null;

    private Singleton() {}

    public static Singlention getInstance() {
        if (instance == null) {
            synchronized(Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

这样如果`instance`已经被赋值，那么就不会进入串行化代码中。仍然需要在串行化代码中再次检查`instance`是否为null，因为整个原子操作就是如果`instance`没有被赋值，就创建一个实例赋值给它。注意，`instance`静态成员变量被`volatile`修饰了，这里主要是为了禁止`getInstance()`方法出现`指令重排`，防止将一个未完成初始化的对象返回出去。

除了使用`synchronized`做并发控制，我们完全可以用静态变量无论如何只会初始化一次的机制来实现让instance只会赋值一次的目标。

``` java
public class Singleton {
    private static final Singleton instance = new Singleton();

    private Singleton() {}

    public static Singlention getInstance() {
        return instance;
    }
}
```

当然，我们也可以让`instance`变成全局常量，即使用`public static final`修饰，这样就不需要`getInstance()`方法了。不过使用这种方法是用空间换时间，无论是否访问`Singleton`实例，都需要实例化一个`Singleton`对象。

#### 使用枚举

枚举是 JDK5 才出现的特性。枚举从JAVA语法上保证了类的实例数量固定。如果我们让枚举的个数只有一个，不就是单例了吗？

``` java
public enum Singleton {
    INSTANCE;
}
```

对的，就是这个简单，并且它也是并发安全的。不过枚举是不能继承其它的类的，因为枚举类已经继承了`java.lang.Enum`类了，这是使用此方法的一个局限。

### 使用单态模式(Monostate)

单态模式是另外一种获取对象单一性的方法，虽然他并没有保证全局只有一个实例。他获取对象单一性的思路是，只要让实例间的状态是共享的，那么从逻辑上说，也就获取到了对象单一性的。

``` java
public class Monostate {
    private static State state; // 使用静态成员变量保存状态

    public void fun() {
        // 访问状态
    }
}
```

通过让静态成员变量保存状态，从而实现了实例间的状态共享。这些实例虽然是不同的，但他们的行为总是表现得就像是同一个一样。和使用单例模式相比，使用单态模式有一个好处是，可以继承单态类。你是不能断承单例类的，因为单例类的构造器是私有的(使用枚举的实现构造器同样也是私有的)。不过每个单态类的实例都是真正的对象，所以会导致更多创建和销毁对象的开销。
{% endraw %}