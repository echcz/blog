---
layout: post
title: "Dokcerfile使用教程"
subtitle: "使用Dockerfile构建Docker镜像"
date: 2020-05-05 15:37:00
author: "Echcz"
header-img: "img/in-post/2020-05-05-dockerfile-use.png"
catalog: true
tags:
  - "docker"
---
{% raw %}
## 介绍

Dockerfile是一个文本文件，用于构建Doker镜像。其内包含了一条条的指令(Instruction)，每条指令描述了当前层(Docker使用分层文件系统)是如何构建的(只有RUN,ADD,COPY才会构建层数)。

## 常用指令的使用

Dockerfile的格式如下:

``` dockerfile
# Comment

# direcive=value
INSTRUCTION arguments
```

其中`Comment`是注释，以`#`号开头; `direcive`是解析器指令，是一种特殊类型的注释，会影响Dockerifle中后续行的处理，不区分大小写，但建议小写; 和 `INSTRUCTION`是指令，不区分大小写，但建议大写; `arguments`是参数，可以有多个参数。

可以看出Dockerfile的格式非常简单，学会Dockerfile的关键是学会Dockerfile的各指令是什么如何使用。

#### FROM

``` dockerfile
FROM <imge>[:<tag>|@<digest>] [as <alias>]
```

`FROM`指令设置基础镜像，其必须作为Dockerfile的第一条指令。`FROM` 可以出现多次，以用于多阶段构建，但只有最后一个才会输出到结果中。

* `tag`和`digest`是可选的，如果省略则使用`latest`
* `alias`用于设置编译阶段的别名，如果省略则是由0开始递增的数字

#### RUN

``` dockerfile
# exec形式
RUN ["<executable>", "<param1>", "<param2>"...]
# Or
# shell形式
RUN <command> <param1> <param2>...
```

`RUN`指令指定`docker build`时执行的程序或命令

* shell形式的`RUN`将作为`/bin/sh -c`的子命令启动，并且不传递信号
* `RUN`指令将在当前镜像之上的新层中执行命令，并提交结果。生成的已提交镜像将用于`Dockerfile`中的下一步

#### LABEL

``` dockerfile
LABEL <key1>=<value1> <key2>=<value2> ...
```

`LABEL`指令向镜像添加元数据(键值对的形式)

* `LABEL`可以出现多次，但推荐合并到单个`LABEL`中
* `LABEL`的值如果需要包含空格请使用引号或反斜杠转义

#### MAINTAINER

``` dockerfile
MAINTAINER <name>
```

`MAINTAINER`指令设置镜像的维护者信息，一般为`name <email>`

* 推荐使用`LABEL`指定维护者信息，如: `LABEL maintainer="jack <jack@email.com>"`

#### EXPOSE

``` dockerfile
EXPOSE <port1> <port2> ...
```

`EXPOSE`指令声明容器在运行时监听的网络端口

* `EXPOSE`并不使容器的端口可以被直接访问，为此需要在容器运行时使用`-p`选项将容器的端口映射到主机中

#### ENV

``` dockerfile
ENV <key> <value>
# Or
ENV <key1>=<value1> <key2>=<value2>...
```

`ENV`指令设置容器的环境变量

#### ADD

``` dockerfile
ADD <src1> <src2>... <dest>
# Or
# 如果路径包含空格，必须使用这种方式
ADD ["<src1>","<src2>"... "<dest>"]
```

`ADD`用于将文件或目录拷贝到镜像中

* `src`必须是相对路径且在构建上下文之内
* `dest`如果的相对路径是相对于`WORKDIR`的
* `src`可以使用通配符(`*`,`?`)
* 如果`dest`是目录，应显式的以`/`结尾
* 如果`src`是URL则会下载文件到`dest`，如果`dest`以`/`结尾，则会下载到`<dest>/<filename>`
* 如果`src`是压缩格式，则会自动解包为目录，不过来自URL的资源不会自动解包

#### COPY

`COPY`与`ADD`类似，只是不支持URL与自动解包

#### CMD

``` dockerfile
# exec形式
CMD ["<executable>", "<param1>", "<param2>"...]
# Or
# shell形式
CMD <command> <param1> <param2>...
```

`CMD`指令设置容器启动时执行的程序或命令，不过可以被`docker run`的命令行选项覆盖

* shell形式的`CMD`将作为`/bin/sh -c`的子命令启动，并且不传递信号
* `CMD`可以出现多次，但只有最后一个有效

#### ENTRYPOINT

``` dockerfile
# exec形式
ENTRYPOINT ["<executable>", "<param1>", "<param2>"...]
# Or
# shell形式
ENTRYPOINT <command> <param1> <param2>...
```

`ENTRYPOINT`指令设置容器启动时执行的程序或命令，与`CMD`类似，但`docker run`指定的参数不是覆盖`ENTRYPOINT`，而是被当成参数传递给`ENTRYPOINT`

* shell形式不会使用`CMD`与命令行参数，并且`ENTRYPOINT`将作为`/bin/sh -c`的子命令启动，并且不传递信号
* `ENTRYPOINT`可以出现多次，但只有最后一个有效
* exec形式可以配合`CMD`指定默认的参数

#### VOLUME

``` dockerfile
VOLUME <path1> <path2>...
# Or
VOLUME <JSON字符串数组>
```

`VOLUME`指令创建指向指定路径的挂载点，以挂载宿主机或其它容器的卷

