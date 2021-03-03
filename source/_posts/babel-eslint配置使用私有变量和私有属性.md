---
title: babel+eslint配置使用私有变量和私有属性
date: 2021-03-03 17:52:22
tags: babel, eslint, 前端工程化
---
在使用eslint为我的工程规范代码的时候，发现类私有域`#`还没有得到支持；好吧，它还在草案阶段。
{% asset_img eslint报错.png eslint报错 %}
所以我们首先要安装一个`npm i --save-dev @babel/eslint-parser`，它可以为eslint还不支持的语法提供支持，安装完后在eslint配置文件中设置`parser: '@babel/parser'`（我这里时.eslintrc.js）
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
  parserOptions: {
    ecmaVersion: 12,
    sourceType: 'module',
  },
  rules: {
  },
};
```
到这里，babel已经能帮助我们的eslint规范它还不支持的语言特性，但是我们提到JavaScript的类私有域还在提案阶段，babel的preset-env还没有支持呢！所有需要安装两个babel插件
```shell
npm i --save-dev @babel/plugin-proposal-class-properties @babel/plugin-proposal-private-methods
```
然后在eslint配置文件添加`plugins: ['classPrivateProperties', 'classPrivateMethods']`，至此报错消失，又可以愉快的使用类私有域了。
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
