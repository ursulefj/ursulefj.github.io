---
title: webpack配置与优化
date: 2021-04-09 17:51:35
categories: tech
tags: webpack
---
在以往开发过程中有用到`vue-cli`、`create-react-app`等脚手架构建管理项目，只是在需要的时候查找相关问题。提升道路上还是有必要系统的进行学习和补充。

## 基本的配置

<br>

### 初始化项目

首先创建就目录初始化npm
`npm init`

下载webpack的两个包
`npm i -D webpack webpack-cli`

配置package.json
```
"script":{
    "build":"webpack src/main.js"
}
```

`npm run build`进行打包，在目录下生成dist文件夹

### 生成自定义配置

现在使用的是默认配置，接下来需要创建可自定义的配置文件。在build文件夹下创建`webpack.config.js`

```
// webpack.config.js

const path = require('path');
module.exports = {
    mode:'development', // 开发模式
    entry: path.resolve(__dirname,'../src/main.js'),    // 入口文件
    output: {
        filename: 'output.js',      // 打包后的文件名称
        path: path.resolve(__dirname,'../dist')  // 打包后的目录
    }
}
```

设置打包命令使用配置文件

```
"script":{
    "build":"webpack --config build/webpack.config.js"
}
```

打包之后的dist下的`main.js`就是实际运行的文件

### 配置html模板

js文件打包好之后，还需要在html文件中进行引用。
>通常在生产模式下考虑资源缓存的问题会在文件名中加入hash串，所以引用也不是固定的。因此可能这样配置
```
module.exports = {
    // 省略其他配置
    output: {
      filename: '[name].[hash:8].js',      // 打包后的文件名称
      path: path.resolve(__dirname,'../dist')  // 打包后的目录
    }
}
```

每次的文件名不同，因此需要动态的将js文件引入到html中，使用`html-webpack-plugin`

`npm i -D html-webpack-plugin`

配置文件如下
```
// webpack.config.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin')
module.exports = {
    mode:'development', // 开发模式
    entry: path.resolve(__dirname,'../src/main.js'),    // 入口文件
    output: {
      filename: '[name].[hash:8].js',      // 打包后的文件名称
      path: path.resolve(__dirname,'../dist')  // 打包后的目录
    },
    plugins:[
      new HtmlWebpackPlugin({
        template:path.resolve(__dirname,'../public/index.html')
      })
    ]
}
```
打包后js文件自动引入到html文件中
![js引入效果](/images/202104/20210412171823.png)

### 清理残留文件
每次执行打包命令时之前的打包文件还会继续存在，不符合需要，因此需要在打包前清楚残留。
`npm i -D clean-webpack-plugin`

配置：
```
const {CleanWebpackPlugin} = require('clean-webpack-plugin')
module.exports = {
    // ...省略其他配置
    plugins:[new CleanWebpackPlugin()]
}
```

### 引用CSS

除了js资源外也需要引入css文件，需要使用一些loader来解析css文件
`npm i -D style-loader css-loader`

如果要使用less、scss等预处理语言，需要额外的loader进行解析，这里我们使用less
`npm i -D less less-loader`

配置：
```
// webpack.config.js
module.exports = {
    // ...省略其他配置
    module:{
      rules:[
        {
          test:/\.css$/,
          use:['style-loader','css-loader'] // 从右向左解析原则
        },
        {
          test:/\.less$/,
          use:['style-loader','css-loader','less-loader'] // 从右向左解析原则
        }
      ]
    }
}
```

#### 添加css前缀

`npm i -D postcss-loader autoprefixer`

配置如下：
```
// webpack.config.js
module.exports = {
    module:{
        rules:[
            {
                test:/\.less$/,
                use:['style-loader','css-loader','postcss-loader','less-loader'] // 从右向左解析原则
           }
        ]
    }
} 
```

有两种方式引入`autoprefixer`
1.在项目根目录下创建一个`postcss.config.js`文件，配置如下：
```
module.exports = {
    plugins: [require('autoprefixer')]  // 引用该插件即可了
}
```

2.直接在`webpack.config.js`里配置
```
// webpack.config.js
module.exports = {
    module:{
        rules:[{
            test:/\.less$/,
            use:['style-loader','css-loader',{
                loader:'postcss-loader',
                options:{
                    plugins:[require('autoprefixer')]
                }
            },'less-loader'] // 从右向左解析原则
        }]
    }
}
```

#### 拆分css
使用`mini-css-extract-plugin`插件把css从js文件中提取出来

`npm i -D mini-css-extract-plugin`

配置如下：
```
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
module.exports = {
  module: {
    rules: [
      {
        test: /\.less$/,
        use: [
           MiniCssExtractPlugin.loader,
          'css-loader',
          'less-loader'
        ],
      }
    ]
  },
  plugins: [
    new MiniCssExtractPlugin({
        filename: "[name].[hash].css",
        chunkFilename: "[id].css",
    })
  ]
}
```

### 引入图片、字体、视频等文件
`file-loader`的主要功能是处理文件名，例如文件名中添加hash值，解析文件路径，并将文件移动到输出的目录中
`url-loader` 一般与file-loader搭配使用，功能与 `file-loader` 类似，可以设置一定的文件大小内返回 base64 编码进行优化，否则使用 `file-loader` 将文件移动到输出的目录中

```
// webpack.config.js
module.exports = {
  module: {
    rules: [
      // ...
      {
        test: /\.(jpe?g|png|gif)$/i, //图片文件
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 10240,
              fallback: {
                loader: 'file-loader',
                options: {
                    name: 'img/[name].[hash:8].[ext]'
                }
              }
            }
          }
        ]
      },
      {
        test: /\.(mp4|webm|ogg|mp3|wav|flac|aac)(\?.*)?$/, //媒体文件
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 10240,
              fallback: {
                loader: 'file-loader',
                options: {
                  name: 'media/[name].[hash:8].[ext]'
                }
              }
            }
          }
        ]
      },
      {
        test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/i, // 字体
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 10240,
              fallback: {
                loader: 'file-loader',
                options: {
                  name: 'fonts/[name].[hash:8].[ext]'
                }
              }
            }
          }
        ]
      },
    ]
  }
}
```

### babel转义

`npm i -D babel-loader @babel/preset-env @babel/core`
上面的`babel-loader`默认只会将ES6/ES7/ES8的语法进行转义，一些api无法进行转换，例如Promise、Maps、Set
因此还需要增加polyfill转换
`npm i @babel/polyfill`
配置如下
```
// webpack.config.js
const path = require('path')
module.exports = {
    entry: ["@babel/polyfill",path.resolve(__dirname,'../src/index.js')],    // 入口文件
}
```
