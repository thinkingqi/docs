# Linux Gradle的安装与升级 #
## Gradle 简介 ## 
Gradle是一个基于JVM的构建工具，它提供如下功能：
1. 像Ant一样，通用灵活的构建工具
2. 可以切换的，基于约定的构建框架
3. 强大的多工程构建支持
4. 基于Apache lvy的强大的依赖惯了
5. 支持maven,lvy仓库
6. 支持传递性依赖管理，而不需要远程仓库或pom.xml和ivy.xml配置文件
7. 对Ant的任务做了很好的集成
8. 基于Groovy,build脚本使用Groovy编写
9. 有广泛的领域模型支持构建

## Gradle 概述 ##
1. 基于声明和基于约定的构建。
2. 依赖型的编程语言。
3. 可以结构化构建，易于维护和理解。
4. 有高级的API允许你在构建执行的整个过程当中，对它的核心进行监视，或者是配置它的行为。
5. 有良好的扩展性。有增量构建功能来克服性能瓶颈问题。
6. 多项目构建的支持。
7. 多种方式的依赖管理。
8. 是第一个构建集成工具。集成了Ant, maven的功能。
9. 易于移值。
10. 脚本采用Groovy编写，易于维护。
11. 通过Gradle Wrapper允许你在没有安装Gradle的机器上进行Gradle构建。
12. 自由，开源。
## Linux下安装 ##
```
wget https://services.gradle.org/distributions/gradle-3.4-bin.zip
unzip gradle-3.4-bin.zip
mv gradle-3.4  /usr/local/gradle
```
## 配置 ##
```
## vim /etc/profile
export GRADLE_HOME=/usr/local/gradle-2.7
PATH=$PATH:$GRADLE_HOME/bin

## source /etc/profile
source /etc/profile
```
## 测试 ##
```
[root@localhost ~]# gradle -v

------------------------------------------------------------
Gradle 3.4
------------------------------------------------------------

Build time:   2017-02-20 14:49:26 UTC
Revision:     73f32d68824582945f5ac1810600e8d87794c3d4

Groovy:       2.4.7
Ant:          Apache Ant(TM) version 1.9.6 compiled on June 29 2015
JVM:          1.7.0_131 (Oracle Corporation 24.131-b00)
OS:           Linux 2.6.32-642.15.1.el6.x86_64 amd64
****
```
## Linux下Gradle升级 ##
根据gradle的安装操作，就可以知道gradle升级很简单，就像安装一样，只不过在最后配置的时候稍微不一样而已。
### 升级 ###
```
##download new version
wget https://services.gradle.org/distributions/gradle-3.4.1-bin.zip

## unzip and mv
unzip gradle-3.4.1-bin.zip
mv gradle-3.4.1 /usr/local/

## vim /etc/profile
export GRADLE_HOME=/usr/local/gradle

##source 
source /etc/profile
```
### 验证 ###
```
[root@localhost ~]# gradle -v

------------------------------------------------------------
Gradle 3.4.1
------------------------------------------------------------

Build time:   2017-02-20 14:49:26 UTC
Revision:     73f32d68824582945f5ac1810600e8d87794c3d4

Groovy:       2.4.7
Ant:          Apache Ant(TM) version 1.9.6 compiled on June 29 2015
JVM:          1.7.0_131 (Oracle Corporation 24.131-b00)
OS:           Linux 2.6.32-642.15.1.el6.x86_64 amd64

```