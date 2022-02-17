# Webpack

## 安装和运行

**安装**：

```shell
# 安装webpack和webpack-cli
# webpack是真正核心，webpack-cli是webpack的脚手架工具
npm i -D webpack webpack-cli
```

**运行**：

```shell
# 直接使用webpack命令会使用全局的webpack，所以使用npx来调用本地模块
npx webpack
```

## 自定义配置

在当前文件夹下创建 `webpack.config.js` 文件。

```js
// webpack.config.js
const { resolve } = require('path')
// 因为webpack运行在node环境进行打包，所以使用CMD规范
module.exports = {
	// 指定项目的入口文件为src下的index.js
    entry: './src/index.js',
    
    // 指定输出
    output: {
        // 指定打包出的文件名
		filename: 'bundle.js',
        // 指定打包的输出路径（必须是绝对地址）
        path: resolve(__dirname, './build')
    }
}
```

其实我们也可以修改这个 webpack.config.js 的名称和路径，比如改成 wk.config.js。

```shell
npx webpack --config ./wk.config.js
```

**Webpack 依赖图**：

Webpack 对我们的项目进行打包时，会根据配置文件或命令找到入口文件，然后从入口文件开始，生成一个依赖关系图，这个依赖图会包含应用程序中所需的所有模块，包括 js、css、图片、字体等，然后遍历图结构，打包一个个模块（根据文件的不同，使用不同的 loader 来解析）。

所以说，如果项目中存在一个文件，但是从入口文件的依赖图找不到这个文件的引用，这个文件是不会被打包的。而对于一个文件中没有用到过的函数，打包时不打包它使用的技术是 tree shaking。

### entry

entry 配置项配置入口文件地址。

### output

output 配置打包输出相关：

- `output.filename`：配置打包出的文件名；
- `output.path`：配置打包的输出路径，必须是一个绝对路径；
- `output.publicPath`：配置打包后 index.html 引用的基本路径，默认值是一个空字符串，所以我们打包后引入 JS 文件时，路径是 `bundle.js`。在开发时，我们也将其设为 `/`，也就是路径为 `/bundle.js`；

```js
const { resolve } = require('path')

module.exports = {
  output: {
    // 文件名
    filename: 'bundle.js',
    // 输出路径
    path: resolve(__dirname, './build'),
    // index.html打包引用的基本路径
    publicPath: '/'
  }
}
```

**为什么 Vue 脚手架打包出来的应用无法直接运行？**

因为 `publicPath` 被配置为 `/`，这是为了防止某些浏览器不会自动拼接这个 `/`，所以脚手架先加上一个，这样浏览器在请求路径为 `/bundle.js` 的文件时，就会使用 当前地址 + `/bundle.js` 来拼出完整的路径。而我们直接打开打包后的文件时，使用的并不是 http 协议，而是 file，这就导致了路径不正确，自然请求不到资源。而当我们把 `publicPath` 改为 `./` 之后，就可以正常的请求资源了。

### mode

mode 配置当前打包的模式。

配置打包的模式，默认是 propduction。

- `"development"` 开发模式

   相当于设置了：

  ```js
  // webpack.config.js
  module.exports = {
      devtool: 'eval',
      cache: true,
      performance: {
          hints: false
      },
      output: {
          pathinfo: true
      },
      optimization: {
          moduleIds: 'named',
          chundIds: 'named',
          mangleExports: false,
          nodeEnv: 'development',
          flagIncludedChunks: flase,
          splitChunks: {
              hidePathInfo: false,
              minSize: 10000,
              maxAsyncRequests: Infinity,
              maxInitialRequesta: Infinity
          },
          emitOnErrors: true,
          checkWasmTypes: false,
          minimize: false,
          removeAvailableModules: false
      },
      plugins: {
          new webpack.DefinePlugin({
          "process.env.NODE_ENV": JSON.stringify("development")
      })
      }
  }
  ```

  

- `"propduction"` 生产模式

  打包出的文件会被压缩丑化。

  相当于设置了：

  ```js
  // webpack.config.js
  module.exports = {
      performance: {
          hints: 'warning'
      },
      output: {
          pathinfo: false
      },
      optimization: {
          moduleIds: 'deterministic',
          chundIds: 'deterministic',
          mangleExports: 'deterministic',
          nodeEnv: 'production',
          flagIncludedChunks: true,
          occurrenceOrder: true,
          concatenateModules: true,
          splitChunks: {
              hidePathInfo: true,
              minSize: 30000,
              maxAsyncRequests: 5,
              maxInitialRequesta: 3
          },
          emitOnErrors: false,
          checkWasmTypes: true,
          minimize: true,
      },
      plugins: [
          new TerserPlugin(/* ... */),
          new webpack.DefinePlugin({
          "process.env.NODE_ENV": JSON.stringify("production")
      }),
          new webpack.optimize.ModuleConcatenationPlugin(),
          new webpack.NoEmitOnErrorsPlugin()
      ]
  }
  ```


### resolve

resolve 用于设置模块如何被解析。

在开发时我们会依赖各种各样的模块，这些模块可能来自自己编写的代码，也可能是来自第三方库。resolve 可以帮助我们从每个 import/require 语句中，找到需要引入的合适模块代码。

Webpack 能解析三种文件路径：

- 绝对路径：不需要进一步解析。
- 相对路径：根据我们资源所在的目录（上下文）获取资源的绝对路径。
- 模块路径：在 resolve.modules 中指定的所有目录中检索，默认值是 `node_modules`。我们可以通过设置别名来替换初识模块路径。

