---
layout: post
title: "Gradle 快速入门 - 1.基础"
subtitle: "JVM 世界的富有突破性的构建工具"
date: 2019-02-13 22:05:00
author: "Echcz"
header-img: "img/in-post/2019-02-06-gradle-quick-start.png"
catalog: true
next:
  url: "2019/02/13/gradle-quick-start-2/"
  title: "Gradle 快速入门 - 2.进阶"
tags:
  - "Gradle"
  - "构建工具"
  - "JVM"
---
{% include post/190213-gradle-quick-start/header.md %}
{% raw %}
## 基本概念

Gradle 本身的领域对象主要有 Project 和 Task。Project 为 Task 提供了执行上下文，所有的 Plugin 要么向Project中添加用于配置的 Property，要么向 Project 中添加不同的 Task。一个Task 表示一个逻辑上较为独立的执行过程，比如编译 Java 源代码，拷贝文件，打包 Jar 文件，甚至可以是执行一个系统命令或者调用 Ant 。另外，一个 Task 可以读取和设置 Project 的 Property 以完成特定的操作。

首先让我们创建一个最简单的 Gradle 的项目，在 Project 中创建一个名为 `helloWorld` 的 DefaultTask 类型的 Task。在项目目录里创建一个名为 `build.gradle` 的文本文件，内容如下：

``` groovy
task helloWorld {
    doLast {
        println "Hello World!!!"
    }
}
```

默认情况下，Gradle 将当前目录下的 build.gradle 文件作为项目的构建文件。文件内容其实就是 Groovy 代码，Gradle 提供了 DSL 方便我们的构建。`task` 是一个 Groovy 方法，`task` 后的大括号里的内容是传递给 `task` 方法的一个闭包，用于配置这个 task。`doLast` 也是一个 Grovvy 方法，表示在执行该 task 时的最终执行传递进来的闭包。

在项目目录下执行：

``` shell
project $ gradle helloWorld
```

gradle 将执行 helloWorld Task 以进行构建。命令行将输出如下：

``` shell
> Task :helloWorld
Hello World!!!

BUILD SUCCESSFUL in 0s
1 actionable task: 1 executed
```

## Task 的创建与配置

### 创建 Task

* 创建 Task 最方便的就是使用 Project 的 task 方法了：

``` groovy
task taskA {
    doLast {
        println "this is taskA"
    }
}
```

这将在 TaskContainer 类型对象 tasks 中创建一个名为 `taskA` 的 Task，用于打印 `it is taskA`。

* 使用 TaskContainer 的 create 方法创建 Task：

``` groovy
tasks.create(name: "taskB") {
    doLast {
        println "this is taskB"
    }
}
```

### 声明 Task 的类型

Task 是有类型的，不同类型的 Task 定义了不同的行为。默认情况下，task 定义的 Task 类型是 DefaultTask。我们可以显示地声明 Task 类型（当然可以自定义 Task 类型，这个下面说）：

``` groovy
task taskCopy(type: Copy) {
    from 'source'
    into 'target'
}
```

`type: Copy` 将taskCopy 声明为 Copy 类型。执行 taskCopy 将把 `source` 目录里的内容复制到 `target` 目录里。*注意：文件路径是相对当前 Project 而言的，也就是 build.gradle 文件所在目录*

### 声明 Task 之间的依赖关系

Task 之间是可以有依赖关系的，这样 Task 之间就会组织成一条条执行链。例如 taskC 依赖 taskB，这样在执行 taskC 时，taskB 会先执行。

* 在定义 Task 时声明它的依赖关系(被依赖的 Task 应先定义)：

``` groovy
task taskC(dependsOn: taskB) {
    doLast {
        println "this is taskC"
    }
}
```

* 在定义 Task 后，再声明依赖关系：

``` groovy
task taskD {
    doLast {
        println "this is taskD"
    }
}

taskD.dependsOn taskB
```

### 配置 Task 的 Property

