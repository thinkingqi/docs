# Hadoop分布式环境搭建
### 环境安装介质
```bash
系统： CentOS 6.8
hadoop版本：   hadoop-2.8.0.tar
JDK版本：  1.8.0_121
```
### 环境介绍：
```bash
主服务器ip：10.195.13.134(master)  NameNode  SecondaryNameNode ResourceManager
从服务器ip：10.195.13.135(slave1)  DataNode NodeManager
从服务器ip: 10.195.13.136(slave2)  DataNode NodeManager
```
### 配置服务器域名
```bash
# vim /etc/hosts
10.195.13.134 master
10.195.13.135 slave1
10.195.13.136 slave2
```
### 修改主机名
```
#master
#vim /etc/sysconfig/network
HOSTNAME=master

#slave1
#vim /etc/sysconfig/network
HOSTNAME=slave1

#slave2
#vim /etc/sysconfig/network
HOSTNAME=slave2
```
### jdk环境变量配置
```bash
#jdk下载
wget http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.tar.gz
#解压
mkdir -p /usr/java
tar zxvf jdk-8u131-linux-x64.tar.gz 
mv  jdk1.8.0_121 /usr/java
#设置环境变量
vim  /etc/profile    追加：
#set java JDK 
export JAVA_HOME=/usr/java
export JRE_HOME=/usr/java/jre
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
export CLASSPATH=$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar
#环境变量生效
source /etc/profile
```
### jdk环境变量验证
```bash
[root@localhost ~]# java -version
java version "1.8.0_121"
Java(TM) SE Runtime Environment (build 1.8.0_121-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.121-b13, mixed mode)
```
### hadoop 文件下载
```bash
wget http://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-2.8.0/hadoop-2.8.0.tar.gz
```
## 配置SSH免密钥登陆
**1.修改ssh配置**
```bash
#vim /etc/ssh/sshd_config
#取消下面四条的注释
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile      .ssh/authorized_keys
UseDNS no
```
**2.修改配置文件需要重启sshd服务**
```bash
#CentOS 6系列重启
/etc/init.d/sshd restart
#CentOS 7系列重启
systemctl restart sshd.service
```
**3.生成公钥和私钥（每台主机都要生成）**
```bash
[root@localhost ~]# ssh-keygen -t rsa

Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
d0:72:e6:55:c0:de:67:33:ce:cc:80:58:5e:4a:db:45 root@localhost
The key's randomart image is:
+--[ RSA 2048]----+
|         .....E  |
|       .  +.. .  |
|      o +*.B .   |
|       *..* + =  |
|        S    O o |
|              =  |
|                 |
|                 |
|                 |
+-----------------+
```
>默认在~/.ssh目录下生产两个文件：
id_rsa:私钥
id_rsa.pub:公钥

