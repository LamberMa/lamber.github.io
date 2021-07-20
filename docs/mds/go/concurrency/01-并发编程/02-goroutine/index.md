## Goroutine简介




## 串行的执行

golang串行的执行

```go
package main

import "fmt"

func helloWorld() {
    fmt.Println("hello goroutine")
}

func main() {
    helloWorld()
    fmt.Println("main thread")
}
```
响应的结果为
```shell
hello goroutine
main thread
```
可以看到上面的结果是先打印`hello goroutine`，然后再打印`main thread`，这个执行结果永远都是这样的，因为操作的结果是串行的。这个是不会变化的。那么接下来我们加一个go
```go
package main

import "fmt"

func helloWorld() {
    fmt.Println("hello goroutine")
}

func main() {
    go helloWorld()
    fmt.Println("main thread")
}
```
然后我们再来看相应的结果
```shell
main thread
```
可以发现，我们的hello goroutine之前输出的不存在了，只有这一条。但是多次频繁尝试我们发现，有的时候也是可以打印出来hello goroutine的，那么这个是什么原因导致的呢？究其原因的话，是因为我们在使用go开启一个goroutine，它也是需要开销和时间的，这里操作的时候就不再是串行的了，而main函数本身就是一个单独的goroutine，因此main函数执行完了以后就直接return了，有可能这个时候helloWorld还没有执行完，因此，我们看到有的时候只输出`main thread`有的时候会输出两个。

最简单的处理方式比如说我在main这个goroutine中sleep一下，等一下hellWorld这个goroutine完成。
```go
package main

import (
    "fmt"
    "time"
)

func hello(i int) {
    fmt.Println("hello goroutine", i)
}

func main() {
    for i := 0; i < 10; i++ {
        go hello(i)
    }
    time.Sleep(time.Second)
}
```

上面的显示结果每一次都是不一样的
```shell
hello goroutine 0
hello goroutine 4
hello goroutine 1
hello goroutine 7
hello goroutine 6
hello goroutine 8
hello goroutine 2
hello goroutine 3
hello goroutine 9
hello goroutine 5
```
这样我们就可以知道，这个是并发执行的，而不是串行执行的。再看一个示例
```go
package main

import (
    "fmt"
    "time"
)

func numbers() {
    for i := 1; i <= 5; i++ {
        time.Sleep(250 * time.Millisecond)
        fmt.Printf("%d", i)
    }
}

func alphabets() {
    for i := 'a'; i <= 'e'; i++ {
        time.Sleep(400 * time.Millisecond)
        fmt.Printf("%c", i)
    }
}

func main() {
    go numbers()
    go alphabets()
    time.Sleep(3000 * time.Millisecond)
    fmt.Println("main thread terminate")
}
```

返回的结果，咱们可以算一算如果两个func这样执行的时候，是不是这个输出的顺序？
```shell
1a23b4c5demain thread terminate
```
