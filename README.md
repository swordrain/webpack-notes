## 功能与作用 ##

1. 打包工具，不光是打js还能打css以及图片等，并且进行压缩
2. 不能完全替代gulp或grunt
3. 能支持各种模块，AMD,commonJS, ES6
4. 配合react.js
5. 官网http://webpack.github.io/

## 安装 ##
```
//全局安装
npm install webpack -g
//开发安装
npm install webpack --save-dev
```
## 基本用法 ##
[参考](https://webpack.github.io/docs/cli.html)

`webpack entryfile destfile`

或者直接执行webpack命令，此时会去当前目录寻找默认配置文件`webpack.config.js`，根据该配置文件执行  
或者使用参数

```
--entry 入口文件
--output-path
--ouput-file
--progress 显示进度
--display-error-details  显示详细的出错信息
--colors  输出结果带色彩
--profiles  输出性能数据，可以在scripts里"stats": "webpack --profile --json > stats.json"
--display-modules  默认情况下node_modules下的模块会被隐藏，加上后显示这些隐藏的模块
--config XXX.js 使用XXX.js作为配置文件，可以为prod环境生成一份专门的配置文件
--watch/-w  监听变动并自动执行打包
--devtool 设置source-map、cheap-module-source-map、eval-source-map、cheap-module-eval-source-map 
-p  压缩混淆脚本，重要
-d  生成map映射文件，告知哪些模块被最终打包到何处
--module-bind 绑定loader
--display-chunks 展示编译后的分块
--display-reasons 显示更多引用模块原因
--sort-chunks-by,--sort-assets-by,--sort-modules-by 将modules/chunks/assets进行列表排序
-help 查看webpack帮忙文档
```

绑定loader的一个例子
```
不用module-bind
//entry.js
require("!style-loader!css-loader!./style.css") // 载入 style.css
document.write('It works.')
document.write(require('./module.js'))
执行webpack entry.js bundle.js

使用module-bind
require("./style.css")
执行webpack entry.js bundle.js --module-bind "css=style-loader!css-loader"
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
在`<script>`标签里引用了打包后的文件

## 配置 ##
默认配置文件是项目根目录下的`webpack.config.js`，如果要自定义，在webpack命令后跟--config [configfile.js]  
一个典型webpack.config.js的示例

```
var path = require('path');
var webpack = require('webpack');

//extract-text-webpack-plugin插件，将你的样式提取到单独的css文件里，如果没有它的话，webpack会将css打包到js当中
var ExtractTextPlugin = require('extract-text-webpack-plugin');
//html-webpack-plugin插件，webpack中生成HTML的插件，可以将打包好的文件动态加载到html中
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
            }, {
            		test: /\.jade$/, loader: "jade-loader"
            }, {
            		test: /\. ejs$/, loader: "ejs-loader"
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

if (process.env.NODE_ENV === 'production') {
  module.exports.devtool = '#source-map'
  // http://vue-loader.vuejs.org/en/workflow/production.html
  module.exports.plugins = (module.exports.plugins || []).concat([
    new webpack.DefinePlugin({
      'process.env': {
        NODE_ENV: '"production"'
      }
    }),
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      }
    }),
    //为组件分配ID，通过这个插件webpack可以分析和优先考虑使用最多的模块，并为它们分配最小的ID
    new webpack.optimize.OccurenceOrderPlugin()
  ])
}
```
其中plugins用来指定插件，entry指定打包的入口文件，output指定出口，module下的loaders指定文件类型和其加载器

### context  

webpack处理entry选项时的基础路径（绝对路径），默认值为`process.cmd()`，即`webpack.config.js`文件所在路径
```
Root
|---config
    |---webpack.config.js
