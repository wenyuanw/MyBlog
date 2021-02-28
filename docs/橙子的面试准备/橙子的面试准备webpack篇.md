http://www.xbhub.com/wiki/webpack/

# webpack核心概念

- **Entry**：入口，Webpack 执行构建的第一步将从 Entry 开始，可抽象成输入。
- **Module**：模块，在 Webpack 里一切皆模块，一个模块对应着一个文件。Webpack 会从配置的 Entry 开始递归找出所有依赖的模块。
- **Chunk**：代码块，一个 Chunk 由多个模块组合而成，用于代码合并与分割。
- **Loader**：模块转换器，用于把模块原内容按照需求转换成新内容。
- **Plugin**：扩展插件，在 Webpack 构建流程中的特定时机注入扩展逻辑来改变构建结果或做你想要的事情。
- **Output**：输出结果，在 Webpack 经过一系列处理并得出最终想要的代码后输出结果。

# 打包（loader）

webpack默认只识别js结尾的文件，当遇到其他格式的文件后，webpack并不知道如何去处理。此时，我们可以定义一种规则，告诉webpack当他遇到某种格式的文件后，去求助于相应的loader。

```javascript
module: {
        rules: [
            {
                test: /\.(png|jpe?g|ico|pdf)$/,
                loader: 'url-loader',
                options: {
                    limit: 40000,
                    name: 'i/[name].[hash].[ext]',
                },
            },
            {
                test: /\.svg(\?v=\d+\.\d+\.\d+)?$/,
                loader: 'url-loader',
                options: {
                    limit: 10000,
                    minetype: 'image/svg+xml',
                    name: 'i/[name].[hash].[ext]',
                },
            },
            {
                test: /\.(gif|eot|ttf|otf|woff2?)$/,
                loader: 'file-loader',
                options: {
                    name: 'i/[name].[hash:8].[ext]',
                },
            },
            {
                test: /(\.jsx?$)|(\.tsx?$)/,
                exclude: /node_modules/,
                loaders: [
                    {
                        loader: 'babel-loader',
                    },
                    {
                        loader: 'ts-loader',
                        options: {
                            // disable type checker - we will use it in fork plugin
                            transpileOnly: true,
                        },
                    },
                ],
            },
            {
                test: /\.(mp4|webm|ogg|mp3|wav|flac|aac)(\?.*)?$/,
                loader: 'url-loader',
                options: {
                    limit: 1024,
                    name: 'i/[name].[hash:8].[ext]',
                },
            },
        ],
    },
```



# 插件（plugins）

## HtmlWebpackPlugin

可以生成创建html入口文件，比如单页面可以生成一个html文件入口，配置N个html-webpack-plugin可以生成N个页面入口。

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin')
  plugins: [
    new HtmlWebpackPlugin({ // 打包输出HTML
      title: 'Hello World app',
      minify: { // 压缩HTML文件
        removeComments: true, // 移除HTML中的注释
        collapseWhitespace: true, // 删除空白符与换行符
        minifyCSS: true// 压缩内联css
      },
      filename: 'index.html',
      template: 'index.html'
    }),
  ]
