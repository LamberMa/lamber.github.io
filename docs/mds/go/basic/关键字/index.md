


## 类型别名

类型别名的语法如下：

```go
type identifier = Type
```
来看一个示例
```go
package main

import "fmt"

type S = string

func main() {
    var s S = "hello world"
    fmt.Println(s)
}
```