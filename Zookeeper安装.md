# Zookeeper集群环境安装过程详解
>`Zookeeper`是一个分布式开源框架，提供了协调分布式应用的基本服务，它向外部应用暴露一组通用服务——分布式同步（Distributed Synchronization）、命名服务（Naming Service）、集群维护（Group Maintenance）等，简化分布式应用协调及其管理的难度，提供高性能的分布式服务。`ZooKeeper`本身可以以单机模式安装运行，不过它的长处在于通过分布式`ZooKeeper`集群（一个`Leader`，多个`Follower`），基于一定的策略来保证`ZooKeeper`集群的稳定性和可用性，从而实现分布式应用的可靠性。

## Zookeeper安装和配置
>`Zookeeper`有三种不同的运行环境，包括：单机环境、集群环境和集群伪分布式环境。
## 环境准备
>配置`java`运行环境
>修改`zookeeper`主机的主机名
>修改`hosts`文件，做静态解析
```sql
10.195.16.10	zk01.kmbear.com
10.195.16.11	zk02.kmbear.com
10.195.16.12	zk03.kmbear.com

```
### 1.Zookeeper的下载与解压
```sql
#zookeeper下载
下载地址：http://www.apache.org/dyn/closer.cgi/zookeeper
```
### 2.下载后解压至安装目录下，本文我们解压到目录：/data/zookeeper/
```sql
#解压
tar -xzvf zookeeper-3.4.10.tar.gz
```
### 3.zookeeper的环境变量的配置：
>在/etc/profile文件中加入如下的内容：
```sql
#set zookeeper environment

export ZOOKEEPER_HOME=/data/zookeeper/zookeeper
export PATH=$PATH:$ZOOKEEPER_HOME/bin:$ZOOKEEPER_HOME/conf
```

### 4.集群部署：
>在`Zookeeper`集群环境下只要一半以上的机器正常启动了，那么`Zookeeper`服务将是可用的。因此，集群上部署`Zookeeper`最好使用奇数台机器，这样如果有5台机器，只要3台正常工作则服务将正常使用。

下面我们将对`Zookeeper`的配置文件的参数进行设置：
>进入`zookeeper-3.4.5/conf`：
```sql
cp zoo_sample.cfg zoo.cfg
vim zoo.cfg
```
>可参考下面的配置
```sql
[root@zk01 conf]# cat zoo.cfg 
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/data/zookeeper/data/
clientPort=2181
server.1=zk01.kmbear.com:2222:2225
server.2=zk02.kmbear.com:2222:2225  
server.3=zk03.kmbear.com:2222:2225
```
>配置文件中`server.id=host:port:port`中的第一个port是从机器（follower）连接到主机器（leader）的端口号，第二个port是进行leadership选举的端口号。
>接下来在`dataDir`所指定的目录下创建一个文件名为`myid`的文件，文件中的内容只有一行，为本主机对应的id值，也就是server.id中的id。例如：在服务器1中的`myid`的内容应该写入`1`。
>id 被称为 Server ID, 用来标识服务器在集群中的序号。同时每台 ZooKeeper 服务器上, 都需要在数据目录(即 dataDir 指定的目录) 下创建一个 myid 文件, 该文件只有一行内容, 即对应于每台服务器的Server ID。
>`ZooKeeper` 集群中, 每台服务器上的 zoo.cfg 配置文件内容一致。
>`server.1` 的 `myid` 文件内容就是 `1`。每个服务器的 `myid` 内容都不同, 且需要保证和自己的 `zoo.cfg` 配置文件中 `server.id=host:port:port`的 id 值一致。

id 的范围是 1 ~ 255。
### 5.远程复制分发安装文件
>将zk01主机的`zookeeper`复制到另外两台主机中。
>*主机修改`datadir`目录下`myid`的`id`。
>集群模式中, 集群中的每台机器都需要感知其它机器, 在 zoo.cfg 配置文件中, 可以按照如下格式进行配置, 每一行代表一台服务器配置。

