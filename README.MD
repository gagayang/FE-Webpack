# webpack 打包

## 简介
	webpack 可以说是目前打包界最流行的。这得益于他的简单配置方案，和它优秀的扩展机制（loader、plugin）。今天我就开撸webpack的。
	
	其实我接触webpack 还是很早的。在15年的时候公司就打算从backbone 到react。grunt 到 webpack的转变。我的主要角色和任务就是webpack的攻坚。
	
### 	为什么要打包
从前端的工程话角度来看，我们的前端越来越庞大。js文件越来越多。以前我们都是直接在html上直接使用

`<script src="xxx.js">`

这样的问题就出现了。

第一我们无法保证js的加载执行顺序。我们只能把一些基础的js 尽量放到前面。但是这样仍然是有问题。

第二 性能问题。同一个浏览器在并行下载资源的数量是有限的。目前应该是在8-10个左右。如果前端资源有40个。那么就会产生并行下载。速度就产生问题。

第三 在老的ie 里面 script的标签是有数量限制的(大概是 30来个具体不记得了)。link的css 单个也是有大小限制。超过限制就不会解释执行。

第四 前端太大 需要压缩。而且有一些混淆的需求出现了。

这样后来就出现了前端模块化

### AMD标准
前端的一种模块化标准。其具体实现是requirejs。在以前还是很流行的。

它的特点是预加载。开始的时候就读入内存。等到使用的时候直接就可以食用了。使用define关键字 把需要的资源都列出来

`define(['a.js','b.js'], funciton(a, b){})`

大概是上面这样。我没有查资料就按照记忆中的写了下。

特点是 第一次加载的时候可能速度慢一点。但是运行起来就快很多了。

### CMD标准

前端的另外一种模块化标准。是具体是实现 seajs 

它的特点就和AMD走了另外一套。它讲究的是懒加载。什么是懒加载呢? 就是当用到了再去加载。感觉是不是不太对劲。它也是define 关键字进行模块化定义。具体记不清楚了。

它的特点就是 懒加载 开始很快但是每次都要运行到这里就再次进行加载过来。

他们两个就是两个极端。感觉都不是特别的合适

### commonjs标准

这里的commonjs 标准其实是从nodejs过来的

使用require 和 export 来进行模块化。 这里就做更多的讨论 因为es6 做了很多的这种借鉴。

### es6 模块化

这里就聊到了我们经常使用的 es6的模块化了

具体来说 我们用 export 关键字来表示外部能引入的资源。 使用import 来引入资源

`import a from xxx.js  export default a`

`impoort { a } form xxx.js export default { a }; export a` 

`import {a as A } from xxx.js`

基本上就是类似上面的使用方式

这里要注意 import 其实是静态引入。

静态引入就是在编译的过程中就会进行查找。 webpack 的treeshark 就是利用这个特性做的。

这里要注意 我先说下 在commonjs 的require 是动态引入。就是在执行到的时候再去引入执行。

还有webpack 3.0以后支持的 import ('') 也是动态执行。这里先记住下

##什么是打包

### 翻译

ECMAScript的标准 特别是6推出以后我们可以用比较少的代码完成以前大量的工作。然后把工作专注在开发商。但是浏览器的厂商根本没有发展的那么快。还有很多的es6以上的标准我们浏览器无法直接使用。还有一些可能需要兼容ie8 (这玩意其实是es3.5的一套语法规则)
这样我们就需要向下翻译。这就是打包的一个方面

另外一个就是我们使用了很多css样式的预编译工具。 比如less scss sass。这样的使用让我们开发样式的时候提供了很大的方便。但是相应的也需要变成浏览器能够识别的css才可以。
所有 打包的第一步就是要进行翻译。

### 压缩混淆

第二步我认为就是为了新能和安全。要把资源进行优化。


通过上面我就总结了 打包的特点。


这样我们就来看一下市面上常用的打包和它们的优缺点

## 常用打包工具

### grunt
这个是我以前第一次接触到的打包工具。它主要特点是 任务式的。这里要我们自己注册任务。比如 less sass 等任务。然后我们在package.json 中script 里面直接调用。

