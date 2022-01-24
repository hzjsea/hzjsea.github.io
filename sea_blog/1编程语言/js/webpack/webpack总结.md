首先我们需要安装webpack安装
npm install webpack webpack-cli --save-dev
会出现node_modules文件夹和package-lock.json文件
其中node_modules存放所有安装包， package-lock记录安装依赖关系
开始吧https://www.webpackjs.com/guides/getting-started/#基本安装
我们还需要调整 package.json 文件，以便确保我们安装包是私有的(private)，并且移除 main 入口。这可以防止意外发布你的代码
在安装一个要打包到生产环境的安装包时，你应该使用 npm install --save，如果你在安装一个用于开发环境的安装包（例如，linter, 测试库等），你应该使用 npm install --save-dev。请在 npm 文档 中查找更多信息。
```bash
cd webpacktest2
npm init -y
npm install webpack webpack-cli --save-dev  
```

## vue-cli 和 webpack 是什么关系
vue-cli是基于nodejs+webpack封装的命令行工具。你可以理解为汇集了各种命令的 bash，或者bat。
原本需要自己配置webpack的相关配置，被cli简化了。并且按照vue的用户习惯整理了一套构建和目录规范。
这样，你只要按照vue-cli的配置规则来，就可以满足很多繁琐的webpack+plugin配置。


## 聊一聊webpack的构建过程
1.首先我们需要创建一个文件夹来存放我们的项目文件  mkdir webpacktest2
2.进入文件，安装webpack 和 webpack-cli  

注意安装过程中，如果你想要安装生产版本的，那就是--save,但如果是开发版的则是--save-dev
npm init -y 会在目录中生成一个package.json的文件
npm install webpack webpack-cli 会在目录中生成一个 node_modules用于存放模块 以及package-lock.json用于控制模块版本

https://www.webpackjs.com/guides/getting-started/#基本安装