```js
module.exports = {
  resolve: {
    // 尝试匹配模块后缀名
    extenions: ['.wasm', '.mjs', '.js', '.json', '.jsx', '.ts', '.vue'],
    // 给一个路径起别名，防止层级较深
    alias: {
      // @表示src目录
      "@": path.resolve(__dirname, './src')
    }
  }
}
```



### externals

指定排除打包哪些库，可以将第三方库排除打包，选择使用 CDN 引入。

```js
module.exports = {
  externals: {
    // 不打包lodash，key填写暴露的全局对象
    lodash: '_',
      dayjs: 'dayjs'
  }
}
```



### optimization

optimization 配置项配置了打包优化相关的配置项。

**splitChunks**：

- chunks: 
  - async：默认值，表示只将异步引入的模块拆分
  - initial：同步，
  - all：所有符合配置参数的模块都会被拆分
- runtimeChunk：配置 runtime 相关的代码是否抽取到一个单独的 chunk 中。runtime 相关的代码指的是在运行环境中，对模块进行解析、加载、模块信息相关的代码，模块信息清单在每次有模块变更(hash 变更)时都会变更, 所以我们想把这部分代码单独打包出来, 配合后端缓存策略, 这样就不会因为某个模块的变更导致包含**模块信息的模块**(通常会被包含在最后一个 bundle 中)缓存失效。`optimization.runtimeChunk = true ` 就是告诉 webpack 是否要把这部分单独打包出来。
- cacheGroups：缓存组

## Loader

使用 loader 有两种方式：配置方式、内联方式（cli 方式不支持了）。

配置方式就是在配置文件中写对应的 loader 规则，内联方式是在引入时直接指定，我们常用的是配置方式，更好管理。

### 打包 CSS 资源

直接打包 CSS 资源是不可以的，Webpack 不知道如何对其进行加载，这时我们需要一个 loader 来帮助我们加载资源。

对于 CSS 资源，我们需要 `css-loader` 加载，但是这个 loader 只负责加载资源，并不会将解析之后的样式插入到页面中，我们还需要将样式插入到我们的页面中。

而将样式引入到页面中，我们需要再使用一个 `style-loader`，这个插件会创建一个 style 标签，再将样式插入到标签中。

**安装**：

```shell
npm i css-loader style-loader -D
```

**配置**：

webpack 应用 loader 时，是从下往上（从右往左）应用 loader 的，所以这里我们需要先应用 css-loader，再应用 style-loader。

内联方式：

```js
import 'style-loader!css-loader!../assets/index.css'
```

配置方式：

我们在 `module.rules` 中添加 loader 规则（rule 对象），rule 对象有以下属性：

- test：loader 匹配规则，是一个正则表达式
- use：使用什么 loader，是一个数组，里面放一个个的 UseEntry，每一个 UseEntry 是一个对象，其中属性有：
  -  loader 属性，表示用什么 loader
  - options 属性，可选，会被传入到 loader 中
  - 如果没有 options 需要传入，可以把 UseEntry 对象简写为字符串：`use: ['style-loader', css-loader']`
- loader：如果只有一个 loader，可以直接简写为 `loader: 'css-loader'`

```js
// webpack.config.js
module.exports = {
    //...
    
    module: {
        // 在module.rules中配置多个loader
        rules: [
            // rule对象
            {
                // 匹配规则，匹配以.css结尾的文件
                test: /\.css$/,
                use: [
                    {
                        loader: 'style-loader'
                    },
                    {
                        loader: 'css-loader'
                    }
                ]
            },
        ]
    }
}
```

### 打包 Less 资源

我们再进一步，尝试打包 less 资源。

less 资源需要先编译为 css 资源，我们需要使用 `less-loader` 来处理 less 资源，再使用 `less` 来对 less 资源进行编译。

```shell
npm i less less-loader -D
```

其实在下载好了 less 后，我们就可以使用命令行的形式编译 less 资源了：

```shell
npx less ./src/css/index.less > ./src/css/index.css
```

但是我们希望 webpack 能自动帮我们打包资源，所以我们配置 loader：

```js
// webpack.config.js
module.exports = {
    // ...
    
    module: {
        rules: [
            {
                test: /\.less$/,
                use: [
                    "style-loader",
                    "css-loader",
                    // 这里不需要使用less，less-loader会自己调用
                    "less-loader"
                ]
            }
        ]
    }
}
```

### 浏览器适配以及 PostCSS

#### 浏览器兼容性

我们使用 browserslist 这个工具来在多个不同的前段工具之间共享目标浏览器和 Node.js 版本的配置。这个工具提供目标浏览器版本，来让 babel、postcss 等工具决定如何编译代码。

我们配置 .browserslistrc 文件，类似于：

```
> 1%
last 2 version
not dead
```

#### PostCSS

PostCSS 是一个通过 JS 来转换样式的工具，这个工具可以帮助我们进行一些 CSS 的转换和适配，比如自动添加浏览器前缀、CSS 样式的重置。但是这些功能需要借助于 PostCSS 对应的插件。

**使用步骤**：

1. 查找 PostCSS 在构建工具中的扩展，比如 webpack 中的 postcss-loader
2. 选择可以添加你需要的 PostCSS 相关的插件

安装 postcss-loader：

```shell
npm i postcss-loader -D
```

给样式添加前缀的插件：autoprefixer

```shell
npm i autoprefixer -D
```

将现代的 CSS 特性转换为大部分浏览器可以识别的 CSS 的插件：postcss-preset-env（包含了 autoprefixer 的特性）

