---
layout: post
title: "处理对象间一对多关系的技巧"
subtitle: "观察者模式"
date: 2020-10-19 10:00:00
author: "Echcz"
header-style: "text"
catalog: true
tags:
  - "设计模式"
---
{% raw %}
## 前言

对象间关系不仅体现在数据结构上，也体现对象的行为上，或者更进一步的说体现在程序设计上。而对象间关系中，一对多的关系是非常普遍的。我们常常会面对这种情况：一个对象发生的某种改变会引发另外一些对象的响应，并且这些响应之间可能并不相同。

我们如何才能对这种情况进行编程呢？试想我们使用`if else`的方式：当需要让另外一些对象做出响应时，我们需要判断那些对象的状态或类型，然后根据这些信息做出相应的动作。如果对象的状态或类型的种类很多呢，我们的`if else`的区域就会非常多，并且当需要对新的状态或类型做出新的响应时，就需要对那个主体对象的代码做出修改，这就违背了开放封闭原则。试想一下在一个大段的`if else`中修改代码的情景，多么可怕。

## 观察者模式

大段的`if else`总是预示着不好的代码，这种代码难以阅读与维护。我们现在想办法消除这个可怕的`if else`结构。上述的`if else`结构的目的就是对不同响应的行为做分区。这样的话，我们对响应的行为进行抽象，就可以消除掉这个`if else`结构了。我们将这个抽象后的响应者称之为`观察者(Observer)`，因为它观察`主体(Subject)`对象，然后做出自己的响应。代码表示如下：

``` java
interface Observer {
    void update(Object state);
}

abstract class Subject {
    @Getter @Setter
    private Object state;
    private List<Observer> observers = new ArrayList<>();

    public void attach(Observer observer) {
        observers.add(observer);
    }

    public void detach(Observer observer) {
        observers.remove(observer);
    }

    public void nodifyObservers() {
        for (Observer observer : observers) {
            observer.update(this.state)
        }
    }
}

class ConcreteSubject extends Subject {
    public void change(Object state) {
        setState(state);
        nodifyObservers();
    }
}

class AObserver implements Observer {
    @Override
    public void update(Object state) {
        // do something
    }
}
```

通过抽象与多态，我们成功的消除了`if else`了。不同的响应封装在不同的实现类中，代码变得非常明晰。并且如果我们需要有新的响应，只需要添加新的`Observer`实现类，代码变得非常容易拓展与维护。

### 推模式与拉模式

上面的`Observer.update()`方法的接收参数是`state`。这被称之为`推模式`，因为主体的状态是由主体主动推送给观察者的。当主体的状态很简单时推模式是不错的，但当主体的对象的状态很多且很复杂时就不适用了。这时可以使用`拉模式`，让观察者按需自己取其依赖的状态。将推摸式改为拉摸式很简单，只需让`Observer.update()`方法的接收参数是`Subjct`本身就行了。

### JAVA核心库提供的观察者模式

在JAVA语言的`java.util`库里面，提供了一个`Observable`类(主体)以及一个`Observer`接口(观察者)，构成JAVA语言对观察者模式的支持。

与上述模式不同的是，JAVA核心库提拱的观察者模式中`Observable`有一个`changed`变量用于指示其状态是否发生了改变，只有当changed为`true`时，`notifyObservers()`方法才会通知注册的观察者。

## 响应式编程

如果你了解响应式编程，就会发现它也使用了观察者模式。当向`Stream`发布新事件时，会把这个事件通知给注册的观察者，然后观察者会调用相应的方法处理这个事件。其实观察者模式广泛的应用到异步编程中，而不仅仅是在响应式编程中用到。
{% endraw %}