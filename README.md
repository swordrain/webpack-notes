
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
[参考](https://webpack.github.io/docs/cli.html)

<p>直接执行webpack命令，会去当前目录寻找默认配置文件webpack.config.js，根据该配置文件执行</p>
<p>或者使用参数</p>
```
--display-error-details  显示详细的出错信息
--colors  输出结果带色彩
--profiles  输出性能数据
--display-modules  默认情况下node_modules下的模块会被隐藏，加上后显示这些隐藏的模块
--config XXX.js 使用XXX.js作为配置文件
--watch/-w  监听变动并自动执行打包
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
var path = require('path');
var webpack = require('webpack');
/*
extract-text-webpack-plugin插件，
将你的样式提取到单独的css文件里，如果没有它的话，webpack会将css打包到js当中
 */
var ExtractTextPlugin = require('extract-text-webpack-plugin');
/*
html-webpack-plugin插件，webpack中生成HTML的插件，
可以将打包好的文件动态加载到html中
 */
var HtmlWebpackPlugin = require('html-webpack-plugin');
  
module.exports = {
    entry: { //配置入口文件，有几个写几个。我这里有两个文件。一个是所有我需要引入的文件，一个是我的入口文件，app.js
    //支持数组形式，将加载数组中的所有模块，但以最后一个模块作为输出,比如下面数组里面的js,全部压缩在了vendor这个文件这里
　　　　vendor: ['react','react-dom','react-router'],
　　　 　app: [ './app.js'],
    },
    output: {
        path: path.join(__dirname, 'dist'),  //输出目录的配置，模板、样式、脚本、图片等资源的路径配置都相对于它.名字可以随便起
        // publicPath: '/dist/',               很多教程的publicPath是这个，你会发觉不能动态加载到html中，会报错，实际上是下面的写法才对
        publicPath: '../',                //模板、样式、脚本、图片等资源对应的server上的路径
        filename: 'js/[name].bundle.js',       //每个页面对应的主js的生成配置。比如我的app.js打包后就为  js/app.bundle.js
        chunkFilename: 'js/[id].bundle.js'   //dundle生成的配置
    },
    module: {
        loaders: [ //加载器，关于各个加载器的参数配置。
　　　　　　　　{
　　　　　　　　test: /\.js$/,
　　　　　　　　loaders: ['react-hot', 'babel'],
　　　　　　　　include: [path.join(__dirname, ''), path.join(__dirname, 'router')]  //我需要打包的js所在的目录
　　　　　　　　},
  
　　　　　　　{
                test: /\.css$/,
                //配置css的抽取器、加载器。'-loader'可以省去
                loader: ExtractTextPlugin.extract('style-loader', 'css-loader')
            }, {
                test: /\.less$/,
                //配置less的抽取器、加载器。中间!有必要解释一下，
                //根据从右到左的顺序依次调用less、css加载器，前一个的输出是后一个的输入
                //你也可以开发自己的loader哟。有关loader的写法可自行谷歌之。
                loader: ExtractTextPlugin.extract('css!less')
            }, {
                //html模板加载器，可以处理引用的静态资源，默认配置参数attrs=img:src，处理图片的src引用的资源
                //比如你配置，attrs=img:src img:data-src就可以一并处理data-src引用的资源了，就像下面这样
                test: /\.html$/,
                loader: "html?attrs=img:src img:data-src"
            }, {
                //文件加载器，处理文件静态资源
                test: /\.(woff|woff2|ttf|eot|svg)(\?v=[0-9]\.[0-9]\.[0-9])?$/,
                loader: 'file-loader?name=./fonts/[name].[ext]'
            }, { 
            	test: /\.scss$/, 
            	loader: 'style!css!sass?sourceMap'}, //字符串的写法，用!分割
            {
                //图片加载器，雷同file-loader，更适合图片，可以将较小的图片转成base64，减少http请求
                //如下配置，将小于8192byte的图片转成base64码
                // base的好处可以看看这篇文章
                // http://www.zhangxinxu.com/wordpress/2012/04/base64-url-image-%E5%9B%BE%E7%89%87-%E9%A1%B5%E9%9D%A2%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/
                test: /\.(png|jpg|gif)$/,
                loader: 'url-loader?limit=8192&name=./img/[hash].[ext]'
            }
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
    },
    plugins: [
        new webpack.ProvidePlugin({ //加载jq
            $: 'jquery',
　　　　　　_:'underscore' //加载underscore
        }),
        new webpack.optimize.CommonsChunkPlugin({
            name: 'vendors',   // 将公共模块提取，生成名为`vendors`bundle
            chunks: ['vendor'], //提取哪些模块共有的部分,名字为上面的vendor
            minChunks: Infinity // 提取至少*个模块共有的部分
        }),
        new ExtractTextPlugin('css/[name].css'), //单独使用link标签加载css并设置路径，相对于output配置中的 publickPath
             
        // 有几个页面就写几个
       new HtmlWebpackPlugin({                        //根据模板插入css/js等生成最终HTML
             favicon:'./images/favicon.ico', //favicon存放路径
             filename:'/view/index.html',    //生成的html存放路径，相对于 path
             template:'./index.html',    //html模板路径
             inject:true,    //允许插件修改哪些内容，包括head与body
             hash:true,    //为静态资源生成hash值
             chunks: ['vendor', 'app'], //需要引入的chunk，不配置就会引入所有页面的资源.名字来源于你的入口文件
             minify:{    //压缩HTML文件
                 removeComments:true,    //移除HTML中的注释
                 collapseWhitespace:false    //删除空白符与换行符
             }
         })
          
        new webpack.HotModuleReplacementPlugin() //热加载
    ],
    //使用webpack-dev-server，提高开发效率
    devServer: {
        contentBase: './',
        host: 'localhost',
        port: 3200, //比如我是监听3200端口
        inline: true, //可以监控js变化
        hot: true, //热启动
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
也可以使用数组形式，注意此时是复数
```
loaders: [{
			test: /\.js?$/,
			exclude: /node_modules/,
			loaders: ['react-hot','babel']
		}]
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

####less资源文件的处理####
安装
```
npm install less-loader --save-dev
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

####ES6语法的处理####
安装
```
npm install --save-dev babel-loader babel-core babel-preset-es2015
```
在根目录下创建.babelrc配置文件
```
{
  "presets": ["es2015"]
}
```
webpack.config.js中的配置
```
{ test: /\.js$/, exclude: /node_modules/, loader: "babel-loader" }
```

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
用于指明程序自动补全识别哪些后缀, 注意一下, extensions 第一个是空字符串! 对应不需要后缀的情况.

###plugins###

####使用html-webpack-plugin自动生成入口文件####
安装
```
npm install html-webpack-plugin --save-dev
```
配置
```
new HtmlWebpackPlugin({
	template: './src/html/index.html',
        filename: 'html/index.html',
        inject: 'body',
        hash: true, // index.js?hash
        cache: true, // if true (default) try to emit the file only if it was changed.
        showErrors: true, // if true (default) errors details will be written into the html page.
        chunks: ['js/index'] // filter chunks
})
```

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
有时候我们希望某些模块走CDN并以&lt;script&gt;的形式挂载到页面上来加载，但又希望能在 webpack 的模块中使用上。这时候我们可以在配置文件里使用 externals 属性来帮忙
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
安装[react-hot-loader](https://github.com/gaearon/react-hot-loader)
```
npm install --save-dev react-hot-loader
```
配置
```
//结合webpack-dev-server
entry: [
    'webpack-dev-server/client?http://0.0.0.0:3000', // WebpackDevServer host and port
    'webpack/hot/only-dev-server', // "only" prevents reload on syntax errors
    './scripts/index' // Your appʼs entry point
]
//放在babel-loader或jsx-loader之前
module: {
    loaders: [
        { test: /\.jsx?$/, loaders: ['react-hot', 'jsx?harmony'], include: path.join(__dirname, 'src') }
    ]
}
//plugin里
plugins: [
    new webpack.HotModuleReplacementPlugin()
]
```

##webpack-dev-server
安装
```
npm install --save-dev webpack-dev-server
```
配置
```
module.exports = {
    entry: {
        'js/index': './src/js/index.js'
    },
    output: {
        path: './build',
        filename: '[name].js'
    },
    devServer: {
        progress: true,
        host: '0.0.0.0',
        port: 8080,
        colors: true,
        inline: true,
        // hot: true,
        contentBase: './src',
        displayErrorDetails: true
    },
    module: {
        loaders: loaders
    },
    plugins: plugins
};
```


##配合grunt/gulp##



##参考##
<ol>
<li>http://www.w2bc.com/Article/50764</li>
<li>http://www.tuicool.com/articles/2qiE7jN</li>
<li>http://www.cnblogs.com/LIUYANZUO/p/5184424.html</li>
<li>http://gaearon.github.io/react-hot-loader/getstarted/</li>
<li>http://www.cnblogs.com/xianyulaodi/p/5314769.html</li>
<li>http://www.cnblogs.com/yangjunhua/p/5615118.html</li>
</ol>
