---
layout: post
title: "Gradle 快速入门 - 2.进阶"
subtitle: "JVM 世界的富有突破性的构建工具"
date: 2019-02-13 22:05:00
author: "Echcz"
header-img: "img/in-post/2019-02-06-gradle-quick-start.png"
catalog: true
previous:
  url: "2019/02/13/gradle-quick-start-1/"
  title: "Gradle 快速入门 - 1.基础"
next:
  url: "2019/02/13/gradle-quick-start-3/"
  title: "Gradle 快速入门 - 3.实战"
tags:
  - "Gradle"
  - "构建工具"
  - "JVM"
---
{% include post/190213-gradle-quick-start/header.md %}
{% raw %}
## 自定义 Property

Gradle 在默认情况下已经为 Project 定义了很多 Property，其中比较常见的有：

* project : Project 本身
* name : Project 的名字
* path : Project 的绝对路径
* description : Project 的描述信息
* buildDir : Project 构建结果存入目录
* version : Project 的版本号

我们可以 project.PROPERTY_NAME 的方式使用 Project 的 Property。如果直接使用 PROPERTY_NAME，Gradle 则会设置 delegate 所指对象的 Property：默认情况下，如果所处的 Task 有则使用 Task 的，如果没有则会使用 Project 的。

我们也可以自定义 Project 或 Task 的 Property：

* 在 build.gradle 文件中定义 Property：

``` groovy
ext.propertyA = "this is propertyA" // 定义 Project 的 Property
ext {
    propertyB = "this is propertyB" // 以闭包的方式定义 Property
}

task taskH {
    ext.propertyA = "this is taskH.propertyA" // 定义 taskH 的 Property

    doLast {
        println propertyA // 打印 taskH 的 propertyA
        println project.propertyA // 打印 Project 的 propertyA
        println propertyB // 打印 Project 的 propertyB
    }
}
```

* 通过命令行参数定义 Property：

``` groovy
task taskI {
    doLast {
        println propertyC
    }
}
```

执行以下命令：

``` shell
project $ gradle -PpropertyC="this is propertyC 1" taskI
```

将打印 `this is propertyC 1`。

* 通过 JVM 系统参数定义 Property：

在 Java 中，我们可以通过 `-D` 参数定义 JVM 系统参数。在 Gradle 中，也可以通过 `-D` 参数向 Project 中传入 Property，只是在传入时要加上 `org.gradle.project.` 作为前缀。对于上面的 taskI，执行以下命令：

``` shell
project $ gradle -Dorg.gradle.project.propertyC="this is propertyC 2" taskI
```

将打印 `this is propertyC 2`。

* 通过系统环境变量设置 Property：

系统环境变量需要以 `ORG_GRADLE_PROJECT_` 作为前缀。对于上面的 taskI，执行以下命令：

``` shell
project $ export ORG_GRADLE_PROJECT_propertyC="this is propertyC 3"
project $ gradle taskI
```

将打印 `this is propertyC 3`。

## 自定义 Task 类型

Gradle 中的 Task 要么是由不同的 Plugin 引入的，要么是我们自己在 build.gradle 文件中直接创建的。在默认情况下，我们所创建的 Task 是 DefaultTask 类型，该类型是一个非常通用的 Task 类型，而在有些时候，我们希望创建一些具有特定功能的 Task ，比如 Copy 和 Jar 等。还有些时候，我们希望定义自己创建的 Task 类型。接下来我们以定义一个简单的 MyTask 为例，讲解如何自定义 Task 类型：

* 在 build.gradle 文件中定义 Task 类型：

``` groovy
class MyTask extends DefaultTask { // 自定义 Task 类型
    @Optional // 表示属性是可选的，即在创建该 Task 时，可以不指定 msg
    String msg = 'I am MyTask'

    @TaskAction // 表示 Task 根执行的动作，即在执行该 Task 时，run 方法将会执行
    def run() {
        println "run: $msg"
    }
}

task taskJ(type: MyTask) // 执行 taskJ 将打印 `run: I am myTask`

task taskK(type: MyTask) { // 执行 taskK 将打印 `run: I am taskK`
    msg = "I am taskK"
}
```

