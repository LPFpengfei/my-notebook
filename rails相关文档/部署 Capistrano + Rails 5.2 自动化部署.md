[原文地址 https://itfun.tv/news/47](https://itfun.tv/news/47)

相关文章： [「Mina + Rails 5.2 自动化部署」](https://ruby-china.org/topics/36910)

## 前言

1. 为什么要用它
    
    这是一个一劳永逸的工作，有了它。你不用每次版本更新都去服务器上git pull，不用bundle install，也不用跑 touch tmp/restart.txt之类的命令，更不需要还在用FTP上传代码了。每次新版本更新，只需要在你本机，运行一条cap production deploy命令。服务器就会自动git pull远程仓库的最新代码，并自动完成一系列部署相关的操作。

2. 环境

这是一个一劳永逸的工作，有了它。你不用每次版本更新都去服务器上git pull，也不用跑 touch tmp/restart.txt之类的命令，更不需要还在用FTP上传代码了。

每次新版本更新，只需要在你本机，运行一条cap production deploy命令。服务器就会自动git pull远程仓库的最新代码，并且完成相关的一系列操作。

## 环境

网上诸多Capistrano都已经过时了，部署中因为版本原因会出现一大堆错误，于是我重新整理了完整的流程。只要你不是完全的Linux小白，稍微对这方面有一点了解，相信你参照我写的流程，都能自己部署出来了。

流程中，所采用的服务器环境如下：

服务器系统：我选用的服务器系统是 Ubuntu Server 16.04，如果你是其他系统，流程可能会有部分不一样。如果你用Rails，那么我只推荐这一个服务器操作系统，坑少、易解决。
Rails版本：我所使用的Rails版本是5.2。如果你用的老版本Rails，会有一些不一样的地方，哪里需要注意的，文里也有说明。
数据库: MySQL 5.7
其他：再就是用了Nginx 和 Passenger了。还介绍了SSH秘钥和Git使用的基础知识。

> Tips: 需要注意的是，有的命令要在服务器上运行，有的是在你电脑本地运行的。注释里都有说明，请务必不要搞错了~！

### 一、新建用户

```shell
# 服务器上
# `--ingroup sudo`是说，新建的用户直接就有执行`sudo`命令的权限。
adduser deploy --ingroup sudo
# 输入新密码。
# 问其他的Full Name之类的，你可以填写，也可以不填，直接回车。

# 切换到deploy用户
sudo su deploy

# 进入`家`目录
cd ~
```

### 二、使用秘钥登录服务器

很多云服务器，大部分默认都是使用账号密码登录的。这样做非常不安全，所以你需要改为SSH秘钥登录。如果你想改为秘钥登录，一种是直接在云主机管理界面，上传自己的秘钥。另一种通用方法，就是我下面的操作了。

相信玩Ruby On Rails的基本都是Mac OS用户了。以下演示都以Mac机为例子操作。

```shell
# Mac 本地
ssh-keygen -t rsa
# 如果不需要加密，就直接全部回车。需要加密，就自己填写密码

cat ~/.ssh/id_rsa.pub
# 会出现一段文字，复制下来
```

```shell
# 服务器上
ssh-keygen -t rsa
# 依然全部直接回车

vi /home/deploy/.ssh/authorized_keys
# 将刚才Mac本地命令行中，复制的那一段文字，粘贴进去，然后按:wq保存离开

chmod 644 /home/deploy/.ssh/authorized_keys
sudo service ssh restart
```

现在你再用SSH连服务器，直接ssh deploy@你的ip，不再需要输入密码了。

### 三、禁用密码登录（可选）

用密码登录服务器， 并不是一个安全的选择。最好方法是直接禁用密码登录，改为必须使用SSH秘钥登录。当然这一步是可选的，和我们学习Capistrano并没有什么关系。

```shell
# 服务器上
sudo vi /etc/ssh/sshd_config

# 将`PasswordAuthentication yes` 修改成 `PasswordAuthentication no`
# :wq退出后
sudo service ssh restart
```

### 四、服务器基础准备工作

```shell
# 服务器上
# 确认一下，当前用户依然是deploy。如果不是，先sudo su deploy

# 更新
sudo apt-get update
sudo apt-get upgrade -y
sudo dpkg-reconfigure tzdata
# 选择时区 Time zone=>Asia=>Shanghai

# 安装Rails所必须的各种常见依赖
sudo apt-get install -y build-essential git-core bison openssl libreadline6-dev curl zlib1g zlib1g-dev libssl-dev libyaml-dev libsqlite3-0 libsqlite3-dev sqlite3  autoconf libc6-dev libpcre3-dev curl libcurl4-nss-dev libxml2-dev libxslt-dev imagemagick nodejs libffi-dev
```

Ubuntu 16.04 的 apt-get install 默认只支持 Ruby 2.3。对新版本Rails 5.2来说，已经无法运行了。所以我们选择使用 rbenv 来安装 Ruby。

### 五、使用rbenv安装Ruby

```shell
# 服务器上
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

rbenv需要ruby-build，才能安装ruby。所以现在来安装它。

```shell
mkdir -p "$(rbenv root)"/plugins
git clone https://github.com/rbenv/ruby-build.git "$(rbenv root)"/plugins/ruby-build
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
source ~/.bashrc

# 看一下，现在可安装的Ruby版本
rbenv install -l
# 我写文章这个时间，最新的是2.5.3

rbenv install 2.5.3

# 设置成全局默认使用
rbenv global 2.5.3

# 看一下是否装成功
ruby -v

# 使用Ruby China的RubyGems（境外服务器请略过）
gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/
gem sources -l
# 确保只有 gems.ruby-china.com

# 接着安装 bundler gem
gem install bundler

# 同样使用gems.ruby-china.com
bundle config mirror.https://rubygems.org https://gems.ruby-china.com
```

### 六、安装MySQL

```shell
# 服务器上
sudo apt-get install mysql-common mysql-client libmysqlclient-dev mysql-server
# 安装过程中，会让你输入密码。自己记好了哦！

# 新建一个数据库，其中deployment是你数据库的名字，可根据需求自行修改
mysql -u root -p
CREATE DATABASE deployment_production CHARACTER SET utf8mb4;

# 退出 mysql console
exit
```

### 七、安装 Nginx + Passenger

```shell
# 服务器上
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 561F9B9CAC40B2F7

sudo apt-get install -y apt-transport-https ca-certificates

# 添加 APT 仓库地址
sudo sh -c 'echo deb https://oss-binaries.phusionpassenger.com/apt/passenger xenial main > /etc/apt/sources.list.d/passenger.list'

sudo apt-get update

# 安装 Passenger + Nginx
sudo apt-get install -y nginx-extras passenger
```

### 八、注册Coding账号

你这里使用GitHub、Coding、码云哪个都行。我这里以Coding为例子演示，之所以用Coding，是考虑到国内企业付费用GitHub的比较少。没用码云，是因为页面没有Coding的好看，这个很重要。

Coding的注册地址https://coding.net

### 九、Git设置SSH公钥

![](https://images.itfun.tv/photo/2018/045786e3f81a9ea92e0deaa46ada61da.jpg)

```shell
# Mac 本地
cat ~/.ssh/id_rsa.pub
# 会出现一段文字，复制下来，粘贴到下面`公钥内容`中，选择`永久有效`。
```

![](https://images.itfun.tv/photo/2018/748322c7bdd52b67a0e1ce50abaf377b.jpg)

这样你在使用Git的时候，就不需要每次输入账号密码了，直接用SSH秘钥。

### 十、新建一个Git项目

![](https://images.itfun.tv/photo/2018/a9b531401a9fbaf59c5b7cd39fa5f479.jpg)

![](https://images.itfun.tv/photo/2018/e34b0754401b7ffd6965e4336c8b9219.jpg)

点击代码浏览后，选择SSH方式，点击复制按钮。

### 十一、新建一个Rails项目

```shell
# Mac 本地
# 如果你已经有开发好的项目，这一步请直接略过。

# 新建一个目录，存放rails项目
mkdir -p ~/Developer/Rails

# 新建一个Rails项目，默认使用MySQL数据库
rails new deployment -d mysql
```

### 十二、修改gemfile，安装Capistrano以及插件

到gemfile的group :development中，添加如下代码，这些都是capistrano的插件

```shell
group :development do
  # ...

  # 其中`capistrano-rails`包含了以下三个插件。
  # gem 'capistrano/bundler'
  # gem 'capistrano/rails/assets'
  # gem 'capistrano/rails/migrations'
  # 你也可以分别一个个加进去，但是何必呢？这些基本都是`rails`部署必须的。
  # 直接用`gem 'capistrano-rails'`这一个就好了。
  gem 'capistrano-rails'

  # 对`passenger`与`rbenv`的支持
  gem 'capistrano-passenger'
  gem 'capistrano-rbenv'
end
```

```shell
# 改完后，Mac 本地，命令行进入自己项目中
cd ~/Developer/Rails/deployment
bundle install
```

### 十三、将代码推送到Git远程仓库

Mac上的Git客户端，常用的有两个。一个是需要收费的Tower，另个一是免费的SourceTree。其实用起来差别不大，只是我个人Tower用的比较顺手，于是直接购买了。当然你也可以不装任何客户端，直接用命令。

我们现在为了演示方便，直接用就用命令行操作

```shell
# 确认命令行当前的路径，在刚才新建的项目中
git add .
git commit -m "初次提交"
# 将下面的地址换成自己刚复制的
git remote add origin git@git.coding.net:aaronryuu/deployment.git
git push -u origin master
```

去coding刷新页面，代码都已经提交上去了

![](https://images.itfun.tv/photo/2018/a03876edd5bf0a195fc0d0d4d7fa1c9f.jpg-large)

### 十四、配置Capistrano

1. 生成capistrano的相关配置文件。

```shell
# Mac本地运行
cap install
```

2. 编辑 Capfile（项目的根目录下）

```shell
# 加上这行
require "capistrano/rails"

# 去掉这两行前面的`#`号 
require "capistrano/rbenv"
require "capistrano/passenger"
```

其他配置可保持默认。

![](https://images.itfun.tv/photo/2018/68cf5eff320b1eea5b8d52080fd1622d.jpg)

3. 编辑 config/deploy.rb

```shell
# 最顶上加这行，注意是「`」号而不是单引号「'」
# 如果你对ssh-add有兴趣，你可以去读这一篇。https://ihower.tw/blog/archives/7837
`ssh-add`

# 项目名称
set :application, "deployment"

# git仓库地址
set :repo_url, "git@git.coding.net:aaronryuu/deployment.git"

# 需要部署到服务器的位置
set :deploy_to, "/home/deploy/deployment"

# 去掉注释，并加上 "config/master.key"
append :linked_files, "config/database.yml", "config/master.key"

# 去掉注释
append :linked_dirs, 'log', 'tmp/pids', 'tmp/cache', 'tmp/sockets', 'public/system'
```

> 注意了：我这里是Rails 5.2的部署，这个版本开始，config/secrets.yml变成了config/master.key。所以，如果你的项目Rails版本低于5.2，那这里应该是

```shell
append :linked_files, 'config/database.yml', 'config/master.key'
```

如果这里不正确处理，后面部署可能会碰到这个错误！

```shell
ArgumentError: Missing `secret_key_base` for 'production' environment, set this string with `rails credentials:edit`
```

![](https://images.itfun.tv/photo/2018/1deed707238cd08faee487bd259f0b25.jpg)

4. 编辑config/deploy/production.rb

```shell
# 改成你自己的ip
server "114.67.72.94", user: "deploy", roles: %w{app db web}, my_property: :my_value

set :ssh_options, {
  keys: %w(~/.ssh/id_rsa),
  forward_agent: true,
  auth_methods: %w(publickey)
}
```

![](https://images.itfun.tv/photo/2018/a52e284e29908441f83f478f980b52fc.jpg)

5. 尝试报错

```shell
# Mac 本地执行
cap production deploy:check
```

出现错误

```shell
ERROR linked file /home/deploy/deployment/shared/config/database.yml does not exist on 114.67.72.94
```

解决方法，是需要手动到服务器上，建config/database.yml，config/secrets.yml这两个文件，并做好配置。
服务器上，会出现releases和shared两个目录。releases是每次部署的文件，shared目录则是一些公用的配置文件。那么我们现在就去shared目录中，添加这两个公用的配置文件。

6. 新建database.yml文件

```shell
# 服务器上
cd ~/deployment/shared/config
vim database.yml
```

里面填写以下内容，当然，将password换成刚才装MySQL时自己填写的密码

```shell
production:
  adapter: mysql2
  pool: 25
  encoding: utf8mb4
  database: deployment_production
  host: localhost
  username: root
  password: itfun
```

7. 新建master.key文件

```shell
vim master.key
```

然后将自己本地项目config/master.key中的内容，复制进去。

8. 再试一次

```shell
# Mac 本地执行
cap production deploy:check
```

这次就没有报错了。

### 十五、正式部署

```shell
# Mac 本地
cap production deploy
```

第一次会比较慢，请耐心等待。这条命令，还会在服务器上建立一个叫current的目录，用symbolic link指向releases目录下最新的版本。

### 十六、配置nginx

1. 配置nginx支持passenger

```shell
# 服务器上
sudo vim /etc/nginx/nginx.conf
```

```shell
# 在此文件最顶部加上
env PATH;

# ...

# 去掉这一行的注释
include /etc/nginx/passenger.conf;
```

2. 新增一个项目配置

```shell
sudo vim /etc/nginx/sites-enabled/deployment.conf
```

```shell
server {
  listen 80;

  # 如果你有域名，并做好了域名解析，直接填域名。
  server_name 114.67.72.94; 

  root /home/deploy/deployment/current/public;

  passenger_enabled on;

  passenger_min_instances 1;

  location ~ ^/assets/ {
    expires 1y;
    add_header Cache-Control public;
    add_header ETag "";
    break;
   }
}
```

```shell
# 重启nginx
sudo service nginx restart
```

### 十七、尝试访问项目

![](https://images.itfun.tv/photo/2018/0654257d63d95bb988a6dc5a3108cc18.jpg)

这是其实是有由两个原因造成的。 第一、因为我们项目，目前还连首页都没有。 第二、Passanger需要指定使用的Ruby路径。查资料了解到，如果是apt-get安装的Ruby则没有这个问题。如果和我一样，用rbenv安装的，则需要手动指定Ruby路径。

> Tips: 如果部署中，你还碰到了其他问题。你可以看一下Nginx 错误日志中，是否有相关提示。路径在/var/log/nginx/error.log


#### 1. Rails的首页


config/routes.rb中，添加

```shell
root 'home#index'
```

```shell
# Mac本地
rails g controller home
```

编辑app/controllers/home_controller.rb

```ruby
class HomeController &lt; ApplicationController
  def index
     render plain: "Capistrano 自动化部署"
  end
end
```

每次修改完代码后，需要部署上线，都是以下这么个流程

```shell
# Mac本地，先确认现在命令行，是在项目目录中。
# 提交到git
git add .
git commit -m "新增首页"
git push

# 重新部署
cap production deploy
```

#### 2. 设置passanger，使用指定的 Ruby 版本

```shell
# 服务器上
# 先看一下ruby所在的路径
which ruby
sudo vim /etc/nginx/passenger.conf
```

```shell
# 屏蔽掉默认配置，添加自己的ruby路径
#passenger_ruby /usr/bin/passenger_free_ruby;
passenger_ruby /home/deploy/.rbenv/shims/ruby;
```

```shell
# 重启nginx
sudo service nginx restart
```

### 十八、成功

![](https://images.itfun.tv/photo/2018/673567dfbccc564d498d5a09ed913a3a.jpg)

### 十九、Capistrano对Sidekiq的支持

Sidekiq需要Redis，所以先去服务器上安装好。

```shell
# 服务器上
sudo apt-get install redis-server
```

```shell
# Mac本地
group :development do
  # ...

 gem 'capistrano-sidekiq'
end
```

安装一下

```shell
bundle
```

Capfile中

```shell
require 'capistrano/sidekiq'
```

重新部署完事~

```shell
cap production 
```
