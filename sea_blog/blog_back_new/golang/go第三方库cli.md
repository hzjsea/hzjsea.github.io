---
title: go第三方库cli
date: 2021-03-11 10:29:30
categories:  [golang]
tags: [golang]
---


<!--more-->


# go第三方库cli

> golang中两个命令行的库，一个是cobra 一个是cli这个库，前者的start多点，但是用起来怪怪的，需要工程化项目，cli则比较轻量级


其实不管是用flag原装的 还是 cobra 还是cli这个库 原理都差不多

用flag去读取键值对参数 将value赋值给已经设定好的变量， 再用对应的方法去处理逻辑 

```golang
package command

import (
	"github.com/urfave/cli"
	"log"
	"os"
)

var (
	i string
	nqa string
	key string
)




func FlagInit()  {

	app := cli.NewApp()
	app.Name = "Snmp Tool"
	app.Usage = "Tool to Switch"
	app.HideVersion = true
	app.Version = "1.0.1"

	app.Commands = []cli.Command{
		{
			Name: "inteface",
			Aliases: []string{"interface"}, // 子命令
			Usage: "single command",
			Category: "Interface info ",
			Flags : []cli.Flag{
				cli.StringFlag{
					Name:  "i, interface, switch-interface",  // 键值对关键词 key
					Usage: "input swith interface",
					Value: "47",
					Destination: &i,
				},
			},
			Action: Action,
		},
		{
			Name: "nqa",
			Aliases: []string{"nqa"}, // 子命令
			Usage: "nqa cal",
			Category: "Nqa info",
			Action: NqaAction,
			Flags: []cli.Flag{
				cli.StringFlag{
					Name: "nqa",  // 键值对关键词 key
					Usage: "nqa collect",
					Value: "",
					Destination: &nqa, // value 对应的值
				},
			},
		},
		{
			Name: "search",
			Aliases: []string{"search"}, // 子命令
			Usage: "./command search --key xxx.xxx.xxx.xxx",
			Category: "OID info",
			Action: func(c *cli.Context) {
				if c.String("oid") == ""{
					cli.ShowAppHelp(c)
				}else {
					mibFile := GetMibFile()
					ParsingFile(mibFile,key)
				}
			},
			Flags: []cli.Flag{
				cli.StringFlag{
					Name: "oid", // 键值对 key
					Usage: "oid search",
					Value: "",
					Destination: &key,
				},
			},
		},

	}

	err := app.Run(os.Args)
	if err != nil {
		log.Fatal(err)
	}
}


func Action(c *cli.Context) error {

	if c.String("i") != ""{
		WatchInterface(i)
	} else {
		cli.ShowAppHelp(c)
	}
	return nil
}


func NqaAction(c *cli.Context) error {
	if c.String("nqa") == ""{
		NqaCollection()
	}else {
		cli.ShowAppHelp(c)
	}
	return nil
}
```