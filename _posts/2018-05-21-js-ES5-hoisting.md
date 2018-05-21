#### 声明提升(Hoisting)

变量和函数的声明会被移动到当前作用域的顶端，但是函数会优先于变量。
实际上变量和函数声明在代码里的位置是不会动的，而是在编译阶段被放入内存中。	

##### 变量声明提升

JavaScript only hoists declarations, not initializations.
If a variable is declared and initialized after using it, the  value will be undefined.For example:

```javascript
console.log(num); // Returns undefined
var num;
num = 6;
```



If you declare the variable after it is used, but initialize it beforehand, it will return the value.

```javascript
num = 6;
console.log(num); // returns 6
var num
```



解决for 循环的作用域问题：

```javascript
// ES5
for (var i=1; i<=5; i++) {
    setTimeout(function timer() {
        console.log(i);
    }, i*1000);
}
// 每隔一秒，输出i为6
```

原因：

和JS处理事件的机制有关，for循环只是每次把setTimeout函数添加到任务队列中，在循环执行完毕之后才去执行setTimeout函数，但这个时候i已经变成6了。

解决方法：

`*` 方法1：

```javascript
// ES6 let每次都会创建一个单独的块级作用域。
for (let i=1; i<=5; i++) {
    setTimeout(function timer() {
        console.log(i);
    }, i*1000);
}
```



`*` 方法二：

```javascript
// ES5 利用闭包：立即执行函数为每次循环的时候创建一个单独的作用域。
for (var i=1; i<=5; i++) {
  (function(j) {
    setTimeout(function timer() {
      console.log(j);
    }, j*1000);
  })(i);
}
```



`*` 方法三：

```javascript
// 给setTimeout函数传递第三个参数。
for (var i=1; i<=5; i++) {
    setTimeout(function(i) {
        console.log(i);
    }, i*1000, i);
}
```

第三个参数是附加参数（可选），一旦定时器到期，这个参数会作为参数传递给function。MDN中对[setTimeout](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/setTimeout)的说明。



##### 函数声明提升

```javascript
f();
function f() {
    console.log('函数被成功调用！')
}
// 函数被成功调用！
```

函数在声明之前被成功调用。原因是JS引擎对函数声明进行了提升。

函数声明提升其实做了两个提升动作：
函数名变量提升: 类似变量声明 function f();
函数定义提升: {  / * function body * /  } .

也就是说通过函数声明的方式定义一个函数会有以下三个步骤：
1.声明函数名变量 f ;
2.创建函数对象 funcObj ;
3.把函数名变量 f 指向函数对象 funcObj ;
而一般的函数声明提升会把这三个步骤都提升到作用域的顶部。



##### 函数声明提升优于变量声明提升

```javascript
a();

var a;
function a() {
    console.log(1);
}
a = function() {
    console.log(2);
}

a();
// 1
// 2
```

上面的代码会被JS引擎解析成下面这样：

```javascript
function a() {
    console.log(1);
}
// var a;
a();
a = function() {
    console.log(2);
}
a();
```

由于在``` var a; ```之前a已经被声明，所以``` var a; ```属于重复声明，被引擎省略掉了。

