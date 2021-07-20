---
layout: post
title: "OpenFunction/builder 本地环境复现"
categories: builder
tags: builder
---

* content
{:toc}


OpenFunction 中的 Builders 组件集成了 Tekton 与 Buildpacks ， 用于完成将用户提交的函数代码制作成可以运行在 Kubernetes 中的应用负载镜像这一部分工作。 Tekton 负责编排整个由代码到应用镜像的制作过程，包含：拉取镜像、准备环境变量、使用 Buildpacks 制作镜像、输出镜像到指定仓库等几个主要步骤。 Buildpacks 负责识别代码的语言类型，然后准备好相关 的编译环境，再将代码通过模板渲染为可运行的应用。

<!--more-->

### 复现 nodejs14 环境

```shell
git clone https://github.com/OpenFunction/build
git clone https://github.com/GoogleCloudPlatform/buildpack-samples.git
```

#### 构建stack镜像


```shell
$ bazel run //builders/node14/stack:build
Starting local Bazel server and connecting to it...
INFO: Analyzed target //builders/node14/stack:build (25 packages loaded, 164 targets configured).
INFO: Found 1 target...
Target //builders/node14/stack:build up-to-date:
  bazel-bin/builders/node14/stack/build
INFO: Elapsed time: 22.479s, Critical Path: 0.40s
INFO: 10 processes: 9 internal, 1 linux-sandbox.
INFO: Build completed successfully, 10 total actions
INFO: Build completed successfully, 10 total actions
> Extracting licenses tar
> Building base node14common image
Sending build context to Docker daemon  3.072kB
Step 1/11 : FROM gcr.io/gcp-runtimes/ubuntu_18_0_4
latest: Pulling from gcp-runtimes/ubuntu_18_0_4
2280635623d3: Pull complete
979342e79548: Pull complete
3c2cba919283: Pull complete
Digest: sha256:de4f8dc8c87ee9662a8c0e0009865784e785e793f35341775df8467cccaa5f05
Status: Downloaded newer image for gcr.io/gcp-runtimes/ubuntu_18_0_4:latest
 ---> 5f94f70d5d7e
Step 2/11 : ARG cnb_uid=1000
 ---> Running in 975c90510d49
Removing intermediate container 975c90510d49
 ---> 9b1e234d9e9c
Step 3/11 : ARG cnb_gid=1000
 ---> Running in 22200fcc317a
Removing intermediate container 22200fcc317a
 ---> 2fc4912c489d
Step 4/11 : ARG stack_id="google"
 ---> Running in 498be48e6aa3
Removing intermediate container 498be48e6aa3
 ---> 3fc18eca09a4
Step 5/11 : RUN apt-get update && apt-get install -y --no-install-recommends   libexpat1   libffi6   libmpdec2   libicu60   libc++1-9   tzdata   && apt-get clean && rm -rf /var/lib/apt/lists/*
 ---> Running in efa889f85f49
Get:1 http://security.ubuntu.com/ubuntu bionic-security InRelease [88.7 kB]
Get:2 http://archive.ubuntu.com/ubuntu bionic InRelease [242 kB]
Get:3 http://security.ubuntu.com/ubuntu bionic-security/main amd64 Packages [1784 kB]
Get:4 http://archive.ubuntu.com/ubuntu bionic-updates InRelease [88.7 kB]
Get:5 http://archive.ubuntu.com/ubuntu bionic-backports InRelease [74.6 kB]
Get:6 http://security.ubuntu.com/ubuntu bionic-security/main Translation-en [329 kB]
Get:7 http://archive.ubuntu.com/ubuntu bionic/main amd64 Packages [1019 kB]
Get:8 http://security.ubuntu.com/ubuntu bionic-security/restricted amd64 Packages [365 kB]
Get:9 http://security.ubuntu.com/ubuntu bionic-security/restricted Translation-en [48.9 kB]
Get:10 http://security.ubuntu.com/ubuntu bionic-security/universe amd64 Packages [1130 kB]
Get:11 http://security.ubuntu.com/ubuntu bionic-security/universe Translation-en [256 kB]
Get:12 http://security.ubuntu.com/ubuntu bionic-security/multiverse amd64 Packages [19.2 kB]
Get:13 http://security.ubuntu.com/ubuntu bionic-security/multiverse Translation-en [4412 B]
Get:14 http://archive.ubuntu.com/ubuntu bionic/main Translation-en [516 kB]
Get:15 http://archive.ubuntu.com/ubuntu bionic/restricted amd64 Packages [9184 B]
Get:16 http://archive.ubuntu.com/ubuntu bionic/restricted Translation-en [3584 B]
Get:17 http://archive.ubuntu.com/ubuntu bionic/universe amd64 Packages [8570 kB]
Get:18 http://archive.ubuntu.com/ubuntu bionic/universe Translation-en [4941 kB]
Get:19 http://archive.ubuntu.com/ubuntu bionic/multiverse amd64 Packages [151 kB]
Get:20 http://archive.ubuntu.com/ubuntu bionic/multiverse Translation-en [108 kB]
Get:21 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 Packages [2132 kB]
Get:22 http://archive.ubuntu.com/ubuntu bionic-updates/main Translation-en [422 kB]
Get:23 http://archive.ubuntu.com/ubuntu bionic-updates/restricted amd64 Packages [389 kB]
Get:24 http://archive.ubuntu.com/ubuntu bionic-updates/restricted Translation-en [52.8 kB]
Get:25 http://archive.ubuntu.com/ubuntu bionic-updates/universe amd64 Packages [1739 kB]
Get:26 http://archive.ubuntu.com/ubuntu bionic-updates/universe Translation-en [371 kB]
Get:27 http://archive.ubuntu.com/ubuntu bionic-updates/multiverse amd64 Packages [26.6 kB]
Get:28 http://archive.ubuntu.com/ubuntu bionic-updates/multiverse Translation-en [6792 B]
Get:29 http://archive.ubuntu.com/ubuntu bionic-backports/main amd64 Packages [10.0 kB]
Get:30 http://archive.ubuntu.com/ubuntu bionic-backports/main Translation-en [4764 B]
Get:31 http://archive.ubuntu.com/ubuntu bionic-backports/universe amd64 Packages [10.3 kB]
Get:32 http://archive.ubuntu.com/ubuntu bionic-backports/universe Translation-en [4588 B]
Fetched 24.9 MB in 7s (3394 kB/s)
Reading package lists...
Reading package lists...
Building dependency tree...
Reading state information...
libffi6 is already the newest version (3.2.1-8).
Suggested packages:
  clang
The following NEW packages will be installed:
  libc++1-9 libc++abi1-9 libexpat1 libicu60 libmpdec2 tzdata
0 upgraded, 6 newly installed, 0 to remove and 1 not upgraded.
Need to get 9664 kB of archives.
After this operation, 42.6 MB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libexpat1 amd64 2.2.5-3ubuntu0.2 [80.5 kB]
Get:2 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libicu60 amd64 60.2-3ubuntu3.1 [8054 kB]
Get:3 http://archive.ubuntu.com/ubuntu bionic/main amd64 libmpdec2 amd64 2.4.2-1ubuntu1 [84.1 kB]
Get:4 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 tzdata all 2021a-0ubuntu0.18.04 [190 kB]
Get:5 http://archive.ubuntu.com/ubuntu bionic-updates/universe amd64 libc++abi1-9 amd64 1:9-2~ubuntu18.04.2 [230 kB]
Get:6 http://archive.ubuntu.com/ubuntu bionic-updates/universe amd64 libc++1-9 amd64 1:9-2~ubuntu18.04.2 [1026 kB]
debconf: delaying package configuration, since apt-utils is not installed
Fetched 9664 kB in 3s (3201 kB/s)
Selecting previously unselected package libexpat1:amd64.
(Reading database ... 4536 files and directories currently installed.)
Preparing to unpack .../0-libexpat1_2.2.5-3ubuntu0.2_amd64.deb ...
Unpacking libexpat1:amd64 (2.2.5-3ubuntu0.2) ...
Selecting previously unselected package libicu60:amd64.
Preparing to unpack .../1-libicu60_60.2-3ubuntu3.1_amd64.deb ...
Unpacking libicu60:amd64 (60.2-3ubuntu3.1) ...
Selecting previously unselected package libmpdec2:amd64.
Preparing to unpack .../2-libmpdec2_2.4.2-1ubuntu1_amd64.deb ...
Unpacking libmpdec2:amd64 (2.4.2-1ubuntu1) ...
Selecting previously unselected package tzdata.
Preparing to unpack .../3-tzdata_2021a-0ubuntu0.18.04_all.deb ...
Unpacking tzdata (2021a-0ubuntu0.18.04) ...
Selecting previously unselected package libc++abi1-9:amd64.
Preparing to unpack .../4-libc++abi1-9_1%3a9-2~ubuntu18.04.2_amd64.deb ...
Unpacking libc++abi1-9:amd64 (1:9-2~ubuntu18.04.2) ...
Selecting previously unselected package libc++1-9:amd64.
Preparing to unpack .../5-libc++1-9_1%3a9-2~ubuntu18.04.2_amd64.deb ...
Unpacking libc++1-9:amd64 (1:9-2~ubuntu18.04.2) ...
Setting up libexpat1:amd64 (2.2.5-3ubuntu0.2) ...
Setting up libicu60:amd64 (60.2-3ubuntu3.1) ...
Setting up tzdata (2021a-0ubuntu0.18.04) ...

Current default time zone: 'Etc/UTC'
Local time is now:      Tue Jul 20 07:04:57 UTC 2021.
Universal Time is now:  Tue Jul 20 07:04:57 UTC 2021.
Run 'dpkg-reconfigure tzdata' if you wish to change it.

Setting up libc++abi1-9:amd64 (1:9-2~ubuntu18.04.2) ...
Setting up libmpdec2:amd64 (2.4.2-1ubuntu1) ...
Setting up libc++1-9:amd64 (1:9-2~ubuntu18.04.2) ...
Processing triggers for libc-bin (2.27-3ubuntu1.4) ...
Removing intermediate container efa889f85f49
 ---> a18f467cb4d4
Step 6/11 : LABEL io.buildpacks.stack.id=${stack_id}
 ---> Running in 5f7f604dbbc1
Removing intermediate container 5f7f604dbbc1
 ---> 62a443e1c868
Step 7/11 : RUN groupadd cnb --gid ${cnb_gid} &&   useradd --uid ${cnb_uid} --gid ${cnb_gid} -m -s /bin/bash cnb
 ---> Running in 8fd4a8d1497d
Removing intermediate container 8fd4a8d1497d
 ---> 30c2f6bb61c5
Step 8/11 : RUN curl --fail --show-error --silent --location --retry 3 https://nodejs.org/dist/v14.16.1/node-v14.16.1-linux-x64.tar.gz | tar xz --directory /usr/local --strip-components=1
 ---> Running in 6faa412a04a2
Removing intermediate container 6faa412a04a2
 ---> b3001a215692
Step 9/11 : ENV CNB_USER_ID=${cnb_uid}
 ---> Running in 67f6c766a057
Removing intermediate container 67f6c766a057
 ---> d01983f496e5
Step 10/11 : ENV CNB_GROUP_ID=${cnb_gid}
 ---> Running in 1d882ff49553
Removing intermediate container 1d882ff49553
 ---> e513919b4f41
Step 11/11 : ENV CNB_STACK_ID=${stack_id}
 ---> Running in 3894a5580a59
Removing intermediate container 3894a5580a59
 ---> 8c14d1b76b45
Successfully built 8c14d1b76b45
Successfully tagged node14common:latest
> Building openfunctiondev/buildpacks-node14-run:v1
Sending build context to Docker daemon   2.56kB
Step 1/4 : ARG from_image
Step 2/4 : FROM ${from_image}
 ---> 8c14d1b76b45
Step 3/4 : ENV PORT 8080
 ---> Running in 74b095d76cbf
Removing intermediate container 74b095d76cbf
 ---> 4cd03933c96a
Step 4/4 : USER cnb
 ---> Running in abc0e3c0680e
Removing intermediate container abc0e3c0680e
 ---> c73ea61ef070
Successfully built c73ea61ef070
Successfully tagged openfunctiondev/buildpacks-node14-run:v1
> Building openfunctiondev/buildpacks-node14-build:v1
Sending build context to Docker daemon  12.34kB
Step 1/5 : ARG from_image
Step 2/5 : FROM ${from_image}
 ---> 8c14d1b76b45
Step 3/5 : COPY licenses/ /usr/local/share/licenses/buildpacks/
 ---> ea73a0be43b9
Step 4/5 : RUN apt-get update && apt-get install -y --no-install-recommends   build-essential   git   python3   tar   zip   unzip   xz-utils   g++-8   gcc-8   zlib1g-dev   libstdc++-8-dev   pkg-config   && apt-get clean && rm -rf /var/lib/apt/lists/*
 ---> Running in a7d5d08df844
Get:1 http://security.ubuntu.com/ubuntu bionic-security InRelease [88.7 kB]
Get:2 http://archive.ubuntu.com/ubuntu bionic InRelease [242 kB]
Get:3 http://security.ubuntu.com/ubuntu bionic-security/main amd64 Packages [1784 kB]
Get:4 http://archive.ubuntu.com/ubuntu bionic-updates InRelease [88.7 kB]
Get:5 http://security.ubuntu.com/ubuntu bionic-security/main Translation-en [329 kB]
Get:6 http://security.ubuntu.com/ubuntu bionic-security/restricted amd64 Packages [365 kB]
Get:7 http://security.ubuntu.com/ubuntu bionic-security/restricted Translation-en [48.9 kB]
Get:8 http://security.ubuntu.com/ubuntu bionic-security/universe amd64 Packages [1130 kB]
Get:9 http://archive.ubuntu.com/ubuntu bionic-backports InRelease [74.6 kB]
Get:10 http://security.ubuntu.com/ubuntu bionic-security/universe Translation-en [256 kB]
Get:11 http://security.ubuntu.com/ubuntu bionic-security/multiverse amd64 Packages [19.2 kB]
Get:12 http://security.ubuntu.com/ubuntu bionic-security/multiverse Translation-en [4412 B]
Get:13 http://archive.ubuntu.com/ubuntu bionic/main amd64 Packages [1019 kB]
Get:14 http://archive.ubuntu.com/ubuntu bionic/main Translation-en [516 kB]
Get:15 http://archive.ubuntu.com/ubuntu bionic/restricted amd64 Packages [9184 B]
Get:16 http://archive.ubuntu.com/ubuntu bionic/restricted Translation-en [3584 B]
Get:17 http://archive.ubuntu.com/ubuntu bionic/universe amd64 Packages [8570 kB]
Get:18 http://archive.ubuntu.com/ubuntu bionic/universe Translation-en [4941 kB]
Get:19 http://archive.ubuntu.com/ubuntu bionic/multiverse amd64 Packages [151 kB]
Get:20 http://archive.ubuntu.com/ubuntu bionic/multiverse Translation-en [108 kB]
Get:21 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 Packages [2132 kB]
Get:22 http://archive.ubuntu.com/ubuntu bionic-updates/main Translation-en [422 kB]
Get:23 http://archive.ubuntu.com/ubuntu bionic-updates/restricted amd64 Packages [389 kB]
Get:24 http://archive.ubuntu.com/ubuntu bionic-updates/restricted Translation-en [52.8 kB]
Get:25 http://archive.ubuntu.com/ubuntu bionic-updates/universe amd64 Packages [1739 kB]
Get:26 http://archive.ubuntu.com/ubuntu bionic-updates/universe Translation-en [371 kB]
Get:27 http://archive.ubuntu.com/ubuntu bionic-updates/multiverse amd64 Packages [26.6 kB]
Get:28 http://archive.ubuntu.com/ubuntu bionic-updates/multiverse Translation-en [6792 B]
Get:29 http://archive.ubuntu.com/ubuntu bionic-backports/main amd64 Packages [10.0 kB]
Get:30 http://archive.ubuntu.com/ubuntu bionic-backports/main Translation-en [4764 B]
Get:31 http://archive.ubuntu.com/ubuntu bionic-backports/universe amd64 Packages [10.3 kB]
Get:32 http://archive.ubuntu.com/ubuntu bionic-backports/universe Translation-en [4588 B]
Fetched 24.9 MB in 7s (3333 kB/s)
Reading package lists...
Reading package lists...
Building dependency tree...
Reading state information...
tar is already the newest version (1.29b-2ubuntu0.2).
The following additional packages will be installed:
  binutils binutils-common binutils-x86-64-linux-gnu cpp cpp-7 cpp-8 dpkg-dev
  g++ g++-7 gcc gcc-7 gcc-7-base git-man libasan4 libasan5 libatomic1
  libbinutils libc-dev-bin libc6-dev libcc1-0 libcilkrts5 libcurl3-gnutls
  libdpkg-perl liberror-perl libgcc-7-dev libgcc-8-dev libgdbm-compat4
  libgdbm5 libglib2.0-0 libgomp1 libisl19 libitm1 liblsan0 libmpc3 libmpfr6
  libmpx2 libperl5.26 libpython3-stdlib libpython3.6-minimal
  libpython3.6-stdlib libquadmath0 libreadline7 libstdc++-7-dev libtsan0
  libubsan0 libubsan1 linux-libc-dev make mime-support patch perl
  perl-modules-5.26 python3-minimal python3.6 python3.6-minimal
  readline-common
Suggested packages:
  binutils-doc cpp-doc gcc-7-locales gcc-8-locales debian-keyring g++-multilib
  g++-7-multilib gcc-7-doc libstdc++6-7-dbg g++-8-multilib gcc-8-doc
  libstdc++6-8-dbg gcc-multilib manpages-dev autoconf automake libtool flex
  bison gdb gcc-doc gcc-7-multilib libgcc1-dbg libgomp1-dbg libitm1-dbg
  libatomic1-dbg libasan4-dbg liblsan0-dbg libtsan0-dbg libubsan0-dbg
  libcilkrts5-dbg libmpx2-dbg libquadmath0-dbg gcc-8-multilib libasan5-dbg
  libubsan1-dbg gettext-base git-daemon-run | git-daemon-sysvinit git-doc
  git-el git-email git-gui gitk gitweb git-cvs git-mediawiki git-svn glibc-doc
  gnupg | gnupg2 bzr gdbm-l10n libstdc++-7-doc libstdc++-8-doc make-doc ed
  diffutils-doc perl-doc libterm-readline-gnu-perl
  | libterm-readline-perl-perl python3-doc python3-tk python3-venv
  python3.6-venv python3.6-doc binfmt-support readline-doc
Recommended packages:
  fakeroot gnupg | gnupg2 libalgorithm-merge-perl less ssh-client manpages
  manpages-dev libfile-fcntllock-perl liblocale-gettext-perl libglib2.0-data
  shared-mime-info xdg-user-dirs file
The following NEW packages will be installed:
  binutils binutils-common binutils-x86-64-linux-gnu build-essential cpp cpp-7
  cpp-8 dpkg-dev g++ g++-7 g++-8 gcc gcc-7 gcc-7-base gcc-8 git git-man
  libasan4 libasan5 libatomic1 libbinutils libc-dev-bin libc6-dev libcc1-0
  libcilkrts5 libcurl3-gnutls libdpkg-perl liberror-perl libgcc-7-dev
  libgcc-8-dev libgdbm-compat4 libgdbm5 libglib2.0-0 libgomp1 libisl19 libitm1
  liblsan0 libmpc3 libmpfr6 libmpx2 libperl5.26 libpython3-stdlib
  libpython3.6-minimal libpython3.6-stdlib libquadmath0 libreadline7
  libstdc++-7-dev libstdc++-8-dev libtsan0 libubsan0 libubsan1 linux-libc-dev
  make mime-support patch perl perl-modules-5.26 pkg-config python3
  python3-minimal python3.6 python3.6-minimal readline-common unzip xz-utils
  zip zlib1g-dev
0 upgraded, 67 newly installed, 0 to remove and 1 not upgraded.
Need to get 86.3 MB of archives.
After this operation, 379 MB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libpython3.6-minimal amd64 3.6.9-1~18.04ubuntu1.4 [534 kB]
Get:2 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 python3.6-minimal amd64 3.6.9-1~18.04ubuntu1.4 [1610 kB]
Get:3 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 python3-minimal amd64 3.6.7-1~18.04 [23.7 kB]
Get:4 http://archive.ubuntu.com/ubuntu bionic/main amd64 mime-support all 3.60ubuntu1 [30.1 kB]
Get:5 http://archive.ubuntu.com/ubuntu bionic/main amd64 readline-common all 7.0-3 [52.9 kB]
Get:6 http://archive.ubuntu.com/ubuntu bionic/main amd64 libreadline7 amd64 7.0-3 [124 kB]
Get:7 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libpython3.6-stdlib amd64 3.6.9-1~18.04ubuntu1.4 [1712 kB]
Get:8 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 python3.6 amd64 3.6.9-1~18.04ubuntu1.4 [203 kB]
Get:9 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libpython3-stdlib amd64 3.6.7-1~18.04 [7240 B]
Get:10 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 python3 amd64 3.6.7-1~18.04 [47.2 kB]
Get:11 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 perl-modules-5.26 all 5.26.1-6ubuntu0.5 [2762 kB]
Get:12 http://archive.ubuntu.com/ubuntu bionic/main amd64 libgdbm5 amd64 1.14.1-6 [26.0 kB]
Get:13 http://archive.ubuntu.com/ubuntu bionic/main amd64 libgdbm-compat4 amd64 1.14.1-6 [6084 B]
Get:14 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libperl5.26 amd64 5.26.1-6ubuntu0.5 [3534 kB]
Get:15 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 perl amd64 5.26.1-6ubuntu0.5 [201 kB]
Get:16 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libglib2.0-0 amd64 2.56.4-0ubuntu0.18.04.8 [1171 kB]
Get:17 http://archive.ubuntu.com/ubuntu bionic/main amd64 xz-utils amd64 5.2.2-1.3 [83.8 kB]
Get:18 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 binutils-common amd64 2.30-21ubuntu1~18.04.5 [197 kB]
Get:19 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libbinutils amd64 2.30-21ubuntu1~18.04.5 [489 kB]
Get:20 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 binutils-x86-64-linux-gnu amd64 2.30-21ubuntu1~18.04.5 [1839 kB]
Get:21 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 binutils amd64 2.30-21ubuntu1~18.04.5 [3388 B]
Get:22 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libc-dev-bin amd64 2.27-3ubuntu1.4 [71.8 kB]
Get:23 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 linux-libc-dev amd64 4.15.0-147.151 [991 kB]
Get:24 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libc6-dev amd64 2.27-3ubuntu1.4 [2585 kB]
Get:25 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 gcc-7-base amd64 7.5.0-3ubuntu1~18.04 [18.3 kB]
Get:26 http://archive.ubuntu.com/ubuntu bionic/main amd64 libisl19 amd64 0.19-1 [551 kB]
Get:27 http://archive.ubuntu.com/ubuntu bionic/main amd64 libmpfr6 amd64 4.0.1-1 [243 kB]
Get:28 http://archive.ubuntu.com/ubuntu bionic/main amd64 libmpc3 amd64 1.1.0-1 [40.8 kB]
Get:29 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 cpp-7 amd64 7.5.0-3ubuntu1~18.04 [8591 kB]
Get:30 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 cpp amd64 4:7.4.0-1ubuntu2.3 [27.7 kB]
Get:31 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libcc1-0 amd64 8.4.0-1ubuntu1~18.04 [39.4 kB]
Get:32 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libgomp1 amd64 8.4.0-1ubuntu1~18.04 [76.5 kB]
Get:33 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libitm1 amd64 8.4.0-1ubuntu1~18.04 [27.9 kB]
Get:34 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libatomic1 amd64 8.4.0-1ubuntu1~18.04 [9192 B]
Get:35 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libasan4 amd64 7.5.0-3ubuntu1~18.04 [358 kB]
Get:36 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 liblsan0 amd64 8.4.0-1ubuntu1~18.04 [133 kB]
Get:37 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libtsan0 amd64 8.4.0-1ubuntu1~18.04 [288 kB]
Get:38 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libubsan0 amd64 7.5.0-3ubuntu1~18.04 [126 kB]
Get:39 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libcilkrts5 amd64 7.5.0-3ubuntu1~18.04 [42.5 kB]
Get:40 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libmpx2 amd64 8.4.0-1ubuntu1~18.04 [11.6 kB]
Get:41 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libquadmath0 amd64 8.4.0-1ubuntu1~18.04 [134 kB]
Get:42 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libgcc-7-dev amd64 7.5.0-3ubuntu1~18.04 [2378 kB]
Get:43 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 gcc-7 amd64 7.5.0-3ubuntu1~18.04 [9381 kB]
Get:44 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 gcc amd64 4:7.4.0-1ubuntu2.3 [5184 B]
Get:45 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libstdc++-7-dev amd64 7.5.0-3ubuntu1~18.04 [1471 kB]
Get:46 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 g++-7 amd64 7.5.0-3ubuntu1~18.04 [9697 kB]
Get:47 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 g++ amd64 4:7.4.0-1ubuntu2.3 [1568 B]
Get:48 http://archive.ubuntu.com/ubuntu bionic/main amd64 make amd64 4.1-9.1ubuntu1 [154 kB]
Get:49 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libdpkg-perl all 1.19.0.5ubuntu2.3 [211 kB]
Get:50 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 patch amd64 2.7.6-2ubuntu1.1 [102 kB]
Get:51 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 dpkg-dev all 1.19.0.5ubuntu2.3 [607 kB]
Get:52 http://archive.ubuntu.com/ubuntu bionic/main amd64 build-essential amd64 12.4ubuntu1 [4758 B]
Get:53 http://archive.ubuntu.com/ubuntu bionic-updates/universe amd64 cpp-8 amd64 8.4.0-1ubuntu1~18.04 [7225 kB]
Get:54 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libasan5 amd64 8.4.0-1ubuntu1~18.04 [366 kB]
Get:55 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libubsan1 amd64 8.4.0-1ubuntu1~18.04 [122 kB]
Get:56 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libgcc-8-dev amd64 8.4.0-1ubuntu1~18.04 [2305 kB]
Get:57 http://archive.ubuntu.com/ubuntu bionic-updates/universe amd64 gcc-8 amd64 8.4.0-1ubuntu1~18.04 [8044 kB]
Get:58 http://archive.ubuntu.com/ubuntu bionic-updates/universe amd64 libstdc++-8-dev amd64 8.4.0-1ubuntu1~18.04 [1534 kB]
Get:59 http://archive.ubuntu.com/ubuntu bionic-updates/universe amd64 g++-8 amd64 8.4.0-1ubuntu1~18.04 [8122 kB]
Get:60 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libcurl3-gnutls amd64 7.58.0-2ubuntu3.13 [218 kB]
Get:61 http://archive.ubuntu.com/ubuntu bionic/main amd64 liberror-perl all 0.17025-1 [22.8 kB]
Get:62 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 git-man all 1:2.17.1-1ubuntu0.8 [804 kB]
Get:63 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 git amd64 1:2.17.1-1ubuntu0.8 [3916 kB]
Get:64 http://archive.ubuntu.com/ubuntu bionic/main amd64 pkg-config amd64 0.29.1-0ubuntu2 [45.0 kB]
Get:65 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 unzip amd64 6.0-21ubuntu1.1 [168 kB]
Get:66 http://archive.ubuntu.com/ubuntu bionic/main amd64 zip amd64 3.0-11build1 [167 kB]
Get:67 http://archive.ubuntu.com/ubuntu bionic/main amd64 zlib1g-dev amd64 1:1.2.11.dfsg-0ubuntu2 [176 kB]
debconf: delaying package configuration, since apt-utils is not installed
Fetched 86.3 MB in 10s (9023 kB/s)
Selecting previously unselected package libpython3.6-minimal:amd64.
(Reading database ... 6483 files and directories currently installed.)
Preparing to unpack .../libpython3.6-minimal_3.6.9-1~18.04ubuntu1.4_amd64.deb ...
Unpacking libpython3.6-minimal:amd64 (3.6.9-1~18.04ubuntu1.4) ...
Selecting previously unselected package python3.6-minimal.
Preparing to unpack .../python3.6-minimal_3.6.9-1~18.04ubuntu1.4_amd64.deb ...
Unpacking python3.6-minimal (3.6.9-1~18.04ubuntu1.4) ...
Setting up libpython3.6-minimal:amd64 (3.6.9-1~18.04ubuntu1.4) ...
Setting up python3.6-minimal (3.6.9-1~18.04ubuntu1.4) ...
Selecting previously unselected package python3-minimal.
(Reading database ... 6722 files and directories currently installed.)
Preparing to unpack .../0-python3-minimal_3.6.7-1~18.04_amd64.deb ...
Unpacking python3-minimal (3.6.7-1~18.04) ...
Selecting previously unselected package mime-support.
Preparing to unpack .../1-mime-support_3.60ubuntu1_all.deb ...
Unpacking mime-support (3.60ubuntu1) ...
Selecting previously unselected package readline-common.
Preparing to unpack .../2-readline-common_7.0-3_all.deb ...
Unpacking readline-common (7.0-3) ...
Selecting previously unselected package libreadline7:amd64.
Preparing to unpack .../3-libreadline7_7.0-3_amd64.deb ...
Unpacking libreadline7:amd64 (7.0-3) ...
Selecting previously unselected package libpython3.6-stdlib:amd64.
Preparing to unpack .../4-libpython3.6-stdlib_3.6.9-1~18.04ubuntu1.4_amd64.deb ...
Unpacking libpython3.6-stdlib:amd64 (3.6.9-1~18.04ubuntu1.4) ...
Selecting previously unselected package python3.6.
Preparing to unpack .../5-python3.6_3.6.9-1~18.04ubuntu1.4_amd64.deb ...
Unpacking python3.6 (3.6.9-1~18.04ubuntu1.4) ...
Selecting previously unselected package libpython3-stdlib:amd64.
Preparing to unpack .../6-libpython3-stdlib_3.6.7-1~18.04_amd64.deb ...
Unpacking libpython3-stdlib:amd64 (3.6.7-1~18.04) ...
Setting up python3-minimal (3.6.7-1~18.04) ...
Selecting previously unselected package python3.
(Reading database ... 7169 files and directories currently installed.)
Preparing to unpack .../00-python3_3.6.7-1~18.04_amd64.deb ...
Unpacking python3 (3.6.7-1~18.04) ...
Selecting previously unselected package perl-modules-5.26.
Preparing to unpack .../01-perl-modules-5.26_5.26.1-6ubuntu0.5_all.deb ...
Unpacking perl-modules-5.26 (5.26.1-6ubuntu0.5) ...
Selecting previously unselected package libgdbm5:amd64.
Preparing to unpack .../02-libgdbm5_1.14.1-6_amd64.deb ...
Unpacking libgdbm5:amd64 (1.14.1-6) ...
Selecting previously unselected package libgdbm-compat4:amd64.
Preparing to unpack .../03-libgdbm-compat4_1.14.1-6_amd64.deb ...
Unpacking libgdbm-compat4:amd64 (1.14.1-6) ...
Selecting previously unselected package libperl5.26:amd64.
Preparing to unpack .../04-libperl5.26_5.26.1-6ubuntu0.5_amd64.deb ...
Unpacking libperl5.26:amd64 (5.26.1-6ubuntu0.5) ...
Selecting previously unselected package perl.
Preparing to unpack .../05-perl_5.26.1-6ubuntu0.5_amd64.deb ...
Unpacking perl (5.26.1-6ubuntu0.5) ...
Selecting previously unselected package libglib2.0-0:amd64.
Preparing to unpack .../06-libglib2.0-0_2.56.4-0ubuntu0.18.04.8_amd64.deb ...
Unpacking libglib2.0-0:amd64 (2.56.4-0ubuntu0.18.04.8) ...
Selecting previously unselected package xz-utils.
Preparing to unpack .../07-xz-utils_5.2.2-1.3_amd64.deb ...
Unpacking xz-utils (5.2.2-1.3) ...
Selecting previously unselected package binutils-common:amd64.
Preparing to unpack .../08-binutils-common_2.30-21ubuntu1~18.04.5_amd64.deb ...
Unpacking binutils-common:amd64 (2.30-21ubuntu1~18.04.5) ...
Selecting previously unselected package libbinutils:amd64.
Preparing to unpack .../09-libbinutils_2.30-21ubuntu1~18.04.5_amd64.deb ...
Unpacking libbinutils:amd64 (2.30-21ubuntu1~18.04.5) ...
Selecting previously unselected package binutils-x86-64-linux-gnu.
Preparing to unpack .../10-binutils-x86-64-linux-gnu_2.30-21ubuntu1~18.04.5_amd64.deb ...
Unpacking binutils-x86-64-linux-gnu (2.30-21ubuntu1~18.04.5) ...
Selecting previously unselected package binutils.
Preparing to unpack .../11-binutils_2.30-21ubuntu1~18.04.5_amd64.deb ...
Unpacking binutils (2.30-21ubuntu1~18.04.5) ...
Selecting previously unselected package libc-dev-bin.
Preparing to unpack .../12-libc-dev-bin_2.27-3ubuntu1.4_amd64.deb ...
Unpacking libc-dev-bin (2.27-3ubuntu1.4) ...
Selecting previously unselected package linux-libc-dev:amd64.
Preparing to unpack .../13-linux-libc-dev_4.15.0-147.151_amd64.deb ...
Unpacking linux-libc-dev:amd64 (4.15.0-147.151) ...
Selecting previously unselected package libc6-dev:amd64.
Preparing to unpack .../14-libc6-dev_2.27-3ubuntu1.4_amd64.deb ...
Unpacking libc6-dev:amd64 (2.27-3ubuntu1.4) ...
Selecting previously unselected package gcc-7-base:amd64.
Preparing to unpack .../15-gcc-7-base_7.5.0-3ubuntu1~18.04_amd64.deb ...
Unpacking gcc-7-base:amd64 (7.5.0-3ubuntu1~18.04) ...
Selecting previously unselected package libisl19:amd64.
Preparing to unpack .../16-libisl19_0.19-1_amd64.deb ...
Unpacking libisl19:amd64 (0.19-1) ...
Selecting previously unselected package libmpfr6:amd64.
Preparing to unpack .../17-libmpfr6_4.0.1-1_amd64.deb ...
Unpacking libmpfr6:amd64 (4.0.1-1) ...
Selecting previously unselected package libmpc3:amd64.
Preparing to unpack .../18-libmpc3_1.1.0-1_amd64.deb ...
Unpacking libmpc3:amd64 (1.1.0-1) ...
Selecting previously unselected package cpp-7.
Preparing to unpack .../19-cpp-7_7.5.0-3ubuntu1~18.04_amd64.deb ...
Unpacking cpp-7 (7.5.0-3ubuntu1~18.04) ...
Selecting previously unselected package cpp.
Preparing to unpack .../20-cpp_4%3a7.4.0-1ubuntu2.3_amd64.deb ...
Unpacking cpp (4:7.4.0-1ubuntu2.3) ...
Selecting previously unselected package libcc1-0:amd64.
Preparing to unpack .../21-libcc1-0_8.4.0-1ubuntu1~18.04_amd64.deb ...
Unpacking libcc1-0:amd64 (8.4.0-1ubuntu1~18.04) ...
Selecting previously unselected package libgomp1:amd64.
Preparing to unpack .../22-libgomp1_8.4.0-1ubuntu1~18.04_amd64.deb ...
Unpacking libgomp1:amd64 (8.4.0-1ubuntu1~18.04) ...
Selecting previously unselected package libitm1:amd64.
Preparing to unpack .../23-libitm1_8.4.0-1ubuntu1~18.04_amd64.deb ...
Unpacking libitm1:amd64 (8.4.0-1ubuntu1~18.04) ...
Selecting previously unselected package libatomic1:amd64.
Preparing to unpack .../24-libatomic1_8.4.0-1ubuntu1~18.04_amd64.deb ...
Unpacking libatomic1:amd64 (8.4.0-1ubuntu1~18.04) ...
Selecting previously unselected package libasan4:amd64.
Preparing to unpack .../25-libasan4_7.5.0-3ubuntu1~18.04_amd64.deb ...
Unpacking libasan4:amd64 (7.5.0-3ubuntu1~18.04) ...
Selecting previously unselected package liblsan0:amd64.
Preparing to unpack .../26-liblsan0_8.4.0-1ubuntu1~18.04_amd64.deb ...
Unpacking liblsan0:amd64 (8.4.0-1ubuntu1~18.04) ...
Selecting previously unselected package libtsan0:amd64.
Preparing to unpack .../27-libtsan0_8.4.0-1ubuntu1~18.04_amd64.deb ...
Unpacking libtsan0:amd64 (8.4.0-1ubuntu1~18.04) ...
Selecting previously unselected package libubsan0:amd64.
Preparing to unpack .../28-libubsan0_7.5.0-3ubuntu1~18.04_amd64.deb ...
Unpacking libubsan0:amd64 (7.5.0-3ubuntu1~18.04) ...
Selecting previously unselected package libcilkrts5:amd64.
Preparing to unpack .../29-libcilkrts5_7.5.0-3ubuntu1~18.04_amd64.deb ...
Unpacking libcilkrts5:amd64 (7.5.0-3ubuntu1~18.04) ...
Selecting previously unselected package libmpx2:amd64.
Preparing to unpack .../30-libmpx2_8.4.0-1ubuntu1~18.04_amd64.deb ...
Unpacking libmpx2:amd64 (8.4.0-1ubuntu1~18.04) ...
Selecting previously unselected package libquadmath0:amd64.
Preparing to unpack .../31-libquadmath0_8.4.0-1ubuntu1~18.04_amd64.deb ...
Unpacking libquadmath0:amd64 (8.4.0-1ubuntu1~18.04) ...
Selecting previously unselected package libgcc-7-dev:amd64.
Preparing to unpack .../32-libgcc-7-dev_7.5.0-3ubuntu1~18.04_amd64.deb ...
Unpacking libgcc-7-dev:amd64 (7.5.0-3ubuntu1~18.04) ...
Selecting previously unselected package gcc-7.
Preparing to unpack .../33-gcc-7_7.5.0-3ubuntu1~18.04_amd64.deb ...
Unpacking gcc-7 (7.5.0-3ubuntu1~18.04) ...
Selecting previously unselected package gcc.
Preparing to unpack .../34-gcc_4%3a7.4.0-1ubuntu2.3_amd64.deb ...
Unpacking gcc (4:7.4.0-1ubuntu2.3) ...
Selecting previously unselected package libstdc++-7-dev:amd64.
Preparing to unpack .../35-libstdc++-7-dev_7.5.0-3ubuntu1~18.04_amd64.deb ...
Unpacking libstdc++-7-dev:amd64 (7.5.0-3ubuntu1~18.04) ...
Selecting previously unselected package g++-7.
Preparing to unpack .../36-g++-7_7.5.0-3ubuntu1~18.04_amd64.deb ...
Unpacking g++-7 (7.5.0-3ubuntu1~18.04) ...
Selecting previously unselected package g++.
Preparing to unpack .../37-g++_4%3a7.4.0-1ubuntu2.3_amd64.deb ...
Unpacking g++ (4:7.4.0-1ubuntu2.3) ...
Selecting previously unselected package make.
Preparing to unpack .../38-make_4.1-9.1ubuntu1_amd64.deb ...
Unpacking make (4.1-9.1ubuntu1) ...
Selecting previously unselected package libdpkg-perl.
Preparing to unpack .../39-libdpkg-perl_1.19.0.5ubuntu2.3_all.deb ...
Unpacking libdpkg-perl (1.19.0.5ubuntu2.3) ...
Selecting previously unselected package patch.
Preparing to unpack .../40-patch_2.7.6-2ubuntu1.1_amd64.deb ...
Unpacking patch (2.7.6-2ubuntu1.1) ...
Selecting previously unselected package dpkg-dev.
Preparing to unpack .../41-dpkg-dev_1.19.0.5ubuntu2.3_all.deb ...
Unpacking dpkg-dev (1.19.0.5ubuntu2.3) ...
Selecting previously unselected package build-essential.
Preparing to unpack .../42-build-essential_12.4ubuntu1_amd64.deb ...
Unpacking build-essential (12.4ubuntu1) ...
Selecting previously unselected package cpp-8.
Preparing to unpack .../43-cpp-8_8.4.0-1ubuntu1~18.04_amd64.deb ...
Unpacking cpp-8 (8.4.0-1ubuntu1~18.04) ...
Selecting previously unselected package libasan5:amd64.
Preparing to unpack .../44-libasan5_8.4.0-1ubuntu1~18.04_amd64.deb ...
Unpacking libasan5:amd64 (8.4.0-1ubuntu1~18.04) ...
Selecting previously unselected package libubsan1:amd64.
Preparing to unpack .../45-libubsan1_8.4.0-1ubuntu1~18.04_amd64.deb ...
Unpacking libubsan1:amd64 (8.4.0-1ubuntu1~18.04) ...
Selecting previously unselected package libgcc-8-dev:amd64.
Preparing to unpack .../46-libgcc-8-dev_8.4.0-1ubuntu1~18.04_amd64.deb ...
Unpacking libgcc-8-dev:amd64 (8.4.0-1ubuntu1~18.04) ...
Selecting previously unselected package gcc-8.
Preparing to unpack .../47-gcc-8_8.4.0-1ubuntu1~18.04_amd64.deb ...
Unpacking gcc-8 (8.4.0-1ubuntu1~18.04) ...
Selecting previously unselected package libstdc++-8-dev:amd64.
Preparing to unpack .../48-libstdc++-8-dev_8.4.0-1ubuntu1~18.04_amd64.deb ...
Unpacking libstdc++-8-dev:amd64 (8.4.0-1ubuntu1~18.04) ...
Selecting previously unselected package g++-8.
Preparing to unpack .../49-g++-8_8.4.0-1ubuntu1~18.04_amd64.deb ...
Unpacking g++-8 (8.4.0-1ubuntu1~18.04) ...
Selecting previously unselected package libcurl3-gnutls:amd64.
Preparing to unpack .../50-libcurl3-gnutls_7.58.0-2ubuntu3.13_amd64.deb ...
Unpacking libcurl3-gnutls:amd64 (7.58.0-2ubuntu3.13) ...
Selecting previously unselected package liberror-perl.
Preparing to unpack .../51-liberror-perl_0.17025-1_all.deb ...
Unpacking liberror-perl (0.17025-1) ...
Selecting previously unselected package git-man.
Preparing to unpack .../52-git-man_1%3a2.17.1-1ubuntu0.8_all.deb ...
Unpacking git-man (1:2.17.1-1ubuntu0.8) ...
Selecting previously unselected package git.
Preparing to unpack .../53-git_1%3a2.17.1-1ubuntu0.8_amd64.deb ...
Unpacking git (1:2.17.1-1ubuntu0.8) ...
Selecting previously unselected package pkg-config.
Preparing to unpack .../54-pkg-config_0.29.1-0ubuntu2_amd64.deb ...
Unpacking pkg-config (0.29.1-0ubuntu2) ...
Selecting previously unselected package unzip.
Preparing to unpack .../55-unzip_6.0-21ubuntu1.1_amd64.deb ...
Unpacking unzip (6.0-21ubuntu1.1) ...
Selecting previously unselected package zip.
Preparing to unpack .../56-zip_3.0-11build1_amd64.deb ...
Unpacking zip (3.0-11build1) ...
Selecting previously unselected package zlib1g-dev:amd64.
Preparing to unpack .../57-zlib1g-dev_1%3a1.2.11.dfsg-0ubuntu2_amd64.deb ...
Unpacking zlib1g-dev:amd64 (1:1.2.11.dfsg-0ubuntu2) ...
Setting up libquadmath0:amd64 (8.4.0-1ubuntu1~18.04) ...
Setting up libgomp1:amd64 (8.4.0-1ubuntu1~18.04) ...
Setting up libatomic1:amd64 (8.4.0-1ubuntu1~18.04) ...
Setting up readline-common (7.0-3) ...
Setting up git-man (1:2.17.1-1ubuntu0.8) ...
Setting up libcc1-0:amd64 (8.4.0-1ubuntu1~18.04) ...
Setting up make (4.1-9.1ubuntu1) ...
Setting up mime-support (3.60ubuntu1) ...
Setting up libreadline7:amd64 (7.0-3) ...
Setting up libcurl3-gnutls:amd64 (7.58.0-2ubuntu3.13) ...
Setting up libtsan0:amd64 (8.4.0-1ubuntu1~18.04) ...
Setting up libglib2.0-0:amd64 (2.56.4-0ubuntu0.18.04.8) ...
No schema files found: doing nothing.
Setting up unzip (6.0-21ubuntu1.1) ...
Setting up linux-libc-dev:amd64 (4.15.0-147.151) ...
Setting up libmpfr6:amd64 (4.0.1-1) ...
Setting up perl-modules-5.26 (5.26.1-6ubuntu0.5) ...
Setting up libgdbm5:amd64 (1.14.1-6) ...
Setting up zip (3.0-11build1) ...
Setting up liblsan0:amd64 (8.4.0-1ubuntu1~18.04) ...
Setting up gcc-7-base:amd64 (7.5.0-3ubuntu1~18.04) ...
Setting up binutils-common:amd64 (2.30-21ubuntu1~18.04.5) ...
Setting up libmpx2:amd64 (8.4.0-1ubuntu1~18.04) ...
Setting up patch (2.7.6-2ubuntu1.1) ...
Setting up xz-utils (5.2.2-1.3) ...
update-alternatives: using /usr/bin/xz to provide /usr/bin/lzma (lzma) in auto mode
update-alternatives: warning: skip creation of /usr/share/man/man1/lzma.1.gz because associated file /usr/share/man/man1/xz.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/unlzma.1.gz because associated file /usr/share/man/man1/unxz.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzcat.1.gz because associated file /usr/share/man/man1/xzcat.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzmore.1.gz because associated file /usr/share/man/man1/xzmore.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzless.1.gz because associated file /usr/share/man/man1/xzless.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzdiff.1.gz because associated file /usr/share/man/man1/xzdiff.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzcmp.1.gz because associated file /usr/share/man/man1/xzcmp.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzgrep.1.gz because associated file /usr/share/man/man1/xzgrep.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzegrep.1.gz because associated file /usr/share/man/man1/xzegrep.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzfgrep.1.gz because associated file /usr/share/man/man1/xzfgrep.1.gz (of link group lzma) doesn't exist
Setting up libmpc3:amd64 (1.1.0-1) ...
Setting up libc-dev-bin (2.27-3ubuntu1.4) ...
Setting up libgdbm-compat4:amd64 (1.14.1-6) ...
Setting up libc6-dev:amd64 (2.27-3ubuntu1.4) ...
Setting up libasan5:amd64 (8.4.0-1ubuntu1~18.04) ...
Setting up libitm1:amd64 (8.4.0-1ubuntu1~18.04) ...
Setting up zlib1g-dev:amd64 (1:1.2.11.dfsg-0ubuntu2) ...
Setting up libubsan1:amd64 (8.4.0-1ubuntu1~18.04) ...
Setting up libisl19:amd64 (0.19-1) ...
Setting up cpp-8 (8.4.0-1ubuntu1~18.04) ...
Setting up libpython3.6-stdlib:amd64 (3.6.9-1~18.04ubuntu1.4) ...
Setting up python3.6 (3.6.9-1~18.04ubuntu1.4) ...
Setting up libasan4:amd64 (7.5.0-3ubuntu1~18.04) ...
Setting up libbinutils:amd64 (2.30-21ubuntu1~18.04.5) ...
Setting up libcilkrts5:amd64 (7.5.0-3ubuntu1~18.04) ...
Setting up libubsan0:amd64 (7.5.0-3ubuntu1~18.04) ...
Setting up libgcc-7-dev:amd64 (7.5.0-3ubuntu1~18.04) ...
Setting up cpp-7 (7.5.0-3ubuntu1~18.04) ...
Setting up libstdc++-7-dev:amd64 (7.5.0-3ubuntu1~18.04) ...
Setting up libperl5.26:amd64 (5.26.1-6ubuntu0.5) ...
Setting up libgcc-8-dev:amd64 (8.4.0-1ubuntu1~18.04) ...
Setting up binutils-x86-64-linux-gnu (2.30-21ubuntu1~18.04.5) ...
Setting up libpython3-stdlib:amd64 (3.6.7-1~18.04) ...
Setting up cpp (4:7.4.0-1ubuntu2.3) ...
Setting up libstdc++-8-dev:amd64 (8.4.0-1ubuntu1~18.04) ...
Setting up python3 (3.6.7-1~18.04) ...
Setting up perl (5.26.1-6ubuntu0.5) ...
Setting up binutils (2.30-21ubuntu1~18.04.5) ...
Setting up gcc-7 (7.5.0-3ubuntu1~18.04) ...
Setting up liberror-perl (0.17025-1) ...
Setting up g++-7 (7.5.0-3ubuntu1~18.04) ...
Setting up libdpkg-perl (1.19.0.5ubuntu2.3) ...
Setting up gcc (4:7.4.0-1ubuntu2.3) ...
Setting up gcc-8 (8.4.0-1ubuntu1~18.04) ...
Setting up g++-8 (8.4.0-1ubuntu1~18.04) ...
Setting up dpkg-dev (1.19.0.5ubuntu2.3) ...
Setting up g++ (4:7.4.0-1ubuntu2.3) ...
update-alternatives: using /usr/bin/g++ to provide /usr/bin/c++ (c++) in auto mode
update-alternatives: warning: skip creation of /usr/share/man/man1/c++.1.gz because associated file /usr/share/man/man1/g++.1.gz (of link group c++) doesn't exist
Setting up git (1:2.17.1-1ubuntu0.8) ...
Setting up build-essential (12.4ubuntu1) ...
Setting up pkg-config (0.29.1-0ubuntu2) ...
Processing triggers for libc-bin (2.27-3ubuntu1.4) ...
Removing intermediate container a7d5d08df844
 ---> 31208d176373
Step 5/5 : USER cnb
 ---> Running in 1da306ce677a
Removing intermediate container 1da306ce677a
 ---> fd20d80427ba
Successfully built fd20d80427ba
Successfully tagged openfunctiondev/buildpacks-node14-build:v1
```

