---
title: golang标准库path
date: 2021-02-07 13:41:10
categories:  [golang]
tags: [golang,golibrary]
---


<!--more-->


# golang标准库path

path这个库就是操作文件路径的库

```golang
func main(){
	paths,err := os.Getwd()

	// /Users/alpaca/GolandProjects/workspace_golang
	if err != nil{
		log.Fatal(err)
	}

	// 判断路径是否是绝对路径
	if path.IsAbs(paths){
		fmt.Println("绝对路径")
	}
	fmt.Println(paths) // /Users/alpaca/GolandProjects/workspace_golang

	//  切割路径
	dir,filename := path.Split(paths)
	fmt.Println(dir, filename) // /Users/alpaca/GolandProjects/  workspace_golang

	// 拼接路径
	basePath := "/testDir/"
	baseFilename := "demo.txt"
	completePath := path.Join(basePath,baseFilename)
	fmt.Println(completePath) // /testDir/demo.txt

	paths = "/Users/alpaca/GolandProjects/workspace_golang"
	filename = path.Base(paths)
	fmt.Println(filename)

	// 返回扩展名
	currentWorkspace,err := os.Open(paths)
	if err != nil{
		log.Fatal(err)
	}
	fileList,_:= currentWorkspace.Readdir(-1)
	for _,value :=range fileList{
		filePath := paths + "/" + value.Name()
		fmt.Println(path.Ext(filePath))
	}
}
```

![](http://noback.upyun.com/2021-02-07-13-55-28.png!)

> 有个很奇怪的问题path 和filepath 有部分的方法是相同的 ，但是却用了不同的方式去实现

![](http://noback.upyun.com/2021-02-07-14-38-09.png!)


## filepath

```golang
func main(){
	currentPath,err := os.Getwd()
	if err != nil{
		log.Fatal(err)
	}
	fmt.Println(currentPath) // /Users/alpaca/GolandProjects/workspace_golang

	fmt.Println(path.Split(currentPath)) // /Users/alpaca/GolandProjects/ workspace_golang
	fmt.Println(filepath.Split(currentPath)) // /Users/alpaca/GolandProjects/ workspace_golang

	fmt.Println(filepath.Join(currentPath, "demo", "test"))
	// /Users/alpaca/GolandProjects/workspace_golang/demo/test

	rootPath, err := os.Getwd()
	if err != nil{
		log.Fatal(err)
	}
	//Walk函数对每一个文件/目录都会调用WalkFunc函数类型值。调用时path参数会包含Walk的root参数作为前缀；就是说，如果Walk函数的root为"dir"，
	//该目录下有文件"a"，将会使用"dir/a"调用walkFn参数。walkFn参数被调用时的info参数是path指定的地址（文件/目录）的文件信息，类型为os.FileInfo。
	filepath.Walk(rootPath, func(path string, info os.FileInfo, err error) error {
		if err != nil{
			return err
		}
		fmt.Println(path)
		fmt.Println(info.Name())
		return nil
	})
}
```