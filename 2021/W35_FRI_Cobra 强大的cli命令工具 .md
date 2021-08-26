# Corba 强大的cli命令工具

## 1、corba简介
	corba是目前最流行的命令行工具编写库，其友好的信息提示，简洁明了的嵌套层级，对命令行参数的便捷解析，让其成为众多优秀软件的cli编写库。
## 2、cobra安装
    go get -v [github.com/spf13/cobra/cobra](http://github.com/spf13/cobra/cobra)
## 3、cobra 的命令
 	下面我们以一个例子来完成cobra的使用说明。此案例基于golang module 包管理机制之下。
#### 1. cobra init demo1 —pkg-name demo 
此命令创建一个名为demo1的文件夹生成带有根命令的目录结构及文件。其中包的引用结构的根节点为demo。
```
    .
    ├── cmd
    │   └── root.go
    ├── LICENSE
    └── main.go
```
#### 2. go mod init demo 
初始化模块对应上命令的—pkg-name值。
此时命令行程序已经完整。go build即可生成可执行文件。
#### 3. cobra add sub1添加一个sub1的子命令
```
	.
	├── cmd
	│   ├── root.go
	│   └── sub1.go
	├── demo
	├── go.mod
	├── go.sum
	├── LICENSE
	└── main.go
```
可以看到cmd目录中增加了sub1.go的文件。重新生成可执行文件，验证子命令的成功添加。

```go
    Usage:
      demo [command]

    Available Commands:
      completion  generate the autocompletion script for the specified shell
      help        Help about any command
      sub1        A brief description of your command

    Flags:
          --config string   config file (default is $HOME/.demo.yaml)
      -h, --help            help for demo
      -t, --toggle          Help message for toggle
```

## 4、参数，标识

### 参数

参数用来确定要运行的子命令，cobra工具自动生成文件只支持一级子命令。但是通过对cmd中代码分析我们发现通过修改cmd/xxx.go文件中的init函数我们可以做到多层级子命令。

```go
func init() {
        rootCmd.AddCommand(sub1Cmd) // => xxxCmd.AddCommand(sub1Cmd)
}
```

> 例：./demo sub1 sub2的命令则需要修改sub2.go中init函数，sub1Cmd.AddCommand(sub2Cmd)。

### 标识

标识通常为命令的执行提供所需的配置和所需数值。

标识可以分为两类：一类是全局标识，另一类是局部标识。

### 全局标识的获取

```jsx
globalstr = xxxCmd.PersistentFlags().String("foo","default","help of foo")
globalstrP = xxxCmd.PersistentFlags().StringP("foo","f","default","help of foo")
xxxCmd.PersistentFlags().StringVar("foo","default","help of foo")
xxxCmd.PersistentFlags().StringVarP(&str, "foo", "f", "", "A help for foo")
```

可以看到获取参数的方法一共四种。

前两种会返回一个类型的指针，后两种会将获得的值存入传入的地址中。

第一种和第三种无法设置shorthand。

### 局部标识获取

局部标识仅仅当此命令运行时会去获取标识的值。

```go
curstr = xxxCmd.Flags().String("foo","default","help of foo")
curstrP = xxxCmd.Flags().StringP("foo","f","default","help of foo")
xxxCmd.Flags().StringVar("foo","default","help of foo")
xxxCmd.Flags().StringVarP(&str, "foo", "f", "", "A help for foo")
```

下面我们通过-h命令查看以上两种。

其中root.go文件中的init函数为

```go
//root.go
func init(){
	rootCmd.PersistentFlags().StringVar(&cfgFile, "config", "", "config file (default is $HOME/.demo.yaml)")
	rootCmd.PersistentFlags().StringVarP(&rootfoo, "flag", "f", "xx", "param")
	rootCmd.Flags().BoolP("toggle", "t", false, "Help message for toggle")
}

//./demo 
Available Commands:
  completion  generate the autocompletion script for the specified shell
  help        Help about any command
  test        A brief description of your command

Flags:
      --config string   config file (default is $HOME/.demo1.yaml)
  -f, --flag string     param (default "xx")
  -h, --help            help for demo
  -t, --toggle          Help message for toggle-h

```

test.go中init函数为

```go
test.go
func init(){
	testCmd.Flags().String("testfoo", "default", "A help for foo")
	testCmd.Flags().StringP("testfoo2", "t", "defalut", "A help for foo")
}

// ./demo test -h
Available Commands:
  test2       A brief description of your command

Flags:
  -h, --help              help for test
      --testfoo string    A help for foo (default "default")
  -t, --testfoo2 string   A help for foo (default "defalut")

Global Flags:
      --config string   config file (default is $HOME/.demo1.yaml)
  -f, --flag string     param (default "xx")
```

我们可以看出子命令./demo test 的帮助文档中存在两个全局标识，其中一个拥有shorthand。

## 5、命令执行体

这一部分比较简单，执行体就在cmd/xxx.go文件中xxxCmd结构体中的Run函数中。

```go
var testCmd = &cobra.Command{
        Use:   "test",
        Short: "A brief description of your command",
        Long: `A longer description that spans multiple lines and likely contains examples
and usage of using your command. For example:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.`,
        Run: func(cmd *cobra.Command, args []string) {
                fmt.Println("test called", viper.GetString("hello"))
        },
}
```

# 总结
Corba库以树形结构的形式组织命令行，根据输入参数和配置项来组织对应的调用函数。
使用步骤为：
* corba init 创建项目。
* go init 创建module。
* corba add xxx 添加子命令。
* 编写cmd/xxx.go中的init函数，组织树形命令结构。
* 编写cmd/xxx.go文件的init函数，添加命令所需各类参数。
* 编写cmd/xxx.go文件中的xxxCmd结构体，编辑命令说明信息及调用的Run函数。

## 参考资料

- https://github.com/spf13/cobra
