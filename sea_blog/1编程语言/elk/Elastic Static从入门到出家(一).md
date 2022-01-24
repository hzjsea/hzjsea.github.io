# Elastic Static从入门到出家(一)

## 安装
ElasticSearch是基于Java的,因此我们需要准备JDK的环境

- 准备JDK环境
	https://www.oracle.com/java/technologies/jdk8-downloads.html
- 下载安装elasticSearch
	https://www.elastic.co/cn/downloads/elasticsearch
- 启动elasticSearch

注意:
elasticSearch简称为es ，在es7.5版本中,java环境已经被适配捆包在了es的安装包中，因此我们不用去安装es的环境，如果有特殊的需要则要自己配置JAVA_HOME

如何启动?
切换到安装目录,使用bin\elasticsearch启动项目,命令行出现started表示启动成功，地址在127.0.0.1:9200
## es中的配置文件
由于学习的内容是5版本的,仅仅只有三个文件，在7版本中存在很多个文件，这里我们先介绍一下这三个文件
## 学习地址
该篇作为Elastic Static的入门篇，地址来自慕课网 https://coding.imooc.com/lesson/181.html#mid=11117