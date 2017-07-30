---
layout:     post
title:      【正则表达式系列】字符组、捕获组、非捕获组
category: blog
tags: 正则表达式
favour: 正则表达式
description: 一些正则中的常用名词以及对应概念，譬如字符组，捕获组、非捕获组、反向引用和单词分界等
---

## 前言

本文属于 正则表达式系列文章之一，更多请前往  [正则表达式系列](http://www.jianshu.com/p/d04be43416fb)

本文介绍一些正则中的常用名词以及对应概念，譬如字符组，捕获组、非捕获组、反向引用和`\s` `\b`等

## 大纲

- 字符组

- 捕获组

- 反向引用

- 非捕获组

- `.`、`\s`和`\S`

- `\b`

## 字符组

`[]`字符组表示在同一个位置可能出现的各种字符，也就是说它的匹配结果只能是一个字符，不能是多个

例如`[hello]`匹配的不是`hello`而是`h或e或l或o`

### 特点

- 结果只会匹配一个字符

- 内部特殊字符无需转义`\` `[` `]` 除外

  - 另外，`^`出现在最开始位置时需要转义

  - `-`前后构成区间范围时需要转义（推荐永远使用转义`\-`）

- `-`表示连字符

- `^`表示排除符

## 示例

__特殊字符无需转义__

```js
[\^*\-+|(a)]
```

这个示例的含义是，匹配以下字符中的任意一个

- `^` `*` `-` `+` `|` `(` `a` `)`

- 可以看到，这些特殊字符在字符组中仅仅就是字符本身

__连字符的作用__

```js
[z-a]
```

匹配这个正则表达式会报错（`Range out of order in character class`）

原因是连字符后面的字符码要大于等于前面的字符码

```js
var str = 'abc-';
var reg = /[a-z-]/g;

// 最后的连字符没有构成区间，所以仅仅表示  - 这个字符
str.match(reg); // ["a", "b", "c", "-"]
```

```js
var str = "aAbc-";
var reg = /[A-a-z]/g;

// 通用，A-a 构成一个区间  之后的 -z 无法构成区间，于是它只能作为-字符本身
str.match(reg); // ["a", "A", "-"]
```

__排除型__

```js
var str = '01ABabc-';
var reg = /[^a-z]/g;

// 匹配了除a-z范围外的任意字符
str.match(reg); // ["0", "1", "A", "B", "-"]
```

排除型字符组用法和普通的一样，但是唯一区别是

- 排除型代表符合条件的都不匹配（功能相反）

__`\b`特殊情况__

```js
var str = 'ab\bd,cdef';
var reg = /[\b]/g;
var reg2 =  /\bc/g; 
var reg3 =  /[\b]c/g; 

str.match(reg); // [""] 匹配\b本身
str.match(reg2); // ["c"]  即匹配,号后面的c
str.match(reg3); // null 因为\b后面没有c
```

- `\b`在字符组以外表示单词边界

- 在字符组内，表示退格符(`\b`)

## 捕获组

捕获组就是把正则表达式中子表达式匹配的内容，保存到内存中以数字编号或显式命名的组里，方便后面引用

例如: `/(a)(b)(c)/`中的捕获组编号为

- 组`0`: `abc`

- 组`1`: `a`

- 组`2`: `b`

- 组`3`: `c`

其中，组`0`是正则表达式整体匹配结果，组`1``2``3`才是子表达式匹配结果

### 特点

- 子表达式捕获组编号从`1`开始，顺序从左到右（例如编号`1`是左侧第一个`()`包裹的子表达式的结果）

- 可以在正则表达式中对前面捕获的内容进行引用（反向引用）

- 也可以在程序中，对捕获组捕获的内容进行引用（比如`replace`中）

### 示例

__子表达式捕获组编号从1开始，顺序从左到右__

```js
var str = "2017-07-29";
var reg = /(\d{4})-(\d{2})-(\d{2})/;

// 非全局模式有捕获组结果
str.match(reg); // ["2017-07-29", "2017", "07", "29", index: 0, input: "2017-07-29"]
```

上述匹配中，除了整个正则表达式的结果外，还有各个捕获组的结果，其中，子表达式的捕获组从编号`1`开始，如下

| 编号 | 捕获组 | 匹配内容 |
| :------------- |:-------------|:-------------|
| 0 | (\d{4})-(\d{2})-(\d{2}) | 2017-07-29 |
| 1 | \d{4} | 2017 |
| 2 | \d{2} | 07 |
| 3 | \d{2} | 29 |

__JS程序内的引用__

- 在`replace`中，`JS`通过`$number`引用捕获组内容

- 在外部匹配引用，`JS`通过`RegExp.$number`引用捕获组内容

```js
var str = '<div id="code1" class="highlight"></div>';
var reg = /<(\w+)[^>]*>/g;

// <div></div>
str = str.replace(reg, "<$1>");
```

可以看到，在去除`div`中的属性时，先是整个匹配`<divxxx>`，然后再把整个内容替换成`<$1>`，其中`$1`就是第一个捕获组结果`div`的引用

注，请不要引用`$0`，因为它不属于子表达式的捕获组，在`replace`中引用`$0`没有任何效果

```js
var str = 'abcd0123ABCD';
var reg = /([a-z]+)(\d+)([A-Z]+)/g;

reg.test(str); // true

console.log(RegExp.$0); // undefined
console.log(RegExp.$1); // abcd
console.log(RegExp.$2); // 0123
console.log(RegExp.$3); // ABCD
```

注，同样`$0`的引用没有内容

## 反向引用

在正则表达式内部对捕获组进行引用称之为反向引用

```js
var str = "boom==boom";
var reg = /(boom)==\1/;

str.match(reg); // ["boom==boom", "boom", index: 0, input: "boom==boom"]
```

可以看到，正则中`\1`的值就是捕获组`1`匹配到的结果`boom`

因此，这个表达式等价于`(boom)==boom`

### 示例

```js
var str = '1234567899';
var str2 = '12345678999';
    
var reg = /^(?:([0-9])(?!\1{2})){1,}$/;

reg.test(str); // true
reg.test(str2); // false
```

上例中的效果是，匹配一个数字，但是数字中不允许连续出现`3`次以上的重复数字

- 用到反向引用可以很好的实现

## 非捕获组

上述可以看到`()`包括的内容默认匹配时都在捕获组中

但是有时候，因为特殊原因用到了`()`，但又没有引用它的必要，这时候就可以用非捕获组声明，防止它作为捕获组，降低内存浪费

- `?:`可以声明一个子表达式为非捕获组

```js
var str = 'abcd0123ABCD';
var reg = /(?:[a-z]+)(\d+)([A-Z]+)/g;

reg.test(str); // true

console.log(RegExp.$0); // undefined
console.log(RegExp.$1); // 0123
console.log(RegExp.$2); // ABCD
```

可以看到，`(?:[a-z]+)`将这个子表达式声明成了非捕获组，因此捕获组的编号直接跳过了它，从下一个`(\d+)`开始

## `.`、`\s`和`\S`

首先说下`.`

- 定义是除`\n`以外的任何字符

- 但是，在一些`Chrome`、`Firefox`等内核中，代表`\n`和`\r`以外的字符

- 如果要匹配`.`本身，请用`\.`

再说说`\s`与`\S`

- `\s`是匹配所有的空白字符，包括`空白`、`换行`、`tab缩进`等所有空白

- `\S`是指除了空白以外的任何字符（和`.`区别下，`.`里面还多了一部分空白）

那如何匹配所有字符呢？

- `(.|\n)`或者是`[\s\S]`(推荐用法)

- 请不要试图使用`[.\n]`或`[\.\n]`，这种写法只表示小数点或`\n`字符中的一个

### 示例

请写一个表达式，去除多行注释

```js
var str = '\
    var a = 1; \r\n\
    /\** \r\n\
    * 这里是注释 \r\n\
    */\r\n\
    var b = 2;\r\n\
    console.log(a)';

var reg = xxx;

str = str.replace(reg, '');
```

解答

```js
var reg = /\/\*{1,}[\s\S]*\*\//g;
```

这里就用的了用`[\s\S]`来匹配所有的字符（因为仅仅是`.`是无法匹配`\r\n`的）

## `\b` 单词边界

`\b`匹配单词边界，不匹配任何字符

简单的说，`\b`匹配的位置，一侧是构成单词的字符，另一侧是非单词字符，字符串的开始或结束

而其中单词的判断就是`\w`的匹配范围（正常`a-zA-Z0-9_`，`JS`举例）

注，有一特例，在字符组中`[\b]`表示的是`退格符`

### 特点

- 零宽，即匹配的是位置而不是字符

- 以`\w`来界定单词

- 字符组中是退格符的意思

### 示例

```js
var str = 'abc_d=efg+hij哈opq%';
    
var reg = /.\b./g;

// ["d=", "g+", "j哈", "q%"]
str.match(reg);
```

可以看到，分别在如下几处有单词分界

- `d`和`=`直接有一个分界

- `g`和`+`之间

- `j`和`哈`

- `q`和`%`

## 附录

### 参考资料

* [正则表达式元字符](http://www.runoob.com/regexp/regexp-metachar.html)

* [正则表达式基础系列博客](http://blog.csdn.net/lxcnn?viewmode=contents)