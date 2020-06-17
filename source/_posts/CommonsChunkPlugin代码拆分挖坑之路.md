---
title: CommonsChunkPlugin代码拆分挖坑之路
date: 2020-06-16 18:20:29
tags:
---

# CommonsChunkPlugin代码拆分挖坑之路

## 目的

vue单文件打包，vendor体积过大，影响页面性能。将vue相关的（vue、vuex、vue-router），element-ui、echarts（echarts、zrender），以单独js文件抽离出来，从而减少vendor体积。同时，抽离不太可能更改的模块，对于项目的版本迭代也有一定的好处（避免不必要的文件hash变化，导致重新请求数据）。

## 开始挖坑

vue脚手架默认commonsChunkPlugin配置：

``````js
// ./build/webpack.prod.conf.js
const webpackConfig = merge(baseWebpackConfig, {
  ...
  plugins: [
    ...
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      minChunks (module) {
        // any required modules inside node_modules are extracted to vendor
        return (
          module.resource &&
          /\.js$/.test(module.resource) &&
          module.resource.indexOf(
            path.join(__dirname, '../node_modules')
          ) === 0
        )
      }
    }),
    new webpack.optimize.CommonsChunkPlugin({
      name: 'manifest',
      minChunks: Infinity
    }),
    new webpack.optimize.CommonsChunkPlugin({
      name: 'app',
      async: 'vendor-async',
      children: true,
      minChunks: 3
    }),
    ...
  ]
});
``````

这里的默认配置是从单一入口：app（entry chunk），将所有的公共插件抽离到vendor上，与webpack相关的运行环境模块抽离到manifest上。

初步的改造是：

``````js
// 从vendor中抽离需要的模块数据
new webpack.optimize.CommonsChunkPlugin({
  name: 'echarts',
  chunks: ['vendor'],
  minChunks (module) {
    // any required modules inside node_modules are extracted to vendor
    return (
      module.resource &&
      /echarts|zrender/.test(module.resource)
    )
  }
}),
new webpack.optimize.CommonsChunkPlugin({
  name: 'element-ui',
  chunks: ['vendor'],
  minChunks (module) {
    // any required modules inside node_modules are extracted to vendor
    return (
      module.resource &&
      /element-ui/.test(module.resource)
    )
  }
}),
new webpack.optimize.CommonsChunkPlugin({
  name: 'vue',
  chunks: ['vendor'],
  minChunks (module) {
    // any required modules inside node_modules are extracted to vendor
    return (
      module.resource &&
      /vue/.test(module.resource)
    )
  }
}),
``````
加上了上面的代码后，打包确实把element-ui、echarts、vue给抽离出来了:

![打包后结果](./1592357780.png)

以为有所成就了，立即发布到测试环境，万万没想到呀，出错了！

![webpackJsonp出错了](./1592358102.png)

![出错具体位置](./1592359913.png)