#### 查看构建镜像

```shell
$ docker images
REPOSITORY                                TAG       IMAGE ID       CREATED          SIZE
openfunctiondev/buildpacks-node14-build   v1        fd20d80427ba   13 minutes ago   581MB
openfunctiondev/buildpacks-node14-run     v1        c73ea61ef070   15 minutes ago   219MB
node14common                              latest    8c14d1b76b45   15 minutes ago   219MB
```

#### 构建builder镜像

```shell
$ bazel build //builders/node14:builder.image
INFO: Analyzed target //builders/node14:builder.image (35 packages loaded, 6740 targets configured).
INFO: Found 1 target...
WARNING: Use of implicit local fallback will go away soon, please set a fallback strategy instead. See --legacy_local_fallback option.
INFO: From Executing genrule //builders/node14:builder.image:
2021/07/20 07:24:25 Checking tools
2021/07/20 07:24:25 pack found at /usr/local/bin/pack
2021/07/20 07:24:25 docker found at /usr/bin/docker
2021/07/20 07:24:25 container-structure-test found at /usr/local/bin/container-structure-test
2021/07/20 07:24:25 Checking pack version
2021/07/20 07:24:25 Found pack version 0.19.0+git-360dbae.build-2550
Extracting builder tar:
./
./builder.toml
./entrypoint.tgz
./label.tgz./nodejs/
./nodejs/functions_framework.tgz
./nodejs/npm.tgz
./nodejs/yarn.tgz
Warning: Command 'pack create-builder' has been deprecated, please use 'pack builder create' instead
Downloading from 'https://github.com/buildpacks/lifecycle/releases/download/v0.11.1/lifecycle-v0.11.1+linux.x86-64.tgz'
5.25 MB/5.25 MB
Successfully created builder image 'of/node14'
Tip: Run 'pack build <image-name> --builder of/node14' to use this builder
Target //builders/node14:builder.image up-to-date:
  bazel-bin/builders/node14/builder.sha
INFO: Elapsed time: 57.057s, Critical Path: 14.50s
INFO: 57 processes: 19 internal, 37 linux-sandbox, 1 local.
INFO: Build completed successfully, 57 total actions
```

