---
title: Vue项目性能优化
date: 2021-03-17 13:46:38
tags: 性能优化
categories:
  - 性能优化
---

Vue项目优化, 以下三部分组成:

> + 基础的 Web 技术层面的优化。
> + webpack 配置层面的优化；
> + Vue 代码层面的优化；

## __一.基础的 Web 技术优化__

### __1.开启 gzip 压缩__

gzip 是 GNUzip 的缩写，最早用于 UNIX 系统的文件压缩。HTTP 协议上的 gzip 编码是一种用来改进 web 应用程序性能的技术，web 服务器和客户端（浏览器）必须共同支持 gzip。

目前主流的浏览器，Chrome，firefox，IE等都支持该协议。常见的服务器如 Apache，Nginx，IIS 同样支持，gzip 压缩效率非常高，通常可以达到 70% 的压缩率，也就是说，如果你的网页有 30K，压缩之后就变成了 9K 左右

### __2.浏览器缓存__

为了提高用户加载页面的速度，对静态资源进行缓存是非常必要的，根据是否需要重新向服务器发起请求来分类，将 HTTP 缓存规则分为两大类（强缓存，协商缓存）

### __3.CDN 的使用__

浏览器从服务器上下载 CSS、js 和图片等文件时都要和服务器连接，而大部分服务器的带宽有限，如果超过限制，网页就半天反应不过来。而 CDN 可以通过不同的域名来加载文件，从而使下载文件的并发连接数大大增加，且CDN 具有更好的可用性，更低的网络延迟和丢包率 。

### __4.使用 Chrome Performance 查找性能瓶颈__

window.performance是W3C性能小组引入的新的API，目前IE9以上的浏览器都支持。

![Performance](/images/performance.png)

字段说明:

<b class="c42b983">connectStart</b> 和 <b class="c42b983">connectEnd</b>: 分别代表TCP建立连接和连接成功的时间节点。

<b class="c42b983">domComplete</b>：html文档完全解析完毕的时间节点。

<b class="c42b983">domContentLoadedEventStart</b> 和 <b class="c42b983">domContentLoadedEventEnd</b>：代表DOMContentLoaded事件触发和完成的时间节点。页面文档完全加载并解析完毕之后,会触发DOMContentLoaded事件，HTML文档不会等待样式文件,图片文件,子框架页面的加载(load事件可以用来检测HTML页面是否完全加载完毕(fully-loaded))。

<b class="c42b983">domInteractive</b>：代表浏览器解析html文档的状态为interactive时的时间节点。domInteractive并非DOMReady，它早于DOMReady触发，代表html文档解析完毕（即dom tree创建完成）但是内嵌资源（比如外链css、js等）还未加载的时间点。

<b class="c42b983">domLoading</b>：代表浏览器开始解析html文档的时间节点。我们知道IE浏览器下的document有readyState属性，domLoading的值就等于readyState改变为loading的时间节点。

<b class="c42b983">domainLookupStart</b> 和 <b class="c42b983">domainLookupEnd</b>：分别代表DNS查询的开始和结束时间节点。如果浏览器没有进行DNS查询（比如使用了cache），则两者的值都等于fetchStart。

<b class="c42b983">fetchStart</b>：是指在浏览器发起任何请求之前的时间值。在fetchStart和domainLookupStart之间，浏览器会检查当前文档的缓存。

<b class="c42b983">loadEventStart</b> 和 <b class="c42b983">loadEventEnd</b>：分别代表onload事件触发和结束的时间节点。

<b class="c42b983">navigationStart</b>：代表浏览器开始unload前一个页面文档的开始时间节点。比如我们当前正在浏览baidu.com，在地址栏输入google.com并回车，浏览器的执行动作依次为：unload当前文档（即baidu.com）->请求下一文档（即google.com）。navigationStart的值便是触发unload当前文档的时间节点。

<b class="c42b983">redirectStart</b> 和 <b class="c42b983">redirectEnd</b>：如果页面是由redirect而来，则redirectStart和redirectEnd分别代表redirect开始和结束的时间节点。

<b class="c42b983">requestStart</b>：代表浏览器发起请求的时间节点，请求的方式可以是请求服务器、缓存、本地资源等。

<b class="c42b983">responseStart</b> 和 <b class="c42b983">responseEnd</b>：分别代表浏览器收到从服务器端（或缓存、本地资源）响应回的第一个字节和最后一个字节数据的时刻。

<b class="c42b983">secureConnectionStart</b>：可选。如果页面使用HTTPS，它的值是安全连接握手之前的时刻。如果该属性不可用，则返回undefined。如果该属性可用，但没有使用HTTPS，则返回0。

<b class="c42b983">unloadEventStart</b> 和 <b class="c42b983">unloadEventEnd</b>：如果前一个文档和请求的文档是同一个域的，则unloadEventStart和unloadEventEnd分别代表浏览器unload前一个文档的开始和结束时间节点。否则两者都等于0。

主要性能指标：

