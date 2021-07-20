### Type关键字

type是go语法里重要的关键字，type绝不只是对应的c或者c++中的typedef。搞清楚type的作用，就容易理解go中的核心概念struct，interface，函数的应用。

#### 1、定义结构体

```go
package main

import "fmt"

type person struct {
	name string // 注意后面不能有逗号
	age  int
}

func main() {
	p1 := person{
		name: "maxiaoyu", // 注意后面要添加逗号
		age:  1,          // 注意如果下面的}提到这里来，那么逗号可以省略。
	}
	p2 := person{
        // 初始化字段也不一定要全部指定，如果不指定会被自动初始化为当前类型的“零值”
		age:  2,
	}
	fmt.Println(p1.name)
	fmt.Println(p1.age)
	fmt.Println(p2.name) // 输出空字符串
	fmt.Println(p2.age)
}
```

#### 2、类型重命名

对类型进行重命名，相当于类型的等价定义，但是是属于不同的两个类型。

```go
package main

import "fmt"

type name string

func main() {
	var myname name = "lamber" // 其实底层就是字符串类型
	l := []byte(myname)
	fmt.Println(len(l))
}
```
我们可以利用这些特性定义一系列的方法，提供一些扩展的功能。

```go
package main

import "fmt"

type name string

func (n name) length() int {
	return len(n)
}

func main() {
	var myname name  = "lamber" // 其实底层就是字符串类型
	fmt.Println(myname.length())
}

```

#### 3、结构体内匿名成员

```go
import "fmt"


type person struct {
	string
}

func main() {
	p := person{
		string: "lamber", // 也可以这样：person{"lamber"}
	}
	fmt.Println(p.string)
}
```

#### 4、定义接口


#### 5、定义函数类型

