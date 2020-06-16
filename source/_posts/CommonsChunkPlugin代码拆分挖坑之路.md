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
加上了上面的代码后，打包确实把element-ui、echarts、vue给抽离出来了