```sql
server.id=host:port:port
```
>server.1 的 myid 文件内容就是 "1"。每个服务器的 myid 内容都不同, 且需要保证和自己的 zoo.cfg 配置文件中 "server.id=host:port:port" 的 id 值一致。
### 6.启动ZooKeeper集群
```sql
[root@zk01 bin]# ./zkServer.sh  start
ZooKeeper JMX enabled by default
Using config: /data/zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```
### 7.检查zookeeper启动是否成功
```sql
[root@zk01 conf]# jps
1155 Jps
1093 QuorumPeerMain
```
>其中，`QuorumPeerMain`是`zookeeper`进程，启动正常。
如上依次启动了所有机器上的`Zookeeper`之后可以通过`ZooKeeper`的脚本来查看启动状态，包括集群中各个结点的角色（或是`Leader`，或是`Follower`），如下所示，是在`ZooKeeper`集群中的每个结点上查询的结果：
```sql
[root@zk02 bin]# ./zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /data/zookeeper/bin/../conf/zoo.cfg
Mode: leader
```
```sql
[root@zk01 bin]# ./zkServer.sh  status
ZooKeeper JMX enabled by default
Using config: /data/zookeeper/bin/../conf/zoo.cfg
Mode: follower
```
>通过上面状态查询结果可见，`zk02`是集群的`Leader`，其余的两个结点是`Follower`。
>另外，可以通过客户端脚本，连接到`ZooKeeper`集群上。对于客户端来说，`ZooKeeper`是一个整体（ensemble），连接到`ZooKeeper`集群实际上感觉在独享整个集群的服务，所以，你可以在任何一个结点上建立到服务集群的连接，例如：
```sql
[root@zk02 bin]# ./zkCli.sh -server zk01.kmbear.com
Connecting to zk01.kmbear.com
2017-05-09 17:04:04,309 [myid:] - INFO  [main:Environment@100] - Client environment:zookeeper.version=3.4.10-39d3a4f269333c922ed3db283be479f9deacaa0f, built on 03/23/2017 10:13 GMT
2017-05-09 17:04:04,312 [myid:] - INFO  [main:Environment@100] - Client environment:host.name=zk02.kmbear.com
2017-05-09 17:04:04,312 [myid:] - INFO  [main:Environment@100] - Client environment:java.version=1.8.0_121
2017-05-09 17:04:04,314 [myid:] - INFO  [main:Environment@100] - Client environment:java.vendor=Oracle Corporation
2017-05-09 17:04:04,314 [myid:] - INFO  [main:Environment@100] - Client environment:java.home=/usr/java/jre
2017-05-09 17:04:04,314 [myid:] - INFO  [main:Environment@100] - Client environment:java.class.path=/data/zookeeper/bin/../build/classes:/data/zookeeper/bin/../build/lib/*.jar:/data/zookeeper/bin/../lib/slf4j-log4j12-1.6.1.jar:/data/zookeeper/bin/../lib/slf4j-api-1.6.1.jar:/data/zookeeper/bin/../lib/netty-3.10.5.Final.jar:/data/zookeeper/bin/../lib/log4j-1.2.16.jar:/data/zookeeper/bin/../lib/jline-0.9.94.jar:/data/zookeeper/bin/../zookeeper-3.4.10.jar:/data/zookeeper/bin/../src/java/lib/*.jar:/data/zookeeper/bin/../conf:/usr/java/lib/tools.jar:/usr/java/lib/dt.jar
2017-05-09 17:04:04,314 [myid:] - INFO  [main:Environment@100] - Client environment:java.library.path=/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
2017-05-09 17:04:04,314 [myid:] - INFO  [main:Environment@100] - Client environment:java.io.tmpdir=/tmp
2017-05-09 17:04:04,314 [myid:] - INFO  [main:Environment@100] - Client environment:java.compiler=<NA>
2017-05-09 17:04:04,314 [myid:] - INFO  [main:Environment@100] - Client environment:os.name=Linux
2017-05-09 17:04:04,315 [myid:] - INFO  [main:Environment@100] - Client environment:os.arch=amd64
2017-05-09 17:04:04,315 [myid:] - INFO  [main:Environment@100] - Client environment:os.version=3.10.0-514.16.1.el7.x86_64
2017-05-09 17:04:04,315 [myid:] - INFO  [main:Environment@100] - Client environment:user.name=root
2017-05-09 17:04:04,315 [myid:] - INFO  [main:Environment@100] - Client environment:user.home=/root
2017-05-09 17:04:04,315 [myid:] - INFO  [main:Environment@100] - Client environment:user.dir=/data/zookeeper/bin
2017-05-09 17:04:04,316 [myid:] - INFO  [main:ZooKeeper@438] - Initiating client connection, connectString=zk01.kmbear.com sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@799f7e29
Welcome to ZooKeeper!
2017-05-09 17:04:04,356 [myid:] - INFO  [main-SendThread(zk01.kmbear.com:2181):ClientCnxn$SendThread@1032] - Opening socket connection to server zk01.kmbear.com/10.195.16.10:2181. Will not attempt to authenticate using SASL (unknown error)
JLine support is enabled
2017-05-09 17:04:04,433 [myid:] - INFO  [main-SendThread(zk01.kmbear.com:2181):ClientCnxn$SendThread@876] - Socket connection established to zk01.kmbear.com/10.195.16.10:2181, initiating session
2017-05-09 17:04:04,464 [myid:] - INFO  [main-SendThread(zk01.kmbear.com:2181):ClientCnxn$SendThread@1299] - Session establishment complete on server zk01.kmbear.com/10.195.16.10:2181, sessionid = 0x15bec5fe4490000, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
[zk: zk01.kmbear.com(CONNECTED) 0] 
```
>至此，`Zookeeper`集群安装大功告成！