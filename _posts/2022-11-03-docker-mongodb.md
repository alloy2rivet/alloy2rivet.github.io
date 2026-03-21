---
layout: post
title: docker mongo temp for service
tags: docker monog 
categories: docker  
---

### Install Docker:
> TencentOS
```bash
// 使用 YUM 安装 Docker
# yum install -y docker-ce
// 启动 Docker
# systemctl start docker
// 设置开机启动
# systemctl enable docker
// 检查 Docker 状态：
# systemctl status docker
```

### 创建存储目录：
```
# mkdir -p /data/storage/mongodb
# chmod 775 /data/storage/mongodb
```

### Install mongosh:
> https://github.com/mongodb-js/mongosh/releases
```bash
# wget https://github.com/mongodb-js/mongosh/releases/download/v2.8.1/mongodb-mongosh-2.8.1.x86_64.rpm
# rpm -ivh mongodb-mongosh-2.8.1.x86_64.rpm
```

### pull image：
> https://hub.docker.com/_/mongo/tags
```bash
# docker pull mongo:8.0.20
```

### start container:
> -d：让容器在后台运行。这是你需要的，因为你希望服务一直挂着提供接口。  
> -it：通常用于交互式操作（如进入终端手动输入命令）。对于“占坑”服务，加上 -it 虽然不报错，但完全没必要，且浪费资源  
> --privileged 会让容器拥有宿主机的所有内核权限（如修改磁盘分区、加载内核模块等）  
> --restart always 是非常正确的，这样只要服务器开着，这两个“伪装”服务就会自动运行，保证 App 不会因为连不上数据库而报错。  
```bash
# docker run -d --name mongodb --restart always -p 27017:27017 --memory=256m -v /data/storage/mongodb:/data/db mongo:8.0.20 --quiet
```

### Test mongo:
```bash
# mongo
test>
test> use admin
switched to db admin
admin> db.createUser({
user: "admin_user",
 pwd: "AFSAR#gasdg3eww",
roles: [{ role: "root", db: "admin" }]
})
{ ok: 1 }
admin> db.auth('admin_user', 'AFSAR#gasdg3eww')
{ ok: 1 }
```