> + DNS查询耗时 = domainLookupEnd - domainLookupStart
> + TCP链接耗时 = connectEnd - connectStart
> + request请求耗时 = responseEnd - responseStart
> + 解析dom树耗时 = domComplete - domInteractive
> + 白屏时间 = domLoading - fetchStart
> + domready时间 = domContentLoadedEventEnd - fetchStart
> + onload时间 = loadEventEnd - fetchStart

## __二.Webpack 层面的优化__

### __1 Webpack 对图片进行压缩__

在 vue 项目中除了可以在 webpack.base.conf.js 中 url-loader 中设置 limit 大小来对图片处理，对小于 limit 的图片转化为 base64 格式，其余的不做操作。

所以对有些较大的图片资源，在请求资源的时候，加载会很慢，我们可以用 image-webpack-loader来压缩图片：

1. 首先，安装 image-webpack-loader:

```
  npm install image-webpack-loader --save-dev
```

2. 然后，在 webpack.base.conf.js 中进行配置：

```json
{
  test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
  use:[
    {
    loader: 'url-loader',
    options: {
      limit: 10000,
      name: utils.assetsPath('img/[name].[hash:7].[ext]')
      }
    },
    {
      loader: 'image-webpack-loader',
      options: {
        bypassOnDebug: true,
      }
    }
  ]
}
```

### __2.减少 ES6 转为 ES5 的冗余代码__

Babel 插件会在将 ES6 代码转换成 ES5 代码时会注入一些辅助函数，例如下面的 ES6 代码：

```
class HelloWebpack extends Component{...}
```

这段代码再被转换成能正常运行的 ES5 代码时需要以下两个辅助函数：

```
babel-runtime/helpers/createClass  // 用于实现 class 语法
babel-runtime/helpers/inherits  // 用于实现 extends 语法
```

在默认情况下， Babel 会在每个输出文件中内嵌这些依赖的辅助函数代码，如果多个源代码文件都依赖这些辅助函数，那么这些辅助函数的代码将会出现很多次，造成代码冗余。

为了不让这些辅助函数的代码重复出现，可以在依赖它们时通过 require('babel-runtime/helpers/createClass') 的方式导入，这样就能做到只让它们出现一次。

babel-plugin-transform-runtime 插件就是用来实现这个作用的，将相关辅助函数进行替换成导入语句，从而减小 babel 编译出来的代码的文件大小。

1. 首先，安装 babel-plugin-transform-runtime ：

```
npm install babel-plugin-transform-runtime --save-dev
```

2. 然后，修改 .babelrc 配置文件为：

```json
"plugins": [
    "transform-runtime"
]
```

### __3.提取公共代码__

如果项目中没有去将每个页面的第三方库和公共模块提取出来，则项目会存在以下问题：

> + 相同的资源被重复加载，浪费用户的流量和服务器的成本。
> + 每个页面需要加载的资源太大，导致网页首屏加载缓慢，影响用户体验。

所以我们需要将多个页面的公共代码抽离成单独的文件，来优化以上问题 。Webpack 内置了专门用于提取多个Chunk 中的公共部分的插件 CommonsChunkPlugin，我们在项目中 CommonsChunkPlugin 的配置如下：

```javascript
  // 所有在 package.json 里面依赖的包，都会被打包进 vendor.js 这个文件中。
  new webpack.optimize.CommonsChunkPlugin({
    name: 'vendor',
    minChunks: function(module, count) {
      return (
        module.resource &&
        /\.js$/.test(module.resource) &&
        module.resource.indexOf(
          path.join(__dirname, '../node_modules')
        ) === 0
      );
    }
  }),
  // 抽取出代码模块的映射关系
  new webpack.optimize.CommonsChunkPlugin({
    name: 'manifest',
    chunks: ['vendor']
  })
```

### __4.模板预编译__

当使用 DOM 内模板或 JavaScript 内的字符串模板时，模板会在运行时被编译为渲染函数。通常情况下这个过程已经足够快了，但对性能敏感的应用还是最好避免这种用法。

预编译模板最简单的方式就是使用单文件组件——相关的构建设置会自动把预编译处理好，所以构建好的代码已经包含了编译出来的渲染函数而不是原始的模板字符串。

如果你使用 webpack，并且喜欢分离 JavaScript 和模板文件，你可以使用 vue-template-loader，它也可以在构建过程中把模板文件转换成为 JavaScript 渲染函数。

### __5.提取组件的 CSS__

当使用单文件组件时，组件内的 CSS 会以 style 标签的方式通过 JavaScript 动态注入。这有一些小小的运行时开销，如果你使用服务端渲染，这会导致一段 “无样式内容闪烁 (fouc) ” 。

将所有组件的 CSS 提取到同一个文件可以避免这个问题，也会让 CSS 更好地进行压缩和缓存。

### __6.优化 SourceMap__

我们在项目进行打包后，会将开发中的多个文件代码打包到一个文件中，并且经过压缩、去掉多余的空格、babel编译化后，最终将编译得到的代码会用于线上环境，那么这样处理后的代码和源代码会有很大的差别，

当有 bug的时候，我们只能定位到压缩处理后的代码位置，无法定位到开发环境中的代码，对于开发来说不好调式定位问题，因此 sourceMap 出现了，它就是为了解决不好调式代码问题的。

