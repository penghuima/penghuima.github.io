---
layout: post
title: "Ruby Buildpack"
categories: builder
tags: Buildpacks builder ruby
---

* content
{:toc}


学着制作ruby 语言的一层buildpack，完成 hello ruby ! 测试

 <!--more-->

## creating a Ruby Cloud Native Buildpack

### Set up your local environment

```shell
mkdir ruby-sample-app

cd ruby-sample-app

touch app.rb
cat > app.rb << EOF
require 'sinatra'

set :bind, '0.0.0.0'
set :port, 8080

get '/' do
  'Hello World!'
end
EOF

touch Gemfile
cat > Gemfile << EOF
source "https://rubygems.org"

git_source(:github) {|repo_name| "https://github.com/#{repo_name}" }

gem "sinatra"
EOF
```

### Building blocks of a Cloud Native Buildpack

```shell
pack buildpack new examples/ruby \
    --api 0.5 \
    --path ruby-buildpack \
    --version 0.0.1 \
    --stacks io.buildpacks.samples.stacks.bionic
    
cd ruby-buildpack/bin
cat > detect << EOF
#!/usr/bin/env bash
set -eo pipefail

exit 1
EOF

cat > build << EOF
#!/usr/bin/env bash
set -eo pipefail

echo "---> Ruby Buildpack"
exit 1
EOF

cd ~
pack config default-builder cnbs/sample-builder:bionic
pack build test-ruby-app --path ./ruby-sample-app --buildpack ./ruby-sample-app/ruby-buildpack
```

### Detecting your application

接下来，您将希望实际检测您的应用程序是一个Ruby应用程序。 为此，您需要检查Gemfile

```shell
cd ruby-sample-app/ruby-buildpack/bin
cat > detect << EOF
#!/usr/bin/env bash
set -eo pipefail

if [[ ! -f Gemfile ]]; then
   exit 100
fi
EOF
```

您还会注意到现在，分析显示在构建输出中。 此步骤是BuildPack Lifecycle的一部分，看起来可以看出任何以前的映像构建是否具有BuildPack可以重新使用的图层。

### Building your application

```shell
# cat >   << EOF   不能出现$ 这种转义符
#!/usr/bin/env bash
set -eo pipefail

echo "---> Ruby Buildpack"

# 1. GET ARGS
layersdir=$1

# 2. CREATE THE LAYER DIRECTORY
rubylayer="$layersdir"/ruby
mkdir -p "$rubylayer"

# 3. DOWNLOAD RUBY
echo "---> Downloading and extracting Ruby"
ruby_url=https://s3-external-1.amazonaws.com/heroku-buildpack-ruby/heroku-18/ruby-2.5.1.tgz
wget -q -O - "$ruby_url" | tar -xzf - -C "$rubylayer"

# 4. MAKE RUBY AVAILABLE DURING LAUNCH
echo -e 'launch = true' > "$layersdir/ruby.toml"

# 5. MAKE RUBY AVAILABLE TO THIS SCRIPT
export PATH="$rubylayer"/bin:$PATH
export LD_LIBRARY_PATH=${LD_LIBRARY_PATH:+${LD_LIBRARY_PATH}:}"$rubylayer/lib"

# 6. INSTALL BUNDLER
echo "---> Installing bundler"
gem install bundler --no-ri --no-rdoc

# 7. INSTALL GEMS
echo "---> Installing gems"
bundle install

```

镜像构建结果

