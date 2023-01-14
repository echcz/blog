---
layout: post
title: "Golang 反射规则"
subtitle: "一旦你理解了这些规则，GO 中的反射就会变得更容易使用，尽管它仍然很微妙"
date: 2011-09-06 00:00:00
author: "Rob Pike"
header-style: "text"
catalog: true
tags:
  - "Golang"
  - "反射"
  - "转载"
---
{% raw %}
* 原文: [https://go.dev/blog/laws-of-reflection](https://go.dev/blog/laws-of-reflection)
* 翻译: Echcz

## 简介

计算机中的反射是指程序检查其自身结构，尤其是通过类型检查的能力；它是一种元编程的形式。它也是混乱的重要来源之一。

在本文中，我们试图通过以 GO 为例来解释反射的工作原理。每个语言的反射模式是不同的（也有一点语言根本就不支持反射），但是本文权针对 GO，故在本文接下来的内容中，“反射”一词应理解为“GO 中的反射”。

> 说明(2022 年 1 月添加)：本博文写于 2011 年，在 GO 支持参数化类型（即泛型）之前。虽然文中没有任何重要的内容因语言发展而变得不再正确，但还是在一些地方做了调整，以避免让熟悉现代 GO 的人感到困惑。

## 类型与接口

因为反射构建于类型系统之上，所以让我们先复习下 GO 中的类型。

GO 是静态类型语言。每个变量都有一个静态类型，也就是类型在编译时是已知且固定的，例如：`float32`, `*MyType`，`[]byte` 等等。如果我们定义:

``` go
type MyInt int
var i int
var j MyInt
```

则 `i` 的类型为 `int`，`j` 的类型为 `MyInt`。虽然变量 `i` 和 `j` 基本类型相同，但它们的静态类型却并不相同，如果不进行转换，则不能相互赋值。

一个重要的类别是接口类型，它表示固定的方法集。（在讨论反射时，我们可以忽略接口定义作为多态代码中的约束这一作用）。一个接口变量能保存任意具体(非接口)值，只要该值实现了接口定义的方法。一对知名的例子是 `io.Reader` 与 `io.Writer`，它是 io 包中 `Reader` 类型与 `Writer` 类型：

``` go
// Reader 是一个封装了基本的 Read 方法的接口
type Reader interface {
  Reader(p []byte) (n int, err error)
}

// Writer 是一个封装基本的 Write 方法的接口
type Writer interface {
  Write(p []byte) (n int, err error)
}
```

任何有 `Read`（或 `Write`）方法签名实现的类型都被称为实现了 `io.Reader`（或 `io.Writer`）。就本讨论而言，这意味着 `io.Reader` 类型的变量可以持有任何有 `Read` 方法的类型的值：

``` go
var r io.Reader
r = os.Stdin
r = bufio.NewReader(r)
r = new(bytes.Buffer)
// 等等
```

必须明确的是，无论 `r` 的具体值是什么，`r` 的类型都是 `io.Reader`：GO 是静态类型语言，`r` 的静态类型就是 `io.Reader`。

接口类型一个极其重要的例子是空接口：

``` go
interface{}
```

它等价的别名是：

``` go
any
```

它表示方法的空集，任何值都可以满足，因为每个值都有零个或多个方法。

有些人说 GO 的接口是动态类型的，这是一种误解。它们是静态类型的：接口类型的变量总是具有相同的静态类型，即使在运行时存储在接口变量中的值的具体类型可能会改变，但该值始终满足接口要求。

我们需要对这一切进行精准的分析，因为反射与接口是密切相关的。

## 接口的表示法

Russ Cox 写了一篇详解 GO 中接口值的[博文](https://research.swtch.com/2009/12/go-data-structures-interfaces.html)。这里对其内容就不再赘述了，但我们还是做个简单总结。

一个接口类型的变量存储了一对项：分配给该变量的具体值与该值的类型描述符。更准确地说，值是实现接口的底层具体数据项，而类型则描述了该数据项的完整类型。例如：

``` go
var r io.Reader
tty, err := os.OpenFile("/dev/tty", os.O_RDWR, 0)
if err != nil {
  return nil, err
}
r = tty
```

我们示意性的表达了 `r`：其包含 (值，类型) 对，即 (`tty`, `*os.File`)。注意，`*os.File` 这个类型还实现了 `Read` 之外的其它方法；尽管接口值只提供了对 `Read` 方法的访问，但里面的值却携带了关于这个值的所有类型信息。这就是为什么我们可以如下处理：

``` go
var w io.Writer
w = r.(io.Writer)
```

这个赋值语句中的表达式是一个类型断言；它断言 `r` 里面的值也实现了 `io.Writer`，所以我们可以把它赋给 `w`。接口的静态类型决定了哪些方法可以被接口变量调用，尽管里面的具体值可能有更大的方法集。

接下来，我们可以如下处理：

``` go
var empty interface{}
empty = w
```

空接口值 `empty` 将再次包含同样的一对：（`tty`, `*os.File`）。这很方便：一个空接口可以容纳任何值，并且包含我们可能需要的关于这个值的所有信息。

(我们在这里不需要类型断言，因为静态地知道 `w` 满足空接口的要求。在我们把一个值从 `Reader` 移到 `Writer` 的例子中，我们需要明确地使用一个类型断言，因为 `Writer` 的方法不是 `Reader` 的子集）。

一个重要的细节是，接口变量内部的一对总是具有（值, 具体类型）的形式，而不是（值, 接口类型）。接口不持有接口值。

现在，让我们开始学习反射。

## 反射第一规则: 反射将 接口值 转为 反射对象

在基本层面上，反射只是一种检查存储在接口变量内的（值, 类型）对的机制。作为入门，我们需要了解 `reflect`包 中的两种类型：类型(`Type`)和值(`Value`)。这两种类型可以访问接口变量的内容，两个简单的函数，叫做 `reflect.TypeOf` 和 `reflect.ValueOf`，可以从一个接口值中检索 `reflect.Type` 和 `reflect.Value` 片段。(另外，从一个 `reflect.Value` 可以很容易地得到相应的 `reflect.Type`，但现在我们还是把 `Value` 和 `Type` 的概念分开。）

让我们从 `TypeOf` 开始：

``` go
package main

import (
    "fmt"
    "reflect"
)

func main() {
    var x float64 = 3.4
    fmt.Println("type:", reflect.TypeOf(x))
}
```

这个程序将打印:

``` txt
type: float64
```

你可能想知道这里的接口在哪里，因为程序看起来是在向 `reflect.TypeOf` 传递 `float64` 变量 `x`，而不是一个接口值。但接口就在那里；如 [godoc](https://go.dev/pkg/reflect/#TypeOf) 显示，`reflect.TypeOf` 方法签名包括一个空接口：

``` go
// TypeOf 接收一个 interface{} 值，返回一个 reflect.Type
func TypeOf(i interface{}) Type
```

当我们调用 `reflect.TypeOf(x)` 时，`x` 首先被存储在一个空接口中，然后被作为参数传递；`reflect.TypeOf` 解析该空接口以恢复类型信息。

当然，`reflect.ValueOf` 函数恢复的是值（从这里开始，我们将省略模板代码，只关注可执行代码）：

``` go
var x float64 = 3.4
fmt.Println("value:", reflect.ValueOf(x).String())
```

这将打印：

``` txt
value: <float64 Value>
```

(我们明确地调用 `String` 方法，因为默认情况下，`fmt`包 会挖掘 `reflect.Value` 以显示其中的具体值。而`String` 方法则不会）。

`reflect.Type` 和 `reflect.Value` 都有很多方法让我们检查和操作它们。一个重要的例子是，`Value` 有一个 `Type` 方法来返回 `reflect.Value` 的 `Type`。另一个例子是 `Type` 和 `Value` 都有一个 `Kind` 方法，该方法返回一个常数，如，`Uint`, `Float64`, `Slice` 等等，以表明存储的是什么类型的项目。另外，`Value` 上有如 `Int` 和 `Float` 这样的名字的方法，让我们提取里面存储的值（如 `int64` 和 `float64`）：

``` go
var x float64 = 3.4
v := reflect.ValueOf(x)
fmt.Println("type:", v.Type())
fmt.Println("kind is float64:", v.Kind() == reflect.Float64)
fmt.Println("value:", v.Float())
```

这将打印：

``` txt
type: float64
kind is float64: true
value: 3.4
```

还有一些方法，如 `SetInt` 和 `SetFloat`，但要使用它们，我们需要了解可设置性。我们放在 反射第三规则 的主题下进行讨论。

反射库有几个特性值得特别指出。首先，为了保持 API 的简单性，`Value` 的 "getter" 和 "setter" 方法操作的是可以容纳该值的最大类型：如所有有符号的整数的 `int64` 。也就是说，`Value` 的 `Int` 方法返回一个 `int64`，`SetInt` 值需要一个 `int64`；实际应用时，需根据使用场景将其转换为实际需要的类型。

``` go
var x uint8 = 'x'
v := reflect.ValueOf(x)
fmt.Println("type:", v.Type())                            // uint8.
fmt.Println("kind is uint8: ", v.Kind() == reflect.Uint8) // true.
x = uint8(v.Uint())                                       // v.Uint 返回 uint64 值.
```

第二个要说的点是，反射对象的 `Kind` 描述了底层类型，而不是静态类型。如果一个反射对象包含一个用户定义的整数类型的值，如：

``` go
type MyInt int
var x MyInt = 7
v := reflect.ValueOf(x)
```

尽管 `x` 的静态类型是 `MyInt` 而不是 `int`, `v` 的类型仍是 `reflect.Int`。换句话说，`Kind` 不能区分 `int` 和 `MyInt`，而 `Type` 可以。

## 反射第二规则: 反射将 反射对象 转为 接口值

如物理中的反射一样，GO 中的反射也会产生它自己的逆向。

给定一个 `reflect.Value`，我们可以使用 `Interface` 方法从中恢复一个接口值；实际上，该方法将类型和值的信息打包回一个接口表示，并返回结果：

``` go
// Interface 返回一个 interface{} 值
func (v Value) Interface() interface{}
```

接下来，我们可以如下处理：

``` go
y := v.Interface().(float64) // y 的类型为 float64.
fmt.Println(y)
```

以打印反射对象 `v` 所代表的 `float64` 值。

不过，我们可以更进一步。`fmt.Println`、`fmt.Printf` 等的参数都是以空的接口值传递的，然后由 `fmt`包 内部解包，就像我们在前面的例子中做的那样。因此，正确打印 `reflect.Value` 的内容只需要将 `Interface` 方法的结果传递给格式化打印例程：

``` go
fmt.Println(v.Interface())
```

(自从这篇文章第一次写完后，`fmt`包 进行了修改，这样它就能自动解析 `reflect.Value`，所以我们可以直接打印 `v`：

``` go
fmt.Println(v)
```

两者结果是一样的，但为了清晰起见，我们在这里保留对 `.Interface()` 的调用)。

由于我们的值是 `float64`，如果我们想，甚至可以使用浮点格式：

``` go
fmt.Printf("value is %7.1e\n", v.Interface())
```

这将打印：

``` txt
3.4e+00
```

同样不需要对 `v.Interface()` 的结果进行类型断言到 `float64`；空接口值里面有具体值的类型信息，`Printf` 会恢复它。

简而言之，`Interface` 方法是 `ValueOf` 函数的逆函数，只是它的结果总是静态类型 `interface{}`。

重申一下。反射从接口值到反射对象，然后再反射回来。

## 反射第三规则: 要修改一个反射对象，其值必须是可设置的

第三条规则是最微妙、最令人困惑的，但如果我们从第一规则出发，就很容易理解了。

下面是一些不起作用的代码，但值得研究：

``` go
var x float64 = 3.4
v := reflect.ValueOf(x)
v.SetFloat(7.1) // 错误，将引发 panic.
```

如果你运行这段代码，这将引发 panic 并抛出如下信息：

``` txt
panic: reflect.Value.SetFloat using unaddressable value
```

问题不在于 `7.1` 这个值是不可寻址的，而在于 `v` 是不可设置的。可设置性是反射值的一个属性，并不是所有的反射值都有这个属性。

`Value` 的 `CanSet` 方法报告一个 `Value` 的可设置性；在如下案例中：

``` go
var x float64 = 3.4
v := reflect.ValueOf(x)
fmt.Println("settability of v:", v.CanSet())
```

打印：

``` txt
settability of v: false
```

对一个不可设置的值调用设置方法会产生错误。但什么是可设置性呢？

可设置性有点像可寻址性，但要更严格些。它是表示一个反射对象是否可以修改用于创建反射对象的实际存储值的属性。可设置性是由反射对象是否持有原始值决定的。当如下处理时：

``` go
var x float64 = 3.4
v := reflect.ValueOf(x)
```

这会把 `x` 的副本传递给 `reflect.ValueOf`，所以作为 `reflect.ValueOf` 的参数创建的接口值是 `x` 的副本，而不是 `x` 本身。因此，如果语句

``` go
v.SetFloat(7.1)
```

就算允许成功，它也不会更新 `x`，即使 `v` 看起来像是由 `x` 创建的。相反，它将更新存储在反射值中的 `x` 的副本，而 `x` 本身将不受影响。这没有作用，只会造成混乱，所以它是非法的，而可设置性就是用来避免这个问题的属性。

这看起来很怪异，其实不然。它实际上是一个熟悉的情况，只是披着不寻常的外衣。想想看，把 `x` 传给一个函数：

``` go
f(x)
```

我们不会期望 `f` 能够修改 `x`，因为我们传递的是 `x` 值的拷贝，而不是 `x` 本身。如果我们想让 `f` 直接修改 `x`，我们必须把 `x` 的地址传给函数（也就是 `x` 的指针）：

``` go
f(&x)
```

这对我们来说是很直接和熟悉的，反射的工作方式也是如此。如果我们想通过反射来修改 `x`，我们必须给反射库一个指向我们想修改的值的指针。

让我们来做这件事。首先我们像往常一样初始化 `x`，然后创建一个指向它的反射值，叫做 `p`：

``` go
var x float64 = 3.4
p := reflect.ValueOf(&x) // 提示：传入 x 的指针
fmt.Println("type of p:", p.Type())
fmt.Println("settability of p:", p.CanSet())
```

这将输出：

``` txt
type of p: *float64
settability of p: false
```

反射对象 `p` 还是不可设置的，但我们想设置的不是 `p`，而是（实际上）`*p`。为了得到 `p` 所指向的东西，我们调用 `Value` 的 `Elem` 方法，它通过指针进行间接操作，并将结果保存在一个叫做 `v` 的反射 `Value` 中：

``` go
v := p.Elem()
fmt.Println("settability of v:", v.CanSet())
```

现在，`v` 是可设置的反射对象，如输出所示：

``` txt
settability of v: true
```

由于它代表 `x`，所以我们终于可以使用 `v.SetFloat` 来修改 `x` 的值：

``` go
v.SetFloat(7.1)
fmt.Println(v.Interface())
fmt.Println(x)
```

正如预期的那样，输出为：

``` txt
7.1
7.1
```

反射可能很难理解，但它所做的正是语言所做的，尽管通过反射类型和值可能会掩盖正在发生的事情。请记住，反射值需要一些东西的地址，以便修改它们所代表的东西。

## 结构体

在我们前面的例子中，`v` 本身并不是一个指针，它只是从一个指针派生出来的。出现这种情况的一个常见方式是使用反射来修改结构体的字段。只要我们有结构体的地址，我们就能修改它的字段。

下面是一个分析结构体值 `t` 的简单例子。我们用结构体的地址创建反射对象，因为我们想修改它。然后我们将 `typeOfT` 设置为它的类型，并使用直接的方法调用遍历字段（详见 [reflect 包](https://go.dev/pkg/reflect/)）。请注意，我们从结构体类型中提取字段的名称，但字段本身是普通的 `reflect.Value` 对象。

``` go
type T struct {
    A int
    B string
}
t := T{23, "skidoo"}
s := reflect.ValueOf(&t).Elem()
typeOfT := s.Type()
for i := 0; i < s.NumField(); i++ {
    f := s.Field(i)
    fmt.Printf("%d: %s %s = %v\n", i,
        typeOfT.Field(i).Name, f.Type(), f.Interface())
}
```

这个程序输出如下：

``` txt
0: A int = 23
1: B string = skidoo
```

这里顺便介绍一下关于可设置性的另一点：`T` 的字段名是大写的（导出的），因为只有结构的导出字段是可设置的。

因为 `s` 包含一个可设置的反射对象，我们可以修改结构体的字段：

``` go
s.Field(0).SetInt(77)
s.Field(1).SetString("Sunset Strip")
fmt.Println("t is now", t)
```

这将返回：

``` txt
t is now {77 Sunset Strip}
```

如果我们修改程序，使 `s` 是由 `t` 而不是 `&t` 创建，那么调用 `SetInt` 和 `SetString` 就会失败，因为 `t` 的字段是不可设置的。

## 总结

这里重复下反射的规则：

* 反射将 接口值 转为 反射对象
* 反射将 反射对象 转为 接口值
* 要修改一个反射对象，其值必须是可设置的

一旦你理解了这些规则，GO 中的反射就会变得更容易使用，尽管它仍然很微妙。它是一个强大的工具，应该谨慎使用，除非绝对必要，否则应避免使用。

还有很多关于反射的内容我们没有涉及——在通道上发送和接收，分配内存，使用分片和映射，调用方法和函数——但是这篇文章已经够长了。我们将在以后的文章中介绍其中的一些主题。
{% endraw %}