SourceMap 的可选值如下（+ 号越多，代表速度越快，- 号越多，代表速度越慢, o 代表中等速度 ）

![SourceMap](/images/yy.png)

开发环境推荐：cheap-module-eval-source-map

生产环境推荐：cheap-module-source-map

原因如下：

> cheap：源代码中的列信息是没有任何作用，因此我们打包后的文件不希望包含列相关信息，只有行信息能建立打包前后的依赖关系。因此不管是开发环境或生产环境，我们都希望添加 cheap 的基本类型来忽略打包前后的列信息；

> module ：不管是开发环境还是正式环境，我们都希望能定位到bug的源代码具体的位置，比如说某个 Vue 文件报错了，我们希望能定位到具体的 Vue 文件，因此我们也需要 module 配置；

> soure-map ：source-map 会为每一个打包后的模块生成独立的 soucemap 文件 ，因此我们需要增加source-map 属性；

> eval-source-map：eval 打包代码的速度非常快，因为它不生成 map 文件，但是可以对 eval 组合使用 eval-source-map 使用会将 map 文件以 DataURL 的形式存在打包后的 js 文件中。在正式环境中不要使用 eval-source-map, 因为它会增加文件的大小，但是在开发环境中，可以试用下，因为他们打包的速度很快。

### __7.构建结果输出分析__

Webpack 输出的代码可读性非常差而且文件非常大，让我们非常头疼。

为了更简单、直观地分析输出结果，社区中出现了许多可视化分析工具。这些工具以图形的方式将结果更直观地展示出来，让我们快速了解问题所在。

接下来讲解我们在 Vue 项目中用到的分析工具：webpack-bundle-analyzer 。

我们在项目中 webpack.prod.conf.js 进行配置：

```javascript
if (config.build.bundleAnalyzerReport) {
  var BundleAnalyzerPlugin =   require('webpack-bundle-analyzer').BundleAnalyzerPlugin;
  webpackConfig.plugins.push(new BundleAnalyzerPlugin());
}
```

执行 $ npm run build \--report 后生成分析报告：

### __8.Vue 项目的编译优化__

如果你的 Vue 项目使用 Webpack 编译，需要你喝一杯咖啡的时间，那么也许你需要对项目的 Webpack 配置进行优化，提高 Webpack 的构建效率。

## __Vue 代码层面的优化__

### __1.v-if 和 v-show 区分使用场景__

### __2.computed 和 watch 区分使用场景__

computed：是计算属性，依赖其它属性值，并且computed的值有缓存，只有它依赖的属性值发生改变，下一次获取computed的值时才会重新计算computed的值；

watch： 更多的是「观察」的作用，类似于某些数据的监听回调 ，每当监听的数据变化时都会执行回调进行后续操作；

运用场景：

> 当我们需要进行数值计算，并且依赖于其它数据时，应该使用 computed，因为可以利用 computed 的缓存特性，避免每次获取值时，都要重新计算；
> 当我们需要在数据变化时执行异步或开销较大的操作时，应该使用 watch，使用 watch 选项允许我们执行异步操作 ( 访问一个 API )，限制我们执行该操作的频率，并在我们得到最终结果前，设置中间状态。这些都是计算属性无法做到的。

### __3.v-for 遍历必须为 item 添加 key，且避免同时使用 v-if__

1. v-for 遍历必须为 item 添加 key
在列表数据进行遍历渲染时，需要为每一项 item 设置唯一 key 值，方便 Vue.js 内部机制精准找到该条列表数据。当 state 更新时，新的状态值和旧的状态值对比，较快地定位到 diff 。

2. v-for 遍历避免同时使用 v-if
v-for 比 v-if 优先级高，如果每一次都需要遍历整个数组，将会影响速度，尤其是当之需要渲染很小一部分的时候，必要情况下应该替换成 computed 属性。

### __4.路由懒加载__

Vue 是单页面应用，可能会有很多的路由引入 ，这样使用 webpcak 打包后的文件很大，当进入首页时，加载的资源过多，页面会出现白屏的情况，不利于用户体验。

如果我们能把不同路由对应的组件分割成不同的代码块，然后当路由被访问的时候才加载对应的组件，这样就更加高效了。这样会大大提高首屏显示的速度，但是可能其他的页面的速度就会降下来。

### __5.第三方插件的按需引入__

我们在项目中经常会需要引入第三方插件，如果我们直接引入整个插件，会导致项目的体积太大，我们可以借助 babel-plugin-component ，然后可以只引入需要的组件，以达到减小项目体积的目的。以下为项目中引入 element-ui 组件库为例：

1. 首先，安装 babel-plugin-component ：
```
npm install babel-plugin-component -D
```

2. 然后，将 .babelrc 修改为：
```json
{
  "presets": [["es2015", { "modules": false }]],
  "plugins": [
    [
      "component",
      {
        "libraryName": "element-ui",
        "styleLibraryName": "theme-chalk"
      }
    ]
  ]
}
```

3.在 main.js 中引入部分组件：
```javascript
  import Vue from 'vue';
  import { Button, Select } from 'element-ui';

  Vue.use(Button)
  Vue.use(Select)
```