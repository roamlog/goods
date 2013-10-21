本文档为在 CentOS 6.3 x86_64 版本上搭建开发环境，主要包括 Gitlab、Redmine、Tomcat 等，安装软件时默认使用的是 root 用户，下文命令中以`#`开头的表示在 Root 用户下，以 `$` 开头的表示是在 git 用户下。

## 安装 Gitlab

#### 1）安装必须的依赖软件包

	# yum -y update
	# yum -y groupinstall ‘Development Tools’
	# yum -y install gcc gcc-c++ make autoconf libyaml-devel gdbm-devel ncurses-devel openssl-devel zlib-devel readline-devel curl-devel expat-devel gettext-devel  tk-devel libxml2-devel libffi-devel libxslt-devel libicu-devel sendmail patch libyaml* pcre-devel sqlite-devel perl vim wget sudo

#### 2）安装附加软件包 EPEL

	# rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm

#### 3）安装 Python 2.7+

Gitlab 要求 Python 2.5.5+（暂不支持 Python 3.0），系统 Python 默认是 2.6.x，

	# mkdir /tmp/gitlab && cd /tmp/gitlab
	# curl --progress http://python.org/ftp/python/2.7.5/Python-2.7.5.tgz | tar xz
	# cd Python-2.7.5
	# ./configure --prefix=/usr/local
	# make && make install

安装好之后，需要做两件事情，替换默认 python 的版本至最新版本，因为系统默认 `PATH` 的寻址路径是 `/usr/local/bin`

	# ln -s /usr/local/bin/python2.7 /usr/local/bin/python

最后看下 Python 版本是否是刚刚安装的版本：

	# python --version

由于 yum 是 python 的一个 module，所以这块修改可能会引起无法调用 yum 脚本，所以需要修改这个文件 `/usr/bin/yum` 的第一行为 `!#/usr/bin/python2.6`

#### 4）安装 Ruby 2.0

	# cd /tmp/gitlab
	# curl --progress http://cache.ruby-lang.org/pub/ruby/2.0/ruby-2.0.0-p247.tar.gz | tar xz
	# cd ruby-2.0.0-p247
	# ./configure --prefix=/usr/local
	# make && make install

ruby 2.0 已经内置 gem，只需要安装 bundler

	# gem install bundler

若在执行 `sudo ruby` 或 `sudo gem` 找不到命令，因为编译的路径配置到了 `/usr/local/bin`，我们只需要做下软链接到 root 用户可以找到的 `$PATH` 路径：

	# ln -s /usr/local/bin/ruby /usr/bin/ruby
	# ln -s /usr/local/bin/gem /usr/bin/gem
	# ln -s /usr/local/bin/bundle /usr/bin/bundle

#### 5）安装 Git
	
	# cd /tmp/gitlab
	# yum -y install git perl-ExtUtils-MakeMaker
	# git clone git://github.com/git/git.git
	# cd git
	# git checkout v1.8.4.1
	# autoconf
	# ./configure –prefix=/usr/local
	# make && make install
	# yum erase git
	# ln -s /usr/local/bin/git /usr/bin/git
	
#### 6）安装 nginx

	# rpm -ivh http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm
	# yum -y install nginx
	# service nginx start
	# iptables -I INPUT -p tcp -m tcp --dport 80 -j ACCEPT
	# service iptables restart
	# chkconfig nginx on

#### 7）安装 Mysql

使用 yum 安装的 mysql 版本为 5.1，但我们想安装 5.6 版本的，我们采用手动下载 rpm 包的方式进行安装
	
	# wget http://dev.mysql.com/get/Downloads/MySQL-5.6/MySQL-shared-5.6.14-1.el6.x86_64.rpm/from/http://cdn.mysql.com/
	
	# wget http://dev.mysql.com/get/Downloads/MySQL-5.6/MySQL-server-5.6.14-1.el6.x86_64.rpm/from/http://cdn.mysql.com/

	# wget http://dev.mysql.com/get/Downloads/MySQL-5.6/MySQL-devel-5.6.14-1.el6.x86_64.rpm/from/http://cdn.mysql.com/
	
	# wget http://dev.mysql.com/get/Downloads/MySQL-5.6/MySQL-client-5.6.14-1.el6.x86_64.rpm/from/http://cdn.mysql.com/
	
开始安装

	# rpm -ivh MySQL-shared-5.6.14-1.el6.x86_64.rpm

	# rpm -ivh MySQL-server-5.6.14-1.el6.x86_64.rpm

	# rpm -ivh MySQL-devel-5.6.14-1.el6.x86_64.rpm

	# rpm -ivh MySQL-client-5.6.14-1.el6.x86_64.rpm
	
安装 Mysql-server 时出错

	libaio.so.1()(64bit) is needed by MySQL-server-5.6.14-1.el6.x86_64

下面是解决办法：

	yum install libaio.x86_64 libaio-devel.x86_64