#### 查看构建镜像

```shell
$ docker images
REPOSITORY                                TAG       IMAGE ID       CREATED          SIZE
openfunctiondev/buildpacks-node14-build   v1        fd20d80427ba   19 minutes ago   581MB
openfunctiondev/buildpacks-node14-run     v1        c73ea61ef070   20 minutes ago   219MB
node14common                              latest    8c14d1b76b45   20 minutes ago   219MB
of/node14                                 latest    65b532e3472e   41 years ago     628MB
gcr.io/gcp-runtimes/ubuntu_18_0_4         latest    5f94f70d5d7e   51 years ago     76.7MB
```

#### 给镜像打标签并上传到仓库

docker login -u xxxxx(dockerhub注册的docker用户名)，登录docker成功

```sh
docker tag of/node14 penghuima/node14:v1
docker login -u penghuima
docker push penghuima/node14:v1
```

#### Test

```shell
$ bazel test //builders/node14/acceptance/...
INFO: Analyzed 2 targets (8 packages loaded, 208 targets configured).
INFO: Found 1 target and 1 test target...
INFO: Elapsed time: 146.990s, Critical Path: 135.67s
INFO: 13 processes: 3 internal, 9 linux-sandbox, 1 local.
INFO: Build completed successfully, 13 total actions
//builders/node14/acceptance:nodejs_fn_test                              PASSED in 134.2s

Executed 1 out of 1 test: 1 test passes.
INFO: Build completed successfully, 13 total actions
```

