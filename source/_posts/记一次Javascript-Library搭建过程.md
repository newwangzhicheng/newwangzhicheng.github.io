---
title: 记一次Javascript-Library搭建过程
date: 2021-03-09 10:55:43
tags: 前端工程化
---
随着前端工程化的普及使用，构建一个现代的JavaScript库是愈发复杂，恰好有构建一个js库的需求，现在就从零开始搭建。
考虑到搭建js库大家都推荐rollup，一是自带tree-shaking，二是打包出来的代码更简洁，冗余代码少。但是webpack已经是前端必学的打包工具了，本次搭建先使用webpack，后续考虑使用rollup重新搭建并做对比。

#### 前端工程化的工具

* 打包工具：webpack
* js编译器：babel
* 代码规范：eslint
* 单元测试：jest
* API文档生成：jsdoc

#### 目录

{% asset_img 目录.png %}

这是目录结构，每个文件夹代表不同的职责：

* build：这里存储babel转化好的代码，待npm包发布后commonjs调用该部分的代码，这是经过转化但是未压缩的代码。jsdoc生成的api文档也暂存在这里
* config：这里存储jsdoc的个性化配置脚本和文档生成文件，如果只是简单的生成jsdoc文档这里可以忽略
* demo：这是可以写API简单应用的示例，告诉使用者如何使用
* dist：webpack+babel生成的浏览器可运行的amd类型的打包好的代码存储在这里
* src：这里是源码
* test：一个合格的库必须要有完备的测试用例，用起来，这将是你走向单元测试的第一步

#### 起步：创建npm项目

首先创建一个npm项目，填写好项目的name，description等。。。。

```shell
npm init
```

#### 安装eslint

先全局安装一个eslint包，也可以使用npx命令，在当前目录下创建eslint配置文件

```shell
// npx eslint --init 或者下面这两行
npm install eslint -g
eslint --init
```

具体的配置可按提示一步一步选择，待选择完成后eslint会安装对应的npm包和插件，并配置到package.json的`devDependenceies`中。我这里使用了import的导入方式，airbnb的代码规范，最终会添加下面三个依赖。

```json
"devDependencies": {
    "eslint": "^7.21.0",
    "eslint-config-airbnb-base": "^14.2.1",
    "eslint-plugin-import": "^2.22.1",
  },
```

还会在项目根目录生成eslint配置文件.eslintrc.js（JSON或JS文件，我这里JS），可以看到配置中env浏览器环境和node环境都是允许的，并且可以lint最新的es2021代码规范；

extends里面是我们所使用的代码规范标准，我这里选择的airbnb（视个人喜好，项目要求而定）

parserOptions可以定制支持的js语言，默认支持ES5的代码，`ecmaVersion`可以将支持的环境提升到12，`sourceType`源码类型默认是script，如果是esm类型可以将其改成`module`，

rules允许自定义eslint部分规则

```
module.exports = {
  env: {
    browser: true,
    es2021: true,
    node: true,
  },
  extends: [
    'airbnb-base',
  ],
  parserOptions: {
    ecmaVersion: 12,
    sourceType: 'module',
  },
  rules: {
  },
};
```

到这里，eslint的基本配置就好了，但是eslint不认识的代码我们还需要配合babel来使用，别急，接下来还会再见。

#### 安装babel

如今webpack已经遇到了诸多对手：rollup，parcel甚至是vite；但是babel却鲜有对手，所以必须要学会babel的配置。

此项目安装babel的目标是只将源码babel转换而不用webpack打包，这部分能用使用commonjs或esm的形式引入即可。

babel主要有2个配置：preset和plugins

首先安装babel的核心依赖

* `@babel/core`是babel的核心，它将高版本的ES代码转化成低版本的ES代码
* `@babel/cli`允许我们以命令行的方式运行babel
* `@babel/preset-env`是babel预设的一组插件，它可以根据targets智能地为浏览器不支持的方法polyfill，配合`useBuiltIns`可以按需加载

```shell
npm install --save-dev @babel/core @babel/cli @babel/preset-env
```

然后手动创建一个babel的配置文件babel.config.json，做一些基础的设置，

这里的targets指定各个浏览器的版本，他们不支持的语法由babel负责转换，`useBuiltIns:usage`指示babel只按需加载用到的polyfill而无需将所有polyfill全部载入；

`corejs`指示corejs的次要版本，当`@babel/preset-env`中的corejs不支持某种polyfill时便可以使用指定版本的polyfill；corejs默认不对提案做polyfill，所以这是可选项，这里我默认把它加上（我也不知道有什么影响，先以防万一用到）

```json
{
  "presets": [
    [
      "@babel/env",
      {
        "targets": {
          "edge": "17",
          "firefox": "60",
          "chrome": "67",
          "safari": "11.1",
          "node": "current"
        },
        "useBuiltIns": "usage",
        "corejs": {
          "version": "3.9.1",
          "proposals": true
        }
      }
    ]
  ]
}
```

注：

写JavaScript库时建议用`@babel/plugin-transform-runitme + @babel/runtime-corejs3`，然后在babel.config.json中设置`"useBuiltIns": false`然后删除紧接着的corejs设置；

具体这两种poly fill的差别可以查看以下两个博文，非常仔细且易于理解