然后发现还是出错，是类库冲突的原因，下面是解决办法：
	
	# wget http://dev.mysql.com/get/Downloads/MySQL-5.6/MySQL-shared-compat-5.6.14-1.el6.x86_64.rpm/from/http://cdn.mysql.com/
	
	# rpm -Uvh MySQL-shared-compat-5.6.14-1.el6.x86_64.rpm
	
Mysql 5.6 的 root 用户现在有一个默认密码放在 `/root/.mysql_secret`，记得把密码改一下

	# service mysql start
	# chkconfig mysql on
	# cat /root/.mysql_secret
	
	使用打印出来的密码进行登录
	
	# mysql -u root -p
	# SET PASSWORD FOR 'root'@'localhost' = PASSWORD('yourpassword');

针对 gitlab 配置 Mysql，添加用户和数据库

	# mysql> CREATE USER 'gitlab'@'localhost' IDENTIFIED BY 'passwordForGitlab';
	# mysql> CREATE DATABASE IF NOT EXISTS `gitlabhq_production` DEFAULT CHARACTER SET `utf8` COLLATE `utf8_unicode_ci`;
	# mysql> GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER ON `gitlabhq_production`.* TO 'gitlab'@'localhost';
	# mysql> exit

#### 8）安装 Redis

Gitlab 使用 redis 处理一些数据，这里就不追求太新的了。

	# yum -y install redis
	# service redis start
	# chkconfig redis on
	
#### 9）添加 git 用户

	# useradd -c 'GitLab' git

CentOS 的命令没有办法直接禁止用户的访问的参数，需要用下面命令：

	# passwd -l git

#### 10）安装 Gitlab-shell

从 root 账户切换到 git 账户下操作，可以省去一些麻烦的输入

	$ su git 
	$ cd /home/git
	$ git clone https://github.com/gitlabhq/gitlab-shell.git
	$ cd gitlab-shell

通过 `git tag` 查看最新版本并切换之

	$ git checkout v1.7.1

编辑配置文件修改你要设定的域名（domain），比如 `http://gitlab.dev/`

	$ cp config.yml.example config.yml
	$ vim config.yml

完成之后执行安装脚本

	$ ./bin/install

#### 11）安装 Gitlab

	$ cd /home/git
	$ git clone https://github.com/gitlabhq/gitlabhq.git gitlab
	$ cd gitlab

通过 `git tag` 查看最新版本并切换之

	$ git checkout v6.1.0

