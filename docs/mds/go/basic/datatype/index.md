




## 字符串类型

```go
// 默认不赋值的话就是一个空字符串
var str string
var str string = "hello world"
```

- 字符串输出占位符为`%s`
- 万能输出占位符为`%v`
- 输出一个字符的占位符为`%c`



赋值的一个省略式写法：
```go
b := a
c := "hello"
```
`:=`这个操作符会首先定义一个b，然后把a的值复制给b。同理下面的命令就是定义一个c，然后把“hello”赋值给c。

所以说当`:=`左侧的变量存在的时候，也就是不是新变量的时候，是会直接报错的，也就是说这个符号只是用来定义新的变量。

### 字符串的两种表现形式

- 双引号，`""`可以包含控制字符，比如说`\n`换行符，在双引号中会被解析为换行符
- 反引号，表示所有的字符都是原样输出，比如`\n`会被原样输出。

### 字符串的常用操作

- 长度：`len(str)`
- 拼接：`+, fmt.Sprintf`
```go
// 使用加号拼接
c = c + c
// 使用Sprintf拼接
c = fmt.Sprintf("%s%s", c, c)
```
- 分割：`strings.Split`
```go
// 依赖strings包，因此一开始就需要引入该包
import (
    "fmt"
    "strings"
)

ips := "172.16.1.1;172.16.1.2;172.16.1.3"
// 切片完成以后其实就是一个数组，可以使用下标去访问
ipSlice := strings.Split(ips, ";")
```
- 包含：`strings.Contains`
```go
// 返回布尔类型的，true or false
strings.Contains(ips, "172.16.1.1")
```
- 前缀或者后缀判断：`strings.HasPrefix，strings.HasSuffix`
```go
strings.HasPrefix(str, "http")
```
- 子串出现的位置：`strings.Index()，strings.LastIndex()`
```go
str := "http://baidu baidu.com"
index1 := strings.Index(str, "baidu") // 结果为7
index2 := strings.LastIndex(str, "baidu") // 结果为13

总结来讲就是Index是从前面数第一个出现的位置，LastIndex是从后面数第一个出现的位置，但是位置都是从0开始，0始终是字符串的开始。
```
- join操作：`strings.Join([]string, sep string)`
```go
var str []string = []string{"a", "b", "c", "d"}
strings.Join(str, ";")
```

### 字符串原理解析

- 字符串底层就是一个byte数组，所以可以和`[]byte`切片类型相互转换
- 字符串之中的字符是不能修改的

```go
// 那么如何进行修改呢？可以先转换为byte切片类型再转换回去
var str = "hello"
var byteSlice []byte
byteSlice = []byte(str)
byteSlice[0] = "0"
str = string(byteSlice)
然后再去访问str就会发现str已经被改变了
```

- 字符串是由byte字节组成，所以字符串的长度是byte字节的长度（注意英文字符是一个字符占用一个字节，一个中文字符是占用3个字节，utf8编码的情况下，那么怎么判断有多少个字符呢？可以看rune类型）
- rune类型是用来标识utf8字符，一个rune字符由一个或者多个字节组成

```go
// rune切片标识每一个切片元素是一个字符而不是字节
// rune其实就是类似于Char类型，32位
var str = "中文2333aaa"
var runeSlice []rune
runeSlice = []rune(str)
fmt.Printf("str 长度为\n"， len(runeSlice))

上面的结果如果是字符串形式的
- 3+3+7 = 13bytes，长度为13
- 如果看rune字符级别的，那么长度应该为9
```

### 字符串的遍历

在golang中，字符串遍历有两种方式，一种是`utf8`遍历，一种是unicode遍历。Unicode是符号集，Utf8则是目前使用最广的一种unicode的实现方式。其它的还有utf16，utf32等。首先来看下面这个遍历。

```go
func main() {
	s := "这是一个测试，呵呵哒，hello"
	for i := 0; i < len(s); i++ {
		fmt.Printf("(%T %#v) ", s[i], s[i])
	}
}
```
上面遍历的结果会显示
```go
(uint8 0xe8) (uint8 0xbf) (uint8 0x99) (uint8 0xe6) (uint8 0x98) (uint8 0xaf) (uint8 0xe4) (uint8 0xb8) (uint8 0x80) (uint8 0xe4) (uint8 0xb8) (uint8 0xaa) (uint8 0xe6) (uint8 0xb5) (uint8 0x8b) (uint8 0xe8) (uint8 0xaf) (uint8 0x95) (uint8 0xef) (uint8 0xbc) (uint8 0x8c) (uint8 0xe5) (uint8 0x91) (uint8 0xb5) (uint8 0xe5) (uint8 0x91) (uint8 0xb5) (uint8 0xe5) (uint8 0x93) (uint8 0x92) (uint8 0xef) (uint8 0xbc) (uint8 0x8c) (uint8 0x68) (uint8 0x65) (uint8 0x6c) (uint8 0x6c) (uint8 0x6f) 
```

我顺带把类型也打印出来了，这个就是utf8的遍历，上面我们说到string本身底层就是一个byte只读切片。也就是说我们在存储字符串的是偶，实际存储的就是存储的字节，我们看到的length其实是底层字节切片的长度。那么我们换一种方式来遍历

```go
func main() {
	s := "这是一个测试，呵呵哒，hello"
	for _, ch := range s {
		fmt.Printf("(%T %#v) ", ch, ch)
	}
}
```
此时得到的结果如下：
```go
(int32 36825) (int32 26159) (int32 19968) (int32 20010) (int32 27979) (int32 35797) (int32 65292) (int32 21621) (int32 21621) (int32 21714) (int32 65292) (int32 104) (int32 101) (int32 108) (int32 108) (int32 111)
```
可以看到得到的是int32类型，也就是rune类型，这个时候ch我们是可以直接用`string(ch)`将正确的文字打印出来，但是如果说用len遍历的时候打印出来的中文全部是乱码。