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