[babel详解（五）:polyfill和runtime](https://blog.liuyunzhuge.com/2019/09/04/babel%E8%AF%A6%E8%A7%A3%EF%BC%88%E4%BA%94%EF%BC%89-polyfill%E5%92%8Cruntime/)

[@babel/preset-env 与@babel/plugin-transform-runtime 使用及场景区别](https://segmentfault.com/a/1190000021188054)

首先安装这两个插件：

```shell
npm install --save-dev @babel/plugin-transform-runitme
npm install --save @babel/runtime-corejs3
```

在plugins中添加插件`@babel/plugin-transform-runitme`，这里的设置`"corejs": 3`告诉babel我要使用``@babel/runtime-corejs3`来为我polyfill啦

```json
"plugins": [
  ["@babel/plugin-transform-runtime",
    {
    	"corejs": 3
    }
  ]
]
```

最后别忘了提醒开发者在自己的项目中安装`@babel/runtime-corejs3`，可以在`perDependencies`添加。

#### eslint + babel：支持最新语法的eslint

到这里你已经可以发现，当你写最新的代码的时候，比如类私有域（最新的提案）的时候eslint提示暂不支持，这时就需要babel来帮忙，将eslint的解析器设置为`@babel/parser`让eslint支持babel合法的语法。

首先安装支持的js类私有域的babel插件和babel-eslint解析器

```shell
npm install --save-dev @babel/eslint-parser @babel/plugin-proposal-class-properties @babel/plugin-proposal-private-methods
```

然后在eslint配置文件（.eslintrc.js）中添加插件和解析器

```js
parser: '@babel/parser',
plugins: ['classPrivateProperties', 'classPrivateMethods'],
```

同样的在babel也需要转换这部分代码，所以在babel.config.json也需要添加对应的插件

```json
"plugins": [
  "@babel/plugin-proposal-class-properties",
  "@babel/plugin-proposal-private-methods"
]
```

可以参考一下此时完整的eslint文件

```js
module.exports = {
  env: {
    browser: true,
    es2021: true,
    node: true,
  },
  extends: [
    'airbnb-base',
  ],
  parser: '@babel/parser',
  plugins: ['classPrivateProperties', 'classPrivateMethods'],
  parserOptions: {
    ecmaVersion: 12,
    sourceType: 'module',
  },
  rules: {
  },
};
```

此时我们的eslint和babel都支持类私有域的lint和转换了

#### webpack

如今webpack + babel已经是很多npm项目的标配了，

这里安装webpack的目的是将我们的源码以umd的形式打包成一个文件，方便浏览器引入；同时也支持amd和commonjs的方式引入。

首先安装webpack的核心包

```shell
 npm install --save-dev webpack webpack-cli
```

如果你需要打包的同时分析打包文件各个依赖的大小，可以安装`webpack-bundle-analyzer`这个插件

```shell
npm install --save-dev webpack-bundle-analyzer
```

为了让我们的webpack在打包的时候自动使用babel转译js代码，需要安装一个babel-loader，如何使用我们将稍后使用

```shell
npm install --save-dev babel-loader
```

然后手动创建一个webpack.config.js

配置如下，我们稍后做一下讲解

```js
const path = require('path');
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;
module.exports = {
  entry: './src/hxol/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'hxol.js',
    library: 'hxol',
    libraryTarget: 'umd',
  },
  module: {
    rules: [
      {
        test: /\.m?js/,
        exclude: /(node_modules)/,
        use: {
          loader: 'babel-loader',
        },
      },
    ],
  },
  plugins: [
    new BundleAnalyzerPlugin(),
  ],
};
```

* entry是程序的入口，很好理解；我们可以将我们需要打包的文件在index.js中import再export出来；webpack会根据依赖自动打包
* output是文件的输出位置和格式
  * path指定输出路径
  * filename指示输出文件名
  * library指示我们暴露的全局变量的名称
  * libraryTarget则指示我们输出的类型，这里使用umd，即amd，cmd，浏览器都可以导入
* module指导webpack如何处理不同的模块，上面的规则规定我们转化js/mjs文件，排除node_modules文件夹并使用babel-loader转译
* plugins则是我们使用webpack可能会用的插件，这个`webpack-bundle-analyzer`可以在打包的时候生成一个各个依赖占用大小的报告

#### jest 单元测试

jest进行单元测试就比较简单了，它会自动寻找babel的配置文件并在测试的时候使用babel转译代码，但是jest会自动不转译node_modules中的代码，有转译需求的小伙伴创建一个jest.config.js配置文件，配置如下即可

```js
module.exports = {
  // 转换node_modules中的代码
  transformIgnorePatterns: [],
};
```

当然jest很智能，但它发现babel有test相关的环境时会优先使用test环境去使用babel转译，这样我们就可以为jest定制一个babel配置，这不是必须的

```js
"env": {
    "test": {
      "presets": [
        [
          "@babel/env",
          {
            "targets": {
              "node": "current"
            },
            "useBuiltIns": "usage",
            "corejs": "3.9.1"
          }
        ]
      ]
    }
  }
```

#### 使用

最后所有插件都安装好了，我们可以在src中写源码了，然后进行转译，打包，api生成。可以在package.json中配置这些脚本。

* 转译：`babel your/src/path --out-dir your/dist/path`，babel会自动使用目录下的babel配置，这是npm publish后node项目引入时的文件地址
* 打包：webpack --mode=production，webpack会自动使用目录下的webpack配置，这是浏览器可以引用的代码，当然node项目也可以引入
* 生成API：jsdoc
* 测试：jest，同样的，会自动使用jest配置文件

#### 最后

书写百变，其义自现。前端工程化的配置光看只能理解表面，多实践才能记住。