**4.与所有的服务器做免密钥登陆(所有主机都要互相免密钥登陆)**
```bash
[root@localhost ~]# ssh-copy-id 10.195.13.135
Address 10.195.13.135 maps to localhost, but this does not map back to the address - POSSIBLE BREAK-IN ATTEMPT!
root@10.195.13.135's password: 
Now try logging into the machine, with "ssh '10.195.13.135'", and check in:

  .ssh/authorized_keys

to make sure we haven't added extra keys that you weren't expecting.
```
**安装以上操作，重复安装其余两台主机。**
## Hadoop 安装
**解压hadoop安装包**
```bash
tar zxvf hadoop-2.8.0.tar.gz
```
**配置环境变量**
```bash
# vim /etc/profile
export HADOOP_HOME=/data/hadoop-2.8.0
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```
**修改hadoop-env.sh文件**
```bash
cd /data/hadoop-2.8.0/etc/hadoop
#vim hadoop-env.sh
export JAVA_HOME=/home/hadoop/jdk1.7.0_71
```
**配置etc/hadoop/slaves文件，增加slave主机名**
```bash
#vim etc/hadoop/slaves
salve1
salve2
```
**配置etc/hadoop/core-site.xml**
```bash
<configuration>
<property>
　　<name>fs.defaultFS</name>
　　<value>hdfs://master:9000</value>
</property>
<!-- Size of read/write buffer used in SequenceFiles. -->
<property>
　　<name>io.file.buffer.size</name>
　　<value>131072</value>
</property>
<!-- 指定hadoop临时目录,自行创建 -->
<property>
　　<name>hadoop.tmp.dir</name>
　　<value>/data/hadoop/tmp</value>
</property>
</configuration>
```
**配置etc/hadoop/hdfs-site.xml**
```bash
<configuration>
<property>
　　<name>dfs.namenode.secondary.http-address</name>
　　<value>master:50090</value>
</property>
<property>
　　<name>dfs.replication</name>
      <value>2</value>
</property>
<!-- 指定namenode数据存放临时目录,自行创建 -->
<property>
　　<name>dfs.namenode.name.dir</name>
　　<value>file:/data/hadoop/hdfs/name</value>
</property>
<!-- 指定datanode数据存放临时目录,自行创建 -->
<property>
　　<name>dfs.datanode.data.dir</name>
　　<value>file:/data/hadoop/hdfs/data</value>
</property>
</configuration>
```
**配置etc/hadoop/yarn-site.xml**
```bash
<configuration>
<property>
　　<name>yarn.nodemanager.aux-services</name>
　　<value>mapreduce_shuffle</value>
</property>
<property>
　　<name>yarn.resourcemanager.address</name>
　　<value>master:8032</value>
</property>
<property>
<name>yarn.resourcemanager.scheduler.address</name>
       <value>master:8030</value>
</property>
<property>
　　<name>yarn.resourcemanager.resource-tracker.address</name>
　　<value>master:8031</value>
</property>     
<property>         
　　<name>yarn.resourcemanager.admin.address</name>
　　<value>master:8033</value>
</property>
<property>
　　<name>yarn.resourcemanager.webapp.address</name>
　　<value>master:8088</value>
</property>
</configuration>
```
**配置etc/hadoop/mapred-site.xml**
>注意： 因为默认没有mapred-site.xml，所以先要复制一份，shell命令如下：
>
`cp mapred-site.xml.template mapred-site.xml`
```bash
<configuration>
<property>
　　<name>mapreduce.framework.name</name>
　　<value>yarn</value>
</property>
</configuration>
```
**创建所需的目录**
```bash
mkdir -p /data/hadoop/hdfs/data
mkdir -p file:/data/hadoop/hdfs/name
mkdir -p /data/hadoop/hdfs/name
mkdir -p /data/hadoop/tmp
```
**将修改好的文件和目录推送到其余的主机**
```bash
rsync -acP * 10.195.13.134:/data/
```
**格式化节点**
```bash
cd hadoop-2.8.0/sbin
hdfs namenode -format
```
**hadoop集群全部启动**
```bash
cd hadoop-2.8.0/sbin
./start-all.sh
```
**启动JobHistoryServer  备注(查看MapReduce历史执行记录，和hadoop关系不大，可忽略此步骤)**
```bash
cd hadoop-2.8.0/sbin
./mr-jobhistory-daemon.sh   start historyserver
```
**查看启动进程是否正常**
>在master节点输入 jps命令，将会显示以下进程：
```bash
[root@master sbin]# jps
2288 ResourceManager
1882 SecondaryNameNode
3452 JobHistoryServer
3516 Jps
```
>在slave1、slave2上输入jps命名，将会显示以下进程：
```bash
[root@slave1 hadoop]# jps
1616 DataNode
1883 Jps
1708 NodeManager
```
```bash
[root@slave2 sbin]# jps
2245 NodeManager
2153 DataNode
2367 Jps
```
**通过web UI访问hadoop**
>http://10.195.13.134:50070  #整个集群
http://10.195.13.134:50090  #SecondaryNameNode的情况
http://10.195.13.134:8088   #resourcemanager的情况
http://10.195.13.134:19888  #historyserver(MapReduce历史运行情况)
