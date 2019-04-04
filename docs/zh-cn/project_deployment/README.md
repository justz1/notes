# 项目部署笔记

## Anaconda 3-5.2.0 安装及配置

> 安装 tmux

```
yum install tmux
```

> CentOS 安装系统依赖包

```
yum -y install openssl-devel zlib-devel bzip2-devel sqlite-devel readline-devel libffi-devel systemtap-sdt-devel
```

> 下载 Anaconda3-5.2.0 

```
wget https://repo.continuum.io/archive/Anaconda3-5.2.0-Linux-x86_64.sh
```

> 也可以本地下载完毕后上传至CentOS(这里需要先执行下面的安装scp)

```
scp Anaconda3-5.2.0-Linux-x86_64.sh root@39.97.40.67:/projects
```

> CentOS 下先安装 scp

```
yum install openssh-clients
```

> 为防止报错, 还需安装bzip2

```
yum install -y bzip2
```

> 编译 Anaconda3-5.2.0

```
sh Anaconda3-5.2.0-Linux-x86_64.sh
```

```
一路 yes + 回车 即可 最后一步不用安装VC包(注:中间有一步 是否将 anaconda 添加到PAHT中, 记得选择yes)
安装完之后需要 source /root/.bashrc ,否则添加的新的 PATH 不会生效
```

```
之后 输入 conda --version 与 python 假如未成功 输入 source /root/.bashrc 刷新环境变量配置即可
```

## 虚拟环境和安装和依赖包安装

> 安装虚拟环境

```
yum install python-setuptools python-devel
pip install virtualenvwrapper
```

> 编辑 .bashrc 文件

```
先通过命令 sudo find / -name virtualenvwrapper.sh
找到virtualenvwrapper.sh 文件所在位置,然后
vim ~/.bashrc
```

```
后在文件结尾输入：
export WORKON_HOME=$HOME/.virtualenvs
source /root/anaconda3/bin/virtualenvwrapper.sh
```

```
最后退出vim后 source ~/.bashrc
```

> 新建虚拟环境

```
mkvirtualenv -p python365 mxforum
```

> 安装依赖包(先编辑 requirements.txt 文件 内容如下)

```
aiomysql==0.0.20
amqp==1.4.9
bcrypt==3.1.6
celery==3.1.18
Flask==1.0.2
Flask-Cors==3.0.4
flower==0.9.3
numpy==1.14.3
pandas==0.23.0
peewee==3.8.2
pika==0.10.0
PyJWT==1.7.1
PyMySQL==0.9.2
redis==2.10.6
requests==2.18.4
tensorflow==1.12.0
tornado==5.0.2
tornado-celery==0.3.5
tornado-redis==2.4.18
```

```
进入项目目录后
pip install -r requirements.txt
```

## mysql 的安装和配置

> 安装 mariadb

```
sudo yum install mariadb-server
```

> 启动和重启

```
udo systemctl start mariadb
sudo systemctl restart mariadb
```

> 查看 mariadb 是否启动

```
ps aux|grep mariadb
```

> 设置bind-ip

```
vim /etc/my.cnf
在 [mysqld]:
下面添加一行:
bind-address = 0.0.0.0
```

> 重启 mariadb

```
sudo systemctl restart mariadb
```

> 设置外部ip可以访问

```
进入 mysql 的命令 mysql -uroot -p 
如果提示输入密码则直接回车(初次进入没有密码)
```

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'db123' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' IDENTIFIED BY 'db123' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

> 设置阿里云的对外端口

> 通过navicat直接远程传输数据和新建数据库


## nginx 和 redis 的安装和配置

> 安装和启动 redis

```
yum install redis

sudo systemctl start redis
```

> 安装和启动 nginx

```
sudo yum install epel-release
sudo yum install nginx

sudo systemctl start nginx
```

> 拷贝 nginx 的配置文件到 nginx 目录下

```
cp MxForum/mxforum_nginx.conf /etc/nginx/conf.d/
应用相应配置
sudo systemctl restart nginx
ps aux|grep nginx
```

> 修改nginx的启动用户为root并重启nginx

