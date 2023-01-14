---
layout: post
title: "协程"
subtitle: "比线程更轻量级的异步概念"
date: 2020-10-23 16:08:00
author: "Echcz"
header-style: "text"
catalog: true
tags:
  - "并发"
  - "异步"
  - "协程"
---
{% raw %}
## 协程是什么

或许你已听说过"协程(Coroutine)"这个词，并且还知道它又名"微线程"。你还会发现在讲述协程时，总是会提到"线程"。但其实协程不是线程，相比于线程其更类似于子例程。我们可以这么说，协程是一个特殊的函数，其可以在某个地方挂起，并且可以重新在挂起处继续运行。虽然协程与进程、线程差别很大，但他们都能实现异步编程，并且协程更加轻量级。

## 协程的优缺点

协程最大的优势就是协程的效率极高。因为协程不是被操作系统管理的，而是完全由程序控制，即是在用户态执行的。协程切换只涉及基本的CPU上下文的切换，简单地说就是把当前协程的CPU寄存器状态保存起来，然后将需要切换进来的协程的CPU寄存器状态加载到CPU寄存器上，其开销相比线程的切换是非常低的。

协程另一个优点是其执行是完全由用户控制的，我们使用起来很容易。相反，线程的执行则是由操作系统控制的，我们无法预测在某个时刻，到底是哪些线程在运行，这时就需要很小心的控制对共享变量的访问。

协程的缺点就是协程只能做到并发，而不能做到并行。在一个进程中，并行的能力是由多线程提供的，一个线程里的多个协程是串行执行的。

## Python中的协程

Python对协程的支持是通过`generator`实现。在`generator`中，我们不但可以通过`for`循环来迭代，还可以不断调用`next()`函数获取由`yield`语句返回的下一个值。

但其实Python的`yield`不但可以返回一个值，还可以接收调用者通过`send()`函数发出的参数。来看一个很简单的案例：

``` python
r = 0

def consumer():
    global r
    while True:
        n = yield
        print('[CONSUMER] consuming {}'.format(n))
        r = r + n

def produce(c):
    c.send(None)
    n = 0
    for i in range(1, 4):
        print('[PRODUCE] producing {}'.format(i))
        c.send(i)
    c.close()

c = consumer()
produce(c)
print('Result is {}'.format(r))
```

其结果输出如下：

``` text
[PRODUCE] producing 1
[CONSUMER] consuming 1
[PRODUCE] producing 2
[CONSUMER] consuming 2
[PRODUCE] producing 3
[CONSUMER] consuming 3
Result is 6
```

这个程序使用了生产者与消费者模式，生产者会依次生产`1`，`2`，`3`给消费者进行消费。消费者只要收到消息后，就会进行处理，将收到的数字与当前结果值进行相加赋给结果值，最终得到的结果值就是`6`，即`1+2+3`。

我们可以看到这个程序的执行是有序的，或是由用户控制的。每当`produce`执行`send()`函数时，协程就切换为`consumer`；每当`consumer`执行`yield`语句时，协程就切换为`produce`。整个流程由一个线程执行，`produce`和`consumer`协作完成任务，体现一个"协"字，而非线程那样的抢占式多任务。

## Go协程(Goroutine)

了解了协程之后，我们还需要了解一下Go协程(Goroutine)与协程(Coroutine)的异同点。

本质上，goroutine就是coroutine。不同的是，Golang在runtime、系统调用等多方面对goroutine调度进行了封装和处理。goroutine会被调度到多线程中去执行，并且当遇到长时间执行或者进行系统调用时，会主动把当前goroutine使用的线程转让出去，让其他goroutine能被调度并执行。也就是说，goroutine的执行是在语言层面进行控制的，而不是在用户层面。其目的是为了更好，更高效的支持高并发量。

goroutine是没有"协"这个特性的，goroutine之间的协作往往是通过通道(channel)实现的。
{% endraw %}