```

## ForkTsCheckerWebpackPlugin

用来实时检测ts类型

## webpack.ContextReplacementPlugin

webpack 打包momentjs时会把所有语言包都打包，这样会使打包文件很大。此插件可以帮助我们只打包需要的语言包，大大减小打包文件大小。

new webpack.ContextReplacementPlugin(/moment[/\\]locale$/, /zh-cn|zh-hk|en/)

限定查找 moment/locale 上下文里符合 /zh-cn|zh-hk|en/ 表达式的文件，因此也只会打包这几种本地化内容。

# 构建过程

`webpack` 的运行流程是一个串行的过程，从启动到结束会依次执行以下流程：

1. 初始化参数：从配置文件和 `Shell` 语句中读取与合并参数，得出最终的参数；
2. 开始编译：用上一步得到的参数初始化 `Compiler` 对象，加载所有配置的插件，执行对象的 `run` 方法开始执行编译；
3. 确定入口：根据配置中的 entry 找出所有的入口文件
4. 编译模块：从入口文件出发，调用所有配置的 Loader 对模块进行翻译，再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理；
5. 完成模块编译：在经过第4步使用 Loader 翻译完所有模块后，得到了每个模块被翻译后的最终内容以及它们之间的依赖关系；
6. 输出资源：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 `Chunk`，再把每个 `Chunk` 转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会；
7. 输出完成：在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统。

在以上过程中，`webpack` 会在特定的时间点广播出特定的事件，插件在监听到感兴趣的事件后会执行特定的逻辑，并且插件可以调用 `webpack` 提供的 API 改变 `webpack` 的运行结果。

## 核心对象

- **`Compile`** 对象：负责文件监听和启动编译。`Compiler` 实例中包含了完整的 `webpack` 配置，全局只有一个 `Compiler` 实例。
- **`compilation`** 对象：当 `webpack` 以开发模式运行时，每当检测到文件变化，一次新的 `Compilation` 将被创建。一个 `Compilation` 对象包含了当前的模块资源、编译生成资源、变化的文件等。`Compilation` 对象也提供了很多事件回调供插件做扩展。
- 这两个对象都继承自 [Tapable](https://github.com/webpack/tapable)。

### compilation简介

`compilation` 实际上就是调用相应的 `loader` 处理文件生成 `chunks`并对这些 `chunks` 做优化的过程。几个关键的事件（Compilation对象this.hooks中）：

1. `buildModule` 使用对应的 `Loader` 去转换一个模块;
2. `normalModuleLoader` 在用 `Loader` 对一个模块转换完后，使用 `acorn` 解析转换后的内容，输出对应的抽象语法树（AST），以方便 `webpack` 后面对代码的分析。
3. `seal` 所有模块及其依赖的模块都通过 `Loader` 转换完成后，根据依赖关系开始生成 `Chunk`。

![preview](https://segmentfault.com/img/remote/1460000016984437?w=1150&h=1610/view)

# webpack打包体积过大优化

**去除不必要的插件**

刚开始用 webpack 的时候，开发环境和生产环境用的是同一个 webpack 配置文件，导致生产环境打包的 JS 文件包含了一大堆没必要的插件，比如 `HotModuleReplacementPlugin`, `NoErrorsPlugin`... 这时候不管用什么优化方式，都没多大效果。所以，如果你打包后的文件非常大的话，先检查下是不是包含了这些插件。

**提取第三方库**

像 react 这个库的核心代码就有 627 KB，这样和我们的源代码放在一起打包，体积肯定会很大。所以可以在 webpack 中设置。

```javascript
{
  entry: {
   bundle: 'app'
    vendor: ['react']
  }

  plugins: {
    new webpack.optimize.CommonsChunkPlugin('vendor',  'vendor.js')
  }
}
```

**代码压缩**

```js
{
  plugins: [
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      }
    })
  ]
}
```

**代码分割**

```javascript
output: {
    path: xxx,
    publicPath: yyy,
    filename: 'bundle.js'
}
```

**设置缓存**

```js
output: {
    path: xxx,
    publicPath: yyy,
    filename: '[name]-[chunkhash:6].js'
}
```

# webpack性能优化

- ### 构建过程提速策略

  #### 不要让 loader 做太多事情——以 babel-loader 为例

  babel-loader 无疑是强大的，但它也是慢的。

  最常见的优化方式是，用 include 或 exclude 来帮我们避免不必要的转译，比如 webpack 官方在介绍 babel-loader 时给出的示例：

  ```
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /(node_modules|bower_components)/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env']
          }
        }
      }
    ]
  }
  ```

  这段代码帮我们规避了对庞大的 node_modules 文件夹或者 bower_components 文件夹的处理。但通过限定文件范围带来的性能提升是有限的。除此之外，如果我们选择开启缓存将转译结果缓存至文件系统，则至少可以将 babel-loader 的工作效率提升两倍。要做到这点，我们只需要为 loader 增加相应的参数设定：

  ```
  loader: 'babel-loader?cacheDirectory=true'
  ```

  以上都是在讨论针对 loader 的配置，但我们的优化范围不止是 loader 们。

  举个🌰，尽管我们可以在 loader 配置时通过写入 exclude 去避免 babel-loader 对不必要的文件的处理，但是考虑到这个规则仅作用于这个 loader，像一些类似 UglifyJsPlugin 的 webpack 插件在工作时依然会被这些庞大的第三方库拖累，webpack 构建速度依然会因此大打折扣。所以针对这些庞大的第三方库，我们还需要做一些额外的努力。

  #### 不要放过第三方库

  第三方库以 node_modules 为代表，它们庞大得可怕，却又不可或缺。

  处理第三方库的姿势有很多，其中，Externals 不够聪明，一些情况下会引发重复打包的问题；而 CommonsChunkPlugin 每次构建时都会重新构建一次 vendor；出于对效率的考虑，我们这里为大家推荐 DllPlugin。

  DllPlugin 是基于 Windows 动态链接库（dll）的思想被创作出来的。这个插件会把第三方库单独打包到一个文件中，这个文件就是一个单纯的依赖库。**这个依赖库不会跟着你的业务代码一起被重新打包，只有当依赖自身发生版本变化时才会重新打包**。

  用 DllPlugin 处理文件，要分两步走：

  - 基于 dll 专属的配置文件，打包 dll 库
  - 基于 webpack.config.js 文件，打包业务代码

  以一个基于 React 的简单项目为例，我们的 dll 的配置文件可以编写如下：

  ```
  const path = require('path')
  const webpack = require('webpack')
  
  module.exports = {
      entry: {
        // 依赖的库数组
        vendor: [
          'prop-types',
          'babel-polyfill',
          'react',
          'react-dom',
          'react-router-dom',
        ]
      },
      output: {
        path: path.join(__dirname, 'dist'),
        filename: '[name].js',
        library: '[name]_[hash]',
      },
      plugins: [
        new webpack.DllPlugin({
          // DllPlugin的name属性需要和libary保持一致
          name: '[name]_[hash]',
          path: path.join(__dirname, 'dist', '[name]-manifest.json'),
          // context需要和webpack.config.js保持一致
          context: __dirname,
        }),
      ],
  }
  ```

  编写完成之后，运行这个配置文件，我们的 dist 文件夹里会出现这样两个文件：

  ```
  vendor-manifest.json
  vendor.js
  ```

  vendor.js 不必解释，是我们第三方库打包的结果。这个多出来的 vendor-manifest.json，则用于描述每个第三方库对应的具体路径，我这里截取一部分给大家看下：

  ```
  {
    "name": "vendor_397f9e25e49947b8675d",
    "content": {
      "./node_modules/core-js/modules/_export.js": {
        "id": 0,
          "buildMeta": {
          "providedExports": true
        }
      },
      "./node_modules/prop-types/index.js": {
        "id": 1,
          "buildMeta": {
          "providedExports": true
        }
      },
      ...
    }
  }  
  ```

  随后，我们只需在 webpack.config.js 里针对 dll 稍作配置：

  ```
  const path = require('path');
  const webpack = require('webpack')
  module.exports = {
    mode: 'production',
    // 编译入口
    entry: {
      main: './src/index.js'
    },
    // 目标文件
    output: {
      path: path.join(__dirname, 'dist/'),
      filename: '[name].js'
    },
    // dll相关配置
    plugins: [
      new webpack.DllReferencePlugin({
        context: __dirname,
        // manifest就是我们第一步中打包出来的json文件
        manifest: require('./dist/vendor-manifest.json'),
      })
    ]
  }
  ```

  一次基于 dll 的 webpack 构建过程优化，便大功告成了！

  #### Happypack——将 loader 由单进程转为多进程

  大家知道，webpack 是单线程的，就算此刻存在多个任务，你也只能排队一个接一个地等待处理。这是 webpack 的缺点，好在我们的 CPU 是多核的，Happypack 会充分释放 CPU 在多核并发方面的优势，帮我们把任务分解给多个子进程去并发执行，大大提升打包效率。

  HappyPack 的使用方法也非常简单，只需要我们把对 loader 的配置转移到 HappyPack 中去就好，我们可以手动告诉 HappyPack 我们需要多少个并发的进程：

  ```
  const HappyPack = require('happypack')
  // 手动创建进程池
  const happyThreadPool =  HappyPack.ThreadPool({ size: os.cpus().length })
  
  module.exports = {
    module: {
      rules: [
        ...
        {
          test: /\.js$/,
          // 问号后面的查询参数指定了处理这类文件的HappyPack实例的名字
          loader: 'happypack/loader?id=happyBabel',
          ...
        },
      ],
    },
    plugins: [
      ...
      new HappyPack({
        // 这个HappyPack的“名字”就叫做happyBabel，和楼上的查询参数遥相呼应
        id: 'happyBabel',
        // 指定进程池
        threadPool: happyThreadPool,
        loaders: ['babel-loader?cacheDirectory']
      })
    ],
  }
  ```

  ### 构建结果体积压缩

  #### 文件结构可视化，找出导致体积过大的原因

  这里为大家介绍一个非常好用的包组成可视化工具——[webpack-bundle-analyzer](https://www.npmjs.com/package/webpack-bundle-analyzer)，配置方法和普通的 plugin 无异，它会以矩形树图的形式将包内各个模块的大小和依赖关系呈现出来，格局如官方所提供这张图所示：

  

  ![img](https://user-gold-cdn.xitu.io/2018/9/14/165d838010b20a4c?imageslim)

  

  在使用时，我们只需要将其以插件的形式引入：

  ```
  const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;
   
  module.exports = {
    plugins: [
      new BundleAnalyzerPlugin()
    ]
  }
  ```

  #### 拆分资源

  这点仍然围绕 DllPlugin 展开，可参考上文。

  #### 删除冗余代码

  一个比较典型的应用，就是 `Tree-Shaking`。

  从 webpack2 开始，webpack 原生支持了 ES6 的模块系统，并基于此推出了 Tree-Shaking。webpack 官方是这样介绍它的：

  > Tree shaking is a term commonly used in the JavaScript context for dead-code elimination, or more precisely, live-code import. It relies on ES2015 module import/export for the static structure of its module system.

  意思是基于 import/export 语法，Tree-Shaking 可以在编译的过程中获悉哪些模块并没有真正被使用，这些没用的代码，在最后打包的时候会被去除。

  举个🌰，假设我的主干文件（入口文件）是这么写的：

  ```
  import { page1, page2 } from './pages'
      
  // show是事先定义好的函数，大家理解它的功能是展示页面即可
  show(page1)
  ```

  pages 文件里，我虽然导出了两个页面：

  ```
  export const page1 = xxx
  
  export const page2 = xxx
  ```

  但因为 page2 事实上并没有被用到（这个没有被用到的情况在静态分析的过程中是可以被感知出来的），所以打包的结果里会把这部分：

  ```
  export const page2 = xxx;
  ```

  直接删掉，这就是 Tree-Shaking 帮我们做的事情。

  相信大家不难看出，Tree-Shaking 的针对性很强，它更适合用来处理模块级别的冗余代码。至于**粒度更细**的冗余代码的去除，往往会被整合进 JS 或 CSS 的压缩或分离过程中。

  这里我们以当下接受度较高的 UglifyJsPlugin 为例，看一下如何在压缩过程中对碎片化的冗余代码（如 console 语句、注释等）进行自动化删除：

  ```
  const UglifyJsPlugin = require('uglifyjs-webpack-plugin');
  module.exports = {
   plugins: [
     new UglifyJsPlugin({
       // 允许并发
       parallel: true,
       // 开启缓存
       cache: true,
       compress: {
         // 删除所有的console语句    
         drop_console: true,
         // 把使用多次的静态值自动定义为变量
         reduce_vars: true,
       },
       output: {
         // 不保留注释
         comment: false,
         // 使输出的代码尽可能紧凑
         beautify: false
       }
     })
   ]
  }
  ```

  有心的同学会注意到，这段手动引入 UglifyJsPlugin 的代码其实是 webpack3 的用法，webpack4 现在已经默认使用 uglifyjs-webpack-plugin 对代码做压缩了——在 webpack4 中，我们是通过配置 optimization.minimize 与 optimization.minimizer 来自定义压缩相关的操作的。

  这里也引出了我们学习性能优化的一个核心的理念——用什么工具，怎么用，并不是我们这本小册的重点，因为所有的工具都存在用法迭代的问题。但现在大家知道了在打包的过程中做一些如上文所述的“手脚”可以实现打包结果的最优化，那下次大家再去执行打包操作，会不会对这个操作更加留心，从而自己去寻找彼时操作的具体实现方案呢？我最希望大家掌握的技能就是，先在脑海中留下“这个xx操作是对的，是有用的”，在日后的实践中，可以基于这个认知去寻找把正确的操作落地的具体方案。

  #### 按需加载

  大家想象这样一个场景。我现在用 React 构建一个单页应用，用 React-Router 来控制路由，十个路由对应了十个页面，这十个页面都不简单。如果我把这整个项目打一个包，用户打开我的网站时，会发生什么？有很大机率会卡死，对不对？更好的做法肯定是先给用户展示主页，其它页面等请求到了再加载。当然这个情况也比较极端，但却能很好地引出按需加载的思想：

  - 一次不加载完所有的文件内容，只加载此刻需要用到的那部分（会提前做拆分）
  - 当需要更多内容时，再对用到的内容进行即时加载

  好，既然说到这十个 Router 了，我们就拿其中一个开刀，假设我这个 Router 对应的组件叫做 BugComponent，来看看我们如何利用 webpack 做到该组件的按需加载。

  当我们不需要按需加载的时候，我们的代码是这样的：

  ```
  import BugComponent from '../pages/BugComponent'
  ...
  <Route path="/bug" component={BugComponent}>
  ```

  为了开启按需加载，我们要稍作改动。

  首先 webpack 的配置文件要走起来：

  ```
  output: {
      path: path.join(__dirname, '/../dist'),
      filename: 'app.js',
      publicPath: defaultSettings.publicPath,
      // 指定 chunkFilename
      chunkFilename: '[name].[chunkhash:5].chunk.js',
  },
  ```

  路由处的代码也要做一下配合：

  ```
  const getComponent => (location, cb) {
    require.ensure([], (require) => {
      cb(null, require('../pages/BugComponent').default)
    }, 'bug')
  },
  ...
  <Route path="/bug" getComponent={getComponent}>
  ```

  对，核心就是这个方法：

  ```
  require.ensure(dependencies, callback, chunkName)
  ```

  这是一个异步的方法，webpack 在打包时，BugComponent 会被单独打成一个文件，只有在我们跳转 bug 这个路由的时候，这个异步方法的回调才会生效，才会真正地去获取 BugComponent 的内容。这就是按需加载。

  按需加载的粒度，还可以继续细化，细化到更小的组件、细化到某个功能点，都是 ok 的。

  等等，这和说好的不一样啊？不是说 Code-Splitting 才是 React-Router 的按需加载实践吗？

  没错，在 React-Router4 中，我们确实是用 Code-Splitting 替换掉了楼上这个操作。而且如果有使用过 React-Router4 实现过路由级别的按需加载的同学，可能会对 React-Router4 里用到的一个叫“Bundle-Loader”的东西印象深刻。我想很多同学读到按需加载这里，心里的预期或许都是时下大热的 Code-Splitting，而非我呈现出来的这段看似“陈旧”的代码。

  但是，如果大家稍微留个心眼，去看一下 Bundle Loader 并不长的源代码的话，你会发现它竟然还是使用 require.ensure 来实现的——这也是我要把 require.ensure 单独拎出来的重要原因。所谓按需加载，根本上就是在正确的时机去触发相应的回调。理解了这个 require.ensure 的玩法，大家甚至可以结合业务自己去修改一个按需加载模块来用。

  这也应了我之前跟大家强调那段话，工具永远在迭代，唯有掌握核心思想，才可以真正做到举一反三——唯“心”不破！