---
title: go第三方库cobra
categories:
  - golang
tags:
  - golang
abbrlink: 2a50f6cb
date: 2020-09-15 10:03:23
---


<!-- more -->


# go库cobra使用

cobra是一个命令提示工具，docker k8s里面的命令都是使用它来完成自动提示的



## 使用
```bash
# 使用golang 创建项目 mkdir gocommand
# 进入文件夹 创建mod包
go mod init gocommand
# 拉取cobra项目
go get -u github.com/spf13/cobra/cobra
# 使用cobra创建项目,cobra自带一套项目模板
cobra init --pkg-name Gocommand
# 添加指令 run
cobra add run 
```

项目框架如下
```go
.
├── LICENSE   // 证书
├── cmd       // 存放app
│   ├── completion.go
│   └── root.go // 子程序调用主程序
├── go.mod   // mod包
├── go.sum   // 
└── main.go     // 主程序
```


### Configuring the cobra generator
生成器配置文件，go同时还提供了一个配置文件,文件地址为`~/.cobra.yaml`
```go
author: Steve Francia <spf@spf13.com>
year: 2020
license:
  header: This file is part of CLI application foo.
  text: |
    {{ .copyright }}

    This is my license. There are many like it, but this one is mine.
    My license is my best friend. It is my life. I must master it as I must
    master my life.
```


## cobra执行流程以及源码分析
当前情况是不使用的配置的情况下
1. 进入`/project/main.go` 这里面他会去引用当前目录下面的cmd文件夹，cmd文件下面存放的就是指令
2. 进入`/project/cmd/root.go` 这里面有三个方法，分别是`init`, `initconfig` ,`execute`, 执行顺序后面再说
3. 进入指令程序，这里创建了一个`cobra add run`, 只有一个`init` 方法

> main.go


```go
package main
import "gocommand/cmd"
func main() {
	cmd.Execute() // 调用了cmd/root.go/Execute(func)
}
```

> cmd/root.go

```go
var cfgFile string

// rootCmd represents the base command when called without any subcommands
var rootCmd = &cobra.Command{  // 一个cmd对象
	Use:   "Gocommand",
	Short: "short desc",
	Long: "long desc"
	//	Run: func(cmd *cobra.Command, args []string) { },
}

func Execute() {  //被main.go 调用
	if err := rootCmd.Execute(); err != nil {
		fmt.Println(err)
		os.Exit(1)
	}
}

func init() {  // 初始化
	cobra.OnInitialize(initConfig)
	rootCmd.PersistentFlags().StringVar(&cfgFile, "config", "", "config file (default is $HOME/.Gocommand.yaml)")
	rootCmd.Flags().BoolP("toggle", "t", false, "Help message for toggle")
}

func initConfig() {} // 如果没有配置文件的话，可以先不看
```

上面的执行顺序分别是
1. init
2. execute
3. initconfig

> cmd/run.go

```go
var runCmd = &cobra.Command{
	Use:   "run",
	Short: "short desc",
	Long: "long desc"
	Run: func(cmd *cobra.Command, args []string) {  // 匿名函数，指令所执行的方法
		fmt.Println("run called")   
	},
}

func init() {
	rootCmd.AddCommand(runCmd)  // rootCmd是root下面声明的结构体 runCmd是run.go下面声明的变量
}

```


## 添加子命令

```bash
cobra add run # 创建子命令
# 取消Run的注释
Run: func(cmd *cobra.Command, args []string) {
	fmt.Println("run called")
}
# 运行
go run main.go run
```

## 添加多个命令继承
命令的继承主要在于`init()`这个函数下面
```bash
func init() {
	rootCmd.AddCommand(runCmd) # 这里表示的是将runCmd 放到rootCmd下面 也就是第一个出来的子命令
}
#  创建一个新的子命令
cobra add child_run
#  修改内容如下
func init() {
	runCmd.AddCommand(childRunCmd)
}
# 运行
go run main.go run childRun
go run main.go childRun # 报错
```

## 添加Flag
```go
func init() {
	rootCmd.AddCommand(runCmd)
	runCmd.Flags().String("name","hello","echo ")
}
// 获取写入的数据
Run: func(cmd *cobra.Command, args []string) {
	fmt.Println(cmd.Flag("name").Value)
}

// 运行程序
go run main.go run --name xxds
```

### 多种类型的flag


