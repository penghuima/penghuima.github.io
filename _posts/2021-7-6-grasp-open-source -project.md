---
layout: post
title: "快速熟悉开源项目"
categories: 开源项目
tags: 开源
---

* content
{:toc}


## 快速熟悉开源项目

如今的互联网越来越发达，各类开源项目如雨后春笋般不断涌现。开源社区和 GitHub 等知名开源代码托管平台，让我们免去了重新造轮子的痛苦，给了我们很多借鉴和学习的机会。在互相协作的过程中，开源项目不断涌现出生机与活力。

快速熟悉一个开源项目一般分为查阅文档、动手实践以及源码阅读三个步骤。

<!--more-->

### Step1: 查阅文档

查阅文档包括查阅文档与博客，且最好是带着问题去阅读。

#### 查阅文档与博客

一个好的开源项目未必会火，但一个火起来的开源项目一定有可取之处，而从众心理又会让更多人去研究它。所以，要熟悉你想研究的开源项目，第一步就是在搜索引擎中查找该项目的 博客和资料。通过快速阅读介绍开源项目架构、使用方法等这类文章，你就能大体了解该项目的意义、功能和基本使用方法。

通过搜索到的资料，如果你觉得该项目就是自己想要的，那么便可以有耐心地开始阅读项目官方提供地文档，从中学习一些具体的下载、安装和使用的方法，以便了解项目的全貌。

#### 带着问题去阅读

阅读文档的过程不能盲目，需要带着如下问题去阅读。

- 这个项目解决了什么问题

- 这个项目涉及了哪些成熟的技术
- 这个项目是否符合我的要求（用户规模、使用场景、性能、安全性等等）？
- 当阅读完文档后，是否能尝试画出大致的架构图？

### Step2: 动手实践

实践是最好的老师，在阅读文档的过程中，按照文档的操作指南亲手实践，不但有助于加深理解，同时还会注意到很多细节，可以更清晰地感受到项目是否符合自己的需求。

#### 搭建项目

实践过程一般都遵循项目的 README.md 文件，进行部署安装和尝试。如果有现成使用项目的示例代码，那么也可以按照示例代码进行尝试。此时若是运行顺利，则可以尝试着根据自己的理解对示例代码进行修改。若是出现问题也无需紧张，只需要将问题的异常信息当成关键词去搜索引擎中查找即可。如果实在找不到解决方案，那么就可以提交到开源项目的邮件列表中，开源社区的人们一般都比较热心，相信很快就可以解决问题。

解答疑惑的好网站这里就不推荐了

#### 深层次改动

有趣的是，很多开源项目一般都会为了方便用户使用，提供 release 的版本。如果基本的部署和使用已经成功的话，强烈建议试着从源码构建和部署该项目。这样你就能从开发、调试到发布整个一体化的全部过程，由此全方位地感受项目的优缺点。

基本的尝试过后，我们就可以开始使用该项目的一些高级功能，如一些高级配置项、较为复杂的 API 等。相信一个运作良好的开源项目，为了方便社区的贡献者们可以快速加入，必然会提供一份较为详尽的指南，你只需要挑选你感兴趣的部分阅读即可。

### Step3: 阅读源码

经过上述两步之后，你必然对项目的大致情况已经了然于胸，想要更深入地了解自然非阅读源码莫属了。

一般阅读源码有两种习惯方式，一是根据命令行地代码调用过程阅读；二是根据架构分模块阅读。

#### 跟着运行过程阅读

刚上手的过程可以使用第一种方式。通过在实践过程中对某个命令或参数的理解，从主干开始，一步一步清理这个命令在运行过程中代码调用的路径。通过 debug 工具观察变量和函数、修改源码打印日志，可以更好地帮助你理解源码。

当清理这个过程后，可以将这个过程用流程图的形式记录下来，从而加深印象，方便下次阅读的时候快速会议和对比。

#### 分模块阅读

在清理了程序运行的基本流程后，可以根据架构上各个模块的作用，挑选你感兴趣的部分阅读，如网络、存储、通信、用户接口、界面等，选择一个模块、深入到实现的细节之中。

此时也可以带着如下几个问题帮助自己理解

- 调用了什么底层库
- 采用了什么设计模式
- 这样写有什么好处？

如果在阅读源码的过程中出现瓶颈，你一时无法理解代码的用意，不妨去阅读一下相关的单元测试。一个好的但愿你测试通常都描述了要测试代码的主要功能和数据边界，通过运行和理解单元测试，可以有效地帮助理解源码。

相信经过以上三步，你必然已经对这个开源项目非常熟悉了。此时，如果你感兴趣，也可以加入其中为开源社区作出一份贡献。

作为程序猿，有事没事多逛 GitHub !