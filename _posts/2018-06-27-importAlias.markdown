---
layout: post
title: "webpack中css import使用alias相对路径"
subtitle: "none"
author: "marillo"
header-img: "img/about-bg.jpg"
catalog: true
tages:
     - web pack
---

#### 起因

在用webpack处理打包时，可将某一目录配置一个别名，代码中就能使用与别名的相对路径引用资源。

在Vue项目中，我们通常使用`vue-webpack`脚手架生成工程模板，然后配置`@`为项目根目录下放资源和源码的`/src`目录的别名，写在webpack.conf.js里

```javascript
  resolve: {
    extensions: ['.js', '.vue', '.json'],
    alias: {
      'vue$': 'vue/dist/vue.esm.js',
      '@': resolve('src'),
      'api': resolve(apiPath)
    }
  },
```

这样我们就可以在 js 文件中用形如`import tool from '@/utils/xxx'`的方式引用`/src/utils/xxx.js`文件，并且webpack能正确识别打包。

但是在 css 文件，如less，sass，stylus中，使用`@import '@/style/theme'`的语法引用相对`@`的目录却会报错，`找不到'@'目录`，说明webpack没有正确识别资源相对路径。

#### 分析

原因是 css 文件会被`css-loader`处理，这里css`@import`后的字符串会被`css-loader`视为绝对路径，因为没有并没有加`css-loader`的 alias，所以会报找不到`@`目录。

#### 解决

在 webpack 中 css import 使用 alias 相对路径的解决办法有两种；

一是直接为 `css-loader` 添加 [ailas](https://github.com/webpack-contrib/css-loader#alias) 的路径，但是在 `vue-webpack` 给的模板中，单独针对这个插件添加配置就显得麻烦冗余了；

二是在引用路径的字符串最前面添加上 `~` 符号，如 `@import "~@/style/theme"`；Webpack 会将以 `~` 符号作为前缀的路径视作依赖模块而去解析，这样 `@` 的 alias 配置就能生效了。

#### 总结

`~` 视为模块解析是 webpack 做的事，不是 css-loader 做的事。

各类非 js 直接引用（`import` `require`）静态资源，依赖相对路径加载问题，都可以用 `~` 语法完美解决；

例如 css module 中： `@import "~@/style/theme"`

css 属性中： `background: url("~@/assets/xxx.jpg")`

html 标签中： `<img src="~@/assets/xxx.jpg" alt="alias">`