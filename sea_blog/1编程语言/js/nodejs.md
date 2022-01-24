# ES6

ECMAScript 是 javascript 的标准
javascript 是 ECMAScript 的实现
node 是 javascript 的服务器运行环境
node 对 ES6 的支持度更高。除了那些默认打开的功能，还有一些语法功能已经实现了，但是默认没有打开。使用下面的命令，可以查看 Node 已经实现的 ES6 特性。
es6地址: http://es6.ruanyifeng.com/#docs/intro

node.js 
https://www.liaoxuefeng.com/wiki/1022910821149312/1023025763380448

# node.js
由于Node.js平台是在后端运行JavaScript代码，所以，必须首先在本机安装Node环境。

## 下载与安装node.js

下载地址:
https://nodejs.org/en/download/

node -v 查看版本

## 下载与安装npm
npm是什么东东？npm其实是Node.js的包管理工具（package manager)
出现的原因与linux中的rpm类似。 包管理 解决依赖等等问题

安装完node.js之后  npm就已经安装完成了。

npm -v 查看版本

<font color='red'>之前的javascript都是在浏览器中运行的，但是之后js将会运行在node的环境中</font>


## 写第一个node.js
```js
// hello.js
'use strict';
console.log('hello,world');
```
运行
node hello.js

**严格模式**
其中在js文件开头加上 'use strict'; 是保证node执行该js文件的时候采用严格模式，但是为了方便，我们可以使用命令行的模式改变
node --use_strict xxx.js


**交互模式**
node的交互模式和python一样，在命令行中输入node
输入代码就行了
```js
> node
Welcome to Node.js v12.13.0.
Type ".help" for more information.
> 'use strict';
'use strict'
> console.log('hello,world');
hello,world
```

## 将node.js转成模块
之前我们说过npm是一个包管理工具，在这个包管理工具当中会存在很多的模块，这些模块是由js编写的，但是从js--->模块的过程中需要进行一些改变，

我们先摘抄一下之前的hello.js的文件
```js
// hello.js
'use strict';
console.log('hello,world');
```
对他进行改造变成模块，将js文件名从hello.js 改成 hello
```js
//hello
'use strict';

var s = 'hello';

function greet(name){
	console.log(s+','+ name + '!');
}
module.exports = greet;
```

首先将 内容定义为变量，再将动作变成方法，将输出直接暴露
暴露的形式为 module.export = variable;

再次使用node hello 的时候已经输出不了内容了，这时候我们需要在创建一个main.js文件来调用这些模块
```js
'use strict';
var greet_main = require('./tt');
var name = 'Michael';
greet_main(name);
```
很简单的理解，首先我们创建一个创建了一个模块hello，这个模块主要的作用是暴露了greet()这个方法,这个方法必须获取一个name的参数。之后我们使用main.js这个文件来调用这个模块 ，首先使用require将这个模块暴露出来的方法递给greet_main,再使用greet_main这个函数。

## CommonJS规范
在这个规范下，每个.js文件都是一个模块，它们内部各自使用的变量名和函数名都互不冲突，例如，hello.js和main.js都申明了全局变量var s = 'xxx'，但互不影响。
暴露与获取：一个模块想要对外暴露变量（函数也是变量），可以用module.exports = variable;，一个模块要引用其他模块暴露的变量，用var ref = require('module_name');就拿到了引用模块的变量。


## node.js对javascript的代码隔离
如果我们在多个js的文件中使用了s 这个变量，即 var s = "xxx"
这样在理论上会导致在引用的过程中造成冲突，但是node解决了这个问题

```js
var s = 'Hello';
var name = 'world';

console.log(s + ' ' + name + '!'); 
```

```js
 (function () {
    // 读取的hello.js代码:
    var s = 'Hello';
    var name = 'world';

    console.log(s + ' ' + name + '!');
    // hello.js代码结束
})();
```
他会在他的外部在包裹一个函数声明 这样全局变量变成了该函数的局部变量
```js
// 准备module对象:
var module = {
    id: 'hello',
    exports: {}
};
var load = function (module) {
    // 读取的hello.js代码:
    function greet(name) {
        console.log('Hello, ' + name + '!');
    }
    
    module.exports = greet;
    // hello.js代码结束
    return module.exports;
};
var exported = load(module);
// 保存module:
save(module, exported); 
```
内容暴露
暴露的方法有两种
**第一种**
```js
function hello(){
    console.log("hello,world");
} 
function greet(name){
    console.log("hello"+name+"!");
}

module.exports = {
    hello: hello;
    greet: greet;
};
```

**第二种**
```js
function hello(){
    console.log("hello,world");
} 
function greet(name){
    console.log("hello"+name+"!");
}
exports.hello = hello;
export.greet = greet;
```

