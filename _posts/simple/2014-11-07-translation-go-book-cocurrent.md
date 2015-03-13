---
layout: post
title: "[GO] 翻译 Go Book 第十章 - 并发"
---
本文翻译自 [Go Book - Chapter 10](http://www.golang-book.com/10/index.htm)

原作者 © 2014 Caleb Doxsey. Cover Art: © 2012 Abigail Doxsey Anderson.

原文按照 CC3.0 协议发布。

此章节讲述了Go的并发模型，算是比较重要的一节，因此想要翻译为中文。

---

大型程序经常是有许多小型子程序构成的。比如说Web服务器程序处理大量的来自浏览器的请求，通过构建HTML页面来进行相应。每一个请求会由一个固定逻辑处理，就像是一段小程序。

如果像这样的大型程序能够让其中的各个组件同时运行是非常理想的（比如同时处理多个网络请求）。同时让多个任务执行就是所谓的“并发”。goroutines和channel，使得Go对并发处理有很好的支持。

### Goroutines
---

goroutine是一个可以同其他函数同时运行的函数。创建一个goroutine很简单，只需要在普通函数调用前加上`go`关键字。

```
package main

import "fmt"

func f(n int) {
    for i := 0; i < 10; i++ {
        fmt.Println(n, ":", i)
    }
}

func main() {
    go f(0)
    var input string
    fmt.Scanln(&input)
}
```

这段程序包含了两个goroutine. 第一个是隐式创建的，就是`main()`函数本身；第二个是通过调用`go f(0)`创建的。正常情况下，调用一个函数会执行函数体内的所有表达式，然后返回到调用的下一行。而调用goroutine后会立即返回到下一行，不会等待当前行执行完成。因此，这段示例代码的末尾加上了`Scanln`函数。否则，不等到打印结束，`main()`函数会退出，继而整个程序退出。（译者注：最初我就是坑在这里的）

Goroutines 是轻量级的异步，可以轻轻松松地创建上千个。所以，通过修改上面的示例代码，我们可以同时运行10个goroutine来做这个工作。

```
func main() {
    for i := 0; i < 10; i++ {
        go f(i)
    }
    var input string
    fmt.Scanln(&input)
}
```

运行这段代码时，你会有一个错觉。你会看到10个数字是顺序打印出来的，仿佛不是在并行处理。使用`time.Sleep`和`rand.Intn`添加一些随机的延时可以更清楚地看到goroutine的并发。

```
package main

import (
    "fmt"
    "time"
    "math/rand"
)

func f(n int) {
    for i := 0; i < 10; i++ {
        fmt.Println(n, ":", i)
        amt := time.Duration(rand.Intn(250))
        time.Sleep(time.Millisecond * amt)
    }
}
func main() {
    for i := 0; i < 10; i++ {
        go f(i)
    }
    var input string
    fmt.Scanln(&input)
}
```

`f`输出0~10的数字，但是每个调用会有一个随机时长的延时，这样就能够看到goroutine的并发。

### Channels
---

Channels 提供了两个goroutine之间互相通信和同步的能力。以下是一个例子：

```
package main

import (
    "fmt"
    "time"
)

func pinger(c chan string) {
    for i := 0; ; i++ {
        c <- "ping"
    }
}
func printer(c chan string) {
    for {
        msg := <- c
        fmt.Println(msg)
        time.Sleep(time.Second * 1)
    }
}
func main() {
    var c chan string = make(chan string)

    go pinger(c)
    go printer(c)

    var input string
    fmt.Scanln(&input)
}
```

这段程序会一直打印`ping`（按回车退出）。`channel`（通道）由关键字`chan`后跟一个要传的东西类型的类型表示（在这个例子中，我们穿的是`string`，所以写成`chan string`)。运算符`<-`用来表示向`channel`中发送或者接受一个消息。`c <- "ping"` 表示把`"ping"`这个字符串发送到通道内。`msg := <- c`表示从通道内接收一条消息，然后赋值给`msg`。

```
msg := <- c
fmt.Println(msg)
```

可以简化为

`fmt.Println(<-c)`

可以通过这种方式，使用`channel`来同步两个goroutine。当`pinger`尝试往通道里面发送消息的时候，他会一直等待（也就是被block住)，直到`printer`准备好接收消息。

在示例程序上添加另外一个发送方:

```
func ponger(c chan string) {
    for i := 0; ; i++ {
        c <- "pong"
    }
}
```

然后修改`main()`函数:

```
func main() {
    var c chan string = make(chan string)

    go pinger(c)
    go ponger(c)
    go printer(c)

    var input string
    fmt.Scanln(&input)
}
```

这样程序会一直输出`ping`,`pong`。

### Channel 方向
---

可以在函数声明中声明通道的方向为发送或者接收，在函数内调用超出函数定义的操作会使编译器报错。（译者注：我一直觉得谷歌设计Go是因为被其他灵活的语言坑出心理阴影了）

发送方的声明如下：

```
func pinger(c chan<- string)
```

接收方的声明如下：

```
func printer(c <-chan string)
```

### Select
---

`Go`有一个语句叫`select`，它和`switch`类似，但是是给`channel`用的。

```
func main() {
    c1 := make(chan string)
    c2 := make(chan string)

    go func() {
        for {
            c1 <- "from 1"
            time.Sleep(time.Second * 2)
        }
    }()
    go func() {
        for {
            c2 <- "from 2"
            time.Sleep(time.Second * 3)
        }
    }()
    go func() {
        for {
            select {
            case msg1 := <- c1:
                fmt.Println(msg1)
            case msg2 := <- c2:
                fmt.Println(msg2)
            }
        }
    }()

    var input string
    fmt.Scanln(&input)
}
```

这个程序有两个通道，一个每两秒打印"from 1"，另一个每三秒打印"from 2"。`select`会选择先准备好的通道，从中接收消息（或者发送）。如果同时都准备好了，`select`会随机选一个。如果都没准备好，`select`会卡在那里，直到有一个通道准备好。

`select`语句应常用来实现超时逻辑：

```
select {
case msg1 := <- c1:
    fmt.Println("Message 1", msg1)
case msg2 := <- c2:
    fmt.Println("Message 2", msg2)
case <- time.After(time.Second):
    fmt.Println("timeout")
}
```

`time.After`会创建一个通道，然后在指定的时间到了之后把当前时间发送到通道内（在这个例子中我们不关心当前时间，所以没有处理值)。我们还可以给`select`指定`default`。

```
select {
case msg1 := <- c1:
    fmt.Println("Message 1", msg1)
case msg2 := <- c2:
    fmt.Println("Message 2", msg2)
case <- time.After(time.Second):
    fmt.Println("timeout")
default:
    fmt.Println("nothing ready")
}
```

如果当前没有任何`channel`准备好，`select`会直接跳到`default`语句。

### 带缓冲的 Channel
---

在创建`channel`的时候，多传入一个参数，可以指定`channel`的缓冲长度：

```
c := make(chan int, 1)
```

普通的`channel`两端是同步的，另一端没有就绪的时候，这一端会一直卡住。但是带缓冲的`channel`是异步的，直到缓冲区已满，发送方和接收方都不会卡主。
