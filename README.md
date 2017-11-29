### 概念(demo02)
webpack 是一个 JavaScript 应用程序的**模块打包器(module bundler)**。它会将一个应用所需的模块，打包成一个或多个 bundle。

四个核心概念
+ 入口(entry)
+ 输出(output)
+ loader
+ 插件(plugin)

#### 入口(entry)
**入口起点**提示 webpack 应该使用哪个模块，来作为构建其内部**依赖图**的开始。

可以通过在 webpack 中配置 ```entry```属性，来指定一个入口起点(或多个入口起点)。

```
module.exports = {
    entry: './src/index.js'
};
```

#### 出口(output)
output 属性告诉 webpack 在哪里输出它所创建的 bundles，以及如何命名这些文件。

```
const path = require('path');

 module.exports = {
    entry: './path/to/my/entry/file.js',
    output: {
        // path.resolve 方法，用于把相对路径，转为绝对路径
        // __dirname 总是返回被执行的 js 所在文件夹的绝对路径
        path: path.resolve(__dirname, 'dist'),
        filename: 'my-first-webpack.bundle.js'
    }
 };
```

```output.path```：想要生成(emit)到哪里

```output.filename```:webpack bundle 的名称

#### loader

loader 让 webpack 能够去处理那些非 JavaScript 文件(webpack 自身只理解 JavaScript)。

本质上，webpack loader 将所有类型的文件，转换为应用程序的依赖图可以直接引用模块。

在更高层面上，在 webpack 的配置中 loader 有两个目标：
1. 识别出应该被对应的 loader 进行转换的文件。(使用```test```属性)；
2. 转换这些文件，从而使其能够添加到依赖图(并且最终添加到 bundle 中)(```use```属性)；

```
const path = require('path');

const config = {
    entry: './path/to/my/entry/file.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'my-first-webpack.bundle.js'
      },
    module: {
        rule: [
            { test: /\.txt$/, use: 'raw-loader' }
        ]
    }
};

module.exports = config;
```

以上配置，对一个单独的 module 对象定义了 ```rules```属性，里面必须包含两个属性：```test```和```use```。这样会告诉 webpack 编译器：

> 嘿，webpack 编译器，当你碰到【在 require() / import 语句中被解析为 '.txt' 的路径】时，在你对它打包前，先使用```raw-loader```转换一下。

#### 插件(plugins)
loader 被用于转换某些类型的模块，而插件则可以用于执行范围更广的任务。插件的范围包括，从打包优化到压缩，一直到重新定义环境中的变量.....

想要使用某个插件，你只需要 require() 它，然后把它添加到 ```plugins```数组中。多数插件可以通过选项(option)自定义。

你也可以在一个配置文件中，因为不同目的而多次使用同一个插件，这时需要通过使用 ```new``` 操作符来创建一个它的实例。

```
const HtmlWebpackPlugin = require('html-webpack-plugin'); // 通过 npm 安装
const webpack = require('webpack'); // 用于访问内置插件
const path = require('path');

const config = {
  entry: './path/to/my/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-first-webpack.bundle.js'
  },
  module: {
    rules: [
      { test: /\.txt$/, use: 'raw-loader' }
    ]
  },
  plugins: [
    new webpack.optimize.UglifyJsPlugin(),
    new HtmlWebpackPlugin({template: './src/index.html'})
  ]
};

module.exports = config;
```

### 入口起点(demo03)

#### 单个入口(简写)语法
```
const config = {
    entry: './path/to/my/entry/file.js'
};
```

entry 属性的单个入口语法，是下面的简写：
```
const config = {
    entry: {
        main: './path/to/my/entry/file.js'
    }
};
```

#### 对象语法
```
const config = {
    entry: {
        app: '.src/app.js',
        vendors: './src/vendors.js',
    }
};
```
对象语法比较繁琐。然而，这是应用程序中定义入口的最可扩展形式。

#### 常见场景
##### 分离 应用程序(app) 和 第三方库(vendor)入口
```
const config = {
    entry: {
        app: '.src/app.js',
        vendors: './src/vendors.js',
    }
};
```
**这是什么？** 从表面上看，这告诉我们 webpack 从 ```app.js``` 和 ```vendors.js``` 开始创建依赖图。这些依赖图是彼此完全分离、相互独立的(每个bundle中都有一个 webpack 引导)。这种方式比较常见于，只有一个入口起点(不包括 vendor)的单页应用程序中。

**为什么？** 此设置允许你使用 ```CommonsChunkPlugin```从【应用程序bundle】中提取 vendor 引用到 vendor bundle ， 并把引用 vendor 的部分替换为 ```__webpack_require__()```调用。如果应用程序 bundle 中没有 vendor 代码，那么你可以在 webpack 中实现被称为 长效缓存 的通用模式。

##### 多页面应用程序
```
const config = {
  entry: {
    pageOne: './src/pageOne/index.js',
    pageTwo: './src/pageTwo/index.js',
    pageThree: './src/pageThree/index.js'
  }
};
```
**这是什么？** 我们告诉 webpack 需要3个独立分离的依赖图。

**为什么？** 在多页应用中，每当页面跳转时，服务器将为你获取一个新的 HTML 文档。 页面重新加载新文档，并且资源被重新下载。然而，这给了我们特殊的机会去做很多事：

+ 使用 ```CommonsChunkPlugin```为每个页面间的应用程序共享代码创建 bundle。 由于入口起点增多，多页面应用能够复用入口起点之间的大量代码/模块，从而可以极大地从这些技术中受益。