```shell
$ pack build test-ruby-app --path ./ruby-sample-app --buildpack ./ruby-sample-app/ruby-buildpack
bionic: Pulling from cnbs/sample-builder
Digest: sha256:b7edf381d738273ae4b382221c49385df28dd551839db584b76f4c65893142e8
Status: Image is up to date for cnbs/sample-builder:bionic
bionic: Pulling from cnbs/sample-stack-run
Digest: sha256:7606c5881eaa0a1aebb65861f12928f9f253c242d46c4ba836b2c002e2bb3cfb
Status: Image is up to date for cnbs/sample-stack-run:bionic
0.11.3: Pulling from buildpacksio/lifecycle
Digest: sha256:d6c578fbdf88f2e2594d9907307b17775e648656f62e1ae810d31c804f928cf9
Status: Image is up to date for buildpacksio/lifecycle:0.11.3
===> DETECTING
[detector] examples/ruby 0.0.1
===> ANALYZING
[analyzer] Previous image with name "test-ruby-app" not found
===> RESTORING
===> BUILDING
#!/usr/bin/env bash
[builder] ---> Ruby Buildpack
[builder] I want to konw the string layersdir:
[builder] ---> Downloading and extracting Ruby
[builder] ---> Installing bundler
[builder] Successfully installed bundler-2.2.24
[builder] 1 gem installed
[builder] ---> Installing gems
[builder] Fetching gem metadata from https://rubygems.org/....
[builder] Resolving dependencies...
[builder] Using bundler 2.2.24
[builder] Fetching ruby2_keywords 0.0.5
[builder] Fetching rack 2.2.3
[builder] Installing ruby2_keywords 0.0.5
[builder] Fetching tilt 2.0.10
[builder] Installing rack 2.2.3
[builder] Installing tilt 2.0.10
[builder] Fetching mustermann 1.1.1
[builder] Fetching rack-protection 2.1.0
[builder] Installing mustermann 1.1.1
[builder] Installing rack-protection 2.1.0
[builder] Fetching sinatra 2.1.0
[builder] Installing sinatra 2.1.0
[builder] Bundle complete! 1 Gemfile dependency, 7 gems now installed.
[builder] Use `bundle info [gemname]` to see where a bundled gem is installed.
===> EXPORTING
[exporter] Adding layer 'examples/ruby:ruby'
[exporter] Adding 1/1 app layer(s)
[exporter] Adding layer 'launcher'
[exporter] Adding layer 'config'
[exporter] Adding label 'io.buildpacks.lifecycle.metadata'
[exporter] Adding label 'io.buildpacks.build.metadata'
[exporter] Adding label 'io.buildpacks.project.metadata'
[exporter] Warning: default process type 'web' not present in list []
[exporter] Saving test-ruby-app...
[exporter] *** Images (8b343d375449):
[exporter]       test-ruby-app
Successfully built image test-ruby-app
```

我想看一下layersdir 字符串内容是什么

```shell
$ pack build test-ruby-app --path ./ruby-sample-app --buildpack ./ruby-sample-app/ruby-buildpack
bionic: Pulling from cnbs/sample-builder
Digest: sha256:b7edf381d738273ae4b382221c49385df28dd551839db584b76f4c65893142e8
Status: Image is up to date for cnbs/sample-builder:bionic
bionic: Pulling from cnbs/sample-stack-run
Digest: sha256:7606c5881eaa0a1aebb65861f12928f9f253c242d46c4ba836b2c002e2bb3cfb
Status: Image is up to date for cnbs/sample-stack-run:bionic
0.11.3: Pulling from buildpacksio/lifecycle
Digest: sha256:d6c578fbdf88f2e2594d9907307b17775e648656f62e1ae810d31c804f928cf9
Status: Image is up to date for buildpacksio/lifecycle:0.11.3
===> DETECTING
[detector] examples/ruby 0.0.1
===> ANALYZING
[analyzer] Restoring metadata for "examples/ruby:ruby" from app image
===> RESTORING
===> BUILDING
[builder] ---> Ruby Buildpack
[builder] I want to konw the string layersdir:
[builder] /layers/examples_ruby
[builder] ---> Downloading and extracting Ruby
[builder] ---> Installing bundler
[builder] Successfully installed bundler-2.2.24
[builder] 1 gem installed
[builder] ---> Installing gems
[builder] Fetching gem metadata from https://rubygems.org/....
[builder] Resolving dependencies...
[builder] Using bundler 2.2.24
[builder] Fetching ruby2_keywords 0.0.5
[builder] Fetching rack 2.2.3
[builder] Installing ruby2_keywords 0.0.5
[builder] Installing rack 2.2.3
[builder] Fetching tilt 2.0.10
[builder] Fetching mustermann 1.1.1
[builder] Installing tilt 2.0.10
[builder] Installing mustermann 1.1.1
[builder] Fetching rack-protection 2.1.0
[builder] Installing rack-protection 2.1.0
[builder] Fetching sinatra 2.1.0
[builder] Installing sinatra 2.1.0
[builder] Bundle complete! 1 Gemfile dependency, 7 gems now installed.
[builder] Use `bundle info [gemname]` to see where a bundled gem is installed.
===> EXPORTING
[exporter] Reusing layer 'examples/ruby:ruby'
[exporter] Adding 1/1 app layer(s)
[exporter] Reusing layer 'launcher'
[exporter] Reusing layer 'config'
[exporter] Adding label 'io.buildpacks.lifecycle.metadata'
[exporter] Adding label 'io.buildpacks.build.metadata'
[exporter] Adding label 'io.buildpacks.project.metadata'
[exporter] Warning: default process type 'web' not present in list []
[exporter] Saving test-ruby-app...
[exporter] *** Images (e4ded680d309):
[exporter]       test-ruby-app
Successfully built image test-ruby-app
```

### Make your application runnable

