# 共享内存
变量，数组，map或结构体。map不能并发读写，会panic，

```go
package main

import (
	"fmt"
	"sync"
)

var a sync.Map

func main() {
	//添加新key
	a.Store("newKey", "newValue")
	if value, exists := a.Load("newKey"); exists {
		fmt.Printf("%s",value)
	} else {
		fmt.Printf("error")
	}
}
```

多线程共享内存来通信。[脏读脏写]()。
通过加[锁]()来访问共享数据。

# CSP是什麼？
CSP 全称是 “Communicating Sequential Processes”。不要通过共享内存来通信，而要通过通信来实现内存共享([channel](./channel.md))。