|---app.js
```

```
module.exports = {
    entry: './app.js',
    context: path.join(__dirname, '..'),
    output: {
        filename: './bundle.js'
    }
}
```

### watch
```
module.exports = {
    entry: './A.js',
    output: {
        filename: './bundle.js'
    },
    watch: true //相当于webpack --watch
}
```

### entry、output
页面入口文件和输出文件配置

```
	entry: "./entry.js" //只会打包一个顺序依赖的模块，输出则根据output配置而定
	entry: ["page1", "page2"] //只会打包一个顺序依赖的模块，合并到最后一个模块时导出
	entry: { //根据入口打包多个顺序依赖的模块，key名会根据在output的配置输出
		page1: "./page1",
		page2: ["./entry1", "./entry2"]
	}
	//对应于entry的输出就叫chunk
	
	output: "bundle.js"
	output: {   //对象形式指定属性
		path: __dirname, //path.resolve(__dirname, 'build')
		filename: "[name].bundle.js" //
		//publicPath 设置资源的访问路径，如"/dist/"
		//library 设置模块导出的类名
		//libraryTarget: 'umd' 设置模块兼容模式
		//umdNamedDefine: true 同上
		//chunckFileName: "[name].js"
		//sourceMapFilename
		//devtoolModuleFilenameTemplate
		//devtoolFallbackModuleFilenameTemplate
		//devtoolLineToLine
		//hotupdateChunkFilename
		//hotUpdateMainFilename
		//jsonpFunction
		//hotUpdateFunction
		//pathinfo
		//sourcePrefix
		//crossOriginLoading
	}
	//filename可使用
	//[name]：代表模块集的名称，与entry配置有关，具体可自行测试
	//[hash]：代表编译hash值，与模块集的代码有关，如果模块集的代码有修改，hash值也会变，这个在生成环境里可以解决客户端的缓存问题，如果需要的是8位的hash可以写成[hash:8]
	//[chunkhash]：代表模块集名称的hash值，注意chunkhash与hash不能同时使用
	
	//chunckFileName可使用
	//[id]: 代表模块集的id
	//[name]: 代表模块集的名称，和require.ensure的第三个参数，具体可以自行测试
	//[hash]: 代表编译hash值，与模块集的代码有关，如果模块集的代码有修改，hash值也会变，这个在生成环境里可以解决客户端的缓存问题
	//[chunkhash]: 代表模块集名称的hash值，注意chunkhash与hash不能同时使用
```

### devtool
`source-map`在一个单独的文件中产生一个完整且功能完全的文件。这个文件具有最好的source map，但是它会减慢打包文件的构建速度

`cheap-module-source-map`在一个单独的文件中生成一个不带列映射的map，不带列映射提高项目构建速度，但是也使得浏览器开发者工具只能对应到具体的行，不能对应到具体的列（符号），会对调试造成不便

`eval-source-map`使用eval打包源文件模块，在同一个文件中生成干净的完整的source map。这个选项可以在不影响构建速度的前提下生成完整的sourcemap，但是对打包后输出的JS文件的执行具有性能和安全的隐患。不过在开发阶段这是一个非常好的选项，但是在生产阶段一定不要用这个选项

`cheap-module-eval-source-map`这是在打包文件时最快的生成source map的方法，生成的Source Map 会和打包后的JavaScript文件同行显示，没有列映射，和eval-source-map选项具有相似的缺点

上述选项由上到下打包速度越来越快，不过同时也具有越来越多的负面作用，较快的构建速度的后果就是对打包后的文件的的执行有一定影响。
在学习阶段以及在小到中性的项目上，eval-source-map是一个很好的选项，不过记得只在开发阶段使用它

### externals
有时候我们希望某些模块走CDN并以`<script>`的形式挂载到页面上来加载，但又希望能在 webpack 的模块中使用上。这时候我们可以在配置文件里使用 externals 属性来帮忙，external的本意就是设置为外部引用，内部不会打包合并
```
{
    externals: {
        // require("jquery") 是引用自外部模块的对应全局变量 jQuery
        "jquery": "jQuery"
    }
}
```
确保CDN文件在webpack打包文件引入之前在html里引入

使用[script.js](https://github.com/ded/script.js)在脚本中加载模块
```
var $script = require("scriptjs");
$script("//ajax.googleapis.com/ajax/libs/jquery/2.0.0/jquery.min.js", function() {
    $('body').html('It works!')
});
```

### loaders
loaders 用于转换应用程序的资源文件，他们是运行在nodejs下的函数 使用参数来获取一个资源的来源并且返回一个新的来源(资源的位置)

* Loader可以通过管道方式链式调用，每个 loader 可以把资源转换成任意格式并传递给下一个 loader ，但是最后一个 loader 必须返回 JavaScript。
* Loader可以同步或异步执行。
* Loader运行在 node.js 环境中，所以可以做任何可能的事情。
* Loader可以接受查询参数，以此来传递配置项给 loader。
* Loader可以通过文件扩展名（或正则表达式）绑定给不同类型的文件。
* Loader可以通过 npm 发布和安装。
* 除了通过 package.json 的 main 指定，通常的模块也可以导出一个 loader 来使用。
* Loader可以访问配置。
* 插件可以让loader拥有更多特性。
* Loader可以分发出附加的任意文件。

[loader列表](http://webpack.github.io/docs/list-of-loaders.html)

`loader`可以通过require来引用
```
require(!style-loader!css-loader!less-loader!./src/css/index.less);
```
还可以通过`module-bind`参数来引用
```
webpack --module-bind jade --module-bind 'css=style!css'
```

也可以通过配置`webpack.config.js`文件

如果使用字符串配置且一个文件类型如果有多个loader，使用!将其分开，如
```
{ test: /\.css$/, loader: 'style-loader!css-loader' }
```
其中'-loader'可以省略，变成 (具体看版本，在新版本中不再可省略，切记)
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
多个loader的处理顺序是从右向左执行

配置项包括 
 
* test：一个匹配loaders所处理的文件的拓展名的正则表达式（必须）
* loader：loader的名称（必须）
* include/exclude:手动添加必须处理的文件（文件夹）或屏蔽不需要处理的文件（文件夹）（可选），node_modules中的文件都是编译好的可以直接加载，exclude它们后可以优化打包速度
* query：为loaders提供额外的设置选项（可选）

#### json文件的处理
安装
```
npm install --save-dev json-loader
```
配置
```
module: {
	loaders: [
	  {
	    test: /\.json$/,
	    loader: "json-loader"
	  }
	]
}
```
使用
```
//config.json
{
  "greetText": "Hi there and greetings from JSON!"
}
//main.js
var config = require('./config.json');
 
