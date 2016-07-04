
## 功能与作用 ##

<ol>
<li>打包工具，不光是打js还能打css以及图片等</li>
<li>能支持各种模块，AMD,commonJS, ES6</li>
<li>配合react.js</li>
<li>官网http://webpack.github.io/</li>
</ol>
## 安装 ##
```
//全局安装
npm install webpack -g
//开发安装
npm install webpack --save-dev
```
## 基本用法 ##
<p>直接执行webpack命令，会去当前目录寻找默认配置文件webpack.config.js，根据该配置文件执行</p>
<p>或者使用参数</p>
```
--display-error-details  显示详细的出错信息
--config XXX.js 使用XXX.js作为配置文件
--watch  监听变动并自动执行打包
-p  压缩混淆脚本，重要
-d  生成map映射文件，告知哪些模块被最终打包到何处
```


最终需要将打包后的文件放入一个html中，当然这一步也可以由插件自动生成，一个典型的html文件是
```
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title>demo</title>
</head>
<body>
    <script src="dist/js/page/common.js"></script>
    <script src="dist/js/page/index.js"></script>
</body>
</html>
```
在&lt;script&gt;标签里引用了打包后的文件

## 配置 ##
<p>默认配置文件webpack.config.js，如果要自定义，在webpack命令后跟--config [configfile.js]</p>
一个webpack.config.js的示例
```
var webpack = require('webpack');
var commonsPlugin = new webpack.optimize.CommonsChunkPlugin('common.js');
 
module.exports = {
    //插件项
    plugins: [commonsPlugin],
    //页面入口文件配置
    entry: {
        index : './src/js/page/index.js'
    },
    //入口文件输出配置
    output: {
        path: 'dist/js/page',
        filename: '[name].js'
    },
    module: {
        //加载器配置
        loaders: [
            { test: /\.css$/, loader: 'style-loader!css-loader' },
            { test: /\.js$/, loader: 'jsx-loader?harmony' },
            { test: /\.scss$/, loader: 'style!css!sass?sourceMap'},
            { test: /\.(png|jpg)$/, loader: 'url-loader?limit=8192'}
        ]
    },
    //其它解决方案配置
    resolve: {
        root: 'E:/github/flux-example/src', //绝对路径
        extensions: ['', '.js', '.json', '.scss'],
        alias: {
            AppStore : 'js/stores/AppStores.js',
            ActionType : 'js/actions/ActionType.js',
            AppAction : 'js/actions/AppAction.js'
        }
    }
};
```
其中plugins用来指定插件，entry指定打包的入口文件，output指定出口，module下的loaders指定文件类型和其加载器



###entry###

###output###


###loaders###
如果使用字符串配置且一个文件类型如果有多个loader，使用!将其分开，如
```
{ test: /\.css$/, loader: 'style-loader!css-loader' }
```
其中'-loader'可以省略，变成
```
{test: /\.css$/, loader: "style!css"}
```

####css资源文件的处理####
安装
```
npm install style-loader css-loader --save-dev
```


####sass/scss资源文件的处理####
安装
```
npm install sass-loader --save-dev
```
配置
```
{test: /\.scss$/, loader: "style!css!sass"}
```

####图片资源文件的处理####
图片资源的引用
```
div.img{
    background: url(../image/xxx.jpg)
}
//或者
var img = document.createElement("img");
img.src = require("../image/xxx.jpg");
document.body.appendChild(img);
```
一个典型的配置
```
{ test: /\.(png|jpg)$/, loader: 'url-loader?limit=8192'}
```
需要安装
```
npm install url-loader -save-dev
```
其中?limit=8192表示图片大小在8k以下的会转换成base64编码
####不符合规范的模块处理(shim)####
参考[exports-loader](https://github.com/webpack/exports-loader)
安装
```
npm install exports-loader --save
```
配置
```
{ test: require.resolve("./src/js/tool/swipe.js"),  loader: "exports?swipe"}
```
使用
```
var swipe = require('./tool/swipe.js');
swipe(); 
```

###resolve###

###plugins###

####使用CommonsChunkPlugin提取公共模块####
如果在不同的文件中各自引用了import React from 'react'，那么打包的时候react模块会被打包多次，需要使用CommonsChunkPlugin将公共的模块提取到一个公共部分
安装
```
```
一个典型的配置
```
var CommonsChunkPlugin = require("webpack/lib/optimize/CommonsChunkPlugin");
module.exports = {
    entry: {
        p1: "./page1",
        p2: "./page2",
        p3: "./page3",
        ap1: "./admin/page1",
        ap2: "./admin/page2"
    },
    output: {
        filename: "[name].js"
    },
    plugins: [
        new CommonsChunkPlugin("admin-commons.js", ["ap1", "ap2"]),
        new CommonsChunkPlugin("commons.js", ["p1", "p2", "admin-commons.js"])
    ]
};
// <script>s required:
// page1.html: commons.js, p1.js
// page2.html: commons.js, p2.js
// page3.html: p3.js
// admin-page1.html: commons.js, admin-commons.js, ap1.js
// admin-page2.html: commons.js, admin-commons.js, ap2.js
```
####使用extract-text-webpack-plugin独立打包样式文件####
[extract-text-webpack-plugin](https://github.com/webpack/extract-text-webpack-plugin)
安装
```
npm install extract-text-webpack-plugin --save-dev
```
典型配置
```
var ExtractTextPlugin = require("extract-text-webpack-plugin");
module.exports = {
    // ...省略各种代码
    module: {
        loaders: [
            {test: /\.js$/, loader: "babel"},
            {test: /\.css$/, loader: ExtractTextPlugin.extract("style-loader", "css-loader")},
            {test: /\.(jpg|png|svg)$/, loader: "url?limit=8192"},
            {test: /\.scss$/, loader: "style!css!sass"}
        ]
    },
    plugins: [
        new webpack.optimize.CommonsChunkPlugin('common.js'),
        new ExtractTextPlugin("[name].css")
    ]
}
```


###external###
有时候我们希望某些模块走CDN并以<script>的形式挂载到页面上来加载，但又希望能在 webpack 的模块中使用上。这时候我们可以在配置文件里使用 externals 属性来帮忙
```
{
    externals: {
        // require("jquery") 是引用自外部模块的
        // 对应全局变量 jQuery
        "jquery": "jQuery"
    }
}
```
确保CDN文件在webpack打包文件引入之前引入
使用[script.js](https://github.com/ded/script.js)在脚本中加载模块
```
var $script = require("scriptjs");
$script("//ajax.googleapis.com/ajax/libs/jquery/2.0.0/jquery.min.js", function() {
    $('body').html('It works!')
});
```


##配合React##


##配合grunt/gulp##



##参考##
http://www.w2bc.com/Article/50764
http://www.tuicool.com/articles/2qiE7jN
 
