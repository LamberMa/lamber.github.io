go中的测试文件必须要以`_test.go`结尾单元测试源码文件可以由多个测试用例（可以理解为函数）组成，每个测试用例的名称需要以 Test 为前缀，例如：

```go
func TestXxx( t *testing.T ){
    //......
}
```

编写测试用例有以下几点需要注意：

- 测试用例文件不会参与正常源码的编译，不会被包含到可执行文件中；
- 测试用例的文件名必须以_test.go结尾；
- 需要使用 import 导入 testing 包；
- 测试函数的名称要以Test或Benchmark开头，后面可以跟任意字母组成的字符串，但第一个字母必须大写，例如 TestAbc()，一个测试用例文件中可以包含多个测试函数；
- 单元测试则以(t *testing.T)作为参数，性能测试以(t *testing.B)做为参数；
- 测试用例文件使用go test命令来执行，源码中不需要 main() 函数作为入口，所有以_test.go结尾的源码文件内以Test开头的函数都会自动执行。
- 函数中通过调用 testing.T 的`Error`,`Errorf`,`FailNow`,`Fatal`,`FatalIf`方法，说明测试不通过，调用`Log`方法用来记录测试的信息。

## Go语言测试分类

Go语言的 testing 包提供了三种测试方式，分别是单元（功能）测试、性能（压力）测试和覆盖率测试。其中XXX为要测试方法的名称。

- 单元测试：TestXXX (t *testing.T)
- 基准（性能/压力）测试：BenchmarkXXX (b *testing.B)
- Main 函数测试：TestMain (m *testing.M)
- 控制台测试 ExampleXXX ()


### 单元测试

举一个例子，比如，当前目录下有两个demo文件
```shell
[root@xeq-vm-197-77-opsorder-test lesson2]# ls -l
total 12
-rw-r--r-- 1 root root  96 Aug  9 14:45 demo.go
-rw-r--r-- 1 root root 134 Aug  9 14:47 demo_test.go
-rw-r--r-- 1 root root  32 Aug  9 14:48 go.mod
```
其中demo的内容为
```go
package demo

// GetArea ...
func GetArea(weight, height int) int {
    return weight * height
}
```
demo_test.go的内容为
```go
package demo

import "testing"

func TestGetArea(t *testing.T) {
	area := GetArea(40, 50)
	if area != 2000 {
		t.Error("失败")
	}
}
```
我们执行测试
```shell
# 直接使用go test默认是不会显示测试通过的信息的，只会显示测试失败的信息，如果说想要查看测试通过的信息可以带上-v参数。
[root@xeq-vm-197-77-opsorder-test lesson2]# go test -v
=== RUN   TestGetArea
--- PASS: TestGetArea (0.00s)
PASS
ok      gostudy/lesson2 0.005s
```

### 压力测试

改造一下`demo_test.go`文件

```go
package demo

import "testing"

func BenchmarkGetArea(t *testing.B) {
    for i := 0; i < t.N; i++ {
        GetArea(40, 50)
    }
}
```
这里benchmark压力测试，用的就是`*testing.B`，接下来我们看执行的结果
```shell
[root@xeq-vm-197-77-opsorder-test lesson2]# go test -bench="."
goos: linux
goarch: amd64
pkg: gostudy/lesson2
cpu: Intel Core Processor (Broadwell)
BenchmarkGetArea-8      1000000000               0.4688 ns/op
PASS
ok      gostudy/lesson2 0.528s
```
这里显示运行了10亿次，每一次循环耗时0.4688 ns。压力测试需要注意以下几个点

- go test 不会默认执行压力测试的函数，如果要执行压力测试需要带上参数-bench


### 覆盖率测试

覆盖率测试能知道测试程序总共覆盖了多少业务代码（也就是 demo_test.go 中测试了多少 demo.go 中的代码），可以的话最好是覆盖100%。

将 demo_test.go 代码改造成如下所示的样子：
```go
package demo

import "testing"
func TestGetArea(t *testing.T) {
    area := GetArea(40, 50)
    if area != 2000 {
        t.Error("测试失败")
    }
}
func BenchmarkGetArea(t *testing.B) {
    for i := 0; i < t.N; i++ {
        GetArea(40, 50)
    }
}
```
然后运行覆盖率测试
```shell
[root@xeq-vm-197-77-opsorder-test lesson2]# go test -cover
PASS
coverage: 100.0% of statements
ok      gostudy/lesson2 0.004s
```

## 工具

### 使用gotests自动生成测试代码

> [gotests](https://github.com/cweill/gotests)支持official Sublime Text 3 plugin，emacs，vscode，Vim还有GoLand。gotest可以为单独一个源文件生成测试，也可以为整个一个目录生成测试文件。默认情况下，gotests将输出打到stdout标准输出。

使用go get下载

```shell
go get -u github.com/cweill/gotests/...
```

gotests使用指南

```shell
[root@xeq-vm-197-77-opsorder-test lesson2]# gotests --help
Usage of gotests:
  -all
        generate tests for all functions and methods
  -excl string
        regexp. generate tests for functions and methods that don't match. Takes precedence over -only, -exported, and -all
  -exported
        generate tests for exported functions and methods. Takes precedence over -only and -all
  -i    print test inputs in error messages
  -nosubtests
        disable generating tests using the Go 1.7 subtests feature
  -only string
        regexp. generate tests for functions and methods that match only. Takes precedence over -all
  -parallel
        enable generating parallel subtests
  -template string
        optional. Specify custom test code templates, e.g. testify. This can also be set via environment variable GOTESTS_TEMPLATE
  -template_dir string
        optional. Path to a directory containing custom test code templates. Takes precedence over -template. This can also be set via environment variable GOTESTS_TEMPLATE_DIR
  -template_params string
        read external parameters to template by json with stdin
  -template_params_file string
        read external parameters to template by json with file
  -w    write output to (test) files instead of stdout
```


## 参考

- [搞定Go单元测试（一）——基础原理](https://juejin.cn/post/6844903853528186894)
- [搞定Go单元测试（二）—— mock框架(gomock)](https://juejin.cn/post/6844903853532381198)
- [搞定Go单元测试（三）—— 断言（testify）](https://juejin.cn/post/6844903853532397581)
- [搞定Go单元测试（四）—— 依赖注入框架(wire)](https://juejin.cn/post/6844903853536575501)
- [Go 性能调优之 —— 基准测试](https://segmentfault.com/a/1190000016354758)
- [如何有效地测试Go代码](https://zhuanlan.zhihu.com/p/360646081)
- [Golang 单元测试：有哪些误区和实践？](https://www.infoq.cn/article/3o9mnjlsejkfifvbmrrn)
- [一个 Golang 项目的测试实践全记录](https://mp.weixin.qq.com/s?__biz=MzU1ODEzNjI2NA==&mid=2247487177&idx=3&sn=e7841ab3cc1c990c73fdca398f8f25c1&source=41#wechat_redirect)
- [go test命令（Go语言测试命令）完全攻略](http://c.biancheng.net/view/124.html)
- [go test 详解](https://maiyang.me/post/2018-11-14-go-test/)
- [如何用好 Go 的测试黑科技](https://learnku.com/articles/39971)
- [Go 怎么写测试用例](https://learnku.com/docs/build-web-application-with-golang/how-113-go-writes-test-cases/3224)
- [Golang Testing 概览 - 基本篇](https://hedzr.com/golang/testing/golang-testing-1/)