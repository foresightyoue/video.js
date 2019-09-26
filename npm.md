npm install 本地安装与全局安装的区别
npm的包安装分为本地安装（local）、全局安装（global）两种，从敲的命令行来看，差别只是有没有-g而已，比如

npm install grunt # 本地安装
npm install -g grunt-cli # 全局安装
这两种安装方式有什么区别呢？从npm官方文档的说明来看，主要区别在于（后面通过具体的例子来说明）：
本地安装
1. 将安装包放在 ./node_modules 下（运行npm时所在的目录）
2. 可以通过 require() 来引入本地安装的包

全局安装
1. 将安装包放在 /usr/local 下
2. 可以直接在命令行里使用

本地安装

1、将安装包放在 ./node_modules 下（运行npm时所在的目录）

比如运行下面命令

npm install grunt --save-dev
那么，就会在当前目录下发现一个node_modules目录，进去后能够看到grunt这个包

casperchenMacBookPro:testUsemin casperchen$ ll
total 200
drwxr-xr-x  16 casperchen  staff   544B 12 14 23:17 node_modules
进入node_modules

casperchenMacBookPro:node_modules casperchen$ ll
total 0
drwxr-xr-x  16 casperchen  staff   544B 12  5 00:49 grunt
2、可以通过 require() 来引入本地安装的包

直接来个例子，我们在项目根目录下创建test.js，里面的内容很简单

var grunt = require('grunt');grunt.log.writeln('hello grunt');
然后在控制台运行test.js

node test.js
然后就会看到如下输出

casperchenMacBookPro:testUsemin casperchen$ node test.js
hello grunt
全局安装

1、将安装包放在 /usr/local 下

运行如下命令

npm install -g grunt-cli
然后进入/usr/local/bin目录，就会发现grunt-cli已经被放置在下面了

casperchenMacBookPro:bin casperchen$ pwd
/usr/local/bin
casperchenMacBookPro:bin casperchen$ ll grunt
lrwxr-xr-x  1 root  admin    39B  8 18 21:43 grunt -> ../lib/node_modules/grunt-cli/bin/grunt
可见，全局模块的真实安装路径在/usr/local/lib/node_modules/下，/usr/local/bin下的可执行文件只是软链接而已

2、可以直接在命令行里使用

实现细节在上面其实就讲到了，通过在`/usr/local/bin下创建软链接的方式实现。这里不赘述

更直观的例子

下面就直接看下，当我们在项目目录下运行grunt task（task为具体的grunt任务名，自行替换）时，发生了什么事情。这里要借助node-inspector。

首先，没接触过node-inspector的童鞋可以参考之前的文章了解下

运行如下命令开启调试

node-inspector &
见到如下输出

casperchenMacBookPro:tmp casperchen$ node-inspector &
[1] 14390
casperchenMacBookPro:tmp casperchen$ Node Inspector v0.6.1
   info  - socket.io started
Visit http://127.0.0.1:8080/debug?port=5858 to start debugging.
接着，在当前任务下运行grunt任务

^CcasperchenMacBookPro:testUsemin casperchen$ node --debug-brk $(which grunt) dev
debugger listening on port 5858
接着，打开chrome浏览器，输入网址http://127.0.0.1:8080/debug?port=5858，就会自动进入断点调试状态。从一旁显示的tiPS可以看到，全局命令grunt其实就是/usr/local/lib/node_modules/grunt-cli/bin/grunt

按下F8接着往下跑，就会进如Gruntfile.js，此时的grunt，是本地安装的一个node包。全局命令跟本地的包名字一样，挺有迷惑性的。
