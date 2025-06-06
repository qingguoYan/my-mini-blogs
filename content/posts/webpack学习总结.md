+++
author = "yqg"
title = "Webpack学习总结"
date = "2025-06-06"
description = "webpack打包流程、打包体积优化、打包速度提升、chunk、代码分割、bundle"
tags = [
    "Webpack", "打包工具"
]
+++

## webpack 打包流程

Webpack 的打包流程是一个复杂的过程，主要包含以下几个步骤：

1. **初始化参数**：

   - 从配置文件和 Shell 语句中读取与合并参数
   - 得到最终的配置对象

2. **开始编译**：

   - 初始化 Compiler 对象
   - 加载所有配置的插件
   - 执行对象的 run 方法开始执行编译

3. **确定入口**：

   - 根据配置中的 entry 找出所有的入口文件
   - 调用 `compilation.addEntry` 将入口文件转换为 dependence 对象

4. **编译模块**：

   - 从入口文件出发，调用所有配置的 Loader 对模块进行转换
   - 再找出该模块依赖的模块，递归本步骤直到所有入口依赖的文件都经过了本步骤的处理

5. **完成模块编译**：

   - 经过第 4 步使用 Loader 翻译完所有模块后，得到了每个模块被翻译后的最终内容以及它们之间的依赖关系

6. **输出资源**：

   - 根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 Chunk
   - 再把每个 Chunk 转换成一个单独的文件加入到输出列表

7. **输出完成**：
   - 确定好输出内容后，根据配置确定输出的路径和文件名
   - 把文件内容写入到文件系统

## webpack 打包体积优化

在实际项目中，打包体积的优化是非常重要的，以下是一些常用的优化策略：

1. **代码压缩**

   - 使用 `terser-webpack-plugin` 压缩 JavaScript 代码
   - 使用 `css-minimizer-webpack-plugin` 压缩 CSS 代码
   - 使用 `html-webpack-plugin` 压缩 HTML 代码

2. **Tree Shaking**

   - 开启 production 模式自动启用
   - 在 package.json 中设置 "sideEffects": false
   - 使用 ES6 的模块语法（import/export）

3. **图片优化**

   - 使用 `image-webpack-loader` 压缩图片
   - 小图片转 base64（url-loader）
   - 使用 webp 格式图片

4. **按需加载**

   - 路由懒加载
   - 组件按需加载
   - 第三方库按需引入

5. **压缩第三方库**

   - 使用 CDN 加载第三方库
   - 使用 `externals` 配置排除不需要打包的库
   - 使用 DllPlugin 单独打包第三方库

6. **Gzip 压缩**
   - 使用 `compression-webpack-plugin` 开启 Gzip 压缩
   - 服务器端配置 Gzip

## webpack 打包速度提升

提升 Webpack 打包速度的策略主要有：

1. **优化 loader 配置**

   - 使用 `include` 或 `exclude` 缩小搜索范围
   - 使用 `cache-loader` 缓存 loader 的执行结果
   - 使用 `thread-loader` 开启多线程打包

2. **优化 resolve 配置**

   ```javascript
   resolve: {
     extensions: ['.js', '.jsx'],
     alias: {
       '@': path.resolve(__dirname, 'src')
     }
   }
   ```

3. **使用 DllPlugin**

   - 将第三方库单独打包
   - 避免重复编译第三方库

4. **开启热更新**

   - 使用 `webpack-dev-server` 的 HMR 功能
   - 只更新变更的模块，加快开发效率

5. **利用缓存**

   - 使用 `cache` 配置项
   - 使用 `hard-source-webpack-plugin`

6. **合理使用 sourceMap**
   - 开发环境使用 `eval-cheap-module-source-map`
   - 生产环境使用 `none` 或 `source-map`

## webpack 配置代理解决跨域问题

在开发环境中，我们经常需要解决跨域问题，Webpack 提供了几种解决方案：

1. **devServer proxy 配置**

   ```javascript
   module.exports = {
     devServer: {
       proxy: {
         "/api": {
           target: "http://localhost:3000",
           pathRewrite: { "^/api": "" },
           changeOrigin: true,
         },
       },
     },
   };
   ```

2. **CORS 配置**

   - 服务器端配置 Access-Control-Allow-Origin
   - 配置其他 CORS 相关头部

3. **使用 nginx 反向代理**

## 什么是 chunk

Chunk 是 Webpack 打包过程中的一个重要概念：

1. **定义**：

   - Chunk 是 Webpack 打包过程中，一个模块的集合
   - 用于生成最终输出的文件 bundle

2. **Chunk 的类型**：

   - Initial Chunk：入口起点的模块集合
   - Async Chunk：异步加载的模块集合
   - Runtime Chunk：运行时代码的集合

3. **Chunk 的生成**：
   - entry 配置会生成 chunk
   - 动态导入会生成新的 chunk
   - SplitChunksPlugin 分离的代码会生成新的 chunk

## 代码分割

代码分割是优化前端应用的重要手段：

1. **入口起点分割**

   ```javascript
   entry: {
     app: './src/app.js',
     vendor: './src/vendor.js'
   }
   ```

2. **动态导入**

   ```javascript
   import("./module").then((module) => {
     // 使用module
   });
   ```

3. **SplitChunksPlugin 配置**
   ```javascript
   optimization: {
     splitChunks: {
       chunks: 'all',
       minSize: 20000,
       minRemainingSize: 0,
       minChunks: 1,
       maxAsyncRequests: 30,
       maxInitialRequests: 30,
       enforceSizeThreshold: 50000,
       cacheGroups: {
         defaultVendors: {
           test: /[\\/]node_modules[\\/]/,
           priority: -10,
           reuseExistingChunk: true,
         },
         default: {
           minChunks: 2,
           priority: -20,
           reuseExistingChunk: true,
         },
       },
     },
   }
   ```

## 什么是 bundle

Bundle 是 Webpack 打包的最终产物：

1. **定义**：

   - Bundle 是由 Webpack 打包出的最终文件
   - 包含了经过加载和编译的最终源文件版本

2. **Bundle 的特点**：

   - 是一个自执行函数
   - 包含模块加载器（module loader）
   - 包含模块定义
   - 包含运行时代码

3. **Bundle 的类型**：

   - 主 bundle：包含入口点的运行时和模块
   - 异步 bundle：通过代码分割生成的 bundle
   - vendor bundle：第三方库的 bundle

4. **Bundle 优化**：
   - 压缩和混淆
   - Tree Shaking
   - Scope Hoisting
   - 代码分割