#### RUN Samples

```shell
$ pack build function-node1 --builder of/node14 --env FUNC_NAME="helloWorld"
v1: Pulling from openfunctiondev/buildpacks-node14-run
228505c46125: Pull complete
52b243a9211b: Pull complete
3c2cba919283: Pull complete
b7fa2f049eab: Pull complete
c76eab094eb6: Pull complete
7041c64b240b: Pull complete
Digest: sha256:e308aa40add1c6454d280d1d46039bdbd941a59a5d9b27e796dc660a70664cbf
Status: Downloaded newer image for openfunctiondev/buildpacks-node14-run:v1
0.11.1: Pulling from buildpacksio/lifecycle
60775238382e: Pull complete
49f2c9c87b35: Pull complete
Digest: sha256:2ca9870d9472400792bdc2d43ded8df2642ffcf937e8cd01685836678c28780b
Status: Downloaded newer image for buildpacksio/lifecycle:0.11.1
===> DETECTING
[detector] 2 of 3 buildpacks participating
[detector] google.nodejs.functions-framework 0.9.1
[detector] google.utils.label                0.0.1
===> ANALYZING
[analyzer] Previous image with name "function-node1" not found
===> RESTORING
===> BUILDING
[builder] === Node.js - Functions Framework (google.nodejs.functions-framework@0.9.1) ===
[builder] --------------------------------------------------------------------------------
[builder] Running "node --check index.js"
[builder] Done "node --check index.js" (41.710527ms)
[builder] Handling functions without dependency on functions-framework.
[builder] Installing application dependencies.
[builder] --------------------------------------------------------------------------------
[builder] Running "npm ci --quiet --production --prefix /layers/google.nodejs.functions-framework/functions-framework"
[builder] added 52 packages in 1.155s
[builder] Done "npm ci --quiet --production --prefix /layers/google.nodejs.f..." (1.723402949s)
[builder] === Utils - Label Image (google.utils.label@0.0.1) ===
===> EXPORTING
[exporter] Adding layer 'google.nodejs.functions-framework:functions-framework'
[exporter] Adding 1/1 app layer(s)
[exporter] Adding layer 'launcher'
[exporter] Adding layer 'config'
[exporter] Adding layer 'process-types'
[exporter] Adding label 'io.buildpacks.lifecycle.metadata'
[exporter] Adding label 'io.buildpacks.build.metadata'
[exporter] Adding label 'io.buildpacks.project.metadata'
[exporter] Setting default process type 'web'
[exporter] Saving function-node1...
[exporter] *** Images (19b2491d66f0):
[exporter]       function-node1
[exporter] Adding cache layer 'google.nodejs.functions-framework:functions-framework'
Successfully built image function-node1
```

