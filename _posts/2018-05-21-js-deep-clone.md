---
layout:       post
title:        "JavaScript对象的深拷贝"
subtitle:     "JavaScript, 前端开发, 基础知识"
date:         2018-05-21
author:       "Chou"
header-img:   "img/post-bg-deep-clone.jpg"
header-mask:  0.3
catalog:      true
multilingual: false
tags:
    - 前端开发
    - JavaScript
    - 知识总结

---



在实际的生产环境中经常需要拷贝一个JS对象，根据不同的数据类型拷贝的方法也不太相同。

首先JavaScript中，数据类型有一下几种：

> 原始类型（基本类型）： String、Number、Boolean、Symbol、Null、Undefined
>
> 引用类型：Object、Array、Date、Function、RegExp 等



浅拷贝和深拷贝的区别： 

浅拷贝只是复制了对象的引用，这个引用指向的还是同一块内存，因此改变拷贝后的对象，原对象的的值也被改变了。

简单的复制语句只能进行浅拷贝，要想做到深拷贝，要用其他的方法。

### 实现浅拷贝

#### 方法一：

`*` 简单的复制语句。

```javascript
// shallowClone
function shallowClone(oldObj) {
    const newObj = {};
    for (let i in oldObj) {
        newObj[i] = oldObj[i];
    }
    return newObj;
}
```

简单测试一下：

```javascript
// 原始数据
var oldObj = {
    a: "Hello",
    b: 1,
    c: true,
    d: ["Foo", "Bar"],
    e: function () {
        alert("Hello World!")
    },
    f: {
        g: 3,
        h: 4	
    }
}

// 进行浅拷贝
var obj = shallowClone(oldObj);

// 检测拷贝后的数据
console.log(obj.a); // Hello
console.log(obj.b); // 1
console.log(obj.c); // true
console.log(obj.d); // ["Foo", "Bar"]
console.log(obj.e); // function () { alert("Hello World!") }
console.log(obj.f); //  { g: 3, h: 4 }

// 更改新数据
obj.a = "Hello,it's me";
obj.b = 2;
obj.c = false;
obj.d = ["Jhon", "Lenoon"];
obj.e = function() {
    alert("I have been changed！");
};
obj.f.g = 5;

// 查看原数据以及是否与新数据相等
console.log(oldObj.a); // Hello // 原对象未被修改
console.log(oldObj.a === obj.a); // false

console.log(oldObj.b); // 1 // 原对象未被修改
console.log(oldObj.b === obj.b);// false

console.log(oldObj.c); // true // 原对象未被修改
console.log(oldObj.c === obj.c); // false

console.log(oldObj.d); // ["Foo", "Bar"] // 原对象未被修改
console.log(oldObj.d === obj.d); // false

console.log(oldObj.e); // function () { alert("Hello World!") } // 原对象未被修改
console.log(oldObj.e === obj.e); // false

console.log(oldObj.f.g); // 5 // 原对象被修改了
console.log(oldObj.f.g === obj.f.g); // true

```



测试结果表明，```obj.f.g```与```obj.f.g```相等，表明他们指向同一个内存地址，所以当改变新的数据之后，原数据也被改变了，这显然不是我们期待的结果。



#### 方法二：

`*`Object.assign()方法



Object.assign()方法用于将所有可枚举属性的值从一个或多个源对象复制到目标对象。它将返回目标对象。



 ```javascript
// 原数据
var oldObj = {
    a: 1,
    b: 2,
    c: 3
}

// 拷贝
var obj = Object.assign(oldObj);

// 检测
console.log(obj.a); // 1
console.log(obj.b); // 2
console.log(obj.c); // 3

// 更改新数据
obj.a = 4;
obj.b = 5;
obj.c = 6;

// 检测
console.log(obj.a === oldObj.a); // true
console.log(oldObj.a); // 4
 ```

上面的检测结果说明，```Object.assign()```也是浅拷贝。



### 实现深拷贝

#### 方法一：

黑科技就是利用```JSON.parse```与```JSON.stringify```结合。

```javascript
var obj = JSON.parse(JSON.stringify(oldObj));
```

