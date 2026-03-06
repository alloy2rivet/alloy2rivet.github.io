---
layout: post
title:  Django+Channels + Daphne WebSocket 内存占用分析优化
tags: django channel daphne
categories: python   
---

django + channel-redis + daphne 开发websocket服务，使用 protocol buffer协议通信。
supervisor管理daphne进程：
```ini
[program:daphne_26035]
environment = ENV="PROD"
directory = /data/services/app
command = /usr/local/bin/daphne -b 127.0.0.1 -p 26035 --proxy-headers --websocket_timeout -1 -v 0 --ping-interval 8640000 app.asgi:application
user = root
autostart = true
autorestart = true
numprocs = 1
stopasgroup = true
killasgroup = true
stderr_logfile = /var/log/stderr.log
stdout_logfile = /var/log/stdout.log
```

top发现 daphne进程 VIRT  RES  占用较高，尤其是在长时间运行后。

pmap 查看进程对应内存使用情况，部分如下：
```bash
00007ff423bf5000  98304K rw---   [ anon ]
00007ff439bfd000  65536K rw---   [ anon ]
00007ff418022000  65400K -----   [ anon ]
0000000001d76000  35664K rw---   [ anon ]
00007ff4373fc000  32768K rw---   [ anon ]
00007ff434bfb000  32768K rw---   [ anon ]
00007ff4323fa000  32768K rw---   [ anon ]
00007ff42fbf9000  32768K rw---   [ anon ]
00007ff42d3f8000  32768K rw---   [ anon ]
00007ff42a3f6000  32768K rw---   [ anon ]
```

# 按进程名筛选
```bash
top -p $(pgrep -d ',' daphne)
```

### 查看进程内存映射情况（Process Memory Map）
```bash
pmap 2892 | tail -n +2 | sort -k2 -n -r | more 
```

### 安装jemalloc
```bash
# CentOS/RHEL
yum install -y jemalloc jemalloc-devel
# Ubuntu/Debian
apt install -y libjemalloc-dev jemalloc
```

### 检查so是否存在：
```bash
find / -name 'libjemalloc.so.2'
```

### supervisor配置增加：
```bash
environment=ENV="PROD",LD_PRELOAD="/usr/lib64/libjemalloc.so.2",MALLOC_CONF="dirty_decay_ms:1000,muzzy_decay_ms:1000",MALLOC_ARENA_MAX=4
```

| 参数                   | 含义              | 作用                             |
| --------------------- | --------------- | ------------------------------ |
| `dirty_decay_ms:1000` | 空闲脏页释放延迟        | 1 秒后才归还给操作系统，减少频繁 purging      |
| `muzzy_decay_ms:1000` | muzzy page 释放延迟 | 同上                             |
| `MALLOC_ARENA_MAX=4`  | 最大 arena 数      | 限制 jemalloc 分配的内存碎片数量          |
| `LD_PRELOAD`          | 使用 jemalloc     | 替换系统 malloc，减少 Python/C 扩展内存碎片 |


### 确认进程正常运行
```bash
ps aux | grep daphne 
```

### 输出包含libjemalloc.so.2则加载成功:
```bash
lsof -p 2892 | grep jemalloc  # 输出包含libjemalloc.so.2则加载成功
```

### 汇总 smaps 里的关键指标
```bash
grep -E "Private_Dirty|Private_Clean|Pss|Rss" /proc/2892/smaps | awk '{sum[$1]+=$2} END {for (i in sum) print i, sum[i]" kB"}'
Pss: 116273 kB  分摊后的实际使用量（考虑共享库）
Private_Dirty: 114328 kB  进程实际占用的物理内存
Private_Clean: 284 kB
Rss: 134660 kB  进程总物理占用
```