```shell
npm i postcss-preset-env -D
```

配置文件：

```js
// webpack.config.js
module.exports = {
    // ...
    module: {
        rules: [
            "style-loader",
            "css-loader",
            // 因为需要传入options，所以写成对象
            {
                loader: "postcss-loader",
                // options是一个对象类型
                options: {
                    postcssOptions: {
                        plugins: [
                            // postcss-preset-env已经包含了添加浏览器前缀的特性，所以不需要autoprefixer
                            // require("autoprefixer"),
                            // 可以简写为字符串
                            "postcss-preset-env"
                        ]
                    }
                }
            }
        ]
    }
}
```

因为我们的 CSS 和 Less 都需要配置 postcss-loader，所以我们可以直接将 postcss 的配置信息提出来，放在根目录下，我们就可以在配置文件简写了。

```js
// webpack.config.js
module.exports = {
    // ...
    
    module: {
        rules: [
            {
                test: /\.css$/,
                use: [
                    "style-loader",
                    "css-loader",
                    "postcss-loader"
                ]
            },
            {
                test: /\.less/,
                use: [
                    "style-loader",
                    "css-loader",
                    "postcss-loader",
                    "less-loader"
                ]
            }
        ]
    }
}
```

```js
// postcss.config.js
module.exports = {
    plugins: [
        require("postcss-preset-env")
    ]
}
```

**问题**：

如果有样式表通过 `@import` 的形式引入，那被引入的样式表并不能被 postcss 添加前缀和 polyfill。因为 `@import` 在 css-loader 中被解析加载，而在这之前 postcss-loader 就已经完成了解析加载，并不会回头再使用 postcss-loader 再加载一次。

所以解决方法是：在 css-loader 中传入一个配置项，让解析完 import 语句后再被它之前 loader 再解析一遍。

```js
// webpack.config.js
module.exports = {
    // ...
    
    module: {
        rules: [
            {
                test: /\.css$/,
                use: [
                    "style-loader",
                    {
                        loader: "css-loader",
                        options: {
                            // 解析import之后，再使用前一个loader再解析一次
                            importLoaders: 1
                        }
                    },
                    "postcss-loader"
                ]
            },
            {
                test: /\.less$/,
                use: [
                    "style-loader",
                    {
                        loader: "css-loader",
                        options: {
                            // 让less-loader和postcss都解析一遍
                            importLoaders: 2
                        }
                    },
                    "postcss-loader",
                    "less-loader"
                ]
            }
        ]
    }
}
```

### 打包其他资源

我们创建一个 img 元素，将 src 设置为我们需要的图片。这里我们不能直接设置路径，而是需要引入后加入 src 属性中，因为 webpack 将需要图片作为模块解析。

```js
// 项目中
const img = new Image()
// file-loader加载完返回一个Module对象，其中的default字段表示模块路径
img.src = require('@assets/img/pic.png').default
document.body.appendChild(img)
```

**file-loader**：

那么现在问题来了，我们又需要一个 loader 来帮助我们加载图片资源，对于这类普通文件，我们可以使用 file-loader 来加载。

```shell
npm i -D file-loader
```

```js
// webpack.config.js
module.exports = {
	// ...
    
    module: {
        rules: [
            {
                test: /\.(png|jpg|jpeg|gif|svg|webp)$/,
                use: [
                    "file-loader"
                ]
            }
        ]
    }
}
```

这样打包之后，图片就可以正常显示了。当然有时我们也会通过 import 来直接引入图片模块：

```js
// 项目中
import imgSource from '@assets/img/pic.png'

const img = new Image()
img.src = imgSource
document.body.appendChild(img)
```

这样也是可以的。

我们可以看到它给我们图片生成的名字很长，是通过 md4 摘要算法生成的。对我们来说，我们可以来指定修改生成的名字：

```js
// webpack.config.js
module.exports = {
    // ...
    
    module: {
        rules: [
            {
                test: /\.(jpg|png)$/,
                use: [
                    {
                        loader: "file-loader",
                        oiptions: {
                            // 指定生成的名字，可以用placeholder来完成
                            // [ext]表示文件的扩展名，[name]是被处理文件的名字，[hash]表示文件的哈希值（摘要），[contentHash]在file-loader中和[hash]是一样的
                            // 把文件放在img文件夹下
                            name: "img/[name]-[hash:6].[ext]"
                        }
                    }
                ]
            }
        ]
    }
}
```

**url-loader**：

url-loader 和 file-loader 的工作方式类似，但是可以把较小的文件，转成 base64 的 URI。

```shell
npm i -D url-loader
```

使用配置和 file-loader 一样，但是会默认把所有图片转成 base64，我们需要给他设置一个限制，不让他转码一些大的图片。

```js
// webpack.config.js
module.exports = {
	// ...
    
    module: {
        rules: [
            {
                test: /\.(jpg|png)$/,
                use: [
                    {
                        loader: "url-loader",
                        options: {
                            // 大于limit不转base64，单位是字节
                            // 小于8K转为base64
                            limit: 8 * 1024
                        }
                    }
                ]
            }
        ]
    }
}
```

在 webpack5 之前我们加载资源的时候主要使用一些 loader，但是在 webpack5 之后，我们可以直接使用资源模块类型（asset module type），来替代上面这些 loader。

资源模块类型：

