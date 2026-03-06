---
layout: post
title: golang定时器timer的几种简易实现
tags: golang time
categories: golang   
---

为写一个go的自动化工具，定时触发执行任务，简单测试了下go定时器的实现。

```golang
package main

import (
	"fmt"
	"time"
)

func teTimer() {
	// Timer使用完后, 再次启用它，需要调用Reset 方法
	userTimer := time.NewTimer(4 * time.Second)
	for {
		select {
		case <-userTimer.C:
			fmt.Printf("teTimer time is:%s\r\n", time.Now().Format("2006-01-02 15:04:05"))
			userTimer.Reset(4 * time.Second)
		}
	}
	userTimer.Stop()
}

func teAfter() {
	// time.After 返回的是一个 channel，不可复用
	userAfter := time.After(6 * time.Second)
	select {
	case <-userAfter:
		fmt.Printf("teAfter time is:%s\r\n", time.Now().Format("2006-01-02 15:04:05"))
	}

}

func teTicker() {
	// Ticker 时间达到后不需要人为调用 Reset 方法，会自动续期。
	userTicker := time.NewTicker(3 * time.Second)
	for {
		<-userTicker.C
		fmt.Printf("teTicker time is:%s\r\n", time.Now().Format("2006-01-02 15:04:05"))
	}
	userTicker.Stop()

}

func main() {
	go teTimer()
	go teAfter()
	go teTicker()

	time.Sleep(time.Second * 300)
}
```

### 打印输出：

```shell
# go run test_timer.go
teTicker time is:2021-12-18 11:18:50
teTimer time is:2021-12-18 11:18:51
teTicker time is:2021-12-18 11:18:53
teAfter time is:2021-12-18 11:18:53
teTimer time is:2021-12-18 11:18:55
teTicker time is:2021-12-18 11:18:56
teTicker time is:2021-12-18 11:18:59
```
