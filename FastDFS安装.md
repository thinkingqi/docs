# FastDFS 安装 #
## 一、FastDFS 简介 ##
FastDFS是一个开源的轻量级分布式文件系统，它对文件进行管理，功能包括：文件存储、文件同步、文件访问（文件上传、文件下载）等，解决了大容量存储和负载均衡的问题。特别适合以文件为载体的在线服务，如相册网站、视频网站等等。
### 1.1 一般架构图 ###
FastDFS系统结构如下图所示：
![](http://i.imgur.com/D247hxV.jpg)
### 1.2 上传示意图 ###
![](http://i.imgur.com/NPdBFM7.jpg)
### 1.3 下载交互过程 ###
`client询问tracker下载文件的storage,参数为文件标识（卷名和文件名）；`
`tracker返回一台可用的storage;`
`client直接和storage通讯完成文件下载。`
### 需要的安装包准备 ###
```
wget https://github.com/happyfish100/fastdfs/archive/V5.10.tar.gz
```
```
git clone https://github.com/happyfish100/fastdfs-nginx-module.git
```
```
git clone https://github.com/happyfish100/libfastcommon.git
```
```
wget http://tengine.taobao.org/download/tengine-2.2.0.tar.gz
```
### tracker配置 ###
```
yum install  gcc gcc-c++ make openssl-devel kernel-devel zlib-devel -y
mkdir -p /data/fastdfs/{tracker,storage} //用于存储
```
### 安装libfastcommon ###
```
git clone https://github.com/happyfish100/libfastcommon.git
cd libfastcommon
./make.sh
./make.sh install
安装libfastcommon，这个是FastDFS必须要安装的东西
```
装完后会看到libfastcommon.so安装到了/usr/lib64/libfastcommon.so，但是FastDFS主程序设置的lib目录是/usr/local/lib，所以需要创建软链接.
```
ln -s /usr/lib64/libfastcommon.so /usr/local/lib/libfastcommon.so
ln -s /usr/lib64/libfastcommon.so /usr/lib/libfastcommon.so
ln -s /usr/lib64/libfdfsclient.so /usr/local/lib/libfdfsclient.so
ln -s /usr/lib64/libfdfsclient.so /usr/lib/libfdfsclient.so
```
## 安装FastDFS ##
解压安装包
```
tar zxvf V5.10.tar.gz
cd fastdfs-5.10
./make.sh
./make.sh install 
```
编辑tracker.conf
```
cd /etc/fdfs
cp tracker.conf.sample tracker.conf
vim tracker.com
```
```
base_path=/data/fastdfs/tracker------日志存储目录
store_group=group2----------group链接中的组名如：fastdfs/M00/00/00/oYYBAFWd9RaAd4nRAADrmdE3r9M406.png
log_level=debug--------------日志打印为debug信息会比较全
use_storage_id = true--------使用的storage_ids.conf配置文件
id_type_in_filename = id-----使用storage表示为id,原值为ip
http.server_port=80----------http端口80用来访问storage
```

把配置文件storage_ids.conf复制到/etc/fdfs/----fastdfs的配置文件目录(一组不用修改)
```
cd /etc/fdfs/
cp storage_ids.conf.sample storage_ids.conf
vim /etc/fdfs/storage_ids.conf
//修改为如下
# <id>  <group_name>  <ip_or_hostname>
100001   fastdfs  192.168.2.209
100002   fastdfs  192.168.2.210

启动tracker----一般先启动tracker再启动storage，如果先启动storage的话会卡住，查看storage的日志就知道了
fdfs_trackerd /etc/fdfs/tracker.conf----启动后会在/data/fastdfs/tracker/下创建logs目录，里面有tracker.log
```
启动后查看下日志 看看有啥问题没有,tracker最后
```
cp /root/fastdfs-5.10/conf/http.conf mime.types /etc/fdfs/ 后期用tracker上传文件测试用
```
```
cd /etc/fdfs/
cp client.conf.sample  client.conf
```
```
vim client.conf
base_path=/data/fastdfs
tracker_server=172.31.0.9:22122
tracker_server=172.31.0.10:22122
log_level=debug
```
配置storage
```
cd /etc/fdfs/
cp storage.conf.sample storage.conf
```
`vim storage.conf`
修改以下配置
```
group_name=group1
base_path=/data/fastdfs
store_path0=/data/fastdfs/storage/
tracker_server=172.31.0.9:22122
tracker_server=172.31.0.10:22122
log_level=debug
http.server_port=80
```
### 在tracker上测试 ##
`cp /etc/fdfs/client.conf.sample /etc/fdfs/client.conf`
`vim client.conf`
修改以下配置项
```
tracker_server=172.31.0.9:22122
tracker_server=172.31.0.10:22122
http.tracker_server_port=80
```

#### 测试 ###
```
执行命令上传图片，如果没有通查看日志，或者看防火墙端口
fdfs_upload_file client.conf /dir/aaaaaaaaaaaa.png(随意找一个文件)
会返回类似这样的
fastdfs/M00/00/00/oYYBAFWd9RaAd4nRAADrmdE3r9M406.png
然后上第一个storage上/data/fastdfs/data/00/00发现文件被上传
```
## 为storage配置nginx ##
`tar zxvf tengine-2.2.0.tar.gz`
编译安装
```
./configure --prefix=/usr/local/nginx --add-module=/root/fastdfs-nginx-module/src
make &&  make install 
```
修改nginx配置文件
`vim /usr/local/nginx/conf/nginx.conf`

server段加上
```
location /group1/M00 {
        root /data/fastdfs/storage/;
        ngx_fastdfs_module;
}
```

复制mod_fastdfs.con配置文件到/etc/fdfs下
`cp /root/fastdfs-nginx-module/src/mod_fastdfs.conf /etc/fdfs/`
编辑mod_fastdfs.con
`vim /etc/fdfs/mod_fastdfs.conf`
修改以下几项
```
base_path=/data/fastdfs
tracker_server=172.31.0.9:22122
tracker_server=172.31.0.10:22122
group_name=group1
store_path0=/data/fastdfs/storage/
url_have_group_name = true---意思是在url中包含名
cp /root/fastdfs-5.10/conf/http.conf /etc/fdfs/
cp /root/fastdfs-5.10/conf/mime.types /etc/fdfs/
```
