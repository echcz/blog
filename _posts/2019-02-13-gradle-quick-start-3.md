---
layout: post
title: "Gradle 快速入门 - 3.实战"
subtitle: "JVM 世界的富有突破性的构建工具"
date: 2019-02-13 22:05:00
author: "Echcz"
header-img: "img/in-post/2019-02-06-gradle-quick-start.png"
catalog: true
previous:
  url: "2019/02/13/gradle-quick-start-2/"
  title: "Gradle 快速入门 - 2.进阶"
tags:
  - "Gradle"
  - "构建工具"
  - "JVM"
---
{% include post/190213-gradle-quick-start/header.md %}
{% raw %}
## 实战演练：快速构建一个 Java 项目

想使用 Gradle 快速构建一个 Java 项目，最简单的方法就是使用 java Plugin。java Plugin 通过向项目中添加一些相互依赖的 Task，构建了生命周期的概念，就像 Maven 一样。

java Plugin 主要引入的 Task 及说明如下：

| Task | 说明 |
|:--:|:-- |
| compileJava | 利用 javac 编译 Java 源文件 |
| processResource | 将项目的资源文件复制到 classes 目录中 |
| classes | 组装 classes 目录 |
| compileTestJava | 利用 javac 编译 Java 测试源文件 |
| processTestResource | 将项目的测试资源文件复制到 classes 目录中 |
| testClasses | 组装测试用的 classes 目录 |
| jar | 打包成 jar 文件 |
| javadoc | 生成 java 帮助文档 |
| test | 执行测试用例 |
| assemble | 组合分析所有的档案文件 |
| check | 执行所有的验证类任务 |
| build | 执行构建 |
| uploadArchives | 上传存档文件 |
| clean | 删除 build 文件，例项目回归到原始状态 |
| cleanTASK_NAME | 删除由任务产生的文件，比如 cleanJar 就是删除任务 jar 产生的文件 |

下面演示创建一个名为 `java-demo` 的 Java 项目：

### 配置 build.gradle 与 settings.gradle 文件

在 build.gradle 文件中添加如下内容：

``` groovy
plugins {
    id 'java' // 使用 java Plugin
}

group 'com.github.echcz.gradle' // 设置项目组
version '1.0' // 设置项目版本

sourceCompatibility = '1.8' // 设置源码编译版本为 JDK 1.8

repositories {
    mavenCentral() // 使用 maven 中央仓库
}
```

在 settings.gradle 文件中添加如下内容：

``` groovy
rootProject.name = 'java-demo'
```

### 创建项目目录结构

在默认情况下，java Plugin 采用了与 Maven 相同的 Java 项目目录结构：

``` shell
java-demo
├── build.gradle
├── settings.gradle
└── src
    ├── main
    │   ├── java
    │   └── resources
    └── test
        ├── java
        └── resources
```

实际上 Gradle 使用了 SourceSet 这个概念用于映射目录结构，用于指定哪些文件要被编译，哪些文件根被排除。java Plugin 默认实现了 两个 SourceSet：`main`，`test`。如果我们想创建一个名为 `api` 的 SourceSet 来存放接口，可以如下声明：

``` groovy
sourceSets {
    api
    main {
        compileClasspath = compileClasspath + files(api.output.classesDir)
    } // 1
   test {
        runtimeClasspath = runtimeClasspath + files(api.output.classesDir)
    } // 2
}

classes.dependsOn apiClasses // 3
```

java Plugin 会每个 SourceSet 创建相应的 Task：对于名为 `api` 的 SourceSet 会为其创建 compileApiJava（对 main 来说是 compileJava），processApiResource（对 main 来说是 processResource） 和 apiClasses（对 main 来说是 classes） 这三个 Task。因为编译 main 中的源码依赖 api 中的类文件，所以需要添加 `3` 处的内容；因为我们需要将 api 编译生成的类文件放在 main 的 classpath 下，所以需要添加 `1` 处的内容；因为运行测试时需要加载 api 中的类文件，所以需要添加 `2` 处的内容。

### 依赖管理

通常一个 Java 项目总会依赖其它项目，可能是第三方类库，也可能是你自己开发的其它 Java 项目。所以依赖管理是使用 Gradle 绕不开的话题。

#### 配置 Repository

在声明对其它项目的依赖时，我们需要配置 Gradle 的 Repository。在配置好依赖后，Gradle 会自动下载这些依赖到本地缓存（默认在 ~/.gradle/caches/modules-2/files-2.1 目录）。Gradle 可以使用 Maven 和 Ivy 的 Repository，也可以使用本地文件系统作为 Repository。正如前文的 build.gradle 文件里的 `repositories` 配置项，就是使用 Maven 的 Repository。

使用过 Maven 的同学会发现 Maven 的中央仓库的下载速度很慢而会使用 阿里云的 Maven 仓库镜像。Gradle 也可以使用阿里云的镜像，其方法是在 build.gradle（对当前项目有效） 或 ~/.gradle/init.gradle 或 ~/.gradle/init.d/\*.gradle（对当前用户有效）或 $GRADLE_HOME/init.d/\*.gradle（对当前系统有效）添加如下内容：

``` groovy
allprojects {
    repositories {
        maven { url 'https://maven.aliyun.com/repository/public/' }
        mavenLocal()
        mavenCentral()
    }
}
```

#### 配置依赖

Gradle 会对依赖进行分组管理，以方便在不同的 Task 使用不同的依赖。每一组依赖称为一个 Configuration，我们先将依赖加入到 Configuration 中，再在配置 Task 时通过configurations.CONFIGURATION_NAME.DEPENDENCY_NAME 使用依赖。例子如下：

``` groovy
configurations {
    myDependency
}

dependencies {
   myDependency 'org.apache.commons:commons-lang3:3.0'
}

task showMyDependency {
    doLast {
        println configurations.myDependency.asPath
    }
}
```

不过 java Plugin 已经为我们定义好 configurations 了，我们只需要给不同的 Configuration 配置相应的依赖就行了。例子如下：

``` groovy
dependencies {
   implementation 'org.apache.commons:commons-lang3:3.0'
   testCompile 'junit:junit:4.12'
}
```

java Plugin 定义的 Configuration 及说明：

| Configuration | 说明 |
|:--:|:-- |
| implementation | 仅实施依赖项，在编译期隐藏自身使用的依赖 |
| compileOnly | 仅编译期依赖项，运行时不提供 |
| compileClasspath | 编译类路径，在编译源码（执行 compileJava 任务）时提供 |
| annotationProcessor | 编译期使用的注释处理器 |
| runtimeOnly | 仅运行期依赖项 |
| runtimeClasspath | 运行时依赖项，包括 implementation 和 runtimeOnly 的元素 |
| testImplementation | 仅测试实施依赖项 |
| testCompileOnly | 仅测试编译期依赖项 |
| testCompileClasspath | 测试编译类路径 |
| testRuntimeOnly | 仅测试运行期依赖项 |
| testRuntimeClasspath | 测试运行时依赖项 |
| archives | 生成制品依赖项，在执行 uploadArchives 任务时提供 |
| default | 项目在运行时所需的工件和依赖项 |

{% endraw %}
{% include post/190213-gradle-quick-start/footer.md %}