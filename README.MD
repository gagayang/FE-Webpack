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


## webpack 源码分析
本次分析以 3.0版本为依托。 4.0以上版本 作者对于写法有比较大的改动。但原理没有改变

首先查看webpack的package.json 这里有个main 是lib/webpack.js 这里就是webpack的入口文件。

webpack.js 分析

首先是引入了一大堆的别的模块 值得注意的主要是这个

	Compiler.js 这个就是所有的编译查找的类
	
	WebpackOptionsDefaulter.js 这个类就是把一些默认的配置搞进去。但是这个并不是很重要。
	
	WebpackOptionsApply.js 这个其实是最重要的部分 这个会把一些插件给加入到webpack中。
	
	
这里要注意说明下。webpack最重要的机制就是它的插件机制。通过它的插件机制它可以提供很多丰富的插件来完善自己提供更加强大的功能。	
	
	这个js就产生了一个compiler实例。然后开始触发设置的hook 完善配置 调用 compiler的 run方法。
	
接下来就要看 compiler.js的代码 上面就是调用了run方法那么我就主要开始分析run干了啥

	这里看到其实他就做了两个事情
	
	1 触发了两个hook 1、before-run 2、run
	
	2 调用自身的 compile方法 
	
这里就看到了其实它主要是用调用了compile方法

但是要主要这里 compiler 继承自一个叫做Tapable的类。这个类提供了hook机制。这个机制其实和事件监听是一个类型就是提供了一个事件队列 然后出发的时候依次调用。这里要注意 hook 分为这么几种
	
	1 同步hook 它在代码中是没有async的结尾的。我把它作为一种通知的hook。它的意思就是 我要做什么事情了。通知到关心的人。但是关心的人并不能对发生的事情有任何阻止。只是一个单纯的通知到。也没有任何返回。
	
	2 异步hook 这个基本上就是以 async结尾的hook。这里就是它的返回结果将会影响下一步。
	
		这里要注意异步的hook 也分两种。1、 串行。每一个都是单纯的一个hook。就是类似普通的异步hook 就是一个个来。
		2、并行的hook 这里就是多个都完成之后才会执行下一步。很类似 promise的all 或者说更类似于 promise的 新的那个allSettled 都是reslove 才会执行下一步。 这个一般是以parallel结尾。
		
那么我们看下 compile方法里面都干了啥

	第一执行了下 before-compile hook 看看有没有人在编译之前还要干啥 是否要检查下。这个是异步hook。
	
	第二个就是一个同步hook compile 
	
	第三步这里就出现了另外一个灰常重要的类 copmilation 就是真正干活的人来了。
	
	第四这就出现了一个灰常重要的hook 这个就是并行的异步hook make 当时我第一次看到的时候也纳闷了很久这个hook在哪里搞得。先不管先看下一步
	
	第五步就是 我干完了 然后调用 compilation的finish方法
	
	第六步 掉用了compilation的eal方法 里面有个异步的hook after-compile
	
	第七部 返回了callback方法
	
这里咱们就看到了其实主要的动作就是那个并行的hook make 那这个玩意是在哪里呢？ 我们从上面的源码分析中似乎并没有发现。不要着急，其实上面我说了有个很重要的WebpackOptionsApply.js 这里就是把很多的插件给默认弄进去了 这里我们就去看看它吧

WebpackOptionsApply.js 

	这里就是一大堆插件 这里我们注意到后个叫做 EntryOptionPlugin.js这个就是一个入口的插件。这里就提供了webpack 打包入口。
	
我们看看这个 入口文件做了什么

	打开之后发现其实他就做了 这么几件事情
	1、注册了‘entry-option’ 事件
	2、判断入口是单入口还是多入口还是回调方法	
	3、根据上面不同来选择不同的插件
	
这里我们就按照单入口的形式进行插件调用	singleEntryPlugin.js

打开这个文件查看这个文件的作用
	1、 注册了一个compilation 的事件。触发之后产生了一个normalModuleFactory的工厂类。这个工厂类就是产生module的。这里要记住下
	2、注册了make事件。这里就要进行读取文件方法了。这个产生一个依赖然后顺着这个依赖进行查找整个的module。
	这里就调用了comilation.addEntry方法。我们看一下addEntry
	
	
comilation.js

这里又回到了这个方法 我们开始查看通过addEntry来做的一系列操作

	1、通过另外一个_addModuleChain 方法来把module进行串联起来
	2、这里的_addModule里面就调用moduleFactory产生模块
	
我们来看看模块产生干了啥

因为我们上面使用的是normalModuleFactory 所以里面产生的是normalModule 我们就看下normalModule的源码

	这里就是有个dobuild 方法这里就要看这个了
	
	这里有个runLoaders 方法 这里就是通过这个类把里面的loader 进行引入和使用。我们可以简单看看
	
	哇哦 里面功能太丰富了。简单来说就是通过配置里面的loader 真正查找loader 然后找到这个nodule真正需要哪一个loader 然后调用编译方法 。把返回值存起来
	
上面的过程简单来说就是make插件做的。是不是感觉很多。其实上面很多都是在编译前期。这里没有很多的hook给开发人员。大多数都是默认就行了。	

接下来就是上面的coplation.js里面的 finish方法这里就做了很多的完成后的动作。其实我们很多的插件都是在这个阶段。这个阶段提供了很多优化方案，

先写到这里吧。 心情很低落。



	
		

	
	
	



	