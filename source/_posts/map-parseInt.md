---
title: map+parseInt
date: 2021-03-03 12:21:01
tags: JavaScript
---
# 对于`['1', '2', '3'].map(parseInt)`的认知和理解

#### map
数组的map方法中的回调参数如下
```js
array.map((item, index, array) => {});
```

#### parseInt
parseInt将一个字符串（如果不是字符串，`tostring`转化为字符串）转换为一个number类型的整数，并可以指定这个字符串的进制；范围2-36，默认为10进制；
如果radix是0 or `undefined`，则string以`0x`, `0X`开头是16进制
```js
parseInt(string, radix = 10);
```
#### 理解
上面的表达式实际上的调用是这样的
```js
['1', '2', '3'].map((item, index) => {
  return parseInt(item, index);
})
```
结果是`[parseInt('1', 0), parseInt('2', 1), parseInt('3', 2)]`即为`[1, NaN, NaN]`

#### 进一步理解
可以看到题目的需求可能是循环遍历一个字符串数字，用`Number`可以满足需求
```js
`['1', '2', '3'].map(Number)`
```
