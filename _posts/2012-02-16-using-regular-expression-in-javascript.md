---
title: JavaScript中的正则表达式操作
author: vivi
layout: post
tags:
  - JavaScript
  - RegExp
---
总是搞混RegExp对象的方法和字符串正则方法，整理备忘一下，抄自JavaScript权威指南

```javascript
var reg = new RegExp("[a-zA-Z]+", "gi");// 或者
var rex  = /[a-zA-Z]+/gim;
```

## RegExp对象方法

```javascript
// 1. exec(string)
// 返回：返回单个匹配，和详细信息
var pattern = /Java/g;
var text = "JavaScript is more fun than Java!";
var result = pattern.exec(text);
while(result != null){
    console.log("match "+result[0]+", start="+result.index+", end="+pattern.lastIndex);
    result = pattern.exec(text);
}

// 2. test(string)
// 返回：如果字符串中含有匹配正则表达式的字符串，那么返回true，如果不包含，那么返回false
var reg = /java/i;
console.log(reg.test("welcome to JavaScript")); // true
// 加上^和$限定就可以用来判定整个字符串是否符合正则表达式
console.log(/^\s*$/.test(""));    // true
console.log(/^\s*$/.test(" 1"));  // false
console.log(/^\s*$/.test("1 "));  // false
console.log(/^\s*$/.test(" 1 ")); // false
console.log(/^\s*$/.test("1 1")); // false
```

## 字符串对象方法
```javascript
// 1. search(regexp)
// 返回：第一个匹配的位置，或者-1; search不支持“g”标记，会直接忽略
"JavaScript".search(/script/i); // return 4

// 2. replace(regexp, "replacement");
// 返回： 替换后的字符串
// 如果没有设置"g"标记，那么只替换第一个匹配的字符串，如果设置了"g"标记，那么会替换所有匹配到的字符串
"javascript javascript".replace(/java/i, "Java"); // Javascript javascript
"javascript javascript".replace(/java/gi, "Java"); //Javascript Javascript
// 可以使用反向引用已匹配到的字符串，如
"var a=1234;".replace(/([a-z]+)=(\d+)/gi, "$1 = $2");// 轻松补上空格

// 3. match(regexp)
// 返回：正则表达式匹配的字符串列表
"1 + 2 = 3".match(/\d+/g);// ["1", "2", "3"]
// 如果设置了g标记，那么返回字符串中所有的匹配
// 如果没有设置g标记，那么只查找第一个匹配，但是返回结果依然是一个列表，list[0]包含匹配的整个字符串， list[1]...list[n]包含第n个分组
var url = /(\w+):\/\/([\w.]+)\/(\S*)/;
var text = "visit http://www.google.com/codeSearch";
var list = text.match(url);
if(list != null){
    console.log(list[0]);
    console.log(list[1]);
    console.log(list[2]);
    console.log(list[3]);
}

// 4. split(regexp) &lt;-- 还支持参数为字符串
// 返回：根据正则表达式分割后的字符串列表
"123,456,789".split(","); // ["123", "456", "789"]
"123  ,  456  , 789".split(/\s*,\s*/); // ["123", "456", "789"]  
```
