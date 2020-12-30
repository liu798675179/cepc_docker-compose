# 配置文档

## 当前环境：CentOS 7.6 64位

## 一、设置阿里云镜像：

### 1、下载新的CentOS-Base.repo 到/etc/yum.repos.d/
```
$ sudo wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```
### 2、运行以下命令生成缓存
```
$ sudo yum clean all
$ sudo yum makecache fast
```

## 二、安装Docker

### 1、删除旧docker
```
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

### 2、安装一些必要的包
```
$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

### 3、设置docker镜像源
```
$ sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

### 4、更新索引
```
$ sudo yum makecache fast
```

### 5、安装最新docker-ce
```
$ sudo yum install docker-ce
```

### 6、测试
```
启动
$ sudo systemctl start docker

运行hello-world
$ sudo docker run hello-world
```

### 7、修改配置文件使用阿里云加速docker镜像
### <font color="#00dddd">需注册阿里云账户并获取镜像加速地址、具体方法自行查找</font><br />

### 修改daemon配置文件/etc/docker/daemon.json来使用加速器
```
$ sudo mkdir -p /etc/docker

ve8ifthx为我个人的地址
$ sudo tee /etc/docker/daemon.json <<-'EOF'

{
  "registry-mirrors": ["https://ve8ifthx.mirror.aliyuncs.com"]
}
EOF

$ sudo systemctl daemon-reload

$ sudo systemctl restart docker
```

## 三、在docker中安装elasticsearch、kibana、cerebro、fastDFS、fastDHT、nginx

### 1、为保证顺利，先安装java 11
```
$ sudo yum install java-11-openjdk.x86_64
```
### 2、为保证顺利，先设置虚拟内存至少为262144(256mb)、524288(512mb)、1048576(1024mb, 4核8G)
```
$ sudo sysctl -w vm.max_map_count=1048576
```

### 3、安装docker-compose
```
$ sudo yum install docker-compose
```
### 4、使用docker-compose安装这些软件

## <font color="#dd0000">注意：安装前，请修改docker-compose.yaml中的</font><br />

### <font color="#dd0000">注意：更改端口，以防攻击</font>

```
IP: 10.206.0.17
- -Dhosts.0.host=http://10.206.0.17:9200
- ELASTICSEARCH_HOSTS=http://10.206.0.17:9200
```
### <font color="#dd0000">将以上ip设置成为本机下的内网ip</font><br />

### a、将docker-compose.yaml文件放入某个文件夹内

### b、在该文件下，运行以下命令：
```
$ sudo docker-compose up
```

### c、运行一下命令查看运行状态：
```
$ sudo docker ps
```
### 过30s后再执行以下命令检查，以保证各软件均启动完成

### d、检查elasticsearch运行情况
```
浏览器中输入：

http:\\服务器外网ip:9200

若在本机运行，输入

http:\\localhost:9200

出现以下类似信息则为安装成功：

{
  "name" : "564ea6d62dae",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "M_qpAgi2RZK7-GuYsIGZzg",
  "version" : {
    "number" : "7.5.2",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "8bec50e1e0ad29dad5653712cf3bb580cd1afcdf",
    "build_date" : "2020-01-15T12:11:52.313576Z",
    "build_snapshot" : false,
    "lucene_version" : "8.3.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

### e、检查kibana运行情况
```
浏览器中输入：

http:\\服务器外网ip:5601

若在本机运行，输入

http:\\localhost:5601
```

### f、检查cerebro运行情况
```
浏览器中输入：

http:\\服务器外网ip:9000

若在本机运行，输入

http:\\localhost:9000
```

### g、检查nginx运行情况
```
浏览器中输入：

http:\\服务器外网ip:80

若在本机运行，输入

http:\\localhost:80
```

### h、检查fastDFS运行情况
```
$ sudo docker exec -it fastdfs /bin/bash 

echo "Hello FastDFS!">index.html

fdfs_test /etc/fdfs/client.conf upload index.html

此时会产生该文件地址的链接

将该链接用浏览器打开即可
```

### Node.js安装
```
yum install -y gcc-c++ make
curl -sL https://rpm.nodesource.com/setup_14.x | sudo -E bash -
yum install nodejs
npm config set registry https://registry.npm.taobao.org
```

### pm2监控示例
```
npm i pm2 -g
pm2 start npm --name "aw-front" -- run start --watch -save
```

### MongoDB安装
```
vim /etc/yum.repos.d/mongodb.repo

[mongodb-org]
name=MongoDB Repository
baseurl=https://mirrors.tuna.tsinghua.edu.cn/mongodb/yum/el$releasever/
gpgcheck=0
enabled=1

esc
:wq!

yum install -y mongodb-org

systemctl enable mongod
systemctl start mongod

db.createUser({ user: "test",pwd: "1234",roles:[{ role: "userAdminAnyDatabase",db: "admin" }]})

// readWrite
```

### HmacSHA256
```
CryptoJS.enc.Hex.stringify(CryptoJS.HmacSHA256('timestamp=' + timestamp, 'YXXVbXP0hCaoXKgS9rDrU98be34F16b4bqFeHwoajZ2o3HlDZvPqUI0e43XV0mE6'))
```

### redis安装
```
sudo yum install epel-release yum-utils
sudo yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
sudo yum-config-manager --enable remi

sudo yum install redis

sudo systemctl enable redis
sudo systemctl start redis

sudo systemctl status redis

/etc/redis.conf

sudo systemctl restart redis
```

### nginx安装
```
yum install nginx

systemctl enable nginx
systemctl start nginx
```

### git安装
```
yum install git
cd /hume/git
mkdir xxx.git && cd xxx.git
git init --bare
# 创建hooks/post-recive
# 更改权限0755
git --work-tree=/home/front/ --git-dir=/home/git/xxx.git checkout -f
# /home/front必须存在
```

### linux sed 文件中每行的开头添加引号末尾处添加引号逗号
```
sed 's/^/"/g;s/$/",/g' input.txt >> output.txt
```