感觉不是很方便。我觉得这个东西应该自动的完成。而不是我们手动的写。

### gulp

这个也是任务式的打包工具。和grunt 类似。它是流式的。这样速度就会快一点。感觉就是老家话蛤蟆吃长虫。吞了一截算一截吧。也不管后面。
同样它也需要手动去注册任务。

### file3

这个是百度推出的一个打包工具。我不太会。但是看了看也是基于流进行工作的。估计是百度系的人比较熟悉吧。

### rollup

有人懂的话希望可以给补充下啊

### webpack

这个是我们目前用的最多的了。
这个是得益于两点

1 react 不停的吹嘘。当然也的确好用。
在15年facebook 刚推出的时候就是他就说 和react是绝配。不停的推荐大家用

2 的确是好用 (感觉是废话)

它的loader 机制就是对应了翻译过程。plugin 机制就正好对应了 优化（甚至还提供更多更强大的功能。）

我们就着重说下webpack呗 其实很多分析已经烂大街了。如果我15年就写 没准已经成大神了 。

它的原理

1. 	万物皆模块。然后就使用loader 进行配置不同的模块进行处理
2. 从入口建立反向查找的树形结构。（从树的主干开始查找到每一个叶子。读入内存中） 

## webpack 配置

	{
		model: 'production', // 传入环境变量 用于让webpack 处理优化策略
		entry: {
			'a': 'xxx/a.js',
			'b': 'xxx/b.js'
		}, // 这里是说入口文件 这里支持三种形式 1 string 这里单页面基本上就这样。 在webpack 4的默认文件中为 src/index.js 2、上面的对象形式。这样的好处就是在生成的文件中可以使用name 有明确的规定。3、使用数组形式[a.js, b.js]
		output: {
			publicPath: '/',
			path: '',
			filename: '',
			libraryTarget: ''
		},
		module: {
			noParse: '',// 这里就是一个优化的点。这里配置的一个正则| function 字符串就是不加入到编译的序列里面。
			rules: {
				test: /\.vue$/,// 这里就是说匹配的后缀名
				loader: ''vue-loader,// 当匹配上之后使用loader
				options: vueLoaderConfig,// 这里就是这个loader的参数
				enforce: '',// loader的种类。我也没有遇到过				exclude:[] //数组类型表示排除这些地方的vue文件
				use:'', 这里其实也是loader的意思 要注意的是webpack采用的是从右往左。从下往上的解析路径。
				
			}
		},
		reslove: {
			alias: {
				'src': path.reslove('src')
			},//别名
			extensions: ['.json', '.js', '.vue'] //隐藏扩展名 在引入的时候不用写自己查找	modules: ''//解析的目录 这里制定了也可以增加打包速度
		},	
		optimization: { // 从webpack4 之后 默认的时候通过mode 来进行优化策略，这里其实我们也可以自定义优化策略 这里就不详细写了
			minimize: true //这里是表示要用一个插件 TerserPlugin 来进行最小化代码。以前我们使用UglifyJsPlugin在webpack 4中不能最小化ecmascript 2015
			minimizer: []//这里就是上面那个插件的配置信息了。很奇怪啊这里的插件不再plugins里面了
		},//这个就是优化 但是优化什么呢？
		plugins: [
			DefinePlugin()// 定义环境变量等信息的插件 
			ExtractTextPlugin()// 把css从每一个model中抽出变成一个css文件
			CopyWebpackPlugin()  // 主要是用来把承载页给生成
			HotModuleReplace() // 热更新插件 后续会说一下热更新原理
			HtmlWebpackPlugin() // 拷贝静态资源的插件
			UglifyJsPlugin() //压缩混淆代码的 
			OptimizeCSSPlugin() //优化css的插件
			HashedModuleIdsPlugin() //使用hash作为moduleid
			CommonsChunkPlugin() // 把公用代码作为vender 来抽出来
			CompressionWebpackPlugin() //使用giz 传输
		]
	}




	