Task 除了执行操作外，还可以有多个 Property，这些 Property 用于配置 Task 的属性，以方便配置 Task 的行为。Gradle 为每个 Task 默认定义了一些 Property，比如 description,logger等。此外，每个特定类型的 Task 也有特定的 Property，比如 Copy 类型的 from 和 to 等。当然，我们也可以动态地向 Task 中添加 Property。通常在执行一个 Task 之前，我们需要先设定 Property。Gradle 提供了多种方法设置 Task 的 Property。

* 在定义 Task 时设置 Property：

``` groovy
task taskE {
    description = "this is taskE"

    doLast {
        println description
    }
}
```

* 通过调用 Task 的 configure 方法设置 Property：

``` groovy
task taskF {
    doLast {
        println description
    }
}

taskF.configure {
    description = "this is taskF"
}
```

Gradle 在执行 Task 时分为两个阶段，首先是配置阶段，然后才是实际的执行阶段。所以在 taskF 时，依然会打印 `this is taskF`。

* 通过闭包的方式设置 Property:

``` groovy
task taskG {
    doLast {
        println description
    }
}

taskG {
    description = "this is taskG"
}
```

实际上，通过闭包的方式在内部也是通过调用 Task 的 configure 方法完成的。

## 增量式构建

每个 Task 都拥有 inputs(输入) 和 outputs(输出) 属性，他们的类型分别是 TaskInputs 和 TaskOutputs。在执行一个 Task 时，如果它的输入与输出与前一次执行时没有发生变化，那么 Gradle 便认为该 Task 是最新的(UP-TO-DATE)，这时，Gradle 将不予执行。这就是增量式构建，通过增量式构建可以节约构建时间。Task 的输入与输出可以是文件或文件夹，也可以是 Project 的 Property，还可以是某个闭包所定义的条件。

下面我们定义一个 Task，用于将 sourceDir 文件夹里的文件的内容合并到 combine.txt 文件里：

``` groovy
task taskCombine {
    def sources = fileTree('sourceDir')
    def target = file('combine.txt')

    doLast {
        target.withPrintWriter { writer ->
            sources.each { source ->
                writer.println source.text
            }
        }
    }
}
```

多次执行 taskCombine Task，Task 都会反复执行，即使上一次执行已经得到了所需的结果。如果 taskCombine 是一个耗时的任务，这样势必会没必要地浪费大量时间。如果将 sources 声明为 taskCombine 的 inputs，将 target 声明为 taskCombine 的 outputs，将会执行增量式构建：

``` groovy
task taskCombine2 {
    def sources = fileTree('sourceDir')
    def target = file('combine.txt')

    inputs.dir sources
    outputs.file target

    doLast {
        target.withPrintWriter { writer ->
            sources.each { source ->
                writer.println source.text
            }
        }
    }
}
```

可以看到后一个 Task 只比前一个 Task 多了两行用于声明 inputs 和 outputs 的代码。当首次执行 taskCombine2 时，Gradle 会执行该 Task，但如果再执行一次，命令行将显示：

``` shell
> Task :taskCombine2 UP-TO-DATE

BUILD SUCCESSFUL in 0s
1 actionable task: 1 up-to-date
```

可以看到 taskCombine2 被标记为 `UP-TO-DATE`，表示该 Task 是最新的，Gradle 不予执行。

如果我们修改了 inputs（对于 taskCombine2 来说就是 sourceDir 目录里的内容）或修改/删除了 outputs（对于 taskCombine2 来说就是 combine.txt 文件），该 Task 就不是最新的了，Gradle 就会重新执行该 Task。我们可以使用 upToDateWhen 方法来决定一个 Task 是否是最新的，该方法接受一个闭包作为检查条件，感兴趣的同学可以自行了解，这里就不详细说明了。

## 多 Project 构建