* 在此指令后对数据卷的数据更改将会被放弃
* 如果在容器运行时没有指定挂载的数据卷将创建随机数据卷
* 如果挂载点目录路径下此前有文件存在，docker run命令会在卷挂载完成后将此前所有文件复制到新挂载的卷中

#### USER

``` dockerfile
USER <user>[:<group>]
```

`USER`指令指定执行镜像中的命令时使用的用户

#### WORKDIR

``` dockerfile
WORKDIR <path>
```

`WORKDIR`指令指定执行镜像中的命令时所在的工作目录

* 如果`WORKDIR`指定的目录不存在将创建此目录，即使它没有在任何后续的指令中使用
* `WORKDIR`可以多次使用，如果使用了相对路径，它将相对前先前的`WORKDIR`指定的路径

#### HEALTHCHECK

``` dockerfile
# 通过在容器内运行命令来检查容器的运行状态
HEALTHCHECK [OPTIONS] CMD <command>
# 禁用从基础镜像继承的任何运行状态检查
HEALTHCHECK NONE
```

`HEALTHCHECK`指令指定以何种方法检查容器是否正常工作

* `HEALTHCHECK`只能有一个，如果指定多个，则只有最后一个生效
* OPTIONS的选项有:
  * `--interval=<duration>`: 探测间隔时长，默认30s
  * `--timeout=<duration>`: 响应超时时长，默认30s
  * `--start-period=<duration>`: 开始探测延迟时长，默认0s
  * `--retries=<times>`: 容忍探测失败次数，默认3
* 命令的退出码会表示容器的状态:
  * 0: 成功，容器可正常提供服务
  * 1: 不健康，容器无法正常工作
  * 2: 保留，不要使用这个状态码

#### ARG

``` dockerfile
ARG <arg>
# 可以使用$引用这个参数,如:
# ARG user
# USER $user
```

`ARG`指定用于传递在命令`docker build --build-arg <arg>=<value>`时指定的参数

* 可以使用`${<arg>:-<default_value>}`格式指定默认值

#### SHELL

``` dockerfile
SHELL ["<excutable>", "<param1>", "<param2>"...]
```

`SHELL`指令用于覆盖用于执行shell形式的命令的默认shell

* `Linux`的默认shell是: `["/bin/sh", "-c"]`
* `Windows`的默认shell是: `["cmd", "/S", "/C"]`

#### ONBUILD

``` dockerfile
ONBUILD <INSTRUCTION>
```

`ONBUILD`是一个特殊的指令，它后面跟的是其它指令，比如`RUN`, `COPY`等，而这些指令，在当前镜像构建时并不会被执行。只有当以当前镜像为基础镜像，去构建下一级镜像的时候才会被执行

## 最佳实践

#### 容器应该是短暂的

通过`Dockerfile`构建的镜像所启动的容器应该尽可能短暂(ephemeral)。短暂意味着可以很快地启动并且终止

#### 使用 .dockerignore 排除构建无关的文件

构建时会将`构建上下文`中的所有文件发送到`dokcer`服务中，使用`.dockerignor`文件可以排除文件，其语法与`.gitignore`一致

#### 使用多阶段构建

多阶段构建可以有效减小镜像体积，特别是对于需编译语言而言，一个应用的构建过程往往如下:

1. 安装编译工具
2. 安装第三方依赖
3. 编译构建应用

而在前两步会有大量的镜像体积冗余，使用多阶段构建可以避免这一问题。以下是一个构建前端应用的一个示例:

``` dockerfile
FROM node:10-alpine as builder

ENV PROJECT_ENV production
ENV NODE_ENV production

WORKDIR /code

ADD package.json /code
RUN npm install --production

ADD . /code
RUN npm run build

# 选择更小体积的基础镜像
FROM nginx:10-alpine
COPY --from=builder /code/public /usr/share/nginx/html
```

#### 避免安装不必要的包

减小体积，减少构建时间。如前端应用使用`npm install --production`只装生产环境所依赖的包

#### 一个容器只做一件事

一般来说，我们期望一个容器只运行一个进程。如果一个应用需要用于多个服务配合，可以使用`docker-compose`

#### 减少镜像层数

只有`RUN`,`COPY`,`ADD`会创建层数，其它指令不会增加镜像体积，因些应尽量减少这三个指令的使用。例如:

``` dockerfile
# 错误的安装依赖的方式
RUN yum install -y node
RUN yum install -y python
RUN yum install -y go

# 应改成这样的形式
RUN yum install -y node python go
```

#### 充分利用构建缓存

在镜像的构建过程中`docker`会遍历`Dockerfile`文件中的所有指令，顺序执行。对于每一条指令，`docker`都会在缓存中查找是否已存在可重用的镜像，否则会创建一个新的镜像。使用`docker build --no-cache`将不使用缓存

* `ADD`和`COPY`将会计算文件的`checksum`是否改变来决定是否利用缓存
* `RUN`仅仅查看命令字符串是否命中缓存，如`RUN apt-get -y update`可能会有问题

以下的示例将充分利用缓存跳过多次构建时`npm`的安装

``` dockerfile
FROM node:10-alpine

WORKDIR /code
ADD package.json /code
# 只要package.json文件不更改，将利用缓存减少从开始到此指令的执行
RUN npm install --production

ADD . /code
RUN npm run build
```

{% endraw %}
