---
title: 提升webpack打包速度的方法
date: 2021-03-25 19:28:40
tags: webpack
categories:
  - webpack
---

## __1.跟上技术的迭代，尽可能使用新版本的webpack,node,npm, yarn__

## __2.在尽可能少的模块上应用loader__

> + 合理的使用include或者exclude可以降低loader的使用频率，提高打包速度
> + 那么图片文件是否需要配置include或者exclude呢，实际上是不需要的，因为无论引入哪里的图片，实际上都需要url-loader来帮我们打包到dist目录下

```javascript
  {
    module: {
      rules: [
        {
          test: /\.js$/,
          exclude: /node_modules/,
          include: path.resolve(__dirname,'../src'),
          loader: 'babel-loader'
        }
      ]
    }
  }
```

## __3.Plugin尽可能精简，并确保可靠__

> + 比如线上环境的配置文件中使用了optimize-css-assets-webpack-plugin这样一个插件，对我们的CSS文件进行了压缩，可是如果我们在开发环境下，是没有必要对代码进行压缩的。

## __4.resolve参数合理配置__

> 1. 引入一个模块的时候，省略这个模块的后缀
> 2. 引入一个文件夹路径时, 自动匹配路径下的index文件或child文件，一般不建议配置; 虽然mainFiles配置解决了我们的问题，但是它也会带来性能上的问题，因为需要在路径下不停的去找我们配置的名字的文件是否匹配,所以一般情况下我们不会配置这个参数
> 3. 设置路径别名; 当看到的是lee这个路径的时候，实际上它是path.resolve(__dirname, '../src/child')这个路径的别名

```javascript
  {
    resolve: {
      extensions: ['.js', '.vue'], // 引入文件省略后缀
      mainFiles: ['index', 'child'], // 引入路径，自动查找文件
      alias: {
        '@': path.resolve(__dirname,'../src') // 设置路径别名
      }
    }
  }
```

## __5.使用DLLPlugin提升打包速度__

> 背景: 我们在引入第三方模块的时候，每次重新打包的时候，webpack都要重新分析这些第三方模块，然后把它们打包到我们的项目之中。

> 解决方案: 我们可以把这些第三方模块单独打包生成一个文件，只在第一次打包的时候去分析这个文件里的代码，之后再打包的时候，直接用这个分析过的结果就可以了，这是一个最理想的优化方式。

具体操作:

1. 创建一个配置文件，姑且命名为webpack.dll.js

```javascript
  const path = require('path')
  const webpack = require('webpack')

  module.exports = {
    entry: {
      vendors: ['vue', 'vue-router', 'vuex']
    },
    output: {
      filename: '[name].dll.js',
      path: path.resolve(__dirname, '../dll'),
      //  打包生成了webpack.dll.js这个文件，通过一个全局变量暴露出来
      library: '[name]'
    },
    plugins: [
      // 用DllPlugin这个插件, 来分析上面生成的库文件
      // 把库里一些第三方模块的映射关系，放到vendors.manifest.json文件中
      new webpack.DllPlugin({
        name: '[name]', // 要分析文件的名字，要跟库保持一致
        path: path.resolve(__dirname, '../dll/[name].manifest.json')
      })
    ]
  }
```

2. 然后再在package.json里再配置一个命令

```json
  {
    "scrtpts": {
      "build:dll": "webpack --config ./webpack.dll.js"
    }
  }
```

3. 然后我们运行打包命令npm run build:dll把引入的三个模块，打包到了dll文件夹下的vendors.dll.js文件

4. 然后再html中引入vendors.dll.js文件; 需要安装一个插件 npm install add-asset-html-webpack-plugin --save,这个插件的作用就是往html-webpack-plugin上再去增加一些静态的资源

5. 结合全局变量和刚才生成的vendors.manifest.json文件, 对我们源代码进行分析，一旦分析出来，使用的模块内容，是在vendors.dll.js里，那么它就会直接去使用vendors.dll.js里的内容了，就不会去node_modules引入第三方模块了

```javascript
  const addAssetHtmlWebpackPlugin = require('add-asset-html-webpack-plugin')
  const webpack = require('webpack')

  module.exports = {
    plugins: [
      new addAssetHtmlWebpackPlugin({
        filePath: path.resolve(__dirname, '../dll/vendors.dll.js')
      }),
      // 使用DllReferencePlugin这个插件, 当去打包src下的index.js时候，会引入一些第三方的模块
      // 当发现我们在引入一些第三方模块时，会到vendors.manifest.json去找第三方模块的映射关系
      // 如果能找到映射关系，它就知道没必要再打包出来，直接从vendors.dll.js里拿过来用就可以了
      // 它会从定义的全局变量中拿，但是如果发现引入的第三方模块不在映射关系里，才会到node_modules中拿过来打包
      new webpack.DllReferencePlugin({
        manifest: path.resolve(__dirname, '../dll/vendors.manifest.json')
      })
    ]
  }
```

## __6.控制包文件的大小__

> + 一些用不到的包，要通过Tree Shaking去除掉
> + 还可以通过splitchunksplugin这样的插件对代码进行拆分，把一个大的文件拆分成小的文件，进行webpack的打包处理，这样也可以提升webpack的打包速度

## __7.thread-loader,parallel-webpack,happypack 多进程进行打包__

> webpack默认是同构NodeJS来运行的，所以是一个单进程的打包过程，有时候，我们可以借助node里的多进程来帮助我们提升webpack的打包速度

## __8.合理使用 sourceMap__

> 打包的时候，生成的sourceMap越详细，打包的速度就越慢，所以我们要思考，不同环境打包的时候，什么样的sourceMap是最合适的，既要保证我们及时发现代码里的问题。
> production模式下配置cheap-module-source-map; 
> development模式下配置eval-cheap-module-source-map

## __9.开发环境内存编译__

> 我们在开发环境的时候使用的是webpack dev server,它在做打包的时候，不会生成dist目录，他会把编译生成的内容放到内存里，内存的读取，肯定要比硬盘的读取快的多，所以通过这种手段，也可以让我们在开发的过程中，webpack的性能得到很大的提升

## __10.开发环境，无用插件剔除__

> 比如项目调试的时候，我们并不需要对代码进行压缩，我们就应该把mode设置成development而不是production,如果开发环境下就压缩的话是没用意义的，这会降低webpack的打包速度