在多 Project 的项目中，我们会操作多个 Project 领域对象。Gradle 提供了强大的多 Project 构建支持。要创建多 Project 的 Gradle 项目，我们首先需要在根 Project 中加入名为 `settings.gradle` 的配置文件，该文件应该包含各个子 Project 的名称。比如，我们有一个根 Project 名为 `root-project`，它包含有两个子 Project，名字分别为 `sub-project1` 和 `sub-project2`，此时对应的文件目录结构如下：

``` shell
root-project
├── build.gradle
├── settings.gradle
├── sub-project1
│   └── build.gradle
└── sub-project2
    └── build.gradle
```

root-project/settings.gradle 文件的内容如下：

``` groovy
include 'sub-project1', 'sub-project2'
```

### 在根 Project 中配置所有 Project

在 root-project/build.gradle 文件中添加如下内容：

``` groovy
allprojects {
    task taskAll {
        doLast {
            println project.name
        }
    }
}
```

执行如下命令：

``` shell
root-project $ gradle taskAll
# 注：gradle :taskAll 将只执行 root-project 里的 taskAll
# 注：gradle :sub-project1:taskAll 将只执行 sub-project1 里的 taskAll
```

命令行将输出：

``` shell
> Task :taskAll
root-project

> Task :sub-project1:taskAll
sub-project1

> Task :sub-project2:taskAll
sub-project2

BUILD SUCCESSFUL in 0s
3 actionable tasks: 3 executed
```

### 在根 Project 中配置所有子 Project

在 root-project/build.gradle 文件中添加如下内容：

``` groovy
subprojects {
    task taskSub {
        doLast {
            println project.name
        }
    }
}
```

执行如下命令：

``` shell
root-project $ gradle taskSub
```

命令行将输出：

``` shell
> Task :sub-project1:taskSub
sub-project1

> Task :sub-project2:taskSub
sub-project2

BUILD SUCCESSFUL in 0s
2 actionable tasks: 2 executed
```

### 在根 Project 中配置某个 Project

在 root-project/build.gradle 文件中添加如下内容：

``` groovy
project(':sub-project1') {
   task taskPro1 {
       doLast {
           println project.name
       }
   }
}
```

执行如下命令：

``` shell
root-project $ gradle taskPro1
```

命令行将输出：

``` shell
> Task :sub-project1:taskPro1
sub-project1

BUILD SUCCESSFUL in 0s
1 actionable task: 1 executed
```

### 以一种更灵活的方式配置 Project

Gradle 其本质就是用 Groovy 语言操作一些领域对象，所以我们可以将 Groovy 的语言特性用在 Gradle 领域对象上，比如我们可以对 Project 进行过滤后再配置：

``` groovy
configure(allprojects.findAll { it.name.startsWith('sub-') }) {
   task taskSub2 {
       doLast {
            println "I am ${project.name}"
       }
   }
}
```

执行如下命令：

``` shell
root-project $ gradle taskSub2
```

命令行将输出：

``` shell
> Task :sub-project1:taskSub2
I am sub-project1

> Task :sub-project2:taskSub2
I am sub-project2

BUILD SUCCESSFUL in 0s
2 actionable tasks: 2 executed
```

### 多 Project 的 Task 之间的依赖配置

多 Project 项目的不同项目的 Task 之间也是可以相互依赖的。在 root-project/build.gradle 文件中添加如下内容：

``` groovy
allprojects {
    task taskA {
        doLast {
            println "this is ${project.name}"
        }
    }

    task taskB {
        doLast {
            println "I am ${project.name}"
        }
    }
}
```

在 root-project/sub-project1/build.gradle 文件中添加如下内容：

``` groovy
taskA.dependsOn ':sub-project2:taskB'
```

执行如下命令：

``` shell
root-project $ gradle :sub-project1:taskA
```

命令行将输出：

``` shell
> Task :sub-project2:taskB
I am sub-project2

> Task :sub-project1:taskA
this is sub-project1

BUILD SUCCESSFUL in 0s
2 actionable tasks: 2 executed
```

{% endraw %}
{% include post/190213-gradle-quick-start/footer.md %}