module.exports = function() {
  var greet = document.createElement('div');
  greet.textContent = config.greetText;
  return greet;
};
```

#### css资源文件的处理
安装
```
npm install style-loader css-loader --save-dev
```
引用
```
require('./main.css');
```
配置
```
{
	test: /\.css$/,
 	loaders: ['style-loader', 'css-loader'], //另一种写法 loader: 'style!css'
	include: APP_PATH
}
```

对css module的支持
```
{
	test: /\.css$/,
	loader: 'style!css?modules'//跟前面相比就在后面加上了?modules
}
```

#### css预处理器的处理
安装
```
npm install --save-dev postcss-loader autoprefixer
```
此时只要
```
...
module: {
	loaders: [
  	...
  	{
    		test: /\.css$/,
    		loader: 'style!css?modules!postcss'
  	}]
},
postcss: [
	require('autoprefixer')//调用autoprefixer插件
],
```

CSS的加载策略

* 建一个或多个css文件，在js的入口处引入
* 建多个css文件，在所需的地方引入
* css的文件与js组件使用一个命名空间（文件名、路径相同），产生一对一的效果


#### sass/scss资源文件的处理
安装
```
npm install sass-loader --save-dev
```
引用
```
require('./main.scss');
```
配置
```
{test: /\.scss$/, loader: "style!css!sass"}
```

#### less资源文件的处理
安装
```
npm install less-loader --save-dev
```
配合使用`autoprefixer`
```
{
  test: /\.less$/,
  loader: 'style-loader!css-loader!autoprefixer-loader!less-loader'
}
```

#### 静态文件的处理
安装
```
npm install file-loader --save-dev
```
用法
```
{
    //文件加载器，处理文件静态资源
    test: /\.(woff|woff2|ttf|eot|svg)(\?v=[0-9]\.[0-9]\.[0-9])?$/,
    loader: 'file-loader?name=./fonts/[name].[ext]'
}
```

#### 图片资源文件的处理
安装(url-loader是对file-loader的封装)
```
npm install url-loader --save-dev
```
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
{ test: /\.(png|jpg)$/, loader: 'url-loader?mimetype=image/png&limit=8192&publicPath=./dist/'}
```
其中limit=8192表示图片大小在8k以下的会转换成base64编码，publicPath会把打包的图片生成到该路径（或者在output里配置publicPath）

该loader还能用来处理字体文件

#### ES6/jsx语法的处理
安装
```
npm install --save-dev babel-loader babel-core babel-preset-es2015 babel-preset-react
```
在根目录下创建.babelrc配置文件（也可以在webpack.config.js里配置，见下）
```
{
  "presets": ["react", "es2015"]
}
```
webpack.config.js中的配置
```
{ 
	test: /\.js$/, //test: /\.js|jsx$/ 
	exclude: /node_modules/, 
	loader: "babel-loader",
	query: {
		presets: ['es2015','react'] //配置babel
   } 
}
```

