---
layout: post
title: "OpenFunction的Ruby函数构建器开发"
categories: builder
tags: builder OpenFunction Ruby
---

* content
{:toc}


### 项目概述

项目目标： 开发并优化函数构建器，使其可以识别 Ruby 语言，并将函数构建成可以运行在 OpenFunction 中的应用镜像。 

项目背景： OpenFunction 是一个云原生的开源函数即服务（Functions-as-a-Service）平台，旨在让用户专注于他们的业务逻辑，而不必担心底层运行环境和基础设施。用户只需要以函数的形式提交业务相关的源代码，即可将服务按需运行在集群中。 Tekton 是一个云原生的 CI/CD 项目。 Cloud Native Buildpacks （以下简称 Buildpacks ）是一个云原生的 OCI 标准镜像制作工具。

项目详情： OpenFunction 的 builders 组件集成了 Tekton 与 Buildpacks ，用于完成将用户提交的函数代码制作成可以运行在 Kubernetes 中的应用负载镜像这一部分工作。 Tekton 负责编排整个由代码到应用镜像的制作过程，包含：拉取镜像、准备环境变量、使用 Buildpacks 制作镜像、输出镜像到指定仓库 等几个主要步骤。 

Buildpacks 负责识别代码的语言类型，然后准备好相关的编译环境，再将代码通过模板渲染为可运行的应用。 我们希望： 它可以支持更多的主流语言（及更多的语言版本） 可以优化代码构建的策略、减少代码构建的时间。

<!--more-->

### 本地环境

- Ubuntu 18.04.4   user:ubuntu
- CPU Core * 8
- Memory * 8G
- Goland 2020.3

### 依赖工具

整个开发过程需要需要 container-structure-test、Bazel、pack、Docker 这些工具，确保这些工具的执行文件处于 $PATH 目录下。

- 安装 Docker

  ```shell
  sudo apt install docker.io
  sudo usermod -aG docker ubuntu
  sudo chmod a+rw /var/run/docker.sock
  sudo systemctl restart docker 
  ```

- 安装pack

  ```shell
  curl -sSL "https://github.com/buildpacks/pack/releases/download/v0.18.1/pack-v0.18.1-linux.tgz" | sudo tar -C /usr/local/bin/ --no-same-owner -xzv pack
  ```

- 安装Bazel (v4.1.0)

  ```shell
  sudo apt install g++ unzip zip
  sudo apt-get install openjdk-11-jdk
  wget https://github.com/bazelbuild/bazel/releases/download/4.1.0/bazel-4.1.0-installer-linux-x86_64.sh
  chmod +x bazel-4.1.0-installer-linux-x86_64.sh
  ./bazel-4.1.0-installer-linux-x86_64.sh --user
  sudo mv $HOME/bin/bazel /usr/local/bin
  ```
  
- 安装 container-structure-test

  ```shell
  curl -LO https://storage.googleapis.com/container-structure-test/latest/container-structure-test-linux-amd64 && chmod +x container-structure-test-linux-amd64 && sudo mv container-structure-test-linux-amd64 /usr/local/bin/container-structure-test
  ```

- 建立一条 python3 软连接(可以不需要)

  ```shell
  sudo ln -s /usr/bin/python3 /usr/bin/python
  ```

使用以下命令验证是否已准备好所有依赖：

```shell
git clone https://github.com/OpenFunction/builder.git
cd builder
bazel test --test_output=errors tools/checktools:main_test
```

如果出现类  INFO: Build completed successfully, 9 total actions 提示信息，说明已经所有依赖安装完成

### Goland 配置远程服务器

连接远程服务器，并配置文件同步路径

