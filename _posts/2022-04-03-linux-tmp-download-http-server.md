---
layout: post
title: Linux服务器启动http临时下载文件
tags: linux http
categories: linux  
---

在 Linux 服务器上**临时启动一个 HTTP 文件下载服务**，最简单、最常见的有 3 种方式。按**启动速度和依赖最少**排序如下。


# 一、最快的方法（推荐）：Python HTTP Server

如果服务器有 Python（大部分都有），一条命令即可。

### Python3

```bash
cd /path/to/files
python3 -m http.server 8000
```

访问地址：

```
http://服务器IP:8000
```

例如：

```
http://192.168.1.10:8000/test.zip
```

特点：

* 自动列出目录
* 支持下载
* 不需要安装软件
* Ctrl+C 即可停止

如果要监听所有 IP：

```bash
python3 -m http.server 8000 --bind 0.0.0.0
```

---

# 二、BusyBox HTTP Server（更轻量）

很多 Linux 都有 busybox。

```bash
cd /path/to/files
busybox httpd -f -p 8000
```

参数：

```
-f  前台运行
-p  端口
```

访问：

```
http://服务器IP:8000
```

停止：

```
Ctrl + C
```

---

# 三、使用 Node.js（如果服务器有 Node）

```bash
npm install -g http-server
```

启动：

```bash
cd /path/to/files
http-server -p 8000
```

---

# 四、只分享一个文件（最方便）

如果只想下载 **单个文件**：

```bash
python3 -m http.server 8000
```

然后访问：

```
http://服务器IP:8000/文件名
```

例如：

```
http://1.2.3.4:8000/app.apk
```

---

# 五、服务器在内网如何下载

如果服务器 IP：

```
10.0.0.5
```

本机下载：

```
http://10.0.0.5:8000/file.zip
```

---

# 六、后台运行（避免断开）

```bash
nohup python3 -m http.server 8000 &
```

查看进程：

```bash
ps aux | grep http.server
```

停止：

```bash
kill 进程ID
```

---

# 七、临时下载神器（最常用）

上传文件到服务器后：

```bash
python3 -m http.server 8000
```

本地直接：

```
wget http://服务器IP:8000/file
```

或者浏览器下载。

---

# 八、进阶（防止别人看到目录）

只允许下载某一个文件：

```bash
python3 -m http.server 8000 --directory /path/file
```

或者新建目录只放下载文件。

---

如果你想用 **PHP** 在 Linux 服务器上临时启动一个 HTTP 文件下载服务，也非常简单。PHP 内置了一个轻量级 Web 服务器，可以直接用来提供文件下载。下面详细说明几种方式。

---

## 1️⃣ 最简单的方式：PHP 内置服务器

进入你想分享的目录：

```bash
cd /path/to/files
```

启动 PHP 内置服务器：

```bash
php -S 0.0.0.0:8000
```

解释：

* `-S 0.0.0.0:8000` → 绑定所有网卡，端口 8000
* 默认文档根目录就是当前目录
* 浏览器访问：`http://服务器IP:8000/`，就能看到文件列表，直接点击下载

**后台运行：**

```bash
nohup php -S 0.0.0.0:8000 > php.log 2>&1 &
```

停止方法：

```bash
ps aux | grep 'php -S'  # 找到进程ID
kill 进程ID
```

---