```shell
#!/usr/bin/env bash
set -eo pipefail

echo "---> Ruby Buildpack"

# 1. GET ARGS
layersdir=$1

# 2. CREATE THE LAYER DIRECTORY
rubylayer="$layersdir"/ruby
mkdir -p "$rubylayer"

# 3. DOWNLOAD RUBY
echo "---> Downloading and extracting Ruby"
ruby_url=https://s3-external-1.amazonaws.com/heroku-buildpack-ruby/heroku-18/ruby-2.5.1.tgz
wget -q -O - "$ruby_url" | tar -xzf - -C "$rubylayer"

# 4. MAKE RUBY AVAILABLE DURING LAUNCH
echo -e 'launch = true' > "$layersdir/ruby.toml"

# 5. MAKE RUBY AVAILABLE TO THIS SCRIPT
export PATH="$rubylayer"/bin:$PATH
export LD_LIBRARY_PATH=${LD_LIBRARY_PATH:+${LD_LIBRARY_PATH}:}"$rubylayer/lib"

# 6. INSTALL BUNDLER
echo "---> Installing bundler"
gem install bundler --no-ri --no-rdoc

# 7. INSTALL GEMS
echo "---> Installing gems"
bundle install

# ========== ADDED ===========
# 8. SET DEFAULT START COMMAND
cat > "$layersdir/launch.toml" <<EOL
[[processes]]
type = "web"
command = "bundle exec ruby app.rb"
EOL
```

```shell
$ docker run --rm -p 8080:8080 test-ruby-app
$  curl http://localhost:8080/
Hello World , Ruby !
```

### Improving performance with caching

通过给依赖增添一个layer ,就可以利用缓冲在构建过程中，仅在必要时下载新的依赖来提高性能

```shell
#!/usr/bin/env bash
set -eo pipefail

echo "---> Ruby Buildpack"

# 1. GET ARGS
layersdir=$1

# 2. CREATE THE LAYER DIRECTORY
rubylayer="$layersdir"/ruby
mkdir -p "$rubylayer"

# 3. DOWNLOAD RUBY
echo "---> Downloading and extracting Ruby"
ruby_url=https://s3-external-1.amazonaws.com/heroku-buildpack-ruby/heroku-18/ruby-2.5.1.tgz
wget -q -O - "$ruby_url" | tar -xzf - -C "$rubylayer"

# 4. MAKE RUBY AVAILABLE DURING LAUNCH
echo -e 'launch = true' > "$layersdir/ruby.toml"

# 5. MAKE RUBY AVAILABLE TO THIS SCRIPT
export PATH="$rubylayer"/bin:$PATH
export LD_LIBRARY_PATH=${LD_LIBRARY_PATH:+${LD_LIBRARY_PATH}:}"$rubylayer/lib"

# 6. INSTALL BUNDLER
echo "---> Installing bundler"
gem install bundler --no-ri --no-rdoc

# 7. INSTALL GEMS
echo "---> Installing gems"
bundlerlayer="$layersdir/bundler"
mkdir -p "$bundlerlayer"
echo -e 'cache = true\nlaunch = true' > "$layersdir/bundler.toml"
bundle config set --local path "$bundlerlayer" && bundle install && bundle binstubs --all --path "$bundlerlayer/bin"

# 8. SET DEFAULT START COMMAND
cat > "$layersdir/launch.toml" <<EOL
# our web process
[[processes]]
type = "web"
command = "bundle exec ruby app.rb"

# our worker process
[[processes]]
type = "worker"
command = "bundle exec ruby worker.rb"
EOL
```

重建一个gemfile

```
cat > Gemfile.lock <<EOF
GEM
  remote: https://rubygems.org/
  specs:
    mustermann (1.0.3)
    rack (2.0.7)
    rack-protection (2.0.7)
      rack
    sinatra (2.0.7)
      mustermann (~> 1.0)
      rack (~> 2.0)
      rack-protection (= 2.0.7)
      tilt (~> 2.0)
    tilt (2.0.9)

PLATFORMS
  ruby

DEPENDENCIES
  sinatra

BUNDLED WITH
   2.0.2
EOF
```

## Package a buildpack

给buildpack做分发，制作镜像之后，推到仓库，随处可用

```sh
$ pack buildpack package my-buildpack --config ./package.toml
hello-world: Pulling from cnbs/sample-package
0cceee8a8cb0: Pull complete
Digest: sha256:577f4c578888e51f2c0b8dcfdcfccc51582f1eafe0dce7997b6194d8799c14ba
Status: Downloaded newer image for cnbs/sample-package:hello-world
Successfully created package my-buildpack
```

