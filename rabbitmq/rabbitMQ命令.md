# rabbitMQ命令

关闭 
rabbitmqctl stop_app

重置 
rabbitmqctl reset

关闭rabbitmq但是不关闭节点 
rabbitmqctl stop_app

该方式启动后终端关闭服务也会关闭，所以不推荐用这种方式启动，

brew services start rabbitmq

rabbitmq-server -detached  启动RabbitMQ节点
rabbitmqctl start_app 启动RabbitMQ应用，而不是节点
rabbitmqctl stop_app  停止
rabbitmqctl status  查看状态
rabbitmqctl add_user mq 123456
rabbitmqctl set_user_tags mq administrator 新增账户
rabbitmq-plugins enable rabbitmq_management  启用RabbitMQ_Management
rabbitmqctl cluster_status 集群状态

rabbitmqctl forget_cluster_node rabbit@rabbit3 节点摘除

rabbitmqctl reset application重置 



# Windows下内建RabbitMQ

1. 添加RABBITMQ_HOME环境变量，值为D:\RabbitMQ Server
2. 修改Hosts，增加虚拟主机（127.0.0.1 node1 ...）
3. sbin目录下copy

~~~ java
copy rabbitmq-server.bat rabbitmq-server-node1.bat
copy rabbitmqctl.bat rabbitmqctl-node1.bat
copy rabbitmq-env.bat rabbitmq-env-node1.bat
copy rabbitmq-plugins.bat rabbitmq-plugins-node1.bat
~~~

4. 在etc下copy

~~~ copy rabbitmq.config.example rabbitmq-node1.config
copy rabbitmq.config.example rabbitmq-node1.config
~~~

5. 修改文件，修改rabbitmq-env-node1.bat，大约16行

~~~ java
set RABBITMQ_CONFIG_FILE=!RABBITMQ_HOME!\etc\rabbitmq-ClusterNode1
set RABBITMQ_BASE=!RABBITMQ_BASE!\rabbitmq-cluster
set RABBITMQ_NODENAME=rabbit1@ClusterNode1
set RABBITMQ_NODE_PORT=5673
set RABBITMQ_DIST_PORT=15673
~~~

端口号，名字要不一样

6. 脚本修改

~~~ java
rabbitmqctl-ClusterNode1.bat　　脚本文件修改：
call "!TDP0!\rabbitmq-env.bat" %~n0　　-》　　call "!TDP0!\rabbitmq-env-ClusterNode1.bat" %~n0

rabbitmq-plugins-ClusterNode1.bat　　脚本文件修改：
call "!TDP0!\rabbitmq-env.bat" %~n0　　-》　　call "!TDP0!\rabbitmq-env-ClusterNode1.bat" %~n0

rabbitmq-server-ClusterNode1.bat　　脚本文件修改：
call "!TDP0!\rabbitmq-env.bat" %~n0　　-》　　call "!TDP0!\rabbitmq-env-ClusterNode1.bat" %~n0
~~~

7. 配置文件rabbitmq-ClusterNode1.config　　配置文件修改，添加内容

![1550472756807](C:\Users\19970326\AppData\Roaming\Typora\typora-user-images\1550472756807.png)

这里的目的是保证所有的节点使用一个Web控制台

8. 启动插件 **rabbitmq-plugins-ClusterNode1 enable rabbitmq_management**

9. 启动节点，执行相关命令

~~~java
rabbitmq-server-ClusterNode1 -detached
~~~

10. 组成集群

~~~
rabbitmqctl-ClusterNode1 stop_app
rabbitmqctl-ClusterNode1 join_cluster rabbit@WK --ram
--ram 表示添加为内存节点
rabbitmqctl-ClusterNode1 start_app
~~~

11. 查看集群状态

~~~
rabbitmqctl cluster_status
~~~



# 分布式机器上的集群

1. **安装Erlang和RabbitMQ**

**Erlang的安装**

yum install epel-release
rpm -ivh https://download1.rpmfusion.org/free/el/updates/6/i386/rpmfusion-free-release-6-1.noarch.rpm

sudo yum install erlang

erl -v

**RabbitMQ的安装(3.7.4版本)**

rpm --import https://dl.bintray.com/rabbitmq/Keys/rabbitmq-release-signing-key.asc

rpm -ivh https://bintray.com/rabbitmq/rabbitmq-server-rpm/download_file?file_path=rabbitmq-server-3.7.4-1.el7.noarch.rpm 

yum install socat

- **启动RabbitMQ**

rabbitmq-server -detached  或者 rabbitmq-server start

- **RabbitMQ-Management安装**

rabbitmq-plugins enable rabbitmq_management

**集群**

- **群节点(对等)通信---erlang Cookie**

erlang分布式的每个节点上要保持相同的.erlang.cookie文件，文件路径：/var/lib/rabbitmq/.erlang.cookie

- **将rabbit2加入到rabbit1（RAM节点，默认Disk节点）**

rabbitmqctl stop_app
rabbitmqctl join_cluster rabbit@rabbit1 <span style="background-color:rgb(255,204,0);">--ram</span>

rabbitmqctl start_app

- **将rabbit3加入到rabbit1(Disk节点)**

rabbitmqctl stop_app
rabbitmqctl join_cluster rabbit@rabbit1
rabbitmqctl start_app

**PS:若希望修改节点类型，则(需要先Stop)**

rabbitmqctl stop_app
rabbitmqctl change_cluster_node_type ram
rabbitmqctl start_app