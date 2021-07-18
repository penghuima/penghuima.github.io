---
layout: post
title: "Bazel 学习笔记"
categories: bazel
tags: bazel linux
---

* content
{:toc}


### 简介

Bazel是Google开源的，类似于Make、Maven或Gradle的构建和测试工具。它使用可读性强的、高层次的构建语言，支持多种编程语言，以及为多种平台进行交叉编译。

Bazel的优势：

- 高层次的构建语言：更加简单，Bazel抽象出库、二进制、脚本、数据集等概念，不需要编写调用编译器或链接器的脚本
- 快而可靠：能够缓存所有已经完成的工作步骤，并且跟踪文件内容、构建命令的变动情况，避免重复构建。此外Bazel还支持高度并行构建、增量构建
- 多平台支持：可以在Linux/macOS/Windows上运行，可以构建在桌面/服务器/移动设备上运行的应用程序
- 可扩容性：处理10万以上源码文件时仍然能保持速度
- 可扩展性：支持Android、C/C++、Java、Objective-C、Protocol Buffer、Python…还支持扩展以支持其它语言

<!--more-->

BUILD文件的作用就是将workspace中的在build文件中声明过的依赖文件连接起来，共同构建为一个整体，存放于build-bin文件中，运行程序时只需运行build-bin文件即可得到输出，其中BUILD的依赖项deps中的文件在srcs文件修改后，再进行构建时，不会再构建deps文件，这样会增加效率便于管理。

**Bazel是一个Blaze的开源项目，是Blaze的一个子集，但是基本功能完全一致。**

#### 如何工作

当运行构建或者测试时，Bazel 会：

1. 加载和目标相关的 BUILD 文件
2. 分析输入及其依赖，应用指定的构建规则，产生一个 Action 图。这个图表示需要构建的目标、目标之间的关系，以及为了构建目标需要执行的动作。Bazel 依据此图来跟踪文件变动，并确定哪些目标需要重新构建
3. 针对输入执行构建动作，直到最终的构建输出产生出来

#### 如何使用

当你需要构建或者测试一个项目时，通常执行以下步骤：

1. 下载并安装Bazel
2. 创建一个工作空间。Bazel 从此工作空间寻找构建输入和 BUILD 文件，同时也将构建输出存放在（指向）工作空间（的符号链接中 bazel-bin ）
3. 编写 BUILD文件，以及可选的 WORKSPACE 文件，告知 Bazel 需要构建什么，如何构建。此文件基于 Starlark 这种 DSL(Domain Specific Language *领域特定语言* )
4. 从命令行调用 Bazel 命令，构建、测试或者运行项目

#### 概念和术语

##### 工作空间 Workspace

工作空间是一个目录，它包含：

1. 构建目标所需要的源码文件，以及相应的 BUILD 文件
2. 指向构建结果的符号链接
3. **WORKSPACE 文件**，可以为空，**可以包含对外部依赖的引用**

##### 包 Package

包是工作空间中主要的代码组织单元，其中包含一系列相关的文件（主要是代码）以及描述这些文件之间关系的BUILD 文件

**包是工作空间的子目录，它的根目录必须包含文件BUILD.bazel或BUILD**。除了那些具有 BUILD 文件的子目录（子包）以外，其它子目录属于包的一部分

##### 目标 Target

包是一个容器，它的元素定义在BUILD文件中，包的元素称为**目标**，包括：

1. 规则（Rule），指定输入集和输出集之间的关系，声明从输入产生输出的必要步骤。**一个规则的输出可以是另外一个规则的输入**
2. 文件（File），可以分为两类：
   1. 源文件
   2. 自动生成的文件（Derived files），由构建工具依据规则生成
3. **包组：**一组包，包组用于限制特定规则的**可见性(visibility)**。包组由函数package_group定义，参数是包的列表和包组名称。你可以在规则的 visibility 属性中引用包组，声明那些包组可以引用当前包中的规则