* 在当前工程中定义 Task 类型：

Gradle 在执行时，会自动地查找 buildSrc 目录下所定义的Task类型，并首先编译该目录下的 groovy 代码以供 build.gradle 文件使用。在当前项目的 buildSrc/src/main/groovy/mytask 目录下创建 MyTask2.groovy 文件，内容如下：

``` groovy
package mytask
import org.gradle.api.*
import org.gradle.api.tasks.*

class MyTask2 extends DefaultTask {
    @Optional
    String msg = 'I am MyTask2'

    @TaskAction
    def todo() {
        println "todo: $msg"
    }
}
```

在 build.gradle 文件中加入如下内容：

``` groovy
// 因为 MyTask2 是定义在 mytask 包下，所以要加  `mytask.` 作为前缀
task taskL(type: mytask.MyTask2) // 执行 taskL 将打印 `todo: I am myTask2`

task taskM(type: mytask.MyTask2) { // 执行 taskM 将打印 `todo: I am taskM`
    msg = "I am taskL"
}
```

* 在单独的项目中定义 Task 类型：

我们可以创建一个 Groovy 项目，定义 Task 类型，并在客户端项目中引入该项目定义的 Task 类型。

首先创建服务项目如下：

在 src/main/groovy/mytask2 目录下创建 Mytask3.groovy 文件，内容如下：

``` groovy
package mytask2
import org.gradle.api.*
import org.gradle.api.tasks.*

class MyTask3 extends DefaultTask {
    @Optional
    String msg = 'I am MyTask3'

    @TaskAction
    def play() {
        println "play: $msg"
    }
}
```

创建 build.gradle 文件，内容如下：

``` groovy
plugins {
    id 'groovy'
    id 'maven'
}

version = '1.0'
group = 'mytasks'
archivesBaseName = 'mytask2'

repositories.mavenCentral()

dependencies {
    implementation gradleApi()
    implementation localGroovy()
}

uploadArchives {
    repositories.mavenDeployer {
        repository(url: 'file:../lib')
    }
}
```

执行以下命令，将生成的构建成品上传到 ../lib 目录中，以供接下来客户端项目使用：

``` shell
mytask2 $ gradle uploadArchives
```

然后在客户端项目的 build.gradle 文件中做如下配置：

``` groovy
buildscript {
    repositories {
        maven {
            url 'file:../lib'
        }
    }

    dependencies {
        classpath group: 'mytasks', name: 'mytask2', version: '1.0'
    }
}

task taskN(type: mytask2.MyTask3) // 执行 taskN 将打印 `play: I am MyTask3`

task taskO(type: mytask2.MyTask3) { // 执行 taskN 将打印 `play: I am taskO`
    msg = "I am taskO"
}
```

## 自定义 Plugin

Gradle 通过使用 Plugin 向 Project 添加 Task，定义 Configurations 和 Property 等。自定义 Plugin 与 自定义 Task 类型的方式类似，都可以：

* 在 build.gradle 文件中定义
* 在当前工程中定义
* 在单独的项目中定义

接下来我只演示在 build.gradle 文件中定义 Plugin，相信大家能根据`自定义 Task 类型` 举一反三：

在 build.gradle 文件中添加如下内容：

``` groovy
plugins {
    id 'ShowInfo' // 应用自定义插件：ShowInfo
}

class ShowInfo implements Plugin<Project> { // 自定义 Plugin
    void apply(Project project) {
        project.ext.myName = "echcz" // 为 Project 添加 Property：myName
        project.task("showInfo") { // 为 Project 添加 Task：showInfo
            doLast {
                println project.myName
                println project.name
                println project.path
                println project.description
                println project.buildDir
                println project.version
            }
        }
    }
}
```

执行以下命令：

``` shell
project $ gradle showInfo
```

将输出：

``` shell
> Task :showInfo
echcz
project
:
null
/home/echcz/project/build
unspecified

BUILD SUCCESSFUL in 0s
1 actionable task: 1 executed
```

{% endraw %}
{% include post/190213-gradle-quick-start/footer.md %}