![image-20210814111538411](https://cdn.jsdelivr.net/gh/penghuima/ImageBed@master/img/blog_file/PicGo-Github-ImgBedimage-20210814111538411.png)

![image-20210814111619937](https://cdn.jsdelivr.net/gh/penghuima/ImageBed@master/img/blog_file/PicGo-Github-ImgBedimage-20210814111619937.png)

> [Create a remote server configuration](https://www.jetbrains.com/help/go/creating-a-remote-server-configuration.html)
>
> [Goland同步插件配置](https://blog.csdn.net/huwh_/article/details/77879152?utm_source=blogxgwz3)

### Build ruby26 stack

#### stack 概述

Stack 的功能是提供基础运行环境，它包含两种镜像：

- build ，为 Lifecycle 的 Build 阶段提供运行环境

- run ，为业务应用提供运行环境

在本项目Openfunction的Ruby函数构建器开发中，需要先准备一个部署有Ruby2.6 runtime 的 stack ，stack 本质为一个基础运行环境，在制作过程中加入 LABEL io.buildpacks.stack.id 即可以指定 stack 的标识。

> 原生 gcp/buildpacks 将下载语言 runtime 的步骤放置在 buildpack 中，即在探测到语言类型后再下载安装对应的语言 runtime ，本项目选择将语言 runtime 安装在 stack 中，以减少函数构建过程中的时间

#### ruby 2.6 runtime

本项目自定义的 ruby2.6 stack ：

```shell
FROM gcr.io/gcp-runtimes/ubuntu_18_0_4
ARG cnb_uid=1000
ARG cnb_gid=1000
ARG stack_id="openfunction.ruby26"
# Required by python/runtime: libexpat1, libffi6, libmpdecc2.
# Required by dotnet/runtime: libicu60
# Required by go/runtime: tzdata (Go may panic without /usr/share/zoneinfo)
RUN apt-get update && apt-get install -y --no-install-recommends \
  libexpat1 \
  libffi6 \
  libmpdec2 \
  libicu60 \
  libc++1-9 \
  tzdata \
  gcc libc6-dev make  zlib1g zlib1g-dev  openssl libssl-dev \
  && apt-get clean && rm -rf /var/lib/apt/lists/*

LABEL io.buildpacks.stack.id=${stack_id}

RUN groupadd cnb --gid ${cnb_gid} && \
  useradd --uid ${cnb_uid} --gid ${cnb_gid} -m -s /bin/bash cnb

# install ruby runtime
RUN cd /tmp && mkdir ruby && cd ruby && curl  https://cache.ruby-lang.org/pub/ruby/2.6/ruby-2.6.6.tar.gz | tar xz \
    && cd ruby-2.6.6 && ./configure && make && make install  \
    && cd ext/zlib \
    && ruby extconf.rb && sed -i 's?$(top_srcdir)?../..?g' Makefile  && make && make install \
    && cd ../openssl \
    && ruby extconf.rb && sed -i 's?$(top_srcdir)?../..?g' Makefile && make && make install \
    && gem uninstall bundle && gem install bundler -v 2.1.4 \
    && cd ../../

ENV CNB_USER_ID=${cnb_uid}
ENV CNB_GROUP_ID=${cnb_gid}
ENV CNB_STACK_ID=${stack_id}
```

> 完整文件请参考 [ruby26/stack](ruby26/stack)

#### 生成 stack

开始在根目录 builder/ 下执行 ruby2.6 stack 构建命令

```shell
bazel run //builders/ruby26/stack:build
```

This command creates three images:

- **ruby26common**
- **openfunctiondev/buildpacks-ruby26-run:v1**
- **openfunctiondev/buildpacks-ruby26-build:v1**

如下所示

```shell
ubuntu@i-a1kwg7lk:~/mph/builder$ docker images
REPOSITORY                                TAG       IMAGE ID       CREATED         SIZE
openfunctiondev/buildpacks-ruby26-build   v1        18a6f7e4c4b6   3 minutes ago   1.04GB
openfunctiondev/buildpacks-ruby26-run     v1        e221e925ecc5   4 minutes ago   791MB
ruby26common                              latest    c21fa60b9e9c   4 minutes ago   791MB
gcr.io/gcp-runtimes/ubuntu_18_0_4         latest    93a9bd0cfd2e   51 years ago    76.7MB
```

验证 ruby 2.6 runtime

```shell
ubuntu@i-a1kwg7lk:~/mph/builder$ docker run -it --name build-runtime openfunctiondev/buildpacks-ruby26-build:v1 /bin/bash
cnb@67882f6e62ab:/$ which ruby
/usr/local/bin/ruby
cnb@67882f6e62ab:/$ bundle -v
Bundler version 2.1.4

ubuntu@i-a1kwg7lk:~/mph/builder$ docker run -it --name run-runtime openfunctiondev/buildpacks-ruby26-run:v1 /bin/bash
cnb@704a89adc952:/$ which ruby
/usr/local/bin/ruby
cnb@704a89adc952:/$ bundle -v
Bundler version 2.1.4
```

#### 上传镜像 

可以将 build 和 run 镜像上传到自己的镜像仓库

docker login -u xxxxx(dockerhub注册的docker用户名)，登录docker成功

```shell
docker login -u penghuima
docker tag openfunctiondev/buildpacks-ruby26-build:v1 penghuima/buildpacks-ruby26-build:v1
docker tag openfunctiondev/buildpacks-ruby26-run:v1 penghuima/buildpacks-ruby26-run:v1
docker push penghuima/buildpacks-ruby26-build:v1
docker push penghuima/buildpacks-ruby26-run:v1
```

### 生成builder

#### builder概述

Builder 描述了一个完整的构建编排逻辑。它本质上是将  Stack 及若干个 Buildpack 组合在一起，形成一个业务逻辑明确的构建工具。Builder是一个镜像，其中包含执行构建所需的所有组件。构建器镜像是通过stack镜像并添加生命周期、构建包和文件来创建的，这些文件配置构建的各个方面，包括构建包检测顺序和运行镜像的位置。

![create-builder diagram](https://buildpacks.io/docs/concepts/components/create-builder.svg)

本项目builder的定义文件 builder.toml内容如下

```shell
description = "Builder for the  Ruby 2.6"

[[buildpacks]]
  id = "google.ruby.functions-framework"
  uri = "ruby/functions_framework.tgz"

[[buildpacks]]
  id = "google.ruby.bundle"
  uri = "ruby/bundle.tgz"

[[buildpacks]]
  id = "google.config.entrypoint"
  uri = "entrypoint.tgz"

[[buildpacks]]
  id = "google.utils.label"
  uri = "label.tgz"

[[order]]

  [[order.group]]
    id = "google.ruby.bundle"

  [[order.group]]
    id = "google.ruby.functions-framework"

  [[buildpacks]]
    id = "google.config.entrypoint"

  [[order.group]]
    id = "google.utils.label"

[stack]
  id = "openfunction.ruby26"
  build-image = "openfunctiondev/buildpacks-ruby26-build:v1"
  run-image = "openfunctiondev/buildpacks-ruby26-run:v1"

[lifecycle]
  version = "0.11.1"
```

- [[buildpacks]] 表示引入的 buildpack ，已经用 bazel 工具将相关代码目录打包成了 tgz 文件，就是封装了 /bin/detect 和 /bin/build 两个功能。
- [[order]] [[order.group]] 表示 buildpack 的执行顺序，从下文 of/ruby26镜像构成的 app镜像运行日志中，可以看到运行检测顺序与 [[order]] 顺序一致
- [stack] 表示引用的 stack 内容，其中stack id 唯一标识我们上文中创建的 stack
- [lifecycle] 表示使用的 lifecycle 模块的版本

#### buildpacks

Buildpack 可以理解为任务单一的构建单元，用于检查您的应用程序源代码并制定构建和运行应用程序的计划。每一个 buildpack 都具备以下三个元素：

- buildpack.toml ，描述了 buildpack 的元数据。
- bin/detect ，Lifecycle 中 Detect 阶段的实现，确定是否应用该buildpack。
- bin/build ，Lifecycle 中 Build 阶段的实现，执行buildpack逻辑。

它的工作原理可以简单理解为：bin/detect 将会判断是否可以进入该 buildpack ，如果可以进入该 buildpack ，那么 Lifecycle 就会指导流程进入 Build 阶段同时触发 bin/build 执行。

每个 buildpack 完成后都会产出一份清单，用于表明自己产出了什么内容（可为空），且如果要完成后续的 buildpack 过程，还需要什么内容（可为空）。 

同时在 Build 阶段，基于复用构建逻辑的原则，buildpack 的作者可以设置不通过的 layer （layer 内保存了所关联的构建过程的结果，如文件、运行时等内容），并设置 build 、launch 、cache 三种属性。Lifecycle 将根据这三个参数的组合来决定是否复用一个 layer。 

以 id 为  "google.ruby.functions-framework" 的 ruby 函数框架这一 buildpack 为例，其元数据在 [buildpack] 中定义，[[stacks]] 用于描述 buildpack 可以运行在哪些 stack 上

```shell
api = "0.2"

[buildpack]
id = "google.ruby.functions-framework"
version = "0.9.1"
name = "Ruby - Functions Framework"

[[stacks]]
id = "openfunction.ruby26"

[[stacks]]
id = "google.ruby27"
```

#### 生成 builder

开始在根目录 builder/ 下执行 ruby2.6 builder 构建命令

```shell
bazel build //builders/ruby26:builder.image
```

This command creates one image:

- **of/ruby26**

生成 builder 日志

```sh
ubuntu@i-a1kwg7lk:~/mph/builder$ bazel build //builders/ruby26:builder.image
INFO: Analyzed target //builders/ruby26:builder.image (54 packages loaded, 6870 targets configured).
INFO: Found 1 target...
INFO: From Executing genrule //builders/ruby26:builder.image:
2021/08/14 14:58:20 Checking tools
2021/08/14 14:58:20 pack found at /usr/local/bin/pack
2021/08/14 14:58:20 docker found at /usr/bin/docker
2021/08/14 14:58:20 container-structure-test found at /usr/local/bin/container-structure-test
2021/08/14 14:58:20 Checking pack version
2021/08/14 14:58:20 Found pack version 0.18.1+git-b5c1a96.build-2373
Extracting builder tar:
./
./builder.toml
./label.tgz
./ruby/
./ruby/functions_framework.tgz
./ruby/bundle.tgz
Warning: Command 'pack create-builder' has been deprecated, please use 'pack builder create' instead
Downloading from 'https://github.com/buildpacks/lifecycle/releases/download/v0.11.1/lifecycle-v0.11.1+linux.x86-64.tgz'
5.25 MB/5.25 MB
Successfully created builder image 'of/ruby26'
Tip: Run 'pack build <image-name> --builder of/ruby26' to use this builder
Target //builders/ruby26:builder.image up-to-date:
  bazel-bin/builders/ruby26/builder.sha
INFO: Elapsed time: 14.209s, Critical Path: 12.22s
INFO: 48 processes: 19 internal, 28 linux-sandbox, 1 local.
INFO: Build completed successfully, 48 total actions
```

### Demo 展示

本节通过展示一个简单的  hello  word 例子来演示 builder 是如何工作的

> git clone https://github.com/penghuima/samples.git    
>
> cd  samples/functions/Knative/hello-world-ruby/

sample里主要有三个文件 app.rb、Gemfile、Gemfile.lock 内容如下

- app.rb

  ```ruby
  require "functions_framework"
  FunctionsFramework.http("Hello-World") do |request|
    "Hello, world!  Hello Ruby!\n"
  end
  ```

- Gemfile

  ```ruby
  source "https://rubygems.org"
  gem "functions_framework", "~> 1.0"
  ```

- Gemfile.lock

  ```ruby
  GEM
    remote: https://rubygems.org/
    specs:
      cloud_events (0.5.1)
      functions_framework (1.0.0)
        cloud_events (>= 0.5.1, < 2.a)
        puma (>= 4.3.0, < 6.a)
        rack (~> 2.1)
      nio4r (2.5.7)
      puma (5.3.2)
        nio4r (~> 2.0)
      rack (2.2.3)
  
  PLATFORMS
    ruby
  
  DEPENDENCIES
    functions_framework (~> 1.0)
  
  BUNDLED WITH
     2.1.4
  ```

#### 生成 app image

原理如下图所示，由 builder镜像和源码文件 生成 app images

![build diagram](https://buildpacks.io/docs/concepts/operations/build.svg)

Download samples:

```shell
git clone https://github.com/penghuima/samples.git    
cd  samples/functions/Knative/hello-world-ruby/
```

Build the function:

```shell
pack build function-ruby --builder of/ruby26 --env FUNC_NAME="Hello-World"
docker run --rm -p8080:8080 function-ruby
```

Visit the function:

```shell
curl http://localhost:8080
```

Output example:

```shell
Hello, world!  Hello Ruby!
```

#### 运行日志

```shell
ubuntu@i-a1kwg7lk:~/mph/samples/functions/Knative/hello-world-ruby$ pack build function-ruby --builder of/ruby26 --env FUNC_NAME="Hello-World"
v1: Pulling from openfunctiondev/buildpacks-ruby26-run
Digest: sha256:f428f2272085771fd21353d3c3ba58ce1e755f47608dc11e2bd19d5f85df4066
Status: Image is up to date for openfunctiondev/buildpacks-ruby26-run:v1
0.11.1: Pulling from buildpacksio/lifecycle
Digest: sha256:2ca9870d9472400792bdc2d43ded8df2642ffcf937e8cd01685836678c28780b
Status: Image is up to date for buildpacksio/lifecycle:0.11.1
===> DETECTING
[detector] google.ruby.bundle              0.9.0
[detector] google.ruby.functions-framework 0.9.1
[detector] google.utils.label              0.0.1
===> ANALYZING
[analyzer] Previous image with name "function-ruby" not found
===> RESTORING
===> BUILDING
[builder] === Ruby - Bundle (google.ruby.bundle@0.9.0) ===
[builder] Installing application dependencies.
[builder] --------------------------------------------------------------------------------
[builder] Running "bundle config --local without development test"
[builder] Done "bundle config --local without development test" (213.701121ms)
[builder] --------------------------------------------------------------------------------
[builder] Running "bundle config --local path .bundle/gems"
[builder] Done "bundle config --local path .bundle/gems" (204.372547ms)
[builder] --------------------------------------------------------------------------------
[builder] Running "bundle lock --add-platform x86_64-linux"
[builder] Fetching gem metadata from https://rubygems.org/...
[builder] Resolving dependencies...
[builder] Writing lockfile to /workspace/Gemfile.lock
[builder] Done "bundle lock --add-platform x86_64-linux" (4.131479657s)
[builder] --------------------------------------------------------------------------------
[builder] Running "bundle lock --add-platform ruby"
[builder] Fetching gem metadata from https://rubygems.org/..
[builder] Writing lockfile to /workspace/Gemfile.lock
[builder] Done "bundle lock --add-platform ruby" (1.109658296s)
[builder] --------------------------------------------------------------------------------
[builder] Running "bundle config --local deployment true"
[builder] Done "bundle config --local deployment true" (206.397619ms)
[builder] --------------------------------------------------------------------------------
[builder] Running "bundle config --local frozen true"
[builder] Done "bundle config --local frozen true" (192.510552ms)
[builder] --------------------------------------------------------------------------------
[builder] Running "bundle config --local without development test"
[builder] Done "bundle config --local without development test" (214.752127ms)
[builder] --------------------------------------------------------------------------------
[builder] Running "bundle config --local path .bundle/gems"
[builder] Done "bundle config --local path .bundle/gems" (218.378794ms)
[builder] --------------------------------------------------------------------------------
[builder] Running "bundle install (NOKOGIRI_USE_SYSTEM_LIBRARIES=1 MALLOC_ARENA_MAX=2 LANG=C.utf8)"
[builder] Fetching gem metadata from https://rubygems.org/..
[builder] Using bundler 2.1.4
[builder] Fetching cloud_events 0.5.1
[builder] Installing cloud_events 0.5.1
[builder] Fetching nio4r 2.5.7
[builder] Installing nio4r 2.5.7 with native extensions
[builder] Fetching puma 5.3.2
[builder] Installing puma 5.3.2 with native extensions
[builder] Fetching rack 2.2.3
[builder] Installing rack 2.2.3
[builder] Fetching functions_framework 1.0.0
[builder] Installing functions_framework 1.0.0
[builder] Bundle complete! 1 Gemfile dependency, 6 gems now installed.
[builder] Gems in the groups development and test were not installed.
[builder] Bundled gems are installed into `./.bundle/gems`
[builder] Done "bundle install (NOKOGIRI_USE_SYSTEM_LIBRARIES=1 MALLOC_ARENA..." (8.561774732s)
[builder] === Ruby - Functions Framework (google.ruby.functions-framework@0.9.1) ===
[builder] --------------------------------------------------------------------------------
[builder] Running "bundle exec functions-framework-ruby --quiet --verify --source app.rb --target Hello-World (MALLOC_ARENA_MAX=2 LANG=C.utf8 RACK_ENV=production)"
[builder] OK
[builder] Done "bundle exec functions-framework-ruby --quiet --verify --sour..." (319.197753ms)
[builder] === Utils - Label Image (google.utils.label@0.0.1) ===
===> EXPORTING
[exporter] Adding layer 'google.ruby.bundle:gems'
[exporter] Adding layer 'google.ruby.functions-framework:functions-framework'
[exporter] Adding 1/1 app layer(s)
[exporter] Adding layer 'launcher'
[exporter] Adding layer 'config'
[exporter] Adding layer 'process-types'
[exporter] Adding label 'io.buildpacks.lifecycle.metadata'
[exporter] Adding label 'io.buildpacks.build.metadata'
[exporter] Adding label 'io.buildpacks.project.metadata'
[exporter] Setting default process type 'web'
[exporter] Saving function-ruby...
[exporter] *** Images (d46c60a6b4b8):
[exporter]       function-ruby
[exporter] Adding cache layer 'google.ruby.bundle:gems'
Successfully built image function-ruby
ubuntu@i-a1kwg7lk:~/mph/samples/functions/Knative/hello-world-ruby$ docker images
REPOSITORY                                TAG       IMAGE ID       CREATED        SIZE
openfunctiondev/buildpacks-ruby26-build   v1        18a6f7e4c4b6   6 hours ago    1.04GB
openfunctiondev/buildpacks-ruby26-run     v1        e221e925ecc5   6 hours ago    791MB
buildpacksio/lifecycle                    0.11.1    67c0e8a0dbed   41 years ago   15.7MB
of/ruby26                                 latest    19d1b18882a7   41 years ago   1.07GB
function-ruby                             latest    d46c60a6b4b8   41 years ago   801MB
ubuntu@i-a1kwg7lk:~/mph/samples/functions/Knative/hello-world-ruby$ docker run --rm -p8080:8080 function-ruby
I, [2021-08-14T09:41:20.648220 #1]  INFO -- : FunctionsFramework v1.0.0
I, [2021-08-14T09:41:20.648325 #1]  INFO -- : FunctionsFramework: Loading functions from "./app.rb"...
I, [2021-08-14T09:41:20.648584 #1]  INFO -- : FunctionsFramework: Looking for function name "Hello-World"...
I, [2021-08-14T09:41:20.648653 #1]  INFO -- : FunctionsFramework: Starting server...
I, [2021-08-14T09:41:20.715425 #1]  INFO -- : FunctionsFramework: Serving function "Hello-World" on port 8080...
ubuntu@i-a1kwg7lk:~$ curl http://localhost:8080
Hello, world!  Hello Ruby!
```

### Q & A

**Q1:** ERROR: failed to add buildpacks to builder: downloading buildpack: extracting from registry 'bundle.tgz': fetching image: image 'bundle.tgz' does not exist on the daemon: not found
Extracting builder tar:

```shell
openfunction6@cloudshell:~/mph/builder$ bazel build //builders/ruby26:builder.image
INFO: Analyzed target //builders/ruby26:builder.image (1 packages loaded, 5 targets configured).
INFO: Found 1 target...
ERROR: /home/openfunction6/mph/builder/builders/ruby26/BUILD.bazel:7:8: Executing genrule //builders/ruby26:builder.image failed: (Exit 1): bash failed: error executing command /bin/bash -c ... (remaining 1 argument(s) skipped)
2021/08/04 09:09:31 Checking tools
2021/08/04 09:09:31 pack found at /usr/local/bin/pack
2021/08/04 09:09:31 docker found at /usr/bin/docker
2021/08/04 09:09:31 container-structure-test found at /usr/local/bin/container-structure-test
2021/08/04 09:09:31 Checking pack version
2021/08/04 09:09:31 Found pack version 0.20.0+git-66a4f32.build-2668
ERROR: failed to add buildpacks to builder: downloading buildpack: extracting from registry 'bundle.tgz': fetching image: image 'bundle.tgz' does not exist on the daemon: not found
Extracting builder tar:
./
./builder.toml
./label.tgz
./ruby/
./ruby/bundle.tgz
./ruby/functions_framework.tgz
Warning: Command 'pack create-builder' has been deprecated, please use 'pack builder create' instead
Downloading from 'https://github.com/buildpacks/lifecycle/releases/download/v0.11.1/lifecycle-v0.11.1+linux.x86-64.tgz'
5.25 MB/5.25 MB
Target //builders/ruby26:builder.image failed to build
Use --verbose_failures to see the command lines of failed build steps.
```

**A1:** 这是由于 builder.toml 里的路径设置不正确导致，正确路径为

```shell
[[buildpacks]]
  id = "google.ruby.bundle"
  uri = "ruby/bundle.tgz"
```



 **Q2:**  [builder] /usr/local/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require': cannot load such file -- zlib (LoadError)

```shell
openfunction6@cloudshell:~/mph/samples/functions/Knative/hello-world-ruby$ pack build function-ruby1 --builder penghuima/ruby26  --env FUNC_NAME="Hello-World"
v1: Pulling from penghuima/buildpacks-ruby26-run
Digest: sha256:241dbbb3eefe5ef0a4ee5d471933ed331eb6a18d20cf5cdeffb6f9da6a44cc35
Status: Image is up to date for penghuima/buildpacks-ruby26-run:v1
0.11.1: Pulling from buildpacksio/lifecycle
Digest: sha256:2ca9870d9472400792bdc2d43ded8df2642ffcf937e8cd01685836678c28780b
[builder] Running "bundle lock --add-platform x86_64-linux"
[builder] /usr/local/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require': cannot load such file -- zlib (LoadError)
[builder]       from /usr/local/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
[builder]       from /usr/local/lib/ruby/2.6.0/bundler/fetcher.rb:6:in `<top (required)>'
[builder]       from /usr/local/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
[builder]       from /usr/local/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
[builder]       from /usr/local/lib/ruby/2.6.0/bundler/cli/lock.rb:21:in `run'
[builder]       from /usr/local/lib/ruby/2.6.0/bundler/cli.rb:643:in `lock'
[builder]       from /usr/local/lib/ruby/2.6.0/bundler/vendor/thor/lib/thor/command.rb:27:in `run'
[builder]       from /usr/local/lib/ruby/2.6.0/bundler/vendor/thor/lib/thor/invocation.rb:126:in `invoke_command'
[builder]       from /usr/local/lib/ruby/2.6.0/bundler/vendor/thor/lib/thor.rb:387:in `dispatch'
[builder]       from /usr/local/lib/ruby/2.6.0/bundler/cli.rb:27:in `dispatch'
[builder]       from /usr/local/lib/ruby/2.6.0/bundler/vendor/thor/lib/thor/base.rb:466:in `start'
[builder]       from /usr/local/lib/ruby/2.6.0/bundler/cli.rb:18:in `start'
[builder]       from /usr/local/lib/ruby/gems/2.6.0/gems/bundler-1.17.2/exe/bundle:30:in `block in <top (required)>'
[builder]       from /usr/local/lib/ruby/2.6.0/bundler/friendly_errors.rb:124:in `with_friendly_errors'
[builder]       from /usr/local/lib/ruby/gems/2.6.0/gems/bundler-1.17.2/exe/bundle:22:in `<top (required)>'
[builder]       from /usr/local/bin/bundle:23:in `load'
[builder]       from /usr/local/bin/bundle:23:in `<main>'
[builder] Sorry your project couldn't be built.
[builder] Our documentation explains ways to configure Buildpacks to better recognise your project:
[builder]  -> https://github.com/GoogleCloudPlatform/buildpacks/blob/main/README.md
[builder] If you think you've found an issue, please report it:
[builder]  -> https://github.com/GoogleCloudPlatform/buildpacks/issues/new
[builder] --------------------------------------------------------------------------------
[builder] ERROR: failed to build: exit status 1
ERROR: failed to build: executing lifecycle. This may be the result of using an untrusted builder: failed with status code: 51
```

**A2：**这是由于 stack 运行时安装少依赖 zlib 导致的,安装ruby时缺少 openssl 和 zlib

```shell
*** Following extensions are not compiled:
dbm:
        Could not be configured. It will not be installed.
        Check ext/dbm/mkmf.log for more details.
gdbm:
        Could not be configured. It will not be installed.
        Check ext/gdbm/mkmf.log for more details.
openssl:
        Could not be configured. It will not be installed.
        /tmp/ruby/ruby-2.6.7/ext/openssl/extconf.rb:97: OpenSSL library could not be found. You might want to use --with-openssl-dir=<dir>                            option to specify the prefix where OpenSSL is installed.
        Check ext/openssl/mkmf.log for more details.
readline:
        Could not be configured. It will not be installed.
        /tmp/ruby/ruby-2.6.7/ext/readline/extconf.rb:62: Neither readline nor libedit was found
        Check ext/readline/mkmf.log for more details.
zlib:
        Could not be configured. It will not be installed.
        Check ext/zlib/mkmf.log for more details.
```

> 解决方案，[参考 1](https://blog.csdn.net/qq_41690477/article/details/82750530)  [参考2](https://blog.csdn.net/Wangdada111/article/details/73382554?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_title~default-1.control&spm=1001.2101.3001.4242)

```shell
# install ruby runtime
RUN cd /tmp && mkdir ruby && cd ruby && curl  https://cache.ruby-lang.org/pub/ruby/2.6/ruby-2.6.6.tar.gz | tar xz \
    && cd ruby-2.6.6 && ./configure && make && make install  \
    && cd ext/zlib \
    && ruby extconf.rb && sed -i 's?$(top_srcdir)?../..?g' Makefile  && make && make install \
    && cd ../openssl \
    && ruby extconf.rb && sed -i 's?$(top_srcdir)?../..?g' Makefile && make && make install \
    && gem uninstall bundle && gem install bundler -v 2.1.4 \
```



**Q3：**缺少 Gemfile.lock文件

**A3:**  补充文件，注意依赖版本问题



**Q4：**stack 里 build 和 run 镜像 bundler 版本不一致，一个为 2.1.4 ，一个为 1.17.2

**A4：** 卸载默认bundler版本，统一版本为2.1.4



