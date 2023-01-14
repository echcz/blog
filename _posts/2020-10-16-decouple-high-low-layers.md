---
layout: post
title: "分离高层算法与低层实现细节的技巧"
subtitle: "从策略模式到状态模式"
date: 2020-10-16 18:00:00
author: "Echcz"
header-style: "text"
catalog: true
tags:
  - "设计模式"
---
{% raw %}
## 前言

当我们在做程序开发时，总能发现程序可以分为高层次部分和低层次部分。高层次部分一般是很稳定的，并且重用度也很高，或者说很通用。而低层次部分则是非常不稳定的，并且往往只是针对某个特定的问题而存在，当问题或需求变化了，我们就需要改变这部分。程序的高层次部分与低层次部分的特性是如此不同，我们如想设计出拥有良好结构的程序，分离这两部分可以说是重中之重。

如何分离高低层次呢？回顾设计模式原则，我们能发现有两条原则可以给我们答案，那就是`开放封闭原则(OCP)`与`依赖倒置原则(DIP)`。通过OCP，我们能封闭对高层次部分的修改，而开放对低层次部分的拓展。通过DIP，我们让低层次去依赖高层次，而不是反之，从而让高层次不受低层次变化的影响。这样我们分别顺着他们的特性去设计程序，让这两部分能独立保持自己的节奏去应对需求与变化。

## 策略模式

思考这样一个问题:

> 从一个数组中找到最大值

这是一个很简单的问题，其算法说明如下:

``` md
1. 比较待定元素与当前元素
2. 如果当前元素比待定元素大，则将当前元素设为待定元素
3. 当下一元素设为当前元素，回到步骤1；如果没有下一元素，则返回待定元素
```

这其实是程序的高层次结构，不管我们要从什么类型的数组中找到最大值，我们都可以使用这个算法。问题出现在，我们如何比较两个元素。不同的需求有不同的比较方法。比如有时需要比较两个数字，而有时需要比较两个人；有时需要根据人的身高进行比较，有时需要根据人的年龄进行比较。我们需要分离取最大值的算法与比较两个元素的细节。这时我们可以这样设计代码：

``` java
// 主要程序结构:

interface Comparator<T> {
    boolean compara(T src, T target);
}

class Algorithm {
    public <T> T getMaxValue(T[] array, Comparator<T> comparator) {
        T result = array[0];
        for (i = 1; i < array.lenght; i++) {
            T current = array[i];
            if (comparator.compara(current, result)) {
                result = current;
            }
        }
        return result;
    }
}

// 根据身高或根据年龄比较人的例子:

@Data
class Person {
    private String name;
    private int height;
    private int age;
}

class HeightComparator implements Comparator<Person> {
    @Override
    public boolean compara(Person src, Person target) {
        return src.getHeight() > target.getHeight()
    }
}

class AgeComparator implements Comparator<Person> {
    @Override
    public boolean compara(Person src, Person target) {
        return src.getAge() > target.getAge()
    }
}

// main方法主要内容:

// max = algrithm.getMaxValue(personArray, new HeightComparator());
// max = algrithm.getMaxValue(personArray, new AgeComparator());
```

通过将比较方法抽象为一个接口，然后让高层算法使用这个接口，低层实现细节实现这个接口，从而实现了分离高低层次，让它们可以独立演化。如果需要新的比较方式，只需要定义新的`Comparator`实现类既可，而`Algorithm`是不需要做任何修改的。

这就是策略模式的核心思想了:

> 将细节抽象化，称之为`策略`，然后让高层算法使用策略，让低层细节实现(继承)策略，从而解耦高层算法与低层细节(此时它们之间没有直接依赖，他们都依赖策略)，使它们可独立演化，让低层部分是可以互换的。

## 状态模式

如果我们更进一步，让策略在执行的过程中可以进行反馈，修改高层算法下一次要使的策略呢？这是不是很像`有限状态机`:

> 有限状态机会在运行的过程中自动从一个状态变为另一个状态，随着状态的改变，其行为也发生改变。

对`策略模式`的进一步拓展，我们就设计出了`状态模式`。为了更好的理解，我们看一下这个简单的案例:

> 有一个门，如果我们是关闭状态，我们通过会发出警告并拒绝通过。我们可以念咒语，让他变成开启状态。开启状态下，可以通过，但通过后会变成关闭状态。

这是一个简单的有限状态机，其有两个状态(开启与关闭)，两个行为(念咒语与通过)，总共对应4个结果:

1. 开启-念咒语: 无影响
2. 开启-通过: 正常通过，变为关闭状态
3. 关闭-念咒语: 变为开启状态
4. 关闭-通过: 拒绝通过，并且发出警告

其代码如下:

``` java
interface State {
    void pass(Door door);
    void mantra(Door door);
}

class Door {
    private final State openedState;
    private final State closedState;
    private State currentState;

    public Door(State openState, State closedState) {
        this.openedState = openState;
        this.closedState = closedState;
        this.currentState = this.openedState;
    }

    public boolean isOpened() {
        return currentState == openedState;
    }

    public void open() {
        currentState = openedState;
    }

    public void close() {
        currentState = closedState;
    }

    public void pass() {
        currentState.pass(this);
    }

    public void mantra() {
        currentState.mantra(this);
    }
}

class OpenedState implements State {
    @Override
    public void pass(Door door) {
        System.out.println("欢迎通行!");
        door.close();
    }
    @Override
    public void mantra(Door door) {
        // do nothing
    }
}

class ClosedState implements State {
    @Override
    public void pass(Door door) {
        System.out.println("非法入侵，禁止通行!");
    }
    @Override
    public void mantra(Door door) {
        door.open();
    }
}

class Main {
    public static void main(String[] args) {
        Door door = new Door(new OpenedState(), new ClosedState());
        door.pass()
        door.mantra()
    }
}
```

这里让`State`也依赖具体类`Door`了，如果希望完全遵循DIP，让细节依赖抽象，可以抽取`Door`的`open()`与`close`方法，形成一个接口，让Door实现它。但我认为目前是没必要的，因为`Door`和`State`同处于高层次中，除非我们发现门也会有多种实现，或者其它的类也需要复用`State`。

虽然增加了反馈链，程序结构也复杂了，但高低层次仍然是分离的，低层次的实现细节仍然是可拓展、可互换的，我们想要增加新的状态或改变某状态下的行为也非常容易实现。

### 后记

分离高层算法与低层实现细节的技巧不仅是策略模式和状态模式，策略模式和状态模式也不仅就是上文所展示的形式，也可以有多种变化与调整。关键是要分析业务与领域，将其分解为高低层次部分，然后根据开放封闭原则(OCP)与依赖倒置原则(DIP)的指引将它们解耦，让它们能独立演化。这样我们能够设计出具有重用性的同时还能保持灵活性的，可拓展的程序。
{% endraw %}