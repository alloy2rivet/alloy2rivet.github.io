---
layout: post
title: wscat测试webscoket
tags: websocket, nodejs
categories: websocket
---

### 安装nodejs
> https://nodejs.org/en/download/current

### 安装wscat
> https://github.com/websockets/wscat

在 Windows 下安装 wscat（一个基于 Node.js 的 WebSocket 命令行测试工具），最直接的方法是通过 npm 进行全局安装。
安装步骤

   1. 安装 Node.js
   由于 wscat 是 Node.js 模块，你必须先安装环境。请前往 Node.js 官网 下载并安装长期支持版 (LTS)。
   2. 验证 npm
   打开命令提示符 (CMD) 或 PowerShell，输入以下命令检查是否安装成功：
   ```cmd
   node -v
   npm -v
   ```
   3. 安装 wscat
   在命令行中执行以下命令进行全局安装：
   ```cmd
   npm install -g wscat
   ```
   4. 测试安装
   安装完成后，输入命令查看版本以确认成功：
   ```cmd
   wscat --version
   ```

### 基本使用示例
安装成功后，你可以直接在命令行连接 WebSocket 服务器：

#### 连接服务器：
```cmd
wscat -c ws://192.168.1.11:12345/sock
```
#### 添加header:
```cmd
wscat -c ws://192.168.1.11:12345/sock -H "Authorization: aa.zzzz.cccc"
```
#### 使用代理：
```cmd
wscat --proxy http://192.168.1.22:1111 -c ws://192.168.1.11:12345/sock
```
#### 作为服务器启动：
```cmd
wscat -l 8080 (监听本地 8080 端口)
```