cobra支持多种类型的flag，同时支持不同的特性，这里拿string来说
![2020-11-20-09-55-06](http://noback.upyun.com/2020-11-20-09-55-06.png)

- `String` 所要填充的内容分别是 (全名,默认值，注释)
- `StrngP` 所要填充的内容分别是 (全名，短名，默认值，注释)
- `StringVar` 所要填充的内容是 (变量地址，全名，默认值，注释)
- `StringVarP` 所要填充的内容是 (变量地址，全名，短名，默认值，注释)

把上面的String换成其他的都适用，比如换成`int intP intVar intVarP`

另外最好的使用方式是使用后面两种
```go
var split int;
runCmd.Flags().IntVar(&split,"split",2,"split string")


// Flags:
//   -h, --help          help for run
//       --name string   help (default "hello")
//       --split int     split string (default 2)
```


### 全局Flag与本地flag
如果需要有一个flag能在所有指令下面都可以使用的，可以设定一个全局Flag，建议设置在`/root.go`下面

```go
// root.go
var version string
var verbose string
func init() {
	cobra.OnInitialize(initConfig)
	rootCmd.PersistentFlags().StringVarP(&version,"version","v","6.12.1","show version")
	rootCmd.PersistentFlags().StringVar(&verbose,"verbose","6.12.1","show verbose")
}

	Run: func(cmd *cobra.Command, args []string) {
		fstatus,_ := cmd.Flags().GetBool("version")
		if fstatus{
			fmt.Println("version is 20.0.1")
		}
	}
	

// go run main.go run --version
// version is 20.0.1

```


## TAB的静态补全与动态补全

cobra支持静态补全与动态补全

静态补全就是 在之前就在数组中设置好值
动态补全就是 可以动态的获取值到数组中

### 静态名称补全

创建如下项目
![2021-01-21-10-17-38](http://noback.upyun.com/2021-01-21-10-17-38.png)

静态名称补全比较简单，只要在想要自动补全的子命令中加入 ValidArgs 字段，传入一组包含预期结果的字符串数组即可，代码如下：
```go
// desc.go 
package cmd

import (
	"fmt"

	"github.com/spf13/cobra"
)

var valids []string = []string{ "pod", "node", "service", "replicationcontroller" }  // 添加的补全内容

// descCmd represents the desc command
var descCmd = &cobra.Command{
	Use:   "desc",
	Short: "A brief description of your command",
	Long: `A longer description that spans multiple lines and likely contains examples
and usage of using your command. For example:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.`,
	ValidArgs:  valids,
	Run: func(cmd *cobra.Command, args []string) {
		fmt.Println("desc called")
	},
}

func init() {
	rootCmd.AddCommand(descCmd)
}

```

```go
// completion.go
// 创建一个completionCmd
cobra add completion
// 复制粘贴 这是cobra官方提供的
var completionCmd = &cobra.Command{
	Use:                   "completion [bash|zsh|fish|powershell]",
	Short:                 "Generate completion script",
	Long: `To load completions:

Bash:

$ source <(yourprogram completion bash)

# To load completions for each session, execute once:
Linux:
  $ yourprogram completion bash > /etc/bash_completion.d/yourprogram
MacOS:
  $ yourprogram completion bash > /usr/local/etc/bash_completion.d/yourprogram

Zsh:

# If shell completion is not already enabled in your environment you will need
# to enable it.  You can execute the following once:

$ echo "autoload -U compinit; compinit" >> ~/.zshrc

# To load completions for each session, execute once:
$ yourprogram completion zsh > "${fpath[1]}/_yourprogram"

# You will need to start a new shell for this setup to take effect.

Fish:

$ yourprogram completion fish | source

# To load completions for each session, execute once:
$ yourprogram completion fish > ~/.config/fish/completions/yourprogram.fish
`,
	DisableFlagsInUseLine: true,
	ValidArgs:             []string{"bash", "zsh", "fish", "powershell"},
	Args:                  cobra.ExactValidArgs(1),
	Run: func(cmd *cobra.Command, args []string) {
		switch args[0] {
		case "bash":
			cmd.Root().GenBashCompletion(os.Stdout)
		case "zsh":
			cmd.Root().GenZshCompletion(os.Stdout)
		case "fish":
			cmd.Root().GenFishCompletion(os.Stdout, true)
		case "powershell":
			cmd.Root().GenPowerShellCompletion(os.Stdout)
		}
	},
}

// 针对不同的版本进行不同的操作
// 比如bash
source <(go run main.go completion bash)
// 重启终端就OK了
```
completion这个命令其实就是生成了一份补全命令的脚本，可以直接run一下看看
```go
go run main.go completion bash
```

> kubectl ge-bash: _get_comp_words_by_ref: command not found

报错内容如上，这个是因为linux系统里面没有bash_completion，需要安装一下
```bash
yum install -y bash-completion
source /usr/share/bash-completion/bash_completion
source <(kubectl completion bash)
```


## cobra源码分析
第一次分析源码，会有点乱

`main.go` 下面没什么好分析的
```go
package main

import "Gocommand/cmd"

func main() {
	cmd.Execute()   // from /cmd/root.go/  Execute()
}

```

```go
package cmd

import (
	"fmt"
	"github.com/spf13/cobra"
	"os"

	homedir "github.com/mitchellh/go-homedir"
	"github.com/spf13/viper"
)

var cfgFile string

var rootCmd = &cobra.Command{
	Use:   "Gocommand",     // 这个不能改，改了运行命令就不一样了
	Short: "short desc",   
	Long: "long desc"
	Run: func(cmd *cobra.Command, args []string) {
		fmt.Println("helloworld")
	 },
}

// 被main.go 调用
func Execute() {
	if err := rootCmd.Execute(); err != nil {
		fmt.Println(err)
		os.Exit(1)
	}
}

func init() {
	cobra.OnInitialize(initConfig)
	rootCmd.PersistentFlags().StringVar(&cfgFile, "config", "", "config file (default is $HOME/.Gocommand.yaml)")
}

func initConfig() {
	if cfgFile != "" {
		viper.SetConfigFile(cfgFile)
	} else {
		home, err := homedir.Dir()
		if err != nil {
			fmt.Println(err)
			os.Exit(1)
		}
		viper.AddConfigPath(home)
		viper.SetConfigName(".Gocommand")
	}

	viper.AutomaticEnv()
	if err := viper.ReadInConfig(); err == nil {
		fmt.Println("Using config file:", viper.ConfigFileUsed())
	}
}
```






```go
func (c *Command) ExecuteC() (cmd *Command, err error) {
    // 判断 ctx字段是否为nil

	if c.ctx == nil {
		c.ctx = context.Background()
	}

    // Regardless of what command execute is called on, run on Root only
    // 前面说过，我们所有的命令构成了一棵树，每次执行都是从根命令开始执行的
	// 如果我们手残在代码的某处直接执行了一个子命令，这个方法也会向上遍历到根，然后执行根命令的
	if c.HasParent() {
		return c.Root().ExecuteC()
	}

    // windows hook
    // windows系统的钩子
	if preExecHookFn != nil {
		preExecHookFn(c)
	}

	// initialize help as the last point possible to allow for user
	// overriding
	c.InitDefaultHelpCmd()

	args := c.args

    // Workaround FAIL with "go test -v" or "cobra.test -test.v", see #155
    // 如果没有设置过命令的调用参数，使用os.Args[1:]作为默认参数
	if c.args == nil && filepath.Base(os.Args[0]) != "cobra.test" {
		args = os.Args[1:]
	}

	// initialize the hidden command to be used for bash completion
	c.initCompleteCmd(args)

    // 找真正要执行的命令，并解析flag
	// cobra为我们提供了两种flag，一种叫 persistant flag，
	// 这种 flag 可以在子命令中也获取到
	// 另外一种叫 local flag
	// 这种 flag 只会在指定命令中被解析
	// 默认情况下，cobra 只会解析目标命令的 local flag，父命令中的local flag将会被忽略
	// 可以将命令的 TraverseChildren 属性置为 true ，cobra 会在执行命令前解析所有命令
	// 的 local flag
	var flags []string
	if c.TraverseChildren {
		cmd, flags, err = c.Traverse(args)
	} else {
		cmd, flags, err = c.Find(args)
	}
	if err != nil {
		// If found parse to a subcommand and then failed, talk about the subcommand
		if cmd != nil {
			c = cmd
		}
		if !c.SilenceErrors {
			c.Println("Error:", err.Error())
			c.Printf("Run '%v --help' for usage.\n", c.CommandPath())
		}
		return c, err
	}

	cmd.commandCalledAs.called = true
	if cmd.commandCalledAs.name == "" {
		cmd.commandCalledAs.name = cmd.Name()
	}

	// We have to pass global context to children command
	// if context is present on the parent command.
	if cmd.ctx == nil {
		cmd.ctx = c.ctx
	}

	err = cmd.execute(flags)
	if err != nil {
		// Always show help if requested, even if SilenceErrors is in
		// effect
		if err == flag.ErrHelp {
			cmd.HelpFunc()(cmd, args)
			return cmd, nil
		}

		// If root command has SilentErrors flagged,
		// all subcommands should respect it
		if !cmd.SilenceErrors && !c.SilenceErrors {
			c.Println("Error:", err.Error())
		}

		// If root command has SilentUsage flagged,
		// all subcommands should respect it
		if !cmd.SilenceUsage && !c.SilenceUsage {
			c.Println(cmd.UsageString())
		}
	}
	return cmd, err
}
```
判断一些字段是否存在，并作出相应的动作

主要是看这两个方法
![2020-09-15-10-40-07](http://noback.upyun.com/2020-09-15-10-40-07.png)

> 先来看看Traverse这个函数

![2020-09-15-10-59-23](http://noback.upyun.com/2020-09-15-10-59-23.png)
遍历四种形式的参数 将他们添加到flags当中去