任何包生成的文件都属于当前包，不能为其它包生成文件。但是可以从其它包中读取输入

##### 标签 Label

引用一个目标时需要使用“标签”。标签的规范化表示  `@project//my/app/main : app_binary`， **冒号前面是所属的包名，后面是目标名**。如果不指定目标名，则默认以包路径最后一段作为目标名，例如：

```shell
//my/app
//my/app:app
```

这两者是等价的。在BUILD文件中，引用当前包中目标时，包名部分可以省略，因此下面四种写法都可以等价：

```shell
# 当前包为my/app
//my/app:app
//my/app
:app
app
```

在BUILD文件中，**引用当前包中定义的规则时，冒号不能省略**。引用当前包中文件时，冒号可以省略。 例如： generate.cc。

但是，**从其它包引用时、从命令行引用时，都必须使用完整的标签 ** ：//my/app:generate.cc

@project 这一部分通常不需要使用，引用外部存储库中的目标时，project 填写外部存储库的名字。

##### 规则 Rule

规则指定**输入和输出**之间的关系，并且说明产生输出的步骤。

规则有很多类型。每个规则都具有一个名称属性，此名称亦即目标名称。对于某些规则，此名称就是产生的输出的文件名。

在BUILD中声明规则的语法时：

```python
规则类型(
    name = "...",
    其它属性 = ...
)
```

##### BUILD 文件

BUILD 文件定义了包的所有元数据。其中的语句被从上而下的逐条解释，某些语句的顺序很重要， 例如**变量必须先定义后使用，但是规则声明的顺序无所谓**。

BUILD 文件仅能包含 ASCII 字符，且不得声明函数、使用 for/if 语句，你可以在 Bazel 扩展,扩展名为.bzl 的文件中声明函数、控制结构。并在BUILD文件中用load语句加载Bazel扩展：

```python
load("//foo/bar:file.bzl", "some_library")
```

上面的语句加载 foo/bar/file.bzl 并添加其中定义的符号 some_libraray 到当前环境中，load 语句可以用来加载规则、函数、常量（字符串、列表等）。

load语句必须出现在顶级作用域，不能出现在函数中。第一个参数说明扩展的位置，你可以为导入的符号设置别名。

规则的类型，一般以编程语言为前缀，例如cc，java，后缀通常有：

1. *_binary 用于构建目标语言的可执行文件
2. *_test 用于自动化测试，其目标是可执行文件，如果测试通过应该退出0
3. *_library 用于构建目标语言的库 

##### 依赖 Dependency

目标 A 依赖 B，就意味着 A 在构建或执行期间需要 B。所有目标的依赖关系构成非环有向图(DAG)称为依赖图。

距离为1的依赖称为直接依赖，大于1的依赖则称为传递性依赖。

依赖分为以下几种：

1. **srcs 依赖：**直接被当前规则消费的文件
2. **deps 依赖：**独立编译的模块，为当前规则提供头文件、符号、库、数据
3. **data 依赖：**不属于源码，不影响目标如何构建，但是目标在运行时可能依赖之

### 安装

