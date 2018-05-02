---
layout:     post
title:      "关于翻转字符串方法的总结"
subtitle:   "JavaScript, 函数"
date:       2018-04-22
author:     "Chou"
header-img: "img/post-bg-js-version.jpg"
catalog:    true
tags:
    - 前端开发
    - JavaScript
    - 算法
---

​	字符串反转问题是常见而且简单的关于字符串的操作，结合其他人的方法和自己的思路简单总结一下，方便以后的学习。

#### 方法一

​	最容易想到的就是利用循环。

​	新建一个空的字符串，循环原字符串的字符，将其拼接成新的字符串。

```   js
let reverseString = function(oldStr) {
    // 新建空字符串，用来储存翻转后的字符串。
    let newStr = ''
    // 递减循环，将原本为空的字符串拼接成目标字符串
    // i = oldStr.length - 1 时对应的是原字符串的最后一个字符。循环到 i = 0 时对应的是原字符串的首字符。
    for (let i = oldStr.length - 1; i >= 0; i--) {
        newStr += oldStr[i]
    }
    // 返回新字符串
    return newStr
}
```

#### 方法二

​	方法一的思路就是利用循环从**最后一位**到**首位**一个一个取字符，进行拼接。所以对应for循环也可以用while来做。

```javascript
let reverseString = function(oldStr) {
    let newStr = '',
        // 将 i 的初始值设为原字符串的长度。
        i = oldStr.length
    while (i > 0) {
        // 利用substring去截取单个字符。
        newStr += oldStr.substring(i - 1, i)
        i--
    }
    return newStr
}
```

#### 方法三

​	现在想用循环来做虽然思路简单，但是写起来代码有点多，看看能不能用内置函数实现一下呢。

​	`+`String.prototype.split()

​	`+`Array.prototype.reverse()

​	`+`Array.prototype.join()

​	split()是一个字符串方法，返回值是数组。该数组的的每一项是字符串的每一个字符。

```javascript
arr = 'abc'.split("")
arr // ['a','b','c']
```

​	reverse()方法将数组进行翻转，新数组是原数组的反向排序。

```javascript
arr = ['a', 'b', 'c']
arr.reverse() // ['c', 'b', 'a']
```

​	join()方法将数组的每一项拼接组成一个字符串。

```javascript
arr = ['a', 'b', 'c']
arr.join("") // 'abc'
```

​	这样想要将任意一个字符串进行翻转就可以像下面这样来实现：

```javascript
let reverseString = function(oldStr) {
    // 先将原字符串用 split 方法进行分割。 
    let splittedStr = oldStr.split("")
    // 再将分割后的数组翻转。
    let reversedStr = splittedStr.reverse()
    // 最后将翻转过的数组用 join 函数拼接起来。
    let joinedStr = reversedStr.join("")
    // 返回结果。
    return joinedStr
}
```

​	上面的整个过程可以简化成： 

```Javascript
let reverseString = function(oldStr) {
    return oldStr.split("").reverse().join("")
}
```

在ES6中利用内置函数的方法会更简洁：

```javascript
let str = 'abc';
[...str].reverse().join(''); // 'cba'
```

或者利用reduce函数

```javascript
let str = 'abc';
[...str].reduceRight( (prev, curr) => prev + curr ); // 'cba'
```

#### 方法四

​	递归往往是最不容易想到的方法（对于我来说。。）

​	这里递归的思想就是把翻转字符串的整个过程看做：

​	第一步：将首字符移到最后一位。

​	第二步：翻转剩余的部分。

​	之后一直重复这样的过程。

​	例如说，原字符串是'abcd'

​	第一步：将 'a' 移到末尾，翻转 'bcd'。 => bcd a

​	第二步：将 'b' 移到末尾，翻转 'cd'。 => cd ba

​	第三步：将 'c' 移到末尾，翻转 'd'。 => d cba

​	结束。

​	实现如下：

```javascript
let reverseString = function(oldStr) {
    if (oldStr === '') {
        return ''
    } else {
        // 将翻转 oldStr 的过程分为两部分：
        // 1.将首字符移到最后一位
        // 2.翻转剩下的部分
        // 一直如此递归...
        return reverseString(oldStr.substr(1)) + oldStr.charAt(0)
    }
}
```

#### 其他方法

​	可以用循环结合内置函数来实现。