#### jshint
安装
```
npm install jshint jshint-loader --save-dev
```
使用
```
var common = { ...
	module: {
		preLoaders: [
			{
				test: /\.js?$/,
				loaders: ['jshint'],
				// define an include so we check just the files we need 
				include: PATHS.app
		}]
	}, 
};
```
配置`.jshintrc`文件
```
{
	"browser": true, 
	"camelcase": false, 
	"esnext": true,   //es6
	"indent": 2, 
	"latedef": false, 
	"newcap": true, 
	"quotmark": "double"
}
```

#### 不符合规范的模块处理(shim)
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
例子
比如我们用到 Pen 这个模块，这个模块对依赖一个 window.jQuery, 可我手头的 jQuery 是 CommonJS 语法的。而 Pen 对象又是生成好了绑在全局的, 可是我又需要通过 require('pen') 获取变量。最终的写法就是做 Shim 处理直接提供支持:
```
{test: require.resolve('jquery'), loader: 'expose?jQuery'}, //暴露到全局
{test: require.resolve('pen'), loader: 'exports?window.Pen'},
```

### preLoaders和postLoaders
处理顺序 preLoaders -> loaders -> postLoaders
```
module: {
...
    //和loaders一样的语法，很简单
    perLoaders: [
        {
            test: /\.jsx?$/,
            include: APP_PATH,
            loader: 'jshint-loader' //事先安装jshint-loader
        }
    ]
}

...
//配置jshint的选项，支持es6的校验
jshint: {
  "esnext": true
},
```

### resolve
`root`设置根路径
`extension`用于指明程序自动补全识别哪些后缀，这样在引用的时候可以略去，注意一下，extensions 第一个是空字符串! 对应不需要后缀的情况.
```
resolve:{
    root:path.resolve(filePath,'/src'),
    extensions:['','.js']
}
```

`alias`设置别名
```
resolve:{
    alias:{
        "RequestModel":path.resolve(__dirname,'src/lib/request.model')
    }
}, //现在可以require('RequestModel')，不用每次都用require('xxx/xxxx/xxxxx/src/lib/request.model')
```

`modulesDirectories `设置模块起始目录
```
resolve: {
    modulesDirectories: [//取相对路径，所以比起 root ，所以会多很多路径。查找module(可选)
         'node_modules',
         'bower_components',
         'lib',
         'src'
    ]
}
```
此时`entry`指定的是`js/home`而不是`./js/home`