运行镜像

```shell
$ docker run --rm -p8080:8080 function-node1
Serving function...
Function: helloWorld   #功能函数的名字
URL: http://localhost:8080/
```

```shell
$ curl http://localhost:8080/
hello, world! nodejs14 !   #index.js的输出内容
```

### 复现go16环境

```
docker tag of/go116 penghuima/go116:v1
docker push penghuima/go116:v1
```

上传镜像之前需要先登录自己的docker用户名，否则被拒

docker login -u xxxxx(dockerhub注册的docker用户名)，登录docker成功

```shell
$ bazel build //builders/go116:builder.image
INFO: Analyzed target //builders/go116:builder.image (0 packages loaded, 0 targets configured).
INFO: Found 1 target...
INFO: From Executing genrule //builders/go116:builder.image:
2021/07/20 01:23:26 Checking tools
2021/07/20 01:23:26 pack found at /usr/local/bin/pack
2021/07/20 01:23:26 docker found at /usr/bin/docker
2021/07/20 01:23:26 container-structure-test found at /usr/local/bin/container-structure-test
2021/07/20 01:23:26 Checking pack version
2021/07/20 01:23:26 Found pack version 0.18.0+git-e00ee4a.build-2328
Extracting builder tar:
./
./builder.toml
./entrypoint.tgz
./label.tgz
./go/
./go/build.tgz
./go/clear_source.tgz
./go/of_functions_framework.tgz
Warning: Command 'pack create-builder' has been deprecated, please use 'pack builder create' instead
Downloading from 'https://github.com/buildpacks/lifecycle/releases/download/v0.11.1/lifecycle-v0.11.1+linux.x86-64.tgz'
5.25 MB/5.25 MB
Successfully created builder image 'of/go116'
Tip: Run 'pack build <image-name> --builder of/go116' to use this builder
Target //builders/go116:builder.image up-to-date:
  bazel-bin/builders/go116/builder.sha
INFO: Elapsed time: 6.150s, Critical Path: 5.67s
INFO: 2 processes: 1 internal, 1 local.
INFO: Build completed successfully, 2 total actions
```