这里需要配置的东西多一些，这里参考[官方的文档](https://github.com/gitlabhq/gitlabhq/blob/master/doc/install/installation.md#configure-it, "官方的文档")，也可以安装我下面的步骤来，复制配置文件，修改 host 相关的配置项，主要是 domain 要和上面的 `http://gitlab.dev` 一致。

	$ cp config/gitlab.yml.example config/gitlab.yml
	$ vim config/gitlab.yml
	
确认 gitlab 以下目录的权限是否正确

	$ mkdir tmp/pids/
	$ mkdir tmp/sockets/
	$ mkdir public/uploads
	$ chown -R git log/
	$ chown -R git tmp/
	$ chmod -R u+rwX  log/
	$ chmod -R u+rwX  tmp/
	$ chmod -R u+rwX  tmp/pids/
	$ chmod -R u+rwX  tmp/sockets/
	$ chmod -R u+rwX  public/uploads/
	
创建 satellites 目录，这个目录是保存各个用户的仓库

	$ mkdir /home/git/gitlab-satellites

Gitlab 是 Rails 程序，Rails 下可选择的 web 服务器非常多，gitlab 选择了 unicorn，复制 unicorn 配置文件

	$ cp config/unicorn.rb.example config/unicorn.rb

设置 ruby web 容器的参数，比如 2GB RAM 服务器可以设置 3 个 worker。unicorn 默认使用 8080 端口，而 Tomcat 也是默认这个端口，我们把 unicorn 的端口修改成 7070。
	
	$ vim config/unicorn.rb
	
配置 gitlab 数据库设置，主要是修改数据库用户名及密码

	$ cp config/database.yml.mysql config/database.yml
	$ vim config/database.yml
	$ chmod o-rwx config/database.yml

设置一些 git 全局参数

	$ exit
	# sudo -u git -H git config --global user.name "GitLab"
	# sudo -u git -H git config --global user.email "gitlab@localhost"
	
安装必需的 Ruby Gems

	$ exit
	# gem install charlock_holmes --version '0.6.9.4'
	# su git
	$ cd /home/git/gitlab
	$ bundle install --deployment --without development test postgres aws
	
初始化数据库数据（执行输入 `Yes` 继续创建）

	$ bundle exec rake gitlab:setup RAILS_ENV=production

设置 gitlab 的 init 脚本

	$ exit
	# cd /home/git/gitlab
	# cp lib/support/init.d/gitlab /etc/init.d/gitlab
	# chmod +x /etc/init.d/gitlab
	
检查 Gitlab 状态
	
	# su git
	$ cd /home/git/gitlab
	$ bundle exec rake gitlab:env:info RAILS_ENV=production

启动 gitlab 服务

	$ exit
	# service gitlab start

再起检查，保证所有项目都是绿色

	# su git
	$ cd /home/git/gitlab
	$ bundle exec rake gitlab:check RAILS_ENV=production
	
#### 12）配置 nginx

根据 nginx 的安装路径适当修改下面的路径即可，我们先把 gitlab 提供的配置文件拷贝过去，修改 `gitlab.conf` 的 `YOUR_SERVER_FQDN` 为上面设置的 domain，就是 `gitlab.dev`。 

	# exit
	# cd /home/git/gitlab
	# mv /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf.bak
	# cp lib/support/nginx/gitlab /etc/nginx/conf.d/gitlab.conf
	# vim /etc/nginx/conf.d/gitlab.conf

把 nginx 用户加入到 git 用户组

	# usermod -a -G git nginx
	# chmod g+rx /home/git

重启各个服务

	# service mysql restart
	# service redis restart
	# service gitlab restart
	# service nginx restart
	
#### 13）开始 Gitlab 之旅

在你本机配置好 hosts 即可访问 `gitlab.dev`，注意：服务器在特定的网络环境下，也需要配置 hosts。

	# echo "服务器ip gitlab.dev" >> /etc/hosts

默认的用户名、密码：

	admin@local.host
	5iveL!fe

## 安装 Redmine

#### 1）下载 Redmine

我们继续使用 git 用户安装 Redmine，我们下载最新版 2.3.3 版

	$ su git
	$ cd /home/git
	$ git clone https://github.com/redmine/redmine
	$ cd redmine

#### 2）配置数据库

新建用户及数据库

	$ CREATE DATABASE redmine CHARACTER SET utf8;
	$ CREATE USER 'redmine'@'localhost' IDENTIFIED BY 'my_password';
	$ GRANT ALL PRIVILEGES ON redmine.* TO 'redmine'@'localhost';	

配置数据库

	$ cp config/database.yml.example config/database.yml
	$ vim config/database.yml
	
	如下所示，主要把 username 和 password 改一下
	
	production:
	adapter: mysql
	database: redmine
	host: localhost
	username: redmine
	password: my_password
	
#### 3）安装 gem

	$ gem install bundler

在安装 gem 之前，先安装 ImageMagick

	# exit
	# yum install libjpeg-devel libpng-devel
	# yum install ImageMagick ImageMagick-devel
	
然后

	# cd /home/git/redmine
	# bundle install --without development test

#### 4）一些初始化操作

	$ su git
	$ cd /home/git/redmine
	$ rake generate_secret_token
	$ RAILS_ENV=production rake db:migrate
	$ RAILS_ENV=production rake redmine:load_default_data
	
	$ mkdir -p tmp tmp/pdf tmp/sockets public/plugin_assets
	$ chown -R git log/
	$ chown -R git tmp/
	$ chmod -R u+rwX  log/
	$ chmod -R u+rwX  tmp/
	$ chmod -R u+rwX  tmp/pdf/
	$ chmod -R u+rwX  tmp/sockets/
	$ chmod -R u+rwX  public/plugin_assets/
	
#### 5）测试验证是否安装成功

	$ ruby script/rails server webrick -e production
	
	访问地址是：http://ip:3000

#### 6）配置 unicorn

首先在 Gemfile 中添加 unicorn:

	$ cd /home/git/redmine
	$ vim Gemfile
	
加上
	
	gem "unicorn"
	
unicorn 配置。官方 Example 配置文件在 [github unicorn](https://github.com/defunkt/unicorn/blob/master/examples/unicorn.conf.rb)，下面这个配置文件我稍做了点修改。

	$ cd config
	$ wget https://raw.github.com/roamlog/goods/master/unicorn.rb
	
注意修改端口，这次修改为 `9090`，另外增加启动文件，该启动文件是为 redmine 定制的。
	
	$ exit
	# cd /etc/init.d/
	# wget https://raw.github.com/roamlog/goods/master/redmine
	# chmod +x redmine
	
	
#### 6）配置 nginx

下载配置文件，放到指定目录

	# cd /etc/nginx/conf.d/
	# wget https://raw.github.com/roamlog/goods/master/redmine.conf

## 大功告成

重新启动相关所有服务

	# service redmine start
	# service gitlab restart
	# service nginx restart

修改本机的 hosts 文件，加上。（注意：服务器在一些特定的网络配置情况下，也需要配置 hosts。
）：
	
	服务器ip redmine.dev
		
访问：

	http://gitlab.dev
	http://redmine.dev

## jdk 安装

jdk 安装比较简单，去 oracle 官网下载指定的 rpm 包，然后安装即可。

	# wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com" "http://download.oracle.com/otn-pub/java/jdk/7u45-b18/jdk-7u45-linux-x64.rpm"
	# rpm -Uvh jdk-7u45-linux-x64.rpm
	
当然还是要配置一下环境变量
	
	# vim /etc/profile
	
	加入
	
	export JAVA_HOME=/usr/java/jdk1.7.0_45
	export PATH=$PATH:$JAVA_HOME/bin

## Tomcat 安装

去 apache 下载 tomcat, 解压即可




	