由`webpackJsonp` is not defined可想而知，是webpack打包后的运行环境出现了问题，[官网介绍](https://webpack.js.org/configuration/output/#outputjsonpfunction)

根据CommonsChunkPlugin的配置可知，webpack运行相关的代码都拆分到了[manifest](https://webpack.js.org/plugins/commons-chunk-plugin/#combining-implicit-common-vendor-chunks-and-manifest-file)中，可以猜测：

1. manifest打包出错了
2. not defined 在平常开发工作中十分常见，运行时webpackJsonp未定义

猜想1中，由于只是单一从vendor里拆出正则对应的chunk，几乎不影响webpack运行环境的抽离，很明显出错的概率很小。

大概率是猜想2的问题，查看了一下打包后的`index.html`文件

![index.html](./1592359518.png)

对比一下，我们其他环境（正常环境）的`index.html`

![正常的index.html](./1592359683.png)

很明显，对于manifest.js的script调用顺序发生了改变，从而导致了webpackJsonp为not defined，所以我们需要改变chunks写入`index.html`的顺序

chunks的写入就是[HtmlWebpackPlugin](https://www.webpackjs.com/plugins/html-webpack-plugin/)的工作了

vue脚手架出来，默认配置：
``````js
// webpack.prod.conf.js
new HtmlWebpackPlugin({
  ...
  // necessary to consistently work with multiple chunks via CommonsChunkPlugin
  chunksSortMode: 'dependency'
}),
``````
chunksSortMode：允许在插入html之前控制chunks的排序

``````
Allows to control how chunks should be sorted before they are included to the HTML. Allowed values are 'dependency' | 'none' | 'auto' | 'manual' | {Function}
``````

'dependency'：按照不同文件的依赖关系来排序，貌似在4.0版本以上看不到这个了
'auto'：默认值
'none'
'manual'：自定义排序

既然一开始 chunksSortMode: 'dependency'报错，说明上述拆分导致了文件间的依赖关系发生了错乱，这点没搞太懂，望各位大佬赐教一下...

搞不懂依赖关系为啥错乱，那么就手动添加chunks的写入顺序：
顺序要注意，app.js依赖于其他插件、库文件，所以必须在`最后`才加载，否则会产生大量的not defined错误

``````js
// webpack.prod.conf.js
new HtmlWebpackPlugin({
  ...
  // necessary to consistently work with multiple chunks via CommonsChunkPlugin
  chunksSortMode: 'manual',
  chunks: ['manifest', 'vendor', 'vue', 'element-ui', 'echarts', 'app']
}),
``````

打包后的文件妥妥的按照既定的顺序：

![修改顺序后打包](./1592378004.png)

然后放到测试环境，就可以正常运行了....

### 疑惑点

有点怪异的是：前一天按照上文配置了，放到测试环境时，却出现了如下的错误（`这里的图是个类似的错误内容截图`，都是manifest.js里出现的'call' of underfined）

![奇怪的错误](./1592378554.png)

所以以为manifest提取的chunks不完整导致的，加了下方的代码

``````js
new webpack.optimize.CommonsChunkPlugin({
  name: 'manifest',
  chunks: ['vendor', 'vue', 'element-ui', 'echarts', 'app'],
  minChunks: Infinity
}),
``````

神奇的没问题了QAQ

然后第二天把这代码去掉重新打包，却没有了上面的问题，各位大佬有啥好的见解呢？

## 简单对比一下splitChunks

vue-cli3对splitChunks的默认配置：

``````js
// code splitting
if (process.env.NODE_ENV !== 'test') {
  webpackConfig
    .optimization.splitChunks({
      cacheGroups: {
        vendors: {
          name: `chunk-vendors`,
          test: /[\\/]node_modules[\\/]/,
          priority: -10,
          chunks: 'initial'
        },
        common: {
          name: `chunk-common`,
          minChunks: 2,
          priority: -20,
          chunks: 'initial',
          reuseExistingChunk: true
        }
      }
    })
}
``````

而我们在vue.config.js上实现对element-ui和vue的拆分十分的简单，主要的就是对priority优先级的处理

``````js
module.exports = {
  ...
  chainWebpack: config => {
    // code splitting
    config.optimization.splitChunks({
      cacheGroups: {
        vendors: {
          name: `chunk-vendors`,
          test: /[\\/]node_modules[\\/]/,
          priority: -10,
          chunks: 'initial'
        },
        common: {
          name: `chunk-common`,
          minChunks: 2,
          priority: -20,
          chunks: 'initial',
          reuseExistingChunk: true
        },
        // 将element-ui单独拆分出来
        element: {
          name: `element-ui`,
          test: /element-ui/,
          priority: 10,
          chunks: 'all'
        },
        // 将vue相关的单独拆分出来
        vue: {
          name: `vue`,
          test: /vue/,
          priority: 20,
          chunks: 'all'
        }
      }
    });
  }
};
``````

相比于CommonsChunkPlugin可谓十分的简单。

以上只是一点小小的见解，如有错误，不吝赐教！！！！！