## 基本模块
**全局对象**
在js中有且仅有一个全局对象window 但是在node.js中这个全局对象叫做global
```js
> global.console
{
  log: [Function: bound consoleCall],
  warn: [Function: bound consoleCall],
  dir: [Function: bound consoleCall],
  time: [Function: bound consoleCall],
  timeEnd: [Function: bound consoleCall],
  timeLog: [Function: bound consoleCall],
  trace: [Function: bound consoleCall],
  assert: [Function: bound consoleCall], 
}
```
**进程**
process代表了node.js的进程，通过process可以拿到许多有用的信息
可以通过>process +tab 来获取一些命令
```js
process.version;  //版本
process.cwd();  //返回当前工作目录
process.chdir('~/hzj/code') //切换当前工作目录
```
JavaScript程序是由事件驱动执行的单线程模型，Node.js也不例外。Node.js不断执行响应事件的JavaScript函数，直到没有任何响应事件的函数可以执行时，Node.js就退出了。

```js
'use strict';
// process.nextTick()将在下一轮事件循环中调用:
process.nextTick(function(){
        console.log("nexttick callback");
})
console.log("nexttinck set") 
```

### fs文件系统
文件系统模块，负责读写文件。fs模块提供异步和同步的方法。

**回顾js中的异步与同步**
```js
//异步执行
$.getJSON('http://example.com/ajax', function (data) {
    console.log('IO结果返回后执行...');
});
console.log('不等待IO结果直接执行后续代码...');
//同步执行
var data = getJSONSync('http://example.com/ajax');
```

**nodejs中的同步异步操作**
```js
//异步读取文件
'use strict';
var fs = require('fs');
fs.readFile('backup.txt','utf-8',function(err,data){
    if(err){
        console.log(err);
    } else{
        console.log(data);
    }
});

//同步执行
// 同步读取文件
'use strict';
var fs = require('fs');
var data = fs.readFileSync('backup.txt','utf-8');
console.log(data)
```

**nodejs写入数据**
```js 
//异步写入数据
'use strict';
var fs = require('fs');
var data = 'helloworld';
fs.writeFile('backup.txt',data,function(err){
    if(err){
        console.log(err);
    }else{
        console.log('ok');
    }
});

//同步写入数据
'use strict';

var fs = require('fs');

var data = 'Hello, Node.js';
fs.writeFileSync('output.txt', data);
```

## stat文件信息
```js
'use strict';

var fs = require('fs');

fs.stat('sample.txt', function (err, stat) {
    if (err) {
        console.log(err);
    } else {
        // 是否是文件:
        console.log('isFile: ' + stat.isFile());
        // 是否是目录:
        console.log('isDirectory: ' + stat.isDirectory());
        if (stat.isFile()) {
            // 文件大小:
            console.log('size: ' + stat.size);
            // 创建时间, Date对象:
            console.log('birth time: ' + stat.birthtime);
            // 修改时间, Date对象:
            console.log('modified time: ' + stat.mtime);
        }
    }
}); 
```


<font color='red'>由于Node环境执行的JavaScript代码是服务器端代码，所以，绝大部分需要在服务器运行期反复执行业务逻辑的代码，必须使用异步代码，否则，同步代码在执行时期，服务器将停止响应，因为JavaScript只有一个执行线程。服务器启动时如果需要读取配置文件，或者结束时需要写入到状态文件时，可以使用同步代码，因为这些代码只在启动和结束时执行一次，不影响服务器正常运行时的异步执行。</font>


## stream
在nodejs中，流也是一个对象，我们只需要响应流的事件就可以了，一共有三种状态，data表示流的数据可以读取了，end事件表示流已经到末尾了没有数据可以读取了，error表示出错的时候。

**读文件**
```js
'use strict';
var fs = require('fs');

var rs = fs.createReadStream('backup.txt','utf-8');

rs.on('end', function(){
	console.log("started")
})

rs.on('data',function(chunk){
	console.log('DATA:')
	console.log('chunk')
});

rs.on('end', function(chunk){
	console.log('END');
})

rs.on('error', function(err){
	console.log('ERROR'+err);
}); 
```
**取文件**
```js
 'use strict';

var fs = require('fs');

var ws1 = fs.createWriteStream("../node_test/backup.txt");
ws1.write("something\n");
ws1.write("EMD");
ws1.end();

var ws2 = fs.createReadStream("../node_test/backup.txt",'utf-8');
ws2.on('data', function(ws2){
	console.log(ws2)
})


//如果不添加utf-8 只显示二进制buffer
var ws3 = fs.createReadStream("../node_test/backup.txt");
ws3.on('data', function(ws3){
	console.log(ws3)
}
```


**pipe管道**
将流内的东西连接起来，其实就是文件的复制
```js
'use strict';
var fs = require('fs');
var rs = fs.createReadStream("backup.txt");
var ws = fs.createWriteStream("backup2.txt");
rs.pipe(ws);
```


## http