- asset/resource 发送一个单独的文件，并导出 URL。之前使用 file-loader 实现。
- asset/inline 导出一个资源的 data URI。之前使用 url-loader 实现。
- asset/source 导出资源的源代码。之前通过 raw-loader 实现。
- asset 在导出一个 data URI 和发送一个单独的文件之间自动选择。之前通过使用 url-loader，并配置资源体积限制实现。

```js
// webpack.config.js
module.exports = {
    // ...
    output: {
      filename: 'bundle.js',
      path: resolve(__dirname, './build'),
      // 设置静态资源打包的名字，有一点不一样的是这里的[ext]本身就有点
      // 但是在这里设置完，相当于给所有asset打包的模块都重新命名了
      assetModuleFilename: "img/[name]-[hash:6][ext]"
    },
    module: {
		rules: [
            // asset/resource
            {
                test: /\.(png|jpg)$/,
                // 相当于之前的file-loader
                type: "asset/resource",
                // 如果我们只想给当前loader生成的文件重命名，我们可以在generator中配置
                generator: {
                    filename: "img/[name]-[hash:6][ext]"
                }
            },
            // asset/inline
            {
                test: /\.(png|jpg)$/,
                // 相当于之前的url-loader，将文件转为base64
                type: "asset/inline"
                // 这里就不能写generator了，因为inline不会生成文件
            },
            // asset
            {
                test: /\.(png|jpg)$/,
                // 相当于之前的url-loader，设置limit
                type: "asset",
                generator: {
                    filename: "img/[name]-[hash:6][ext]"
                },
                parser: {
                    dataUrlCondition: {
                        // 设置限制
                        maxSize: 8 * 1024
                    }
                }
            }
        ]
    }
}
```

## Plugin

Loader 是用于指定的模块类型进行转换，Plugin 可以用于执行更加广泛的任务，比如打包优化、资源管理、环境变量注入等。

### 生成 HTML 模板

使用 HtmlWebpackPlugin 来自动生成 html 文件，并引入打包的 js 资源。

```shell
npm i html-webpack-plugin
```

```js
// webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
    // ...
    
    plugins: [
        new HtmlWebpackPlugin({
            // 配置自定义title
            title: 'my webpack app',
            // 指定我们自己的html模板
            template: './public/index.html'
        })
    ]
}
```



### 删除之前的 build 文件夹

我们可以使用 CleanWebpackPlugin 这个插件来在每次打包时删除之前打包的文件夹。

```shell
npm i -D clean-webpack-plugin
```

```js
// webpack.config.js
// 导出了一个类
const { CleanWebpackPlugin } = require('clean-webpack-plugin')

module.exports = {
    // ...
    
    plugins: [
        // 传入插件
        new CleanWebpackPlugin()
    ]
}
```

### 创建全局常量

可以使用 DefinePlugin 这个插件来创建全局常量，这是一个 webpack 内置的插件，不需要单独安装。

```js
// webpack.config.js
const { DefinePlugin } = require('webpack')

module.exports = {
    plugins: [
        new DefinePlugin({
            // 这里取值是拿出字符串里的内容，所以需要包裹两层引号
            BASE_URL: "'./'"
        }) 
    ]
}
```

这里我们就成功地向全局注入了一个 `BASE_URL` 的常量，值为 `./`。

### 复制公共资源

如果我们有一些资源，在 public 文件夹下，我们希望直接复制到 dist 下。这时我们可以选择 CopyWebpackPlugin 这个插件。

```shell
npm i copy-webpack-plugin -D
```

```js
// webpack.config.js
const CopyWebpackPlugin = require('copy-webpack-plugin')

module.exports = {
    plugins: [
        new CopyWebpackPlugin({
            // 匹配哪些文件夹
            patterns: [
                {
                    // 表示我们要从public文件夹复制
                    from: 'public',
                    globOptions: {
                        // 配置哪些文件忽略
                        ignore: [
                            // index.html会自己生成，不需要复制
                            "**/index.html",
                            // mac电脑经常生成的记录文件
                            "**/.DS_Store"
                        ]
                    }
                }
            ]
        })
    ]
}
```

## Webpack 模块化原理

我们在开发时是可以随便使用两种模块化规范的（ESM 和 CJS），这些就是 Webpack 帮助我们实现的。

### CJS 模块化原理

对于导入语句，Webpack 将我们的模块包裹在了一个对象 `__webpack_module__` 中，对象的 key 是模块的路径，value 是一个函数，传入 module，函数内部将暴露的内容添加到 module.exports 上。实现了一个 `__webpack_require__` 函数，用于引入模块，第一次加载模块时，会将模块缓存进 `__webpack_module_cache__` 对象中，以后再加载就直接从这个缓存对象中取。加载模块就是调用 `__webpack_module__` 中保存的函数，并会传入 module，module.exports 等参数，返回的值就是我们暴露的对象。

###  ESM 模块化原理

定义了一个对象，和 CJS 模块一样，对象里保存了模块路径和模块。也定义了模块缓存。然后对 `__webpack_require__` 赋了一个新属性 d，值为一个函数，这个函数将 exports 做了一个代理，从 exports 中取属性的时候，本质上是从一个对象中取，这个对象中对应的值为一个函数，返回对应模块。还有一个 新属性 o，也是一个函数，以及一个属性 r，也是一个函数，用来标志导出的对象是一个模块，如果模块是一个 ESM，会在这里标记为 __esmodule。

## Source-Map

如果我们开发时有代码报错，但是我们的代码被 webpack 打包过，无法直接找到报错位置。这时我们就需要 source-map 文件来还原我们原来的文件。