```
vim /etc/nginx/nginx.conf
```

## Erlang、RabbitMQ 和 celery 的安装和配置

```
附参考链接:https://blog.csdn.net/yin767833376/article/details/81223491#commentBox
```

### 编译安装方式(安装 Erlang)(安装 RabbitMQ 前需要先安装好 Erlang)

> 依赖环境的安装-如果需要用编译安装erlang语言环境，需要安装C++编译。

```
yum -y install make gcc gcc-c++ kernel-devel m4 ncurses-devel openssl-devel unixODBC unixODBC-devel httpd python-simplejson
```

> erlang语言环境的安装(rabbitMq是用erlang分布式语言开发的)

```
安装文件获取
wget http://erlang.org/download/otp_src_19.2.tar.gz
解压erlang安装包
tar -xzvf otp_src_19.2.tar.gz
进入erlang目录
cd otp_src_19.2
编译安装erlang语言环境 prefix=/usr/local/erlang 为安装目录
./configure  --prefix=/usr/local/erlang --enable-smp-support  --enable-threads  --enable-sctp --enable-kernel-poll --enable-hipe  --with-ssl --without-javac
```

```
注:
erlang语言编译配置选项：
–prefix 指定安装目录 
–enable-smp-support启用对称多处理支持（Symmetric Multi-Processing对称多处理结构的简称）
–enable-threads启用异步线程支持
–enable-sctp启用流控制协议支持（Stream Control Transmission Protocol，流控制传输协议）
–enable-kernel-poll启用Linux内核poll
–enable-hipe启用高性能Erlang –with-ssl 启用ssl包 –without-javac 不用java编译
```

> 开始安装编译

```
make && make install
```

> 配置erlang环境变量

```
vim /etc/profile

export PATH=$PATH:/usr/local/erlang/bin

source /etc/profile
```

> 测试erlang安装是否成功(会输出 erlang安装版本号)

```
erl
```

### 下载安装 RabbitMQ

> 下载安装

```
cd /usr/local  //切换到计划安装RabbitMQ的目录，我这里放在/usr/local
wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.1/rabbitmq-server-generic-unix-3.6.1.tar.xz  //下载RabbitMQ安装包
xz -d rabbitmq-server-generic-unix-3.6.1.tar.xz
tar -xvf rabbitmq-server-generic-unix-3.6.1.tar
```

> 解压后多了个文件夹rabbitmq-server-3.6.1 ，重命名为rabbitmq以便记忆。

```
mv rabbitmq_server-3.6.1/ rabbitmq
```

> 配置rabbitmq环境变量

```
vim /etc/profile

#set rabbitmq environment
export PATH=$PATH:/usr/local/rabbitmq/sbin

source /etc/profile
```

> 启动服务

```
rabbitmq-server -detached //启动rabbitmq，-detached代表后台守护进程方式启动。
```

> 查看状态(查看是否启动成功)

```
rabbitmqctl status
```

> 其他相关命令

```
启动服务：rabbitmq-server -detached【 /usr/local/rabbitmq/sbin/rabbitmq-server  -detached 】
查看状态：rabbitmqctl status【 /usr/local/rabbitmq/sbin/rabbitmqctl status  】
关闭服务：rabbitmqctl stop【 /usr/local/rabbitmq/sbin/rabbitmqctl stop  】
列出角色：rabbitmqctl list_users
```

### 安装 celery

> 安装 celery

```
pip install celery
```

> 运行 celery (运行相应 task 文件)(在 task.py 同级文件夹下)

```
celery -A task worker --loglevel=info
```

> 解决报错:celery 不能用root用户启动问题 C_FORCE_ROOT environment

> 如果使用root用户启动celery会遇到下面的问题

```
Running a worker with superuser privileges when the worker accepts messages serialized with pickle is a very bad idea!
If you really want to continue then you have to set the C_FORCE_ROOT environment variable (but please think about this before you do).
```

> 解决方法

```
task.py 文件上导入包中增加这行

from celery import platforms
platforms.C_FORCE_ROOT = True  #加上这一行
```
