### plugins
[plugin列表](http://webpack.github.io/docs/list-of-plugins.html)

#### 自动安装plugin的plugin
安装
```
npm install npm-install-webpack-plugin --save-dev
```
使用
```
const NpmInstallPlugin = require('npm-install-webpack-plugin');
...
plugins: [
      new webpack.HotModuleReplacementPlugin(),
      new NpmInstallPlugin({
        save: true // --save
      })
]
```

#### clean
[clean-webpack-plugin](https://github.com/johnagan/clean-webpack-plugin)  
安装 
```
npm install --save-dev clean-webpack-plugin
```
用法
```
const CleanWebpackPlugin = require('clean-webpack-plugin')

// webpack config
{
  plugins: [
    new CleanWebpackPlugin(paths [, {options}])
  ]
}
```

#### 版权声明插件
```
plugins: [
    new webpack.BannerPlugin("Copyright Flying Unicorns inc.")
  ],
```

#### 使用UglifyJsPlugin混淆压缩代码
```
plugins:[
	new webpack.optimize.UglifyJsPlugin({
	    compress: {
	    	// 去除代码块内的告警语句
	       warnings: false //webpack2中默认是false
	    },
	    except: ['$super', '$', 'exports', 'require']    //排除关键字
	}),
	// 优先考虑使用最多的模块，并为它们分配最小的ID
	new webpack.optimize.OccurenceOrderPlugin() //webpack2中默认启用
]
```

#### 使用NoErrorsPlugin允许错误不打断程序
```
	new webpack.NoErrorsPlugin()
```

#### 使用ProvidePlugin将模块名称关联
```
new webpack.ProvidePlugin({ //加载jq
   $: 'jquery',
	_:'underscore' //加载underscore
	"window.jQuery": "jquery"
})
```
使用时
```
//这个也不需要了 因为$, jQuery, window.jQuery都可以直接使用了
//import $ from 'jquery';
import './plugin.js';
...
myPromise.then((number) => {
  $('body').append('<p>promise result is ' + number + ' now is ' + moment().format() + '</p>');
  //call our jquery plugin!
  $('p').greenify();
});
```

#### 使用TransferWebpackPlugin把指定文件夹下的文件复制到指定的目录
```
	new TransferWebpackPlugin([ //把指定文件夹下的文件复制到指定的目录
      {from: 'www'}
	], path.resolve(__dirname,"src"))
```

#### 使用html-webpack-plugin自动生成入口文件
安装
```
npm install html-webpack-plugin --save-dev
```
[参考](https://github.com/jantimon/html-webpack-plugin)  
配置
```
new HtmlWebpackPlugin({ //要生成几个html就排列几个HtmlWebpackPlugin到plugins的数组里
	title: 'Hello World App',
	template: './src/html/index.html',
	filename: 'html/index.html',// index-[hash].html
	inject: 'body', //true | 'head' | 'body' | false ,注入所有的资源到特定的 template 或者 templateContent 中，如果设置为true或者body，所有的javascript资源将被放置到body元素的底部，'head' 将放置到head元素中。
	chunks: ['js/index'] // filter chunks，只把js/index这个chunk打包后的js注入到html
	favicon:  
	minify:{ //压缩HTML文件
		removeComments:true, //移除HTML中的注释
		collapseWhitespace:true //删除空白符与换行符
	}
	hash: true | false, 如果为 true, 将添加一个唯一的 webpack 编译 hash 到所有包含的脚本和 CSS 文件，对于解除 cache 很有用。
	cache: true | false，如果为 true, 这是默认值，仅仅在文件修改之后才会发布文件。
	showErrors: true | false, 如果为 true, 这是默认值，错误信息会写入到 HTML 页面中
	chunks: 允许只添加某些块 比如在其中一个针对Web的HtmlWebpackPlugin里使用chunks: ['app', 'vendors']，在另一个针对Mobile里使用chunks: ['mobile', 'vendors']
	chunksSortMode: 允许控制块在添加到页面之前的排序方式，支持的值：'none' | 'default' | {function}-default:'auto'
	excludeChunks: 允许跳过某些块，(比如，跳过单元测试的块)
})
```

模板里面也可以传参，比如

```
//webpack
plugins: [
    new HtmlWebpackPlugin({
      title:'test',
      template:'template/template.html',//模板文件为'template.html'
      dateData: new Date() 
    })
  ]
  
//template
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title><%= htmlWebpackPlugin.options.title  %></title>
</head>
<body>
<div><%=htmlWebpackPlugin.options.dateData %></div>
</body>
</html>
```

如果引入插件还可以在模板里使用ejs语法

安装
```
npm install ejs-compiled-loader
```

配置
```
plugins: [
  new HtmlWebpackPlugin({
  template:'ejs-compiled-loader!template/template.html'})
]
```

设置多入口
```
 plugins: [
    new HtmlWebpackPlugin({
          filename:'a.html',
          template:'src/template/template.html',
          title:'this is a',
          chunks:['a']
    }),
    new HtmlWebpackPlugin({
          filename:'b.html',
          template:'src/template/template.html',
          title:'this is b',
          chunks:['b']
    })
  ]
```

内联形式加载，不再是链接，而是将文件内容直接放入html中

安装
```
npm install --save-dev html-webpack-inline-source-plugin
```

配置
```
var webpack = require('webpack');
var HtmlWebpackPlugin = require('html-webpack-plugin');
var HtmlWebpackInlineSourcePlugin = require('html-webpack-inline-source-plugin');

module.exports = {
  entry: './entry.js',
  output:{
    path: __dirname,//出口路径
    filename: 'js/[id]-[name]-[hash].js'//出口名称
  },
  plugins: [
    new HtmlWebpackPlugin({
        inlineSource: '.(js|css)$'
    }),
    new HtmlWebpackInlineSourcePlugin()
  ]
}
```

#### 使用CommonsChunkPlugin提取公共模块
如果在不同的文件中各自引用了`import React from 'react'`，那么打包的时候react模块会被打包多次，需要使用CommonsChunkPlugin将公共的模块提取到一个公共部分
安装
```
//自带插件，不需要安装
```
[参考](http://webpack.github.io/docs/list-of-plugins.html#commonschunkplugin)  
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
        new CommonsChunkPlugin("admin-commons.js" /*chunkname*/, ["ap1", "ap2"] /*filename*/),
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

看这个更清晰
```
var config = {
  entry: {
    app: path.resolve(__dirname, 'app/main.js'),

    // 当 React 作为一个 node 模块安装的时候，
    // 我们可以直接指向它，就比如 require('react')
    vendors: ['react']
  },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].js'
  },
  module: {
    loaders: [{
      test: /\.js$/,
      exclude: [node_modules_dir],
      loader: 'babel'
    }]
  },
  plugins: [
    new webpack.optimize.CommonsChunkPlugin({
		names: ['vendor', 'manifest']
    })
  ]
};
```

* options.name or options.names(string|string[]): 公共模块的名称
* options.filename (string): 公开模块的文件名（生成的文件名）
* options.minChunks (number|Infinity|function(module,count) - boolean): 为number表示需要被多少个entries依赖才会被打包到公共代码库；为Infinity 仅仅创建公共组件块，不会把任何modules打包进去。并且提供function，以便于自定义逻辑。
* options.chunks(string[]):只对该chunks中的代码进行提取，即对应entry里配置对象的key。
* options.children(boolean):如果为true,那么公共组件的所有子依赖都将被选择进来
* options.async(boolean|string):如果为true,将创建一个 option.name的子chunks（options.chunks的同级chunks） 异步common chunk
* options.minSize(number):所有公共module的size 要大于number，才会创建common chunk

#### 使用extract-text-webpack-plugin独立打包样式文件
[extract-text-webpack-plugin](https://github.com/webpack/extract-text-webpack-plugin)  
打包在js中的style和用`<link>`引用的style不同的是，js中的style生效可能会滞后，而`<link>`中的style在页面打开时立即生效
安装
```
npm install extract-text-webpack-plugin --save-dev
```
用法
```
new ExtractTextPlugin([id: string], filename: string, [options])
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
        new ExtractTextPlugin("[name].[chunkhash].css") //跟着入口配置来
        //new ExtractTextPlugin("style.css", {allChunks: true})  //全打成一个style.css文件
    ]
}
```

#### assets-webpack-plugin插件解决html文件的版本号的问题
安装
```
npm install assets-webpack-plugin --save-dev
```
[参考](https://github.com/kossnocorp/assets-webpack-plugin)  
使用
```
var AssetsPlugin = require('assets-webpack-plugin');

plugins: [
	new AssetsPlugin({
	    filename: 'build/webpack.assets.js',
	    processOutput: function (assets) {
	        return 'window.WEBPACK_ASSETS = ' + JSON.stringify(assets);
	    }
	})
]
```


## 功能开关
有些代码我们只想在开发环境使用(比如 log), 或者 dogfooging 的服务器里边(比如内部员工正在测试的功能). 在你的代码中, 引用全局变量吧:
```
if (__DEV__) {
  console.warn('Extra logging');
}
// ...
if (__PRERELEASE__) {
  showSecretFeature();
}
```
然后配置 Webpack 当中的对应全局变量:
```
// webpack.config.js

// definePlugin 接收字符串插入到代码当中, 所以你需要的话可以写上 JS 的字符串
var definePlugin = new webpack.DefinePlugin({
  __DEV__: JSON.stringify(JSON.parse(process.env.BUILD_DEV || 'true')),
  __PRERELEASE__: JSON.stringify(JSON.parse(process.env.BUILD_PRERELEASE || 'false'))
});

module.exports = {
  entry: './main.js',
  output: {
    filename: 'bundle.js'       
  },
  plugins: [definePlugin]
};
```
然后你在控制台里用 BUILD_DEV=1 BUILD_PRERELEASE=1 webpack 编译. 注意一下因为 webpack -p 会执行 uglify dead-code elimination, 任何这种代码都会被剔除, 所以你不用担心秘密功能泄漏.

## 配合React
安装babel-loader
```
npm install --save-dev babel-loader babel-preset-react
```
配置见上文

如果要配合`react-hot-loader`(已过时)  
安装[react-hot-loader](https://github.com/gaearon/react-hot-loader)
```
npm install --save-dev react-hot-loader
```
如果是使用`jsx-loader`，需要安装`npm install jsx-loader`（现在更推荐使用babel解析jsx）
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
        { test: /\.jsx?$/,   //test: /\.js?$/
        	loaders: ['react-hot', 'jsx?harmony'], //loaders: ['react-hot', 'babel'],
        	include: path.join(__dirname, 'src') 
        }
    ]
}
//plugin里
plugins: [
    new webpack.HotModuleReplacementPlugin(),
    new webpack.NoErrorsPlugin() //防报错的插件
]
```

配合`babel-plugin-react-transform`   
[参考](https://github.com/gaearon/babel-plugin-react-transform)
安装
```
npm install --save-dev babel-plugin-react-transform
```
再安装具体功能的transform（使组件实时刷新后不丢失状态）
```
npm install --save-dev react-transform-hmr react-transform-catch-errors redbox-react
```
目前一共有如下几种transform

* react-transform-hmr - enables hot reloading using HMR API
* react-transform-catch-errors - catches errors inside render()
* react-transform-debug-inspector - renders an inline prop inspector
* react-transform-render-visualizer - highlight components when updated
* react-transform-style - support style and className styling for all components
* react-transform-log-render - log component renders with passed props and state
* react-transform-count-renders - counts how many times your components render

安装babel的[hot loader](https://github.com/danmartinez101/babel-preset-react-hmre)模块使transform生效
```
npm install babel-preset-react-hmre --save-dev
```
在`.babelrc`中配置
```
{
  "presets": [
    "es2015",
    "react",
	], 
	"env": {
    	"development": {
      		"presets": [
        		"react-hmre"
      		]
		} 
	}
}
```
不用`babel-preset-react-hmre`的`.babelrc`配置示例
```
{
  "presets": ["react", "es2015"],
  "env": {
    // this plugin will be included only in development mode, e.g.
    // if NODE_ENV (or BABEL_ENV) environment variable is not set
    // or is equal to "development"
    "development": {
      "plugins": [
        // must be an array with options object as second item
        ["react-transform", {
          // must be an array of objects
          "transforms": [{
            // can be an NPM module name or a local path
            "transform": "react-transform-hmr",
            // see transform docs for "imports" and "locals" dependencies
            "imports": ["react"],
            "locals": ["module"]
          }, {
            // you can have many transforms, not just one
            "transform": "react-transform-catch-errors",
            "imports": ["react", "redbox-react"]
          }, {
            // can be an NPM module name or a local path
            "transform": "./src/my-custom-transform"
          }]
          // by default we only look for `React.createClass` (and ES6 classes)
          // but you can tell the plugin to look for different component factories:
          // factoryMethods: ["React.createClass", "createClass"]
        }]
      ]
    }
  }
}
```

## webpack-dev-server
安装  
```
npm install --save-dev webpack-dev-server
```

直接启动 `webpack-dev-server --progress --colors` 

用`webpack.config.js`配置  
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
        port: 8080, //设置默认监听端口，如果省略，默认为”8080“
        colors: true, //设置为true，使终端输出的文件为彩色的
        inline: true, //设置为true，当源文件改变时会自动刷新页面
	compress: true, //gzip压缩
        hot: true, //热加载
	quiet: true, //当它被设置为true的时候，控制台只输出第一次编译的信息，当你保存后再次编译的时候不会输出任何内容，包括错误和警告
	overlay: true, //编译出错在浏览器上显示错误信息
        stats: 'errors-only', //console里的显示，其他"minimal"，"normal"，"verbose"
        contentBase: './dist', //默认webpack-dev-server会为根文件夹提供本地服务器，如果想为另外一个目录下的文件提供本地服务器，应该在这里设置其所在目录
        //contentBase: path.join(__dirname) 此时入口为http://localhost:8080/(index.html),不指定入口则为http://localhost:8080/webpack-dev-server/
        historyApiFallback: true, //在开发单页应用时非常有用，它依赖于HTML5 history API，如果设置为true，所有的跳转将指向index.html
        displayErrorDetails: true,
        proxy: {  //代理,用来跨域
          '/api/*': {
              target: 'http://localhost:5000',
              secure: false,
	      changeOrigin: true
          }
        }
    },
    module: {
        loaders: loaders
    },
    plugins: [ new webpack.HotModuleReplacementPlugin() ]
};
```

用`package.json`配置
```
{
  "scripts": {
    "build": "webpack",
    "dev": "webpack-dev-server --devtool eval --progress --colors --hot --content-base build"
  }
}
```

1. webpack-dev-server - 在 localhost:8080 建立一个 Web 服务器
2. --devtool eval - 为你的代码创建源地址。当有任何报错的时候可以让你更加精确地定位到文件和行号
3. --progress - 显示合并代码进度
4. --colors - Yay，命令行中显示颜色！
5. --content-base build - 指向设置的输出目录

webpack-dev-server是在内存里生成打包文件，所以在本地路径上找不到打包后的文件

webpack-dev-server有两种模式支持自动刷新——iframe模式和inline模式

* 在iframe模式下：页面是嵌套在一个iframe下的，在代码发生改动的时候，这个iframe会重新加载
* 在inline模式下：一个小型的webpack-dev-server客户端会作为入口文件打包，这个客户端会在后端代码改变的时候刷新页面

使用iframe模式，无需额外配置，只需在浏览器输入`http://localhost:8080/webpack-dev-server/index.html`

使用inline模式有两种方式：命令行和nodejs API

1. 命令行方式，在运行时，加上`--inline`选项(或者在`webpack.config.js`指定)，此时访问`http://localhost:8080`
2. nodejs API 方式 ，需要手动把`webpack-dev-server/client?http://localhost:8080`加到配置文件的入口文件处

## webpack-dev-middleware
安装
```
npm install webpack-dev-middleware --save-dev
```
[参考](https://blog.risingstack.com/using-react-with-webpack-tutorial/)

## webpack-merge用于配置分离
[webpack-merge](https://github.com/survivejs/webpack-merge)  
安装
```
npm install --save-dev webpack-merge
```

## 开发和生产
```
"scripts": {
	"dev": "webpack-dev-server --devtool eval --progress --colors --hot --content-base build",
	"deploy": "NODE_ENV=production webpack -p --config webpack.production.config.js"
}
```

## 配合grunt/gulp
参考 [Grunt配置](http://webpack.github.io/docs/usage-with-grunt.html) [gulp配置](http://webpack.github.io/docs/usage-with-gulp.html)

gulp  
```
var gulp = require('gulp');
var webpack = require('gulp-webpack');
var webpackConfig = require('./webpack.config');
gulp.task("webpack", function() {
    return gulp
        .src('./')
        .pipe(webpack(webpackConfig))
        .pipe(gulp.dest('./build'));
});
```
或者
```
gulp.task("webpack", function(callback) {
    // run webpack
    webpack({
        // configuration
    }, function(err, stats) {
        if(err) throw new gutil.PluginError("webpack", err);
        gutil.log("[webpack]", stats.toString({
            // output options
        }));
        callback();
    });
});
```

## Webpack分析可视化工具
[网址](http://chrisbateman.github.io/webpack-visualizer/)

## 参考

* http://www.jianshu.com/p/1c4fd72b84e8
* http://www.w2bc.com/Article/50764
* https://fakefish.github.io/react-webpack-cookbook
* http://zhaoda.net/webpack-handbook
* http://www.cnblogs.com/leinov/p/5241185.html
* http://www.tuicool.com/articles/2qiE7jN
* http://www.cnblogs.com/LIUYANZUO/p/5184424.html
* http://gaearon.github.io/react-hot-loader/getstarted/
* http://www.cnblogs.com/xianyulaodi/p/5314769.html
* http://www.cnblogs.com/yangjunhua/p/5615118.html
* https://zhuanlan.zhihu.com/p/20367175
* https://zhuanlan.zhihu.com/p/20397902
* https://segmentfault.com/a/1190000002767365
* https://segmentfault.com/a/1190000002551952
* http://www.cnblogs.com/leinov/p/5330944.html
* https://segmentfault.com/a/1190000002552008
* http://web.jobbole.com/85396/
* http://www.cnblogs.com/giveiris/p/5237080.html
* http://www.cnblogs.com/wdlhao/p/5801918.html
* http://www.cnblogs.com/wdlhao/p/5807157.html
* http://web.jobbole.com/87408/
* http://www.07net01.com/2015/08/890558.html
* http://www.cnblogs.com/yangjunhua/p/5680168.html
* http://www.cnblogs.com/hh54188/p/6633671.html
* https://segmentfault.com/a/1190000005612506
* https://segmentfault.com/a/1190000005666159
* http://www.cnblogs.com/yincheng/p/webpack.html
* http://blog.csdn.net/q1056843325/article/details/54600090
* http://www.cnblogs.com/tugenhua0707/p/5576262.html
* http://www.cnblogs.com/sloong/p/5584684.html
* http://www.cnblogs.com/sloong/p/5689162.html
* https://zhuanlan.zhihu.com/p/20522487
* http://web.jobbole.com/84847/
* http://www.cnblogs.com/zhengrunlin/p/7070479.html
* http://www.cnblogs.com/xiaohuochai/p/7007391.html
