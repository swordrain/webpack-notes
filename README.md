
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
一个文件类型如果有多个loader，使用!将其分开，如
```
{ test: /\.css$/, loader: 'style-loader!css-loader' }
```


###plugins###

## 插件 ##

##配合React##


 
