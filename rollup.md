rollup也是一款打包工具，比webpack要轻量许多，用于弥补gulp的无tree-shaking是很好的选择，最大的用途是打包生产一个库文件，比如sdk.js之类。虽然webpack也可以做到，但是webpack较重，打包后的文件有部分webpack内部代码，如__webpack__require之类的函数定义，给人一种不干净的感觉。而rollup打出来的包就很干净，没有其他冗余代码。

1、使用方式
首先安装rollup
```
npm i rollup
```
再看rollup的几种调用形式

命令行运行
```
rollup src/main.js -o rel/bundle.js  -f cjs //将main.js(es5)编译输出至bundle.js(commonjs)
```
配置文件形式
```
//新建一个rollup.config.js
export default {
  entry: 'src/main.js',
  format: 'cjs',
  dest: 'rel/bundle.js' // 输出文件
};
```
然后执行
```
rollup -c   //当你的配置文件另有其名(dev),执行 rollup -c rollup.config.dev.js
```
模块形式(方便配合gulp等其他工具)
如将rollup.config.js 写出这样
```
var rollup = require( 'rollup' );
var babel = require('rollup-plugin-babel');

rollup.rollup({
    entry: 'src/main.js',
    plugins: [ babel() ]
}).then( function ( bundle ) {
    bundle.write({
        format: 'umd',
        moduleName: 'main', //umd或iife模式下，若入口文件含 export，必须加上该属性
        dest: 'rel/bundle.js'
    });
});
```
就可用node rollup.config.dev.js 直接执行

2、配置项
input
入口文件地址
output
```
output:{
    file:'bundle.js', // 输出文件
    format: 'cjs,  //  五种输出格式：amd /  es6 / iife / umd / cjs
    name:'A',  //当format为iife和umd时必须提供，将作为全局变量挂在window(浏览器环境)下：window.A=...
    sourcemap:true  //生成bundle.map.js文件，方便调试
}
```
plugins
最常用的就是babel插件了,比较不爽的是，babel 的预设不像 webpack 可以直接写在配置文件里，而还是得独立写个“src/.babelrc”（注意我们可以写在 src 下，而不是非得放在项目根目录下），还得确保装了插件：
```
npm i rollup-plugin-babel
```
```
// import babel from 'rollup-plugin-babel';
plugins:[babel()]
```
```
// .babelrc        npm i -D babel-preset-env babel-plugin-external-helpers
{
  "presets": [
    ["env", {
      "modules": false
    }]
  ],
  "plugins": ["external-helpers"]     //只引入一次babel的helper函数
}
external
external:['lodash'] //告诉rollup不要将此lodash打包，而作为外部依赖
global
global:{
    'jquery':'$'              //告诉rollup 全局变量$即是jquery
}
```
3、注意点
rollup无法识别node_modules中的包，需要安装插件npm install --save-dev rollup-plugin-node-resolve，然后在plugins中使用：
```
import resolve from 'rollup-plugin-node-resolve';
export default {
  input: 'src/main.js',
  output: {
    file: 'bundle.js'
    format: 'cjs'
  },
  plugins: [ resolve() ]
};
```
node_modules中的包大部分都是commonjs格式的，要在rollup中使用必须先转为ES6语法，为此需要安装插件 rollup-plugin-commonjs

4、附一份react-redux开源项目的rollup配置文件
```
import nodeResolve from 'rollup-plugin-node-resolve'     // 帮助寻找node_modules里的包
import babel from 'rollup-plugin-babel'                             // rollup 的 babel 插件，ES6转ES5
import replace from 'rollup-plugin-replace'                       // 替换待打包文件里的一些变量，如 process在浏览器端是不存在的，需要被替换
import commonjs from 'rollup-plugin-commonjs'              // 将非ES6语法的包转为ES6可用
import uglify from 'rollup-plugin-uglify'                              // 压缩包

const env = process.env.NODE_ENV

const config = {
  input: 'src/index.js',
  external: ['react', 'redux'],                           // 告诉rollup，不打包react,redux;将其视为外部依赖
  output: {
    format: 'umd',       　　　　　　　　　　 // 输出 ＵＭＤ格式，各种模块规范通用
    name: 'ReactRedux',　　　　　　　　　// 打包后的全局变量，如浏览器端 window.ReactRedux　
    globals: {
      react: 'React',                                         // 这跟external 是配套使用的，指明global.React即是外部依赖react
      redux: 'Redux'
    }
  },
  plugins: [
    nodeResolve(),
    babel({
      exclude: '**/node_modules/**'
    }),
    replace({
      'process.env.NODE_ENV': JSON.stringify(env)
    }),
    commonjs()
  ]
}

if (env === 'production') {
  config.plugins.push(
    uglify({
      compress: {
        pure_getters: true,
        unsafe: true,
        unsafe_comps: true,
        warnings: false
      }
    })
  )
}

export default config
```
