# Hbase 伪分布式安装 #
## 本课内容 ##
1. Hbase 安装模式介痥
2. 安装前准备
3. Linux安装配置
4. Hadoop安装配置
5. Hbase安装配置
6. Hbase HDFS目录分析

## Hbase 安装模式介绍 ##
### 单机模式 ###
1. Hbase 不使用HDFS，仅使用本地文件系统
2. Zookeeper与Hbase运行在同一个JVM中
### 分布式模式 ###
#### 伪分布式模式 ###
1. 所有进程运行在同一个节点上，不同进程运行在不同的JVM中
2. 比较适合实验测试
#### 完全分布式模式 ####
1. 进程运行在多个服务器集群中
##### 分布式依赖于HDFS文件系统，因此部署Hbase之前一定要有个正常运行的HDFS集群 #####
## 安装顺序 ##
### Linux环境搭建 ###
### Hadoop环境搭建 ###
### Hbase环境搭建 ###

## 安装前准备 ##
1. Linux:CentOS 6.5
2. JDK:jdk-7u79-linux-x64
3. Hadoop：hadoop=2.5.2.tar.gz
4. Hbase:hbase-1.2.3-bin.tar.gz

## Linux环境搭建 ##
1. 关闭防火墙和SELINUX
2. 配置IP和DNS
3. 配置主机名
4. 设置SSH免密码登陆
5. 安装JDK
## Hadoop和Hbase安装配置 ##
### 安装Hadoop ###
### 安装Hbase ###

## Hadoop安装配置 ##
### 安装hadoop ###
```
下载 hadoop : http://hadoop.apache.org/releases.html 
安装 tar –zxf hadoop-2.5.2.tar.gz –C ../softwares/ 
配置HDFS,YARN 
格式化HDFS
启动 
    sbin/start-dfs.sh 
    sbin/start-yarn.sh 
JPS 查看进程， WEB 界面查看，验证安装
```
### Hbase安装配置 ###
```
下载解压
修改hbase-env.sh – export JAVA_HOME=/usr/java/jdk1.7.0_79
修改Hbase-site.xml
```





