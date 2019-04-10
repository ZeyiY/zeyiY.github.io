---
layout: contos
title: echoip步骤
date: 2019-03-18 23:21:02
tags:
---

## Contos 环境 Docker安装 **echoip**步骤

背景：因为最近的一些个人的想法，恰好需要使用根据Ip获取地理位置，在百度和Google中查询了很多种接口，要么收费，要么有次数限制，萌新囊中羞涩，故而使用https://github.com/mpolden/echoip 这个服务了，并且使用了Docker 部署的方式进行使用。在部署过程中踩了很多坑，故进而记录，以防以后使用。

### 环境准备

#### 首先要安装docker环境，并运行起来。

```
uname -r
```

Docker 要求 CentOS 系统的内核版本高于 3.10 ，查看本页面的前提条件来验证你的CentOS 版本是否支持 Docker 。

安装一些必要的系统工具

```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

添加软件源信息：

```
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

更新 yum 缓存：

```
sudo yum makecache fast
```

安装 Docker-ce：

```
sudo yum -y install docker-ce
```

启动 Docker 后台服务

```
sudo systemctl start docker
```

测试运行 hello-world

```
docker run hello-world
```

#### 安装git

```
sudo yum install git
```

安装Vim

```
sudo yum install vim
```



### Install docker-ce

Clone the project

```
git clone https://github.com/alaluces/Docker-EchoIP.git echoip
cd echoip
```

Download the geoip db and extract to geoip folder

```
cd geoip
wget https://geolite.maxmind.com/download/geoip/database/GeoLite2-City.tar.gz
wget https://geolite.maxmind.com/download/geoip/database/GeoLite2-Country.tar.gz
tar xvf GeoLite2-City.tar.gz
tar xvf GeoLite2-Country.tar.gz
mv GeoLite2-City_20190318/GeoLite2-City.mmdb ..
mv GeoLite2-Country_20190318/GeoLite2-Country.mmdb ..

```

Edit the startup script to modify your preferences

```
vim start.sh

Application Options:
  -f, --country-db=FILE        Path to GeoIP country database
  -c, --city-db=FILE           Path to GeoIP city database
  -l, --listen=ADDR            Listening address (default: :8080)
  -r, --reverse-lookup         Perform reverse hostname lookups
  -p, --port-lookup            Enable port lookup
  -t, --template=FILE          Path to template (default: index.html)
  -H, --trusted-header=NAME    Header to trust for remote IP, if present (e.g. X-Real-IP)

Help Options:
  -h, --help                   Show this help message
```

Build the container

```
docker build -t echoip .
```

Run the image you built

```
docker run -d --rm --name echoip -p8080:8080 echoip
```

If built successfully, it can be viewed on:

```
http://ip:8080/json
```

![gratisography-ski-lift-summer.jpg](https://i.loli.net/2019/03/18/5c8fb80fb113e.jpg)