source-map 是从已转化的代码，映射到原始的源文件，是浏览器可以重构原始源并在调试器中显示重建的原始源。

### 使用

1. 根据源文件，生成 source-map 文件，webpack 在打包时，可以通过配置生成 source-map。
2. 在转换后的代码，最后添加一个注释，指向 source-map 文件（配置好 wenpack 会自己加的）。

```js
// webpack.config.js
module.exports = {
    devtool: 'source-map'
}
```

### 配置

devtool 可以设置很多值（现在有26个），主要的关键字有：

- `(none)`：什么值都不设置，不生成 source-map
- `false`：不生成相关的 source-map
- `'eval'`：会将打包的代码放进 eval 函数，在代码末尾会被添加注释，在浏览器中会被转回源代码。
- `'source-map'`：生成一个独立的 source-map 文件，并且在 bundle 文件中有一个注释，指向 source-map 文件。开发工具会根据打包代码最后的注释找到这个文件解析。但是构建速度很慢。
- `'eval-source-map'`：也会生成 source-map，但是 source-map 是以 DataUrl 添加到 eval 函数后面（原先生成的 .map 文件以 base64 编码插入到 eval 后面）。
- `'inline-source-map'`：所有的 source-map 文件以 base64 的形式放在打包的代码最后的注释里。
- `'cheap-source-map'`：会生成 source-map，但是会更高效一些，因为它没有生成列映射，因为在开发中，我们只需要行信息就可以定位到错误了。
- `'cheap-module-source-map'`：会生成 source-map，类似 cheap-source-map，但是对源自 loader 的 source-map 处理会更好（比如用 babel 转化过代码，cheap-source-map 就不能映射回源文件，而是被 babel 转化的代码）。
- `'hidden-source-map'`：还会生成 source-map，但是在打包的代码中不会有注释指明 map 地址，即 map 不会生效。
- `'nosources-source-map'`：没有源的 source-map，生成的 source-map 只有错误信息提示，不会生成源代码文件。

组合规则：

- `inline- | hidden- | eval`：三个值选一个
- `nosources`：可选
- `cheap` 可选，并可以跟 module 的值

`[inline-|hidden-|eval-][nosources-][cheap-[module-]]source-map`

### 最佳实践

开发阶段：选择 `source-map` 或 `cheap-source-map`。

测试阶段：同开发阶段。

生产阶段：false 或 不写。映射在生产环境既没有用，也防止别人还原代码。

## Babel 深入解析

在开发中，我们很少接触 babel，但是 babel 对于前端开发来说，是不可缺少的一部分。在开发和学习中，我们希望使用更新的语法，包括 TS，这些都是离不开 babel 的。

babel 是一个工具链，主要用于旧浏览器环境中将 ES6+ 代码转换为向后兼容的 JS 代码，包括语法转换、源代码转换、polyfill 实现目标缓解缺少的功能。

### 安装

```shell
# 安装babel的内核
npm i @babel/core -D
# 命令行工具（不需要就不安装）
npm i @babel/cli -D
```

babel 安装完是不知道要如何转化我们的代码的，所以我们需要安装一些其他的插件来帮助 babel 转化。

```shell
# 转化箭头函数的插件
npm i -D @babel/plugin-transform-arrow-functions
# 转化ES6块级作用域的插件
npm i -D @babel/plugin-transform-block-scoping
```

我们可以很轻易地发现这样做是不可能用于开发的，因为要配置的语法实在太多了，所以我们使用预设：

```shell
# 安装预设
npm i -D @babel/preset-env 
```

我们把 babel 的配置写入 webpack 中：

```js
// webpack.config.js
module.exports = {
    module: {
        rules: [
            {
                test: /\.js$/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: [
                            '@babel/preset-env'
                        ]
                    }
                }
            }
        ]
    }
}
```

`@babel/preset-env` 默认情况下是根据 browserslist 的配置进行转化。

### 原理

JS 代码在执行时，会先经过 parsing 变为 AST，再通过 transformation 变为字节码，字节码就会被 JS 引擎处理为机器码执行。

而 babel 会解析 JS 源代码，生成源代码的 AST 后，遍历 AST，生成一个目标代码的 AST 树，再通过 generator 生成目标 JS 代码，本质上就是一个编译器。

### 抽取配置

我们可以抽取 babel 的配置信息到单独的文件中，这样我们可以避免多次书写配置。

配置文件名可以为：babel.config.json（或 js 文件）/ .babelrc.json。

.babelrc.json 早期使用较多，但是对于配置多包管理项目比较麻烦。

babel.config.json 可以直接作用于多包管理的子包，更加推荐。

```js
// babel.config.js
module.exports = {
    presets: [
        '@babel/preset-env'
    ]
}
```

### Polyfill

polyfill 是垫片的意思，在这里 polyfill 是用来给我们的浏览器打补丁，来让一些老旧的浏览器适配一些新特性，比如 Promise 就需要一个 polyfill。

```shell
# 原先我们可以直接引入这个包，但是现在推荐我们分开安装两个包
npm i @babel/polyfill
# core-js 是一些核心的补丁
npm i core-js
# 包含了generator、async、await这些polyfill，Facebook在维护这个库
npm i regenerator-runtime
```

**使用**：

```js
// babel.config.js
module.exports = {
    presets: [
        ["@babel/preset-env", {
            // 那些属性需要构建进去，也就是怎么使用polyfill
            // 如果填false，即不对polyfills做任何操作，就是全部引入，填'useage'是按用户代码来引入（按需引入），填'entry'是按照目标浏览器来进行polyfill 
            useBuiltIns: 'useage'
        }]
    ]
}
```

