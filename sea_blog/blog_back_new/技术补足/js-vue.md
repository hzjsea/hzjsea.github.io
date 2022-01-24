---
title: js技能补足
categories: [js]
tags: [js]
abbrlink: cb97dd69
date: 2020-07-10 18:29:34
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [js补足](#js补足)
  - [npm与yarn的区别](#npm与yarn的区别)
    - [使用yarn](#使用yarn)
    - [yarn命令使用](#yarn命令使用)
  - [Vue-cli 使用](#vue-cli-使用)
    - [vue项目配置](#vue项目配置)
    - [插件与管理](#插件与管理)
    - [vue项目配置之vue.config.js](#vue项目配置之vueconfigjs)
  - [axios使用](#axios使用)

<!-- /code_chunk_output -->
<!-- more -->



# js补足
js这个东西怪怪的

## npm与yarn的区别

Yarn是由Facebook、Google、Exponent 和 Tilde 联合推出了一个新的 JS 包管理工具 ，正如官方文档中写的，Yarn 是为了弥补 npm 的一些缺陷而出现的。”这句话让我想起了使用npm时的坑了
- `npm install` 速度慢 特别是新的项目拉下来要等半天，删除node_modules，重新install的时候依旧如此
- 同一个项目，安装的时候无法保持一致性。由于package.json文件中版本号的特点，下面三个版本号在安装的时候代表不同的含义

```bash
"5.0.3",
"~5.0.3",
"^5.0.3"
```
“5.0.3”表示安装指定的5.0.3版本，“～5.0.3”表示安装5.0.X中最新的版本，“^5.0.3”表示安装5.X.X中最新的版本。这就麻烦了，常常会出现同一个项目，有的同事是OK的，有的同事会由于安装的版本不一致出现bug。
- 安装的时候，包会在同一时间下载和安装，中途某个时候，一个包抛出了一个错误，但是npm会继续下载和安装包。因为npm会把所有的日志输出到终端，有关错误包的错误信息就会在一大堆npm打印的警告中丢失掉，并且你甚至永远不会注意到实际发生的错误。

### 使用yarn
- 速度快
    - 并行安装
        并行安装：无论 npm 还是 Yarn 在执行包的安装时，都会执行一系列任务。npm 是按照队列执行每个 package，也就是说必须要等到当前 package 安装完成之后，才能继续后面的安装。而 Yarn 是同步执行所有任务，提高了性能。
    - 离线模式
        如果之前已经安装过一个软件包，用Yarn再次安装时之间从缓存中获取，就不用像npm那样再从网络下载了
        
- 安装版本统一
    为了防止拉取到不同的版本，Yarn 有一个锁定文件 (lock file) 记录了被确切安装上的模块的版本号。每次只要新增了一个模块，Yarn 就会创建（或更新）yarn.lock 这个文件。这么做就保证了，每一次拉取同一个项目依赖时，使用的都是一样的模块版本。npm 其实也有办法实现处处使用相同版本的 packages，但需要开发者执行 npm shrinkwrap 命令。这个命令将会生成一个锁定文件，在执行 npm install 的时候，该锁定文件会先被读取，和 Yarn 读取 yarn.lock 文件一个道理。npm 和 Yarn 两者的不同之处在于，Yarn 默认会生成这样的锁定文件，而 npm 要通过 shrinkwrap 命令生成 npm-shrinkwrap.json 文件，只有当这个文件存在的时候，packages 版本信息才会被记录和更新

- 输出间接
    更简洁的输出：npm 的输出信息比较冗长。在执行 npm install  的时候，命令行里会不断地打印出所有被安装上的依赖。相比之下，Yarn 简洁太多：默认情况下，结合了 emoji直观且直接地打印出必要的信息，也提供了一些命令供开发者查询额外的安装信息。。
    
### yarn命令使用
```bash
npm                             yarn 

npm install                      yarn
npm install react --save          yarn add react
npm uninstall react --save        yarn remove react
npm install react --save-dev      yarn add react --dev
npm update --save                yarn upgrade
```

##  Vue-cli 使用
安装
```bash
yarn global add @vue/cli
vue create my-project
# 使用图形界面
vue ui
```

### vue项目配置
全局CLI配置
有些针对`@vue/cli`的全局配置，例如你惯用的包管理器和你本地保存的 preset，都保存在 home 目录下一个名叫`.vuerc`的 JSON 文件。你可以用编辑器直接编辑这个文件来更改已保存的选项。
你也可以使用`vue config`命令来审查或修改全局的CLI配置

```bash
vue config                                                                                     
Resolved path: /Users/alpaca/.vuerc
 {
  "useTaobaoRegistry": true,
  "packageManager": "npm",
  "presets": {
    "first": {
      "useConfigFiles": true,
      "plugins": {
        "@vue/cli-plugin-babel": {},
        "@vue/cli-plugin-router": {
          "historyMode": true
        },
        "@vue/cli-plugin-eslint": {
          "config": "base",
          "lintOn": [
            "save"
          ]
        }
      },
      "cssPreprocessor": "dart-sass"
    }
  },
  "latestVersion": "4.2.3",
  "lastChecked": 1584209986740
}
```

### 插件与管理
Vue CLI 使用了一套基于插件的架构。如果你查阅一个新创建项目的 `package.json`，就会发现依赖都是以 `@vue/cli-plugin-` 开头的

<font color='red'>需要注意的内容，在下载插件之前，可以先提交自己项目的最新状态到git仓库，因为新的插件可能会影响vue文件的布局或者其他改动</font>

```bash
# 安装插件
vue add eslint 
```
插件的安装流程， 将`@vue/eslint` 解析为完整的包 `@vue/cli-plugin-eslint` 然后使用npm安装

```bash
vue add apollo # -> vue add @vue/cli-plugin-apollo 
```

### vue项目配置之vue.config.js

全局CLI配置
有些针对 `@vue/cli` 的全局配置，例如你惯用的包管理器和你本地保存的 preset，都保存在 home 目录下一个名叫 `.vuerc` 的 JSON 文件。你可以用编辑器直接编辑这个文件来更改已保存的选项。
你也可以使用 `vue config` 命令来审查或修改全局的 CLI 配置
```bash
alpaca:技术补足/ (master) $ vue config
 {
  "useTaobaoRegistry": true,
  "packageManager": "npm",
  "presets": {
    "first": {
      "useConfigFiles": true,
      "plugins": {
        "@vue/cli-plugin-babel": {},
        "@vue/cli-plugin-router": {
          "historyMode": true
        },
        "@vue/cli-plugin-eslint": {
          "config": "base",
          "lintOn": [
            "save"
          ]
        }
      },
      "cssPreprocessor": "dart-sass"
    }
  },
  "latestVersion": "4.2.3",
  "lastChecked": 1584209986740
}
```


`vue.config.js`是一个可选的配置文件，如果项目的 (和 package.json 同级的) 根目录中存在这个文件，那么它会被 `@vue/cli-service` 自动加载

```bash
module.exports = {
    
}
```

`vue.config.js`中的一些配置
https://cli.vuejs.org/zh/config/#%E5%85%A8%E5%B1%80-cli-%E9%85%8D%E7%BD%AE




## axios使用
```bash
# 安装
npm install axios
# or 
yarn add axios
```