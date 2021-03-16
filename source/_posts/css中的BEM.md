---
title: css中的BEM
date: 2021-03-16 19:44:21
tags: css
---
#### 什么是BEM？
BEM是Block，Element，Modifier的简称；是一种在项目中书写css类名的方式；
它的思想是将用户界面分割成不同的部分，然后更好的重用那些重复的部分。

BEM书写的css类名由三个部分组成`Block-name__Element-name_Modifier-name`，使用单破折号`-`连接一个部分的词组，举例`search-form`因为它的用两个单词代表一个块Block，所以在单词间使用；
用双下划线`__`连接块和元素；用单划线`_`连接块和修饰符，元素和修饰符。

#### Block
Block的中文翻译块，概念上相当于一个名词空间，一个包含众多属性的类，可以是用户界面的一个模块；
比如页头的class类名可以是`header`，一个搜索的表单的类名是`search-form`；

这个叫块的类名是可以嵌套的，也就是说一个外层元素的类名是一个只包含Block类型的class，内层元素依然可以是一个只包含Block类型的class，嵌套的层级没有限制；
因为一个块是可以重用的，所以它的css属性不应该影响它周围的环境，所以Block设置外边距margin，位置position都是不允许的。
```html
<div class="body">
  <div class="main-content">
    <div class="search-form"></div>
  </div>
</div>
```

#### Element
Element是一个元素，概念上相当于类中的属性，是一个模块的一部分的名称，它和块用双下划线连接`__`；
比如一个搜索表单中的搜索框可以是`search-form__input`，查询按钮可以是`search-form__query-button`;

元素这种类型的类名不能单独存在，必须配合块Block使用，不配合它自己不就变成Block了吗；
元素间的类名没有明显的嵌套关系，你可以自由组合他们；`search-form__input`可以在`search-form__body`内部，也可以是它的平级; 
类名中一个块Block只能有一个元素Element，不能这么使用`search-form__body__input`，如果你真的想语义化表达一个body中的输入框可以尝试`search-form__body-input` 。
```html
<div class="search-form">
  <div class="search-form__body">
    <input type="text" class="search-form__input">
  </div>
  <input type="text" class="search-form__input">
</div>
```
#### Block与Element之间的选择

##### 什么时候使用Block？
当这部分代码可能被重用，并且不受周围的组件的实现影响的话，就可以创建一个块类型的class。

##### 什么时候使用Element？
当这部分重用的代码不能单独使用，必须依赖一个外部的环境；这个代表外部的环境代码块就可以用Block的类名，而这部分代码可以使用元素Element；
例如一个表单的查询按钮的样式不能单独存在，必须被一个表单包裹，这个表单的类名可以是`search-form`，这个查询按钮的类名可以是`search-form__query-button`。

当我们需要把代码块分成更小的部分的时候，由于BEM规则下不允许出现`block__elem1__elem2`这样的类名；所以可以把外部代码块的类名定义成一个Block，内部的各个小代码块是Element，如一个搜索框分为用户输入部分和提示部分，就可以定义`input-box__input`和`input-box__prompt`。

#### Modifier
Modifier是一个修饰符，用来修饰块Block或元素Element的外观，状态，行为；
如果想表示一个元素或块的某个状态，可以用`block_modifier`，`block__element_modifier`的形式；举个例子，想要描述一个表单中被点击过的按钮，它的样式可以这么表示`form__button_clicked`。想描述一个横向排列的按钮块可以这么表示`button-group_horizontal`；
如果想表示一个元素或某个块的某个属性的状态呢，可以用`key_value`的模式，他们也用单下划线连接；用公式可以表示为`block_modifier-key_modifier-value`；如果想要表示一个表单的高度是200px，可以这么写`form_height_200`。

表示修饰符的类名不能单独与块或元素使用，只能起到修饰的作用，比如表示一个高度为200px的表单不能这么写，缺少了块
```html
<div class="form_height_200"></div>
```
必须要包含一个块或元素，这才是正确的写法，要记住Modifier只起到修饰的作用
```html
<div class="form form_height_200"></div>
```

#### 混合使用
让我们结合一个示例来描述
```html
<!-- `header` block -->
<div class="header">
    <!--
        The `search-form` block is mixed with the `search-form` element
        from the `header` block
    -->
    <div class="search-form header__search-form"></div>
</div>
```
上面这个例子中，我们在header块中重用了`search-form`这个块，`search-form`是描述搜索表单的一个块，在header块中我们需要在`search-form`的基础上添加一些只在header中特有的样式，所以可以添加一个`header__search-form`，两个组合使用可以重用`search-form`中的样式，在`header__search-form`中又可以定制化一些样式。

一个使用BEM的组成的组件可以是这样子的
{% asset_img 一个BEM的组成.jpeg %}