安装主要参考官网[安装教程](https://docs.bazel.build/versions/4.0.0/install-ubuntu.html#install-with-installer-ubuntu)

本机安装环境为 Ubuntu 18.04 64bit

安装方式：源码安装

1. 安装依赖

   ```shell
   $ apt install g++ unzip zip   #下载 g++ 解压缩工具
   $ apt-get install openjdk-11-jdk  #下载JDk 11
   ```

2. 下载源码并安装

   ```shell
   #第一步github上下载所需的bazel版本如4.0.0
   $ chmod +x bazel-4.0.0-installer-linux-x86_64.sh   #添加权限
   $ ./bazel-4.0.0-installer-linux-x86_64.sh --user   #安装执行
   ```

3. 设置环境变量

   ```shell
   vim ~/.bashrc   #打开当前登录用户家路径下的文件
   export PATH="$PATH:$HOME/bin"  #在 ~/.bashrc文件末尾追加此内容，然后重启
   $ echo $PATH #查看PATH路径
   ```

### 入门

#### 构建C++项目

[官方c++教程](https://docs.bazel.build/versions/4.0.0/tutorial/cpp.html)已经很详细了

##### 示例项目

执行下面的命令下载示例项目：

```shell
git clone https://github.com/bazelbuild/examples/
```

你可以看到stage1、stage2、stage3 这几个WORKSPACE：

```shell
examples
└── cpp-tutorial
    ├──stage1
    │  ├── main
    │  │   ├── BUILD
    │  │   └── hello-world.cc
    │  └── WORKSPACE
    ├──stage2
    │  ├── main
    │  │   ├── BUILD
    │  │   ├── hello-world.cc
    │  │   ├── hello-greet.cc
    │  │   └── hello-greet.h
    │  └── WORKSPACE
    └──stage3
       ├── main
       │   ├── BUILD
       │   ├── hello-world.cc
       │   ├── hello-greet.cc
       │   └── hello-greet.h
       ├── lib
       │   ├── BUILD
       │   ├── hello-time.cc
       │   └── hello-time.h
       └── WORKSPACE
```

本节后续内容会依次使用到这三个WORKSPACE。**从图中可以看到stage1、stage2、stage3 分别是包，根目录包含 BUILD，其次在包 stage3 中 lib 是子包（包含BUILD）**

##### 通过Bazel构建

第一步是创建工作空间。工作空间中包含以下特殊文件：

1. WORKSPACE，此文件位于根目录中，将当前目录定义为Bazel工作空间
2. BUILD，告诉Bazel项目的不同部分如何构建。工作空间中包含BUILD文件的目录称为包

当Bazel构建项目时，所有的输入和依赖都必须位于工作空间中。除非被链接，不同工作空间的文件相互独立没有关系。

每个 BUILD 文件包含若干 Bazel 指令，其中最重要的指令类型是构建规则（Build Rule），构建规则说明如何产生期望的输出--例如可执行文件或库。 BUILD中的每个构建规则也称为目标（Target），目标指向若干源文件和依赖，也可以指向其它目标。

下面是stage1的BUILD文件：

```shell
#cpp-tutorial/stage1/main/BUILD
cc_binary(
    name = "hello-world",
    srcs = ["hello-world.cc"],
)
```

这里定义了一个名为 hello-world 的目标，它使用了内置的 cc_binary 规则。该规则告诉Bazel，从源码hello-world.cc 构建一个自包含的可执行文件。

执行下面的命令可以触发构建：

```shell
#   //main: BUILD文件相对于工作空间的位置
#          hello-world 是BUILD文件中定义的目标
bazel build //main:hello-world
```

构建完成后，工作空间根目录会出现bazel-bin等目录，它们都是指向 $HOME/.cache/bazel 某个后代目录的符号链接。执行：

```shell
bazel-bin/main/hello-world
```

可以运行构建好的二进制文件。

查看依赖图

Bazel会根据BUILD中的声明产生一张依赖图，并根据这个依赖图实现精确的增量构建。

```shell
apt install graphviz xdot
bazel query --nohost_deps --noimplicit_deps 'deps(//main:hello-world)' --output graph | xdot
```

##### 指定多个目标 

大型项目通常会划分为多个包、多个目标，以实现更快的增量构建、并行构建。工作空间stage2包含单个包、两个目标：

```shell
#cpp-tutorial/stage2/main/BUILD
# 首先构建hello-greet库，cc_library是内建规则
cc_library(
    name = "hello-greet",
    srcs = ["hello-greet.cc"],
    # 头文件
    hdrs = ["hello-greet.h"],
)

# 然后构建hello-world二进制文件
cc_binary(
    name = "hello-world",
    srcs = ["hello-world.cc"],
    deps = [
        # 提示Bazel，需要hello-greet才能构建当前目标
        # 依赖当前包中的hello-greet目标
        ":hello-greet",
    ],
)
```

##### 使用多个包

工作空间stage3更进一步的划分出新的包，提供打印时间的功能：

```shell
#cpp-tutorial/stage3/lib/BUILD
cc_library(
    name = "hello-time",
    srcs = ["hello-time.cc"],
    hdrs = ["hello-time.h"],
    # 让当前目标对于工作空间的main包可见。默认情况下目标仅仅被当前包可见
    visibility = ["//main:__pkg__"],
)
```

```shell
#cpp-tutorial/stage3/main/BUILD
cc_library(
    name = "hello-greet",
    srcs = ["hello-greet.cc"],
    hdrs = ["hello-greet.h"],
)
 
cc_binary(
    name = "hello-world",
    srcs = ["hello-world.cc"],
    deps = [
        # 依赖当前包中的hello-greet目标
        ":hello-greet",
        # 依赖工作空间根目录下的lib包中的hello-time目标
        "//lib:hello-time",
    ],
)
```

在BUILD文件或者命令行中，你都使用标签（Label）来引用目标。

#### 构建 Java 项目

自行参考官网 [java教程](https://docs.bazel.build/versions/4.0.0/tutorial/java.html)

### 目录结构

```shell
workspace-name>/                          # 工作空间根目录
  bazel-my-project => <...my-project>     # execRoot的符号链接，所有构建动作在此目录下执行
  bazel-out => <...bin>                   # outputPath的符号链接
  bazel-bin => <...bin>                   # 最近一次写入的二进制目录的符号链接，即$(BINDIR)
  bazel-genfiles => <...genfiles>         # 最近一次写入的genfiles目录的符号链接，即$(GENDIR)
 
/home/user/.cache/bazel/                  # outputRoot，所有工作空间的Bazel输出的根目录
  _bazel_$USER/                           # outputUserRoot，当前用户的Bazel输出的根目录
    install/
      fba9a2c87ee9589d72889caf082f1029/   # installBase，Bazel安装清单的哈希值
        _embedded_binaries/               # 第一次运行时从Bazel可执行文件的数据段解开的可执行文件或脚本
    7ffd56a6e4cb724ea575aba15733d113/     # outputBase，某个工作空间根目录的哈希值
      action_cache/                       # Action cache目录层次
      action_outs/                        # Action output目录
      command.log                         # 最近一次Bazel命令的stdout/stderr输出
      external/                           # 远程存储库被下载、链接到此目录
      server/                             # Bazel服务器将所有服务器有关的文件存放在此
        jvm.out                           # Bazel服务器的调试输出
      execroot/                           # 所有Bazel Action的工作目录
        <workspace-name>/                 # Bazel构建的工作树
          _bin/                           # 助手工具链接或者拷贝到此
          bazel-out/                      # outputPath，构建的实际输出目录
            local_linux-fastbuild/        # 每个独特的BuildConfiguration实例对应一个子目录
              bin/                        # 单个构建配置二进制输出目录，$(BINDIR)
                foo/bar/_objs/baz/        # 命名为//foo/bar:baz的cc_*规则的Object文件所在目录
                  foo/bar/baz1.o          # //foo/bar:baz1.cc对应的Object文件
                  other_package/other.o   # //other_package:other.cc对应的Object文件
                foo/bar/baz               # //foo/bar:baz这一cc_binary生成的构件
                foo/bar/baz.runfiles/     # //foo/bar:baz生成的二进制构件的runfiles目录
                  MANIFEST
                  <workspace-name>/
                    ...
              genfiles/                   # 单个构建配置生成的源文件目录，$(GENDIR)
              testlogs/                   # Bazel的内部测试运行器将日志文件存放在此
              include/                    # 按需生成的include符号链接树，符号链接bazel-include指向这里
            host/                         # 本机的BuildConfiguration
        <packages>/                       # 构建引用的包，对于此包来说，它就像一个正常的WORKSPACE 
```

### 外部依赖

Bazel允许依赖其它项目中定义的目标，这些来自其它项目的依赖叫做“外部依赖“。**当前工作空间的WORKSPACE 文件声明从何处下载外部依赖的源码。**

外部依赖可以有自己的1-N个BUILD文件，其中定义自己的目标。当前项目可以使用这些目标。例如下面的两个项目结构：

```shell
/
  home/
    user/
      project1/
        WORKSPACE
        BUILD
        srcs/
          ...
      project2/
        WORKSPACE
        BUILD
        my-libs/
```

如果 project1 需要依赖定义在 project2/BUILD 中的目标 :foo，则可以在其 WORKSPACE 中声明一个存储库（repository），名字为 project2，位于 /home/user/project2 。然后，可以在BUILD中通过标签`@project2//:foo` 引用目标foo。

除了依赖来自文件系统其它部分的目标、下载自互联网的目标以外，用户还可以编写自己的存储库规则（repository rules ）以实现更复杂的行为。

WORKSPACE 的语法格式和 BUILD 相同，但是允许使用[不同的规则集](https://docs.bazel.build/versions/master/be/workspace.html)。

Bazel会把外部依赖下载到 $(bazel info output_base)/external目录中，要删除掉外部依赖，执行：

```shell
bazel clean --expunge
```

#### 外部依赖类型

##### Bazel项目

可以使用 local_repository、git_repository 或者 http_archive 这几个规则来引用。

引用本地Bazel项目的例子：

```shell
local_repository(
    name = "coworkers_project",
    path = "/path/to/coworkers-project",
)
```

在BUILD中，引用coworkers_project中的目标//foo:bar时，使用标签 @coworkers_project//foo:bar 

##### 非Bazel项目

可以使用new_local_repository、new_git_repository或者new_http_archive这几个规则来引用。你需要自己编写BUILD文件来构建这些项目。

引用本地非Bazel项目的例子：

```shell
new_local_repository(
    name = "coworkers_project",
    path = "/path/to/coworkers-project",
    build_file = "coworker.BUILD",
)

cc_library(
    name = "some-lib",
    srcs = glob(["**"]),
    visibility = ["//visibility:public"],
)&nbsp;
```

在BUILD文件中，使用标签@coworkers_project//:some-lib引用上面的库。 

##### 外部包

对于Maven仓库，可以使用规则maven_jar/maven_server来下载JAR包，并将其作为Java依赖。

多查看一下 [规则](https://docs.bazel.build/versions/4.0.0/rules.html) ，如如何引用 [go规则](https://github.com/bazelbuild/rules_go)  包括 go_repository

#### 依赖拉取

默认情况下，执行bazel Build时会按需自动拉取依赖，你也可以禁用此特性，并使用bazel fetch预先手工拉取依赖。

#### 依赖缓存

Bazel会缓存外部依赖，当WORKSPACE改变时，会重新下载或更新这些依赖。

### 常见规则用例

通配符

可以使用glob语法为目标添加多个文件：

```shell
pkg_tar(
    name = "licenses",
    srcs = glob(["**/licenses.yaml"]),  #通配符，使用glob语法可以为目标添加多个文件
    extension = "tar",
    strip_prefix = ".",   
)
```

args 以内存的方式储存命令

```shell
args = [
        "$(location build.sh)",
        "$(location //licenses:licenses.tar)",
    ],
```

pkg_tar() 生成tar包 压缩包

```shell
pkg_tar(
    name = "licenses",
    srcs = glob(["**/licenses.yaml"]),
    extension = "tar",
    strip_prefix = ".", #文件根目录 A relative path may start with "./" (or be ".") but cannot use ".." to go up level(s)
)
```

> 借鉴前人[笔记](https://blog.gmem.cc/bazel-study-note)，如有侵权，删除 



