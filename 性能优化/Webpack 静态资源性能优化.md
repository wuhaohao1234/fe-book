:::info
webpack 性能优化包括构建速度性能优化和打包结果页面性能优化，后者主要策略有

+ 提出无用代码，减少构建体积
+ 提取公共依赖，多脚本复用
+ 首屏静态资源分离，异步加载
+ 文件添加哈希，浏览器缓存复用

:::

## Tree Shaking
[https://webpack.js.org/guides/tree-shaking/](https://webpack.js.org/guides/tree-shaking/)

Tree Shaking 是一种用于移除 JavaScript 中未使用代码的技术。它依赖于 ES6 模块的静态结构，允许在构建过程中确定哪些部分代码没有被实际引用，进而从最终打包的文件中删除这些未使用的代码。就像摇晃一颗树，未用到的代码就像树叶一样落下

### 工作原理
1. 静态分析：ES6 模块（使用 import 和 export 语法）提供了静态的模块依赖关系，在代码解析时候确定依赖关系
2. 标记和移除未使用代码：在构建过程中，Webpack 和依赖的工具（如 Terser）会通过静态分析模块的引入和导出，标记哪些模块是不被使用的，然后删除未被引用的代码块，从而减小打包体积。

### webpack 启用 tree shaking
首先要确保使用 ES module，然后修改 webpack.config.js mode 设置为 `production`后 webpack 自动开启 tree shaking

在 package.json 中加入 sideEffects 字段，这个字段告诉 webpack 哪些文件具有副作用，不应该被 tree shaking 剔除。可以设置为 false 表示所有模块都没有副作用，或者指定某些文件

```json
{
  "name": "your-project",
  "version": "1.0.0",
  "sideEffects": [
    "./src/some-side-effectful-file.js"
  ]
}
```

如果使用了 @babel/preset-env 需要在 babel 配置中关闭ES6 模块语法转换成 CommonJS 模块语法

```json
{
  "presets": [
    ["@babel/preset-env", {
      "modules": false
    }]
  ]
}
```

### 移除未使用的 CSS
除了常规的 CSS 压缩，可以使用 PurgeCSS 通过扫描 HTML、JavaScript 等文件，找到真正使用的 CSS 类名，并移除未使用的部分，让 CSS 有类似 Tree Shaking 的效果

PurgeCSS 是 postcss 插件，项目需要 postcss 配置 `postcss.config.js`

```javascript
const purgecss = require('@fullhuman/postcss-purgecss');

module.exports = {
  plugins: [
    require('autoprefixer'),
    purgecss({
      content: ['./src/**/*.html', './src/**/*.js'], // 指定包含CSS类名的文件路径
      defaultExtractor: content => content.match(/[\w-/:]+(?<!:)/g) || [],
    }),
  ],
};
```

如果不放心移除未被扫描出来的 CSS，可以使用 critters-webpack-plugin，插件通过解析 HTML 和 CSS 文件来确定哪些 CSS 规则在文档中被使用，将使用的 CSS 规则（关键 CSS）提取并内联到 HTML 文件的 <head> 中，没有使用到的 CSS 异步加载，从而减少首次加载时对外部 CSS 文件的依赖，从而提高首屏渲染速度

## Code Splitting
Code Splitting 是为了根据加载实际和代码复用，将代码拆分成多个小块后 bundle 的性能优化手段

### Entry Points 分割
最简单的方法是根据页面路由配置多个入口点，将不同的入口使用到的模块分割到单独的文件中。每个入口点会生成一个新的 bundle 文件

```javascript
const path = require('path');

module.exports = {
  mode: 'production',
  entry: {
    main: './src/index.js',
    vendor: './src/vendor.js'
  },
  output: {
    filename: '[name].bundle.js', // 生成 main.bundle.js 和 vendor.bundle.js
    path: path.resolve(__dirname, 'dist')
  }
};
```

### SplitChunksPlugin
SplitChunksPlugin 是 Webpack 4 引入的一个内置插件，用于智能分割代码块。可以将共享的模块提取到一个单独的文件中，避免同一个模块在不同的 bundle 中重复出现

Webpack 通过 cacheGroups 选项来配置不同缓存组，以便灵活地管理代码分割的策略。不使用 cacheGroups 的时候，Webpack 会使用默认配置生成缓存组提取共享模块

+ defaultVendors: 针对 node_modules 下的第三方模块
+ default: 针对项目内共享的模块

通过 cacheGroups 可以指定特定的模块或文件夹以特定的方式进行分割

```javascript
const path = require('path');

module.exports = {
  mode: 'production',
  entry: {
    main: './src/index.js',
    another: './src/anotherModule.js'
  },
  output: {
    filename: '[name].bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        reactVendor: {
          test: /[\\/]node_modules[\\/](react|react-dom)[\\/]/,
          name: 'react-vendor',
          chunks: 'all',
        },
        utilityVendor: {
          test: /[\\/]node_modules[\\/](lodash|moment)[\\/]/,
          name: 'utility-vendor',
          chunks: 'all',
        },
      },
    },
  },
};
```

通过这些配置，Webpack 会将 React 和 ReactDOM 模块打包到 react-vendor 文件中，而将 Lodash 和 Moment.js 模块打包到 utility-vendor 文件中。

SplitChunksPlugin 提供了很多可以配置的选项，用来精细化控制代码分割行为：

```javascript
module.exports = {
  mode: 'production',
  optimization: {
    splitChunks: {
      chunks: 'all', // 选择哪些 chunk 进行分割，默认值是 'async'
      minSize: 20000, // 生成 chunk 的最小体积，单位是字节
      maxSize: 0, // 生成 chunk 的最大体积，单位是字节，不限制大小
      minChunks: 1, // 模块被引用的最少次数才会被分割
      maxAsyncRequests: 30, // 按需加载时最大的并行请求数
      maxInitialRequests: 30, // 入口点的最大并行请求数
      automaticNameDelimiter: '~', // 文件名的连接符
      cacheGroups: {
        defaultVendors: {
          test: /[\\/]node_modules[\\/]/, // 匹配哪些模块被分组到这个 cache group
          priority: -10, // 缓存组的优先级
          reuseExistingChunk: true, // 如果当前 chunk 包含已从主 bundle 分离出的模块，则重用它
        },
        default: {
          minChunks: 2,
          priority: -20,
          reuseExistingChunk: true,
        },
      },
    },
  },
};
```

:::info
chunks 有三个取值

+ initial：只有入口点的代码会被分割，而动态加载的代码不会被应用分割规则
+ async：只有动态导入的代码块会应用分割规则，初始加载的代码不会被影响
+ all：初始加载和动态加载的模块都会被分割，最大化覆盖分割效果

:::

### Dynamic Imports
动态导入使用 ES6 import() 语法，在代码运行时而非编译时决定导入哪个模块，按需加载模块，Webpack 会自动处理这种语法并生成按需加载的 chunk。这种方式非常适用于需要延迟加载的模块，例如非首屏内容、路由懒加载等。

```javascript
document.getElementById('loadButton').addEventListener('click', () => {
  import(/* webpackChunkName: "lazy-loaded-module" */ './lazyModule.js')
    .then(({ default: lazyModule }) => {
      lazyModule();
    })
    .catch(err => console.error('Error loading the module:', err));
});
```

动态导入在 React 中非常有用，可以实现组件的懒加载：

```javascript
import React, { Suspense, lazy } from 'react';

const LazyComponent = lazy(() => import('./LazyComponent'));

function App() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <LazyComponent />
      </Suspense>
    </div>
  );
}

export default App;
```

为了提升动态 import 模块的家在性能，可以利用 webpackPrefetch 和 webpackPreload 指令，告诉浏览器在适当的时候预加载相关模块：

+ webpackPrefetch: true：在浏览器空闲时加载。
+ webpackPreload: true：在父 chunk 加载时立即加载。

```javascript
// Prefetch
import(/* webpackPrefetch: true */ './moduleA');

// Preload
import(/* webpackPreload: true */ './moduleB');
```

### 首屏 CSS 提取


## 图片优化
在大部分时候图片优化应该通过专业的图片工具和 CDN 处理，在页面只负责使用，不应该通过构建工具处理，小型的个人项目可以通过 webpack 做一些简单优化

+ 图片压缩：image-webpack-loader
+ 小图转 base64：url-loader
+ 图片格式：image-minimizer-webpack-plugin
    - webp：imagemin-webp
    - avif：imagemin-avif
+ 雪碧图：webpack-spritesmith

```javascript
const path = require('path');
const ImageMinimizerPlugin = require('image-minimizer-webpack-plugin');
const SpritesmithPlugin = require('webpack-spritesmith');

module.exports = {
  mode: 'production',
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
  },
  module: {
    rules: [
      {
        test: /\.(png|jpg|jpeg|gif)$/i,
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 8192, // 小于8KB的图片将转换为Base64内联
              name: '[name].[hash].[ext]',
              outputPath: 'images',
            },
          },
          {
            loader: 'image-webpack-loader',
            options: {
              mozjpeg: {
                progressive: true,
                quality: 65,
              },
              pngquant: {
                quality: [0.65, 0.90],
                speed: 4,
              },
              gifsicle: {
                interlaced: false,
              },
              webp: {
                quality: 75,
              },
              avif: {
                quality: 50,
              },
            },
          },
        ],
      },
      {
        test: /\.(svg)$/i,
        use: [
          {
            loader: 'file-loader',
            options: {
              name: '[name].[hash].[ext]',
              outputPath: 'images',
            },
          },
          {
            loader: 'image-webpack-loader',
            options: {
              svgo: {
                plugins: [
                  { removeViewBox: false },
                  { cleanupIDs: false },
                ],
              },
            },
          },
        ],
      },
    ],
  },
  plugins: [
    // 雪碧图插件配置
    new SpritesmithPlugin({
      src: {
        cwd: path.resolve(__dirname, 'src/icons'), // 图标图片路径
        glob: '*.png',
      },
      target: {
        image: path.resolve(__dirname, 'src/sprite/sprite.png'),
        css: [
          [path.resolve(__dirname, 'src/sprite/sprite.css'), { format: 'css_template' }],
        ],
      },
      apiOptions: {
        cssImageRef: '~sprite.png',
      },
      spritesmithOptions: {
        padding: 4,
      },
      customTemplates: {
        css_template: function (data) {
          return data.sprites.map(sprite => {
            return `.icon-${sprite.name} { 
              background-image: url(${sprite.escaped_image}); 
              background-position: ${sprite.px.offset_x} ${sprite.px.offset_y}; 
              width: ${sprite.px.width}; height: ${sprite.px.height}; 
            }`;
          }).join('\n');
        }
      }
    }),
    new ImageMinimizerPlugin({
      minimizerOptions: {
        plugins: [
          ['mozjpeg', { progressive: true, quality: 65 }],
          ['optipng', { enabled: true }],
          ['pngquant', { quality: [0.65, 0.90], speed: 4 }],
          ['gifsicle', { interlaced: false }],
          ['imagemin-webp', { quality: 75 }],
          ['imagemin-avif', { quality: 50 }],
        ],
      },
      generator: [
        {
          type: 'asset',
          implementation: ImageMinimizerPlugin.imageminGenerate,
          filename: 'images/[name][ext].webp',
          options: {
            plugins: ['imagemin-webp'],
          },
        },
        {
          type: 'asset',
          implementation: ImageMinimizerPlugin.imageminGenerate,
          filename: 'images/[name][ext].avif',
          options: {
            plugins: ['imagemin-avif'],
          },
        },
      ],
    }),
  ],
};
```

## CDN 处理
### 通过 CDN 加载第三方库
为了避免将某些第三方库打包到最终的 bundle 中，可以使用 Webpack 的 externals 选项，将指定的第三方库排出，在 HTML 通过 CDN 引用

```javascript
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
  },
  externals: {
    jquery: 'jQuery', // 全局变量 jQuery，来自于 CDN
    lodash: '_',     // 全局变量 Lodash，来自于 CDN
  },
};
```

### 通过 public path 指定静态资源 CDN 地址
`publicPath` 是 webpack 非常重要的配置选项，定义了项目中所有输出文件的公共路径（public URL）

```javascript
const path = require('path');
const isProduction = process.env.NODE_ENV === 'production';

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
    publicPath: isProduction ? 'https://cdn.example.com/' : '/',
  },
  // ...
};
```

+ Dynamic Import 默认使用 import 语句所在文件路径，但如果设置了 publicPath 则会使用 publicPath（也可以通过全局变量 `__webpack_public_path__`指定）
+ 上文提到的图片优化也会使用 publicPath 设置处理后的图片地址

