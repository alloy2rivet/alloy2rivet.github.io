---
layout: post
title: python websockets 两个库实现
tags: python, python
categories: python
---

### websockets:
```python
"""
pip install websockets
"""
import asyncio
import websockets

# WebSocket 地址
WS_URL = "wss://aa.bb.cc/ws/market"

# HTTP 代理（如果不需要代理，设为 None）
HTTP_PROXY = "http://192.168.1.123:18888"

# Ping 间隔（秒）
PING_INTERVAL = 10

async def ping_loop(ws):
    while True:
        try:
            await ws.send("ping")
            print("Sent ping")
        except Exception as e:
            print("Send error:", e)
            break
        await asyncio.sleep(PING_INTERVAL)

async def recv_loop(ws):
    try:
        async for message in ws:
            print("Received:", message)
    except websockets.ConnectionClosed:
        print("Connection closed")
    except Exception as e:
        print("Receive error:", e)

async def main():
    proxy_uri = HTTP_PROXY if HTTP_PROXY else None

    async with websockets.connect(
        WS_URL,
        proxy=proxy_uri  # websockets 11+ 版本支持 proxy 参数
    ) as ws:
        print("Connected to WebSocket")
        # 同时运行发送 ping 和接收消息
        await asyncio.gather(ping_loop(ws), recv_loop(ws))

if __name__ == "__main__":
    asyncio.run(main())

```
### websocket-client:
```python
# ws_ping.py
import websocket
import threading
import time
import os

# WebSocket 地址
WS_URL = "wss://aa.bb.cc/ws/market"

# HTTP 代理，格式 http://host:port，如果不使用代理可以设为 None
HTTP_PROXY = "http://192.168.1.123:18888"

# Ping 间隔（秒）
PING_INTERVAL = 30

def on_message(ws, message):
    print("Received:", message)

def on_error(ws, error):
    print("Error:", error)

def on_close(ws, close_status_code, close_msg):
    print("WebSocket closed")

def on_open(ws):
    def run(*args):
        while True:
            try:
                ws.send("ping")
                print("Sent ping")
            except Exception as e:
                print("Send error:", e)
                break
            time.sleep(PING_INTERVAL)
    threading.Thread(target=run, daemon=True).start()

if __name__ == "__main__":
    # 配置代理
    if HTTP_PROXY:
        proxy_host, proxy_port = HTTP_PROXY.replace("http://", "").split(":")
        ws = websocket.WebSocketApp(
            WS_URL,
            on_open=on_open,
            on_message=on_message,
            on_error=on_error,
            on_close=on_close,
        )
        ws.run_forever(http_proxy_host=proxy_host, http_proxy_port=int(proxy_port))
    else:
        ws = websocket.WebSocketApp(
            WS_URL,
            on_open=on_open,
            on_message=on_message,
            on_error=on_error,
            on_close=on_close,
        )
        ws.run_forever()
```


