# Jenkins 部署 #
## 相关安装包Download ##
```
#jdk download
wget http://download.oracle.com/otn-pub/java/jdk/8u121-b13/e9e7ea248e2c4826b92b3f075a80e441/jdk-8u121-linux-x64.tar.gz

#jenkins download
wget https://mirrors.tuna.tsinghua.edu.cn/jenkins/war-stable/2.32.3/jenkins.war

#tomcat download
wget http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-8/v8.5.12/bin/apache-tomcat-8.5.12.tar.gz
```
## jdk 环境变量 ##
```
#set java JDK 
export JAVA_HOME=/usr/java
export JRE_HOME=/usr/java/jre
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
export CLASSPATH=$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar

source /etc/profile(环境变量生效)
```
```
#jdk环境变量验证
[root@localhost ~]# java -version
java version "1.8.0_121"
Java(TM) SE Runtime Environment (build 1.8.0_121-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.121-b13, mixed mode)
```
## Tomcat 安装 ##
```
tar zxvf apache-tomcat-8.5.12.tar.gz
cd apache-tomcat-8.5.12/webapps
rm -rf *
cp  jenkins.war  apache-tomcat-8.5.12/webapps/
```
### 配置Tomcat字符集 ###
```
在Tomcat下的\conf\server.xml增加下面
   <Connector port="80" protocol="HTTP/1.1" maxThreads="200" minSpareThreads="25" maxSpareThreads="75"
               connectionTimeout="20000" acceptCount="50" debug="0" 
               redirectPort="8443" 
			   URIEncoding="UTF-8"/>
```
####  配置jvm内存 ####
```
vim catalina.sh
#!/bin/sh
JAVA_OPTS= '-Xms2048M -Xmx2048M -XX:PermSize=512M -XX:MaxNewSize=512M -XX:MaxPermSize=512M'
```
```
-client，-server
这两个参数用于设置虚拟机使用何种运行模式，一定要作为第一个参数，client模式启动比较快，但运行时性能和内存管理效率不如server模式，通常用于客户端应用程序。相反，server模式启动比client慢，但可获得更高的运行性能。
在windows上，缺省的虚拟机类型为client模式，如果要使用server模式，就需要在启动虚拟机时加-server参数，以获得更高性能，对服务器端应用，推荐采用server模式，尤其是多个CPU的系统。在Linux，Solaris上缺省采用server模式。
    此外，在多cup下，建议用server模式

-Xms<size>
设置虚拟机可用内存堆的初始大小，缺省单位为字节，该大小为1024的整数倍并且要大于1MB，可用k(K)或m(M)为单位来设置较大的内存数。初始堆大小为2MB。加“m”说明是MB，否则就是KB了。 
例如：-Xms6400K，-Xms256M
-Xmx<size>
设置虚拟机 的最大可用大小，缺省单位为字节。该值必须为1024整数倍，并且要大于2MB。可用k(K)或m(M)为单位来设置较大的内存数。缺省堆最大值为64MB。
例如：-Xmx81920K，-Xmx80M
当应用程序申请了大内存运行时虚拟机抛出java.lang.OutOfMemoryError: Java heap space错误，就需要使用-Xmx设置较大的可用内存堆。
PermSize/MaxPermSize：定义Perm段的尺寸，即永久保存区域的大小，PermSize为JVM启动时初始化Perm的内存大小；MaxPermSize为最大可占用的Perm内存大小。在用户生产环境上一般将这两个值设为相同，以减少运行期间系统在内存申请上所花的开销
```
#### 配置jenkins家目录 ####
```
vim catalina.sh
#!/bin/sh
export CATALINA_OPTS="-DJENKINS_HOME=/data/jenkins"
export JENKINS_JAVA_OPTIONS="-Djava.awt.headless=true -Dhudson.ClassicPluginStrategy.noBytecodeTransformer=true"
```
### Tomcat启动 ###
```
./catalina.sh start 
```

