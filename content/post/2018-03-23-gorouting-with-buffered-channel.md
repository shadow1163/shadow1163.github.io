---
title: Golang 并发定量协程数量
date: 2018-03-23T10:15:12+08:00
tags: [golang]
---
最近需要用到golang做一个并发测试，之前大概看了一遍并发，对基本的流程有了一个了解了。我的需求是启动固定数量的客户端程序，然后等待程序运行完成后收集分析一下结果即可。参照网上的代码如下：
```
package main

import (
	"fmt"
)

func Afunction(ch chan int) {
	fmt.Println("finish")
	<-ch
}

func main() {
	ch := make(chan int, 10)
	for i := 0; i < 1000; i++ {
		ch <- 1
		go Afunction(ch)
	}
}
```
运行了一下感觉没问题就没仔细检查了。结果领导发现运行结果差10个，然后才来仔细看程序，在main函数中的for循环调用完函数后并不会阻塞等待被调用的函数执行完成。main函数退出后，并发的协程也会被自动回收，就导致了结果差10个。
解决的办法是在for循环后添加代码等待协程执行完成，然后退出。有两种方法。

### sync.WaitGroup
```
package main

import (
  "fmt"
  "sync"
)

var waitgroup sync.WaitGroup

func Afunction(ch chan int) {
	fmt.Println("finish")
	<-ch
  waitgroup.Done()
}

func main() {
	ch := make(chan int, 10)
	for i := 0; i < 1000; i++ {
		ch <- 1
    waitgroup.Add(1)
		go Afunction(ch)
	}
  waitgroup.Wait()
}
```
由于要限制协程的数量，所以不能不用channel。虽然程序有些臃肿，但还是工作的。

### channel
```
package main

import (
	"fmt"
)

func Afunction(ch chan int) {
	fmt.Println("finish")
	<-ch
}

func main() {
	ch := make(chan int, 10)
	for i := 0; i < 1000; i++ {
		ch <- 1
		go Afunction(ch)
	}
  for {
    if len(ch) {
      break
    }
  }
}
```
另外可以通过判断channel的缓存数量来等待协程的运行完成。总之使用并发要非常小心。
