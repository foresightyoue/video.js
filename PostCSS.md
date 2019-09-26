PostCSS是一个用javascript转换CSS的工具，有了它可放心使用CSS未来的语法等。
特点：
增加代码的可读性：可自动添加属性前缀
可使用最新的CSS语法，如变量定义
可模块化CSS，CSS Modules
可检查CSS语法错误，避免出错
强大的格子系统LostGrid
首先介绍下原生CSS变量基础知识：

一、CSS变量
1. 变量声明：
变量名前加两根连词线（–），如--text-size: 14px;

注：变量名的大小写是敏感的。

2. var()函数：
用于读取变量值，如font-size: var(--text-size);
var()函数还可以接受第二个参数，表示变量的默认值，如font-size: var(--text-size, 16px);

注：变量值只能用作属性值，不能用作属性名。

3. 变量值类型：
若变量值为一个字符串，则可以与其他字符串拼接。
--bar: 'hello';
--foo: var(--bar)' world';

若变量值为数值，不能与数值单位直接连用，需用calc()函数连接。
```
.foo {
  --gap: 20;

  /* 无效 */
  margin-top: var(--gap)px;

  margin-top: calc(var(--gap) * 1px);
}
```

若变量值带有单位，则不能写成字符串。
```
/* 无效 */
.foo {
  --foo: '20px';
  font-size: var(--foo);
}

/* 有效 */
.foo {
  --foo: 20px;
  font-size: var(--foo);
}
```

4. 作用域：
CSS变量的作用域是它所在选择器的有效范围，读取时优先级最高的声明生效。
```
<style>
  //全局变量
  :root { --color: blue; }
  div { --color: green; }
  #alert { --color: red; }
  * { color: var(--color); }
</style>

<p>蓝色</p>
<div>绿色</div>
<div id="alert">红色</div>
```
5. 响应式布局：
在响应式布局的media中声明变量，可使得不同的屏幕宽度有不同的变量值。
```
body {
  --primary: #7F583F;
  --secondary: #F7EFD2;
}

a {
  color: var(--primary);
  text-decoration-color: var(--secondary);
}

@media screen and (min-width: 768px) {
  body {
    --primary:  #F7EFD2;
    --secondary: #7F583F;
  }
}
```
6. javascript操作
```
// 设置变量
document.body.style.setProperty('--primary', '#7F583F');

// 读取变量
document.body.style.getPropertyValue('--primary').trim();
// '#7F583F'

// 删除变量
document.body.style.removeProperty('--primary');
```
二、安装配置
1. gulp配置
(1) 全局安装PostCSS: npm install postcss -g
(2) 项目安装gulp-postcss: npm install --save-dev gulp-postcss
(3) gulpfile.js文件设置：
```
var gulp = require('gulp');
var postcss = require('gulp-postcss');
gulp.task('css', function () {
    var processors = [ ]; //放置PostCSS插件
    return gulp.src('./src/*.css')
        .pipe(postcss(processors))
        .pipe(gulp.dest('./dest'));
});
```
(4) 添加PostCSS插件：Autoprefixer(处理浏览器私有前缀),cssnext(使用CSS未来语法),precss(CSS预处理函数)
```
npm install --save-dev autoprefixer cssnext precss
```
gulpfile.js文件修改：
```
var gulp = require('gulp');
var postcss = require('gulp-postcss');
var autoprefixer = require('autoprefixer');
var cssnext = require('cssnext');
var precss = require('precss');

gulp.task('css', function () {
    //放置PostCSS插件
    var processors = [
        autoprefixer, cssnext, precss
    ];
    return gulp.src('./src/*.css')
        .pipe(postcss(processors))
        .pipe(gulp.dest('./dest'));
});
```
2. webpack配置
(1) 安装相应包
```
npm install --save-dev postcss postcss-loader postcss-cssnext postcss-import
```
(2) 配置webpack.config.js
```
const webpack = require('webpack');
const path = require('path');
const ExtractTextPlugin = require('extract-text-webpack-plugin');
module.exports = {
  context: path.resolve(__dirname, 'src'),
  entry: {
    app: './app.js';
  },
  module: {
    loaders: [
      {
        test: /\.css$/,
        use: ExtractTextPlugin.extract({
          use: [
            {
              loader: 'css-loader',
              options: { importLoaders: 1 },
            },
            'postcss-loader',
          ],
        }),
      },
    ],
  },
  output: {
    path: path.resolve(__dirname, 'dist/assets'),
  },
  plugins: [
    new ExtractTextPlugin('[name].bundle.css'),
  ],
};
```
(3) 根目录配置postcss.config.js
```
module.exports = {
  plugins: {
    'postcss-import': {},
    'postcss-cssnext': {
      browsers: ['last 2 versions', '> 5%'],
    },
  },
};
```