简单测试一下：

```javascript
// 原数据
var oldObj = {
    a: 1,
    b: 2,
    c: 3
}

// 拷贝
var obj = JSON.parse(JSON.stringify(oldObj));

// 更改原数据
obj.a = 4;
obj.b = 5;
obj.c = 6;

// 检测
console.log(obj.a === oldObj.a); // false
console.log(oldObj.a); // 1
```

与之前的浅拷贝相比这次明显成功进行了深拷贝。

但是很明显不是最完美的解决办法，因为此方法用的是JSON的```parse()```和```stringify()```,所以只能对能够被json直接表示的数据结构，包括Number、String、Boolean、Array扁平对象。不过大多数的场景应该够用了。

那测试一下对其他的数据类型进行深拷贝会有什么样的结果呢。



```javascript
// 普通函数
function say() {
    console.log("Hello！");
};

// 构造函数
function person(pname) {
    this.name = pname;
}

const Messi = new person('Messi');

// 原数据
const oldObj = {
    a: say,
    b: new Array(1),
    c: new RegExp('ab+c', 'i'),
    d: Messi
    };
const newObj = JSON.parse(JSON.stringify(oldObj));

// 无法复制函数
console.log(newObj.a, oldObj.a); // undefined [Function: say]

// 稀疏数组复制错误
console.log(newObj.b[0], oldObj.b[0]); // null undefined

// 无法复制正则对象
console.log(newObj.c, oldObj.c); // {} /ab+c/i

// 构造函数指向错误
console.log(newObj.d.constructor, oldObj.d.constructor); // [Function: Object] [Function: person]

// 对象循环引用会抛出错误
const oldObj = {};
oldObj.a = oldObj;
const newObj = JSON.parse(JSON.stringify(oldObj));
console.log(newObj.a, oldObj.a); // TypeEror: COnverting circular structor to JSON

```



由测试结果可以看出，这个方法的一些缺陷。



`#`不能深拷贝函数、正则表达式

`#`拷贝之后的构造函数的指向错误，所有的constructor都指向Object。

`#`如果对象有循环引用，会出错。



#### 方法二：

递归拷贝

```javascript
function deepClone(oldObj, newObj) {
    var obj = newObj || {};
    for (var i in oldObj) {
        var prop = oldObj[i];
        
        // 避免循环引用导致的死循环，如oldObj[i] = oldObj
        if (prop === obj) {
            continue;
        }
        if (typeof oldObj[i] === 'object') {
            obj[i] = (prop.constructor === Array) ? [] : {};
            arguments.callee(prop, obj[i]);
        } else {
            // 不是Object类型的话，不存在深浅拷贝，直接简单复制。
            obj[i] = oldObj[i]; 
        }
    }
    return obj;
}
```



递归拷贝还是不能复制正则表达式，构造函数还是指向Object。



#### 方法三：

利用```Object.create()```，可以进行深拷贝。而且函数、正则表达式、稀疏数组都可以被拷贝，构造函数的指向和循环引用问题都得到解决。



```javascript
// 普通函数
function say() {
    console.log("Hello！");
};

// 构造函数
function person(pname) {
    this.name = pname;
}

const Messi = new person('Messi');

// 原数据
const oldObj = {
    a: say,
    b: new Array(1),
    c: new RegExp('ab+c', 'i'),
    d: Messi
    };
const newObj = Object.create(oldObj);

// 复制普通函数
console.log(newObj.a, oldObj.a); // [Function: say] [Function: say]

// 复制稀疏数组
console.log(newObj.b[0], oldObj.b[0]); // undefined undefined

// 复制正则对象
console.log(newObj.c, oldObj.c); // /ab+c/i /ab+c/i

// 构造函数指向正确
console.log(newObj.d.constructor, oldObj.d.constructor); // [Function: person] [Function: person]
```





```javascript
// 对象循环引用不会抛出错误
const oldObj = {};
oldObj.a = oldObj;
const newObj = Object.create(oldObj);
console.log(newObj.a, oldObj.a); // {a: {…}} {a: {…}}
```