#### jenkins密码 ####
```
cat /root/.jenkins/secrets/initialAdminPassword
```
### jenkins控制台 ###
浏览器打开jenkins的web地址
```
http://10.195.16.28:8080/jenkins/
```
jenkins会自动生成一个默认密码
![](http://i.imgur.com/zwNOtlr.png)
选择默认安装
![](http://i.imgur.com/RtkavUX.png)
等待安装完
![](http://i.imgur.com/37dzmqn.png)
创建用户
![](http://i.imgur.com/sNn83Lx.png)
进入jenjins
![](http://i.imgur.com/gHR6Mhq.png)
![](http://i.imgur.com/QtqUzvs.png)

### jenkins 插件安装 ###
![](http://i.imgur.com/JoXMf1a.png)
![](http://i.imgur.com/t913oLe.png)
#### 实用插件 ####
```
iOS专用：Xcode integration
Android专用：Gradle plugin
Gitlab插件：GitLab Plugin 和 Gitlab Hook Plugin
Git插件： Git plugin
GitBuckit插件： GitBuckit plugin
签名证书管理插件: Credentials Plugin 和Keychains and Provisioning Profiles Management
FTP插件: Publish over FTP
脚本插件: Post-Build Script Plug-in
修改Build名称/描述(二维码)： build-name-setter / description setter plugin
获取仓库提交的commit log： Git Changelog Plugin
自定义全局变量: Environment Injector Plugin
自定义邮件插件： Email Extension Plugin
获取当前登录用户信息： build-user-vars-plugin
显示代码测试覆盖率报表： Cobertura Plugin
来展示生成的单元测试报表，支持一切单测框架，如junit、nosetests等： Junit Plugin
其它： GIT plugin / SSH Credentials Plugin
```
![](http://i.imgur.com/p9q3O0i.png)
### Maven安装 ###
```
wget http://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz

tar zxvf apache-maven-3.3.9-bin.tar.gz -C /usr/local/maven
```
#### Maven环境变量配置 ####
vim .bash_profile
```
MAVEN_HOME=/usr/local/maven
export MAVEN_HOME
export PATH=${PATH}:${MAVEN_HOME}/bin
```
#### Maven 版本验证 ####
```
[root@localhost ~]# mvn -version
Apache Maven 3.3.9 (bb52d8502b132ec0a5a3f4c09453c07478323dc5; 2015-11-11T00:41:47+08:00)
Maven home: /usr/local/maven
Java version: 1.8.0_121, vendor: Oracle Corporation
Java home: /usr/java/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "2.6.32-642.15.1.el6.x86_64", arch: "amd64", family: "unix"

```
### jenkins 组件 ###
jdk
![](http://i.imgur.com/w2FWqI7.png)
maven
![](http://i.imgur.com/aOlFdPR.png)
## 参数化构建过程 ##
### git分支选择 ###
在版本发布的时候选择git分支和代码发布失败代码回滚
![](http://i.imgur.com/EPTEsCW.png)
![](http://i.imgur.com/whi738I.png)
在shell脚本中做war包备份，让rollback的变量和BUILD_NUMEBER变量一致（下文中会介绍）
添加git的分支选择和代码回滚的构建参数
![](http://i.imgur.com/vTle3gY.png)
![](http://i.imgur.com/BmWxm6M.png)
![](http://i.imgur.com/mNkZb2a.png)
接下来需要在git那里修改我们的BRANCH变量
![](http://i.imgur.com/bAYfZ9x.png)
这样就完成分支的选择，代码的回滚会在下面的shell中实现
![](http://i.imgur.com/Sg5MnhO.png)
大家根据自己的环境修改
![](http://i.imgur.com/2MippNW.png)
```
#/bin/bash
project="kemi"
name="kemi"
backupcode="/data/backcode/$project/$BUILD_NUMBER"
rollcode="/data/backcode/$project"
jar="kemi-0.0.1-SNAPSHOT.jar"
mkdir -p $backupcode
if [ "$rollback" -gt "0" ]; then
cp -r "$rollcode"/"$rollback"/"$jar"   /srv/salt/deploy/"$name"/
fi
if [ "$rollback" -eq "0" ]; then
cp -r  "$JENKINS_HOME"/workspace/"$project"/target/"$jar"    $backupcode
cp -r  "$JENKINS_HOME"/workspace/"$project"/target/"$jar"    /srv/salt/deploy/"$name"/
fi
salt  "app*.kmbear.com" state.sls deploy."$name"
rm -rf  "$JENKINS_HOME"/workspace/"$project"/target/"$jar"
rm -rf /srv/salt/deploy/"$name"/"$jar"

```
脚本中可以调用jenkins的变量
```
The following variables are available to shell scripts

BRANCH_NAME
For a multibranch project, this will be set to the name of the branch being built, for example in case you wish to deploy to production from master but not from feature branches; if corresponding to some kind of change request, the name is generally arbitrary (refer to CHANGE_ID and CHANGE_TARGET).
CHANGE_ID
For a multibranch project corresponding to some kind of change request, this will be set to the change ID, such as a pull request number, if supported; else unset.
CHANGE_URL
For a multibranch project corresponding to some kind of change request, this will be set to the change URL, if supported; else unset.
CHANGE_TITLE
For a multibranch project corresponding to some kind of change request, this will be set to the title of the change, if supported; else unset.
CHANGE_AUTHOR
For a multibranch project corresponding to some kind of change request, this will be set to the username of the author of the proposed change, if supported; else unset.
CHANGE_AUTHOR_DISPLAY_NAME
For a multibranch project corresponding to some kind of change request, this will be set to the human name of the author, if supported; else unset.
CHANGE_AUTHOR_EMAIL
For a multibranch project corresponding to some kind of change request, this will be set to the email address of the author, if supported; else unset.
CHANGE_TARGET
For a multibranch project corresponding to some kind of change request, this will be set to the target or base branch to which the change could be merged, if supported; else unset.
BUILD_NUMBER
The current build number, such as "153"
BUILD_ID
The current build ID, identical to BUILD_NUMBER for builds created in 1.597+, but a YYYY-MM-DD_hh-mm-ss timestamp for older builds
BUILD_DISPLAY_NAME
The display name of the current build, which is something like "#153" by default.
JOB_NAME
Name of the project of this build, such as "foo" or "foo/bar".
JOB_BASE_NAME
Short Name of the project of this build stripping off folder paths, such as "foo" for "bar/foo".
BUILD_TAG
String of "jenkins-${JOB_NAME}-${BUILD_NUMBER}". All forward slashes (/) in the JOB_NAME are replaced with dashes (-). Convenient to put into a resource file, a jar file, etc for easier identification.
EXECUTOR_NUMBER
The unique number that identifies the current executor (among executors of the same machine) that’s carrying out this build. This is the number you see in the "build executor status", except that the number starts from 0, not 1.
NODE_NAME
Name of the agent if the build is on an agent, or "master" if run on master
NODE_LABELS
Whitespace-separated list of labels that the node is assigned.
WORKSPACE
The absolute path of the directory assigned to the build as a workspace.
JENKINS_HOME
The absolute path of the directory assigned on the master node for Jenkins to store data.
JENKINS_URL
Full URL of Jenkins, like http://server:port/jenkins/ (note: only available if Jenkins URL set in system configuration)
BUILD_URL
Full URL of this build, like http://server:port/jenkins/job/foo/15/ (Jenkins URL must be set)
JOB_URL
Full URL of this job, like http://server:port/jenkins/job/foo/ (Jenkins URL must be set)
SVN_REVISION
Subversion revision number that's currently checked out to the workspace, such as "12345"
SVN_URL
Subversion URL that's currently checked out to the workspace.
```
## SaltStack简单安装 ##
### Saltstack Master安装 ##
在jenkins主机上安装salt-master
由于我的机器上已经安装了我就不安装了
master安装
```
yum install salt-master.noarch -y
```
minion端安装
```
yum install salt-minion.noarch -y
```
salt使用主机名来分发的，我们需要修改主机名。minion端也要修改，hosts文件也要添加（如果机器多可以使用内网DNS服务）
```
vim /etc/sysconfig/network

NETWORKING=yes
HOSTNAME=devops.xxx.com

```
master配置文件修改
```
vim /etc/salt/master 
```
注意master文件的格式
```
[root@devops deploy]# cd /etc/salt/
[root@devops salt]# cat master  | grep -v  "^#" | grep -v "^$"
  interface: 0.0.0.0
  publish_port: 4505
  user: root
  max_open_files: 100000
  worker_threads: 5
  ret_port: 4506
  pidfile: /var/run/salt-master.pid
  loop_interval: 60
  output: nested
  show_timeout: True
  color: True
  sock_dir: /var/run/salt/master
  job_cache: True
  minion_data_cache: True
  base:
   - /srv/salt/
```
minion修改的比较少
```
[root@testtomcat salt]# cat minion  | grep -v "^#" | grep -v "^$"
  master: devops.kmbear.com
  master_port: 4506
  user: root
```
### master上添加minion ###
由于我已经添加进来了就不演示添加了，salt-key -a 主机名是添加minion主机
```
[root@devops deploy]# salt-key -L
minions:
    - testH5.kmbear.com
    - testtomcat.kmbear.com
minions_denied:
minions_pre:
minions_rejected:
```
### master 编写sls文件 ###
```
mkdir -p /srv/salt/deploy
vim kemibkapp.sls
```
```
clean-code:
  cmd.run:
    - name: /root/kemibkapp/bin/catalina.sh  stop &&  rm -rf /root/kemibkapp/webapps/*
kemibkapp-file:
  file.managed:
    - name: /root/kemibkapp/webapps/kemibkapp-0.0.1-SNAPSHOT.war
    - source: salt://deploy/kemibkapp/kemibkapp-0.0.1-SNAPSHOT.war

start-code:
  cmd.run:
    - name: /root/kemibkapp/bin/catalina.sh  start

```