当我们选择 useage 时，要注意可能有一些第三方库自己已经实现了 polyfill 了，有可能会产生一些冲突。一般我们会排除一些文件来不使用 babel 打包的。

```js
// webpack.config.js
module.exports = {
    module: {
        rules: [
            {
                test: /\.js$/,
                // 第三方库不使用 babel 打包
                exclude: /node_modules/,
                use: "babel-loader"
            }
        ]
    }
}
```

 如果我们选择 entry，我们还需要在源代码中引入一下这两个库：

```js
// index.js
import "core-js/stable";
import "regenerator-runtime/runtime";
```

我们也可以指定 core-js 的版本，防止默认版本和我们安装的不一样，最终报错：

```js
// babel.config.js
module.exports = {
    presets: [
        ["@babel/preset-env", {
            useBuiltIns: 'useage',
            // 指定core-js版本为3
            corejs: 3
        }]
    ]
}
```

### React 的 JSX

在编写 react 代码时，我们可以使用 babel 来转化 jsx。

我们需要安装这个预设：

```shell
npm i -D @babel/preset-react
```

配置：

```js
// babel.config.js
module.exports = {
    presets: [
        "@babel/preset-react"
    ]
}
```

### TypeScript

babel 也是可以对 ts 进行编译的。

我们可以用 ts-loader 来在 webpack 统一加载 ts 文件，原理就是当我们打包时匹配到 ts 文件，就会调用 tsc 来处理文件，所以我们需要安装 typescript 这个包。

```shell
# 安装一下ts-loader和typescript
npm i typescript ts-loader -D
# 初始化tsconfig.js
tsc --init
```

```js
// webpack.config.js
module.exports = {
    module: {
        rules: [
            {
                test: /\.ts$/,
                use: {
                    loader: 'ts-loader'
                }
            }
        ]
    }
}
```

这样我们就可以对 ts 文件进行统一的编译打包了，但是这么做我们没有办法对打包的 ts 文件进行 polyfill。所以我们还可以不使用 ts-loader，选择 babe-loader 来加载 ts 文件（但其实我们可以加载完ts-loader后用babel-loader再加载一次？最好不要混用）。

如果我们使用 babel 来加载 ts，那我们可以不依赖 ts-loader 和 typescript。

```shell
npm i @babel/preset-typescript
```

```js
// babel.config.js
module.exports = {
    presets: [
        ["@babel/preset-env", {
            useBuiltIns: 'usage',
            corejs: 3
        }],
        ["@babel/preset-typescript"]
    ]
}
```

**最佳实践**：

在开发时，我们有两种选择，要么使用 ts-loader 来进行处理，要么用 babel 的 ts 预设来处理。

- babel-loader：使用 babel 来处理 ts 不会进行类型检测（类型擦除），如果有类型错误还是可以编译成功的
- ts-loader：有类型检查，但是没有 polyfill。

所以，我们可以使用 babel-loader 来加载 ts，但是我们在这之前先使用 tsc 来检测代码（`tsc --noEmit` 执行这条命令来检查错误，但是不输出文件）。也可以建一个脚本来实时检测变化和类型检查：`tsc --noEmit --watch`。

## ESLint

ESLint 是一个静态代码分析工具，可以帮助我们建立一个统一的团队代码规范。

```shell
npm i eslint -D
```

### 创建配置文件

```shell
npx eslint --init
```

如果想关掉那个错误，就可以复制那个错误，把它填写进 rules 中，选项填为 true。

```js
module.exports = {
    // ...
    
    rules: {
        // 关掉未使用变量报错
        'no-unused-var': 'off'
        // 除了off，我们还可以选为warn，这样只会警告，而不报错，或者选error，代表报错
        'quotes': ['warn', 'single']
    }
}
```

具体配置规范我们可以去查官网。

### webpack配置

我们可以配置一个 loader，来让每次打包的时候处理一次。

```shell
npm i eslint-loader -D
```

```js
// webpack.config.js
module.exports = {
    module: {
        rules: [
            {
                test: /\.js$/,
                use: [
                    "babel-loader",
                    "eslint-loader"
                ]
            }
        ]
    }
}
```

### 插件

eslint：来对代码静态分析，我们可以直接在编写代码时看到报错。

prettier：快速格式化代码，我们可以建一个文件来具体配置格式化规范

```js
// .prettierrc
{
    // 使用单引号
    "singleQuote": true
}
```

具体 prettier 配置可以去官网查。

## devServer 和 HMR

在开发时，我们不可能每次修改源代码都重新打包，我们需要一个开发时自动打包的工具。

### watch 模式

如果我们只是想监听源代码的改变并重新打包，我们可以执行 `webpack --watch` 命令来监视源代码变化。

但是这样并不会帮助我们启动本地服务，而且有代码改变后会全部打包并写入文件，所以我们还希望帮我们自动启动一个本地服务，并能帮我们自动刷新页面。

### devServer

**安装**：

```shell
npm i -D webpack-dev-server
```

**启动**：

```shell
# webpack 5
npx webpack serve
```

启动后会帮助我们打开一个本地服务器，默认会跑在 8080 端口。dev-server 不会生成打包文件，而是会打包进内存中。

**配置**：

`devServer` 配置本地服务相关：

- `publicPath`：本地服务所在的文件夹，默认值是 `/`，如果我们将其改为 `/xxx`，那我们就需要通过 `http://localhost:8080/xxx` 才能访问到打包后的资源，并且这个时候，我们打包出的 index.html 中的资源路径也是访问不到的，需要将 `output.publicPath` 也设置为 `/xxx` 才行。官方也建议将这两个属性设置为相同的值。

