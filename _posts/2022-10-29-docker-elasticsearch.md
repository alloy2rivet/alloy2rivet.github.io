---
layout: post
title: docker elasticsearch temp for service
tags: docker elasticsearch 
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

### 调整sysctl.conf:
> Elasticsearch/OpenSearch 官方明确要求 ≥262144
- 临时生效：sudo sysctl -w vm.max_map_count=262144
- 永久生效：在 /etc/sysctl.conf 加 vm.max_map_count=262144
```bash
# sysctl -p
```

### 创建存储目录：
```
# mkdir -p /data/storage/elasticsearch
# chmod 775 /data/storage/elasticsearch
```

### pull image：
> https://hub.docker.com/_/elasticsearch
```bash
# docker pull elasticsearch:8.19.12
```

### start container:
> -d：让容器在后台运行。这是你需要的，因为你希望服务一直挂着提供接口。  
> -it：通常用于交互式操作（如进入终端手动输入命令）。对于“占坑”服务，加上 -it 虽然不报错，但完全没必要，且浪费资源。    
> --privileged 会让容器拥有宿主机的所有内核权限（如修改磁盘分区、加载内核模块等）。  
> --restart always 是非常正确的，这样只要服务器开着，这两个“伪装”服务就会自动运行，保证 App 不会因为连不上数据库而报错。  
```bash
# docker run -d --name elasticsearch --restart always -v /data/storage/elasticsearch:/usr/share/elasticsearch/data -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e "xpack.security.enabled=false" -e "ELASTIC_PASSWORD=1234567" -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" elasticsearch:8.19.12
```

### Test ElasticSearch:
```bash
# curl -H 'Content-Type:application/json'  http://127.0.0.1:9200
# curl -H 'Content-Type:application/json' -u 'elastic:1234567' http://127.0.0.1:9200
```
