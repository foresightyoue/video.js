最新Karma4.0安装和使用
官网：https://karma-runner.github.io/4.0/intro/installation.html

介绍
Karma本质上是一个Web服务器的工具，该服务器针对连接的每个浏览器执行测试源代码，检查每个浏览器的测试结果然后通过命令行显示给开发人员，以便他们可以看到哪些浏览器通过了测试或者没有通过测试。
可以用以下方式捕获浏览器：

手动，通过访问Karma服务器正在监听的URL（通常http://localhost:9876/）
或者让Karma知道在运行Karma时要启动哪些浏览器（通过配置）
Karma还会监视配置文件中指定的所有文件，并且每当任何文件发生更改时，它都会通过向测试服务器发送信号来通知所有捕获的浏览器再次运行测试代码来触发测试运行。然后，每个浏览器将源文件加载到iFrame中，执行测试并将结果报告回服务器。

服务器从所有捕获的浏览器中收集结果并将其呈现给开发人员。

这只是一个非常简短的概述，因为当使用Karma时，知道Karma内部如何工作不是必须的。如果想深入了解，请阅读这篇论文：thesis

1、安装
Karma4.0适用的node版本6.x，8.x和10.x
安装Karam和相应的插件

# Install Karma:
```
npm install karma --save-dev
```
# Install plugins that your project needs:
```
npm install karma-jasmine karma-chrome-launcher jasmine-core --save-dev
```
安装命令行工具
```
npm install -g karma-cli
```
1
2、生成配置文件
使用karma init生成配置文件，运行该命令后会出现如下提示：
```
$ karma init my.conf.js

Which testing framework do you want to use?
Press tab to list possible options. Enter to move to the next question.
> jasmine

Do you want to use Require.js?
This will add Require.js plugin.
Press tab to list possible options. Enter to move to the next question.
> no

Do you want to capture a browser automatically?
Press tab to list possible options. Enter empty string to move to the next question.
> Chrome
> Firefox
>

What is the location of your source and test files?
You can use glob patterns, eg. "js/*.js" or "test/**/*Spec.js".
Press Enter to move to the next question.
> *.js
> test/**/*.js
>

Should any of the files included by the previous patterns be excluded?
You can use glob patterns, eg. "**/*.swp".
Press Enter to move to the next question.
>

Do you want Karma to watch all the files and run the tests on change?
Press tab to list possible options.
> yes

Config file generated at "/Users/vojta/Code/karma/my.conf.js".
```

3、运行Karma
运行Karma时配置文件可以作为第一个参数传入。
默认情况下karma会在当前目录下寻找：

./karma.conf.js
./karma.conf.coffee
./karma.conf.ts
./.config/karma.conf.js
./.config/karma.conf.coffee
./.config/karma.conf.ts
你也可以指定自定义的文件，例如：

karma start my.conf.js
1
可以使用命令行参数覆盖配置文件中的配置参数
```
karma start my.conf.js --log-level debug --single-run
```
1
4、集成Grunt/Gulp
grunt-karma
gulp-karma
5、配置文件
karma的配置文件是一个node模块，该模块导出一个函数，该函数接受一个config参数，可以使用这个参数来配置对象:
```
// karma.conf.js
module.exports = function(config) {
  config.set({
    basePath: '../..',
    frameworks: ['jasmine'],
    //...
  });
};

# karma.conf.coffee
module.exports = (config) ->
  config.set
    basePath: '../..'
    frameworks: ['jasmine']
    # ...

// karma.conf.ts
module.exports = (config) => {
  config.set({
    basePath: '../..',
    frameworks: ['jasmine'],
    //...
  });
}

```
注意：在karma底层使用ts-node来转化TypeScript为Javascript，如果tsconfig.json把模块配置js模块，可能会得到SyntaxError: Unexpected token的错误，这是因为js模块在node中是不支持的，要解决这个问题需要使用commonjs模块，我们可以覆盖模块配置：
```
// karma.conf.js
require('ts-node').register({
  compilerOptions: {
    module: 'commonjs'
  }
});
require('./karma.conf.ts');
```
5.1、配置选项
这里列举了一些比较常用的配置选项，全部配置选项请参考：全部配置选项。
一些常见的karam插件请参考：常见的Karma插件。
```
autoWatch
Type: Boolean
Default: true
CLI: --auto-watch, --no-auto-watch
每当文件改变的时候监视文件并执行测试用例

autoWatchBatchDelay
Type: Number
Default: 250
将多个文件的改变集中到一个批次运行，避免抖动。该选项告诉Karam在最后一个文件改变到新开始一轮测试要等待多久，单位毫秒。

basePath
Type: String
Default: ‘’
设置相对定位文件的根目录

browsers
Type: Array
Default: []
CLI: --browsers Chrome,Firefox, --no-browsers
可设置的值
```
```
Chrome (launcher requires karma-chrome-launcher plugin)
ChromeCanary (launcher requires karma-chrome-launcher plugin)
ChromeHeadless (launcher requires karma-chrome-launcher plugin ^2.1.0)
PhantomJS (launcher requires karma-phantomjs-launcher plugin)
Firefox (launcher requires karma-firefox-launcher plugin)
Opera (launcher requires karma-opera-launcher plugin)
IE(launcher requires karma-ie-launcher plugin)
Safari (launcher requires karma-safari-launcher plugin)
设置一个浏览器列表用于测试，当karma运行时，会打开该数组设置的所有浏览器进行测试，也可以手动访问karma服务器(默认http://localhost:9876/)。在命令行使用--no-browsers可以覆盖该值为一个空列表。
client.useIframe
Type: Boolean
Default: true
true：karma运行在一个iFrame，false：karma会运行在一个新窗口。
```
```
client.runInParent
Type: Boolean
Default: false
如果设置为true，karma会运行在原窗口内，而不是使用iFrame或者打开一个新的窗口，它将会动态加载测试脚本。

client.clearContext
Type: Boolean
Default: true
true：karma在测试完成时将会清除浏览器窗口的内容。flase则不会，这在嵌入jasmine测试模版的时候很有用。

colors
Type: Boolean
Default: true
CLI: --colors, --no-colors
是否使输出字体带有颜色(报告或者日志)

files
Type: Array
Default: []
设置加载到浏览器的文件列表，详情请参考：config/files

exclude
Type: Array
Default: []
需要排除的文件列表

frameworks
Type: Array
Default: []
你想要使用的框架列表，通常设为['jasmine'], ['mocha'] or ['qunit']…

listenAddress
Type: String
Default: ‘0.0.0.0’ or LISTEN_ADDR
服务器地址

hostname
Type: String
Default: ‘localhost’
主机名

port
Type: Number
Default: 9876
CLI: --port 9876
设置服务器端口，如果端口被占用，则会自动加1直到有可用的端口。
```
```
logLevel
Type: Constant
Default: config.LOG_INFO
CLI: --log-level debug
可设置的值：

config.LOG_DISABLE
config.LOG_ERROR
config.LOG_WARN
config.LOG_INFO
config.LOG_DEBUG
plugins
Type: Array
Default: [‘karma-*’]
插件列表，一个插件可以是一个字符串，或者一个对象，或者使用require引入，详情请参考：插件列表。默认Karma会加载node_modules下所有以karma-*开头的插件。

reporters
Type: Array
Default: [‘progress’]
CLI: --reporters progress,growl
可能的值：

dots
progress
额外的reporters, 例如growl, junit, teamcity or coverage 可以通过插件加载。

singleRun
Type: Boolean
Default: false
CLI: --single-run, --no-single-run
如果设置为true，则karma只会执行一次。
```