- `contentBase`：设置 dev-server 对外服务的内容来源。如果我们打包后的资源又依赖于其他的一些资源，需要指定从哪里查找这个内容。建议使用绝对路径

- `host`：配置主机地址，比如希望其他子网的用户能访问，可以配置为 `0.0.0.0`（监听 IPV4 上的所有地址）。

- `port`：监听哪一个端口。

- `proxy`：配置代理信息，在开发环境下解决跨域

  ```js
  module.exports = {
    devServer: {
      proxy: {
        "/api": {
        	// 将所有/api请求代理到这个地址
          target: "http://123.xxx.xx.xx:8000",
          // 以/api开头的会被重写为空字符串，就可以去除/api了
          pathRewrite: {
            "^/api": ""
          },
          // 这个配置项开启会改变我们的请求源地址
          // 比如我们的项目在本地的a地址，服务器在b地址，我们被代理的请求会显示我们是a地址，但是有时服务器会校验请求来源，如果不希望我们被校验失败，我们可以开启这个选项，请求就会显示我们也是b地址了。
          changeOrigin: true,
          // 解决开发时前端history路由刷新问题，本质上就是404时返回了index.html，前端拿到index.html自己进行路由。
          historyApiFallback: true
        }
      }
    }
  }
  ```

  

`devServer.publicPath` 是服务开启到哪个路径上，`output.publicPath` 是 `index.html` 中引用的资源路径前缀；至于 `devServer.contentBase`，是 dev-server 返回的静态资源文件夹，即 `app.get(devServer.publicPath, express.static(devServer.contentBase))`。

**原理**：

自己实现一个本地服务：

首先安装一下需要的依赖：

```shell
npm i -D webpack-dev-middleware express
```

然后开始搭建服务：

```js
// server.js
const express = require('express')
// webpack
const webpack = require('webpack')
// webpack dev 中间件
const webpackDevMiddleware = require('webpack-dev-middleware')

const app = express()

// webpack传入配置信息，返回一个compiler
const compiler = webpack(require('./webpack.config.js'))

// 把compiler再传给webpack dev中间件，生成一个express中间件
const middleware = webpackDevMiddleware(compiler)

app.use(middleware)

app.listen(8080, () => {
  console.log('8080 is listening...')
})
```

### HMR 模块热替换

HMR（Hot Module Replacement），指的是在应用程序运行的过程中，替换、添加、删除模块，而无需重新刷新整个页面。这样做可以很大的提高我们的开发性能和效率。

- HMR 不会重新刷新整个页面
- 只更新变更的内容，提高开发效率
- 修改了 CSS 和 JS 源代码，会立即在浏览器中更新，相当于直接在 devtools 修改样式

webpack-dev-server 默认情况下使用 live reloading 技术，即每次改变实时刷新页面。如果我们希望启用 HMR，我们需要添加一下配置：

```js
// webpack.config.js
module.exports = {
  devServer: {
    // 开启HMR，但是我们还需要指定哪些模块需要HMR
		hot: true
  }
}

// 源代码中：
// 让math模块支持HMR
module.hot.accept("./math.js", () => {
	console.log('math 模块发生了更新')
})
```

我们可以发现这么做非常麻烦，所以我们在开发中，还是会使用框架支持好的 HMR，比如 vue-loader、react-refresh 都是支持 HMR 的。

**原理**：

devServer 会创建两个服务：提供静态资源的服务（express）和 Socket 服务（net.Socket）。

Webpack compiler 将打包出来的 bundle.js 给静态资源服务器，来让浏览器访问。因为对这个服务器的访问都是 http 访问，服务器没有办法对浏览器推送新的，修改后的模块，这时就需要一个长链接服务器（Socket 服务）来推送资源。

webpack-dev-server 创建的第二个服务器就是 HMR Server，与浏览器中的 HMR runtime 建立了一个 socket 长链接，来将新的资源推送给浏览器。

## 环境分离



现在我们希望将 Webpack 在开发和生产时的配置分开，这样我们就可以通过两个不同的命令来加载不同的 Webpack 配置。

```json
// package.json
{
	"script": {
    // 开发时运行这个脚本，指定开发时配置文件
    "serve": "webpack serve --config ./config/webpack.dev.js",
    // 打包时运行这个脚本，指定生产时配置文件
    "build": "webpack --config ./config/webpack.prod.js"
  }
}
```

这样我们将配置拆分为三个配置：

- webpack.common.js 共有的配置
- webpack.dev.js 开发时配置
- webpack.prod.js 生产时配置

判断环境：

设置过 mode 后，DefinePlugin 会将 `process.env.NODE_ENV` 设置为 `"production" / "development"`，我们在源代码中可以根据这个值来判断。

**合并配置**：

我们还需要把 common 和开发或生产的配置合并起来，这里我们还需要一个库：webpack-merge。

```shell
npm i -D webpack-merge
```

```js
// webpack.prod.js
// 这里以生产环境举例，开发环境同理。

// 引入webpack-merge
const { merge } = require('webpack-merge')
// 引入common配置
const commonConfig = require('./webpack.common.js')

module.exports = merge(commonConfig, {
  // 生产环境配置
  mode: 'production'
})
```

## 代码分离

现在我们打包出来的所有代码都在一个 JS 文件中，这会导致一个文件的加载速度非常慢，我们可以对打包的代码进行分离，来控制资源加载优先级，提高代码加载性能。

