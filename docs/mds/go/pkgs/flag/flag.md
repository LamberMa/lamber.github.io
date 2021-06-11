> 在golang中有很多方式来处理命令行参数，比如常见的有pflag，cobra（Kubernetes中用到了该库）等。当然我们还可以使用系统默认的`os.Args`。但是在Go
标准库中提供了flag库可以实现该功能，我们可以简单来看一下。