跑 sample

```shell
$ pack build function-go --builder of/go116 --env FUNC_NAME="HelloWorld"
v1: Pulling from openfunctiondev/buildpacks-go116-run
228505c46125: Pull complete
52b243a9211b: Pull complete
3c2cba919283: Pull complete
ed29cdb7332a: Pull complete
899246fbdba5: Pull complete
b74e51cfc140: Pull complete
Digest: sha256:a69808bfe76d869cf0d92506adac919c906673e26ca00acba486e06cd2e3a867
Status: Downloaded newer image for openfunctiondev/buildpacks-go116-run:v1
0.11.1: Pulling from buildpacksio/lifecycle
60775238382e: Pull complete
49f2c9c87b35: Pull complete
Digest: sha256:2ca9870d9472400792bdc2d43ded8df2642ffcf937e8cd01685836678c28780b
Status: Downloaded newer image for buildpacksio/lifecycle:0.11.1
===> DETECTING
[detector] 3 of 5 buildpacks participating
[detector] openfunction.go.of-functions-framework 0.0.1
[detector] google.go.build                        0.9.0
[detector] google.utils.label                     0.0.1
===> ANALYZING
[analyzer] Previous image with name "function-go" not found
===> RESTORING
===> BUILDING
[builder] === Go - OpenFunction Functions Framework (openfunction.go.of-functions-framework@0.0.1) ===
[builder] --------------------------------------------------------------------------------
[builder] Running "go run /cnb/buildpacks/openfunction.go.of-functions-framework/0.0.1/converter/get_package/main.go -dir /workspace/serverless_function_source_code (GOCACHE=/tmp/serverless_function_app536643583)"
[builder] helloDone "go run /cnb/buildpacks/openfunction.go.of-functions-framewor..." (1.03171436s)
[builder] go.sum not found, generating using "go mod tidy"
[builder] functions-framework not specified in go.mod, using default
[builder] --------------------------------------------------------------------------------
[builder] Running "go get github.com/OpenFunction/functions-framework-go@v0.0.0-20210628081257-4137e46a99a6"
[builder] go: downloading github.com/OpenFunction/functions-framework-go v0.0.0-20210628081257-4137e46a99a6
[builder] go get: added github.com/OpenFunction/functions-framework-go v0.0.0-20210628081257-4137e46a99a6
[builder] Done "go get github.com/OpenFunction/functions-framework-go@v0.0.0..." (6.552133836s)
[builder] === Go - Build (google.go.build@0.9.0) ===
[builder] --------------------------------------------------------------------------------
[builder] Running "go list -f {{if eq .Name \"main\"}}{{.Dir}}{{end}} ./..."
[builder] /workspace
[builder] Done "go list -f {{if eq .Name \"main\"}}{{.Dir}}{{end}} ./..." (526.50687ms)
[builder] --------------------------------------------------------------------------------
[builder] Running "go build -o /layers/google.go.build/bin/main ./. (GOCACHE=/layers/google.go.build/gocache CGO_ENABLED=0)"
[builder] Done "go build -o /layers/google.go.build/bin/main ./. (GOCACHE=/l..." (24.611873954s)
[builder] === Utils - Label Image (google.utils.label@0.0.1) ===
===> EXPORTING
[exporter] Adding layer 'openfunction.go.of-functions-framework:functions-framework'
[exporter] Adding layer 'google.go.build:bin'
[exporter] Adding 1/1 app layer(s)
[exporter] Adding layer 'launcher'
[exporter] Adding layer 'config'
[exporter] Adding layer 'process-types'
[exporter] Adding label 'io.buildpacks.lifecycle.metadata'
[exporter] Adding label 'io.buildpacks.build.metadata'
[exporter] Adding label 'io.buildpacks.project.metadata'
[exporter] Setting default process type 'web'
[exporter] Saving function-go...
[exporter] *** Images (52c5a6e51f85):
[exporter]       function-go
Successfully built image function-go
```