Webpack 常用的代码分离有三种：

- 入口起点：使用 entry 配置手动分离代码。

  ```js
  module.exports = {
    entry: {
      // 2个入口文件
      main: './src/main.js',
      index: './src/index.js'
    },
    output: {
      // 文件名使用placeholder来添加名字
      filename: '[name].bundle.js'
    }
  }
  ```

- 防止重复：使用 Entry Dependencies 或者 SplitChunksPlugin 去重和分离代码。

  如果我们配置了多个入口，多个入口还依赖了同一个库，那么这个库会被打包多次。这时我们还需要配置优化：

  ```js
  // Entry Dependencies方式分离，不是很推荐
  module.exports = {
    entry: {
      // 指明main和index都需要依赖这些
      main: { import: './src/main.js', dependOn: 'shared' },
      index: { import: './src/index.js', dependOn: 'shared' },
      // 让这些依赖单独打包
      shared: ['lodash', 'dayjs']
    }
  }
  
  // SplitChunksPlugin方式
  // 该插件Webpack默认集成安装，我们可以手动修改他的默认配置
  module.exports = {
    optimization: {
  		splitChunks: {
        // 默认值是async，表示只有代码里异步加载模块才会对代码进行分离，initial表示对同步进行分离，all表示都进行分离
        chunks: 'all',
        // 最小值，默认是20000，单位是字节。表示如果拆分了一个包，那么我们拆分出的包最小也得是minSize（太小的包不拆，因为会影响网络传输）
        minSize: 20000,
        // 最大值，如果有包大于maxSize，会对他再次拆分，拆成不小于minSize的包
        maxSize: 20000,
        // 表示引入的包至少被导入几次才会拆分
        minChunks: 2,
        cacheGroups: {
          vendor: {
            // 匹配哪些第三方库，这里就是node_modules，这里需要一个路径
            test: /[\\/]node_modules[\\/]/,
            filename: "[id]_vendors.js"
          }
        }
      }
    }
  }
  ```

  

- 动态导入：通过模块的内联函数调用分离代码。

  **只要是异步导入（`import()`函数）的模块，都是分离打包的**。

  还可以使用魔法注释来确定 chunk 的 name。

  ```js
  const Foo = () => import(
    // 指定打包名
    /* webpackChunkName: 'foo' */ 
    // 对包预下载，会在父chunk下载结束时立即下载（浏览器闲置时下载）
    /* webpackPrefetch: true */
    // 对包预加载，会在父chunk下载时并行下载，preLoad和prefetch只能写一个
    /* webpackLoad: true */
    './Foo.vue')
  ```
  
  通过异步导入，我们可以实现代码懒加载。

## CDN

 

CDN 是内容分发网络，是指通过相互连接的网络系统，利用最靠近每个用户的服务器，更快的将文件发送给用户，来提高性能。

在开发时，我们使用 CDN 主要是两种方式：

- 打包所有的静态资源，放到 CDN 服务器，用户的所有资源通过 CDN 服务器加载。
- 一些第三方的资源放到 CDN 服务器上。 

想要使用 CDN，首先要买 CDN 服务器。

**静态资源**：

我们可以直接修改 publicPath，在打包时添加自己的 CDN 地址，这样所有的静态资源都会访问静态资源。

**第三方库**：

如果我们不配置，直接打包，那么我们的第三方库实际上是一起打包了，相当于放进了我们自己的服务器。

所以我们在打包时，排除需要放入 CDN 的第三方库，在 index.html 中直接引入 CDN 链接。

```js
// webpack.config.js
module.exports = {
  // ...
  externals: {
    // 不打包lodash，key填写暴露的全局对象
    lodash: '_',
    dayjs: 'dayjs'
  }
}
```

 **小知识**：sctipt 的三个属性：

- 不写：浏览器会暂停 HTML 的解析，去下载并执行 JS 代码，执行完再继续解析。
-  async：浏览器等到遇到 script 标签先去下载，同时继续解析 HTML，下载好立即执行 JS 代码，会导致多个 async 脚本之间的执行顺序不确定。
- defer：浏览器下载 script 文件，并继续解析 HTML，当 HTML 全部解析完成，才会执行代码，多个 defer 之间按照顺序执行，不会乱。

## 提取 CSS

MiniCssExtractPlugin 可以帮助我们将 CSS 提取到一个单独的文件中。

```js
const MiniCssExtractPlugin = reqire('mimi-css-extract-plugin')

module.exports = {
  modules: {
    rules: [
      {
        test: /\.css$/,
        use: [isProduction ? MiniCssExtractPlugin.loader : "style-loader", "css-loader"]
      }
    ]
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: "css/[name].[hash:8].css"
    })
  ]
}
```

## Hash

在我们给打包的文件命名时，会使用 placeholder，placeholder 中有几个属性比较相似：

- hash：通过 MD4 散列函数处理后，生成一个 128 位的 hash 值（32 个十六进制）。对整个项目进行散列，适合项目变动就需要更新的文件名。
- chunkhash：只会根据自己的依赖树进行散列，建议不需要根据整个项目变动的文件使用。
- contenthash：根据包的内容进行散列，建议一些不依赖其他模块的文件使用。

hash 值生成和整个项目有关，只要修改了项目，那么多个入口打包出来的文件名的 hash 都会变，导致浏览器缓存失效。

建议对打包出来的 output.filename 使用 chunkhash，对每一个单独的 chunk 使用 chunkhash。
