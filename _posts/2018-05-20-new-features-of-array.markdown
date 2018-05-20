---
layout:       post
title:        "ES6中数组的新特性"
subtitle:     "JavaScript, ES6, 基础知识"
date:         2018-05-20
author:       "Chou"
header-img:   "img/post-bg-new-features-of-array.jpg"
header-mask:  0.3
catalog:      true
multilingual: false
tags:
    - 前端开发
    - JavaScript
    - 知识总结
---
####ES6's new array methods

#####(1). Array.of()

​      RegSardless of the number and type of parameters,all the incoming parameters of the method will be the data in the array.

  ``` javascript
let items = Array.of(1, '2', [3, 4]);
console.log(items); // [1, '2', [3, 4]]
console.log(items.length); // 3
  ```



#####(2). Array.from()

​      Convert **array-like objects** and **iterable objects** into arrays.

  ```javascript
// Array-like objects
 
let trans = function(values) {
    return Array.from(arguments, item => item + 1);
    }

console.log(trans(1, 2, 3)); // [2, 3, 4]
  ```

   ```javascript
// Iterable objects

let obj = {
   	*Symbol.iterator {
   	   	yield 1;
   	   	yield 2;
   	   	yield 3;
   	    }
    }
   
console.log(Array.from(obj)); // [1, 2, 3]
   ```



#####(3). Array.prototype.find() 

​      The find() method returns the value of the first element in the array that satisfies the provided testing function. 
     Otherwise undefined is returned.

```javascript
let arr = Array.of(1, 5, 8, 9);
console.log(arr.find(item => item > 5)); // 8
console.log(arr.find(item => item > 9)); // undefined
```



 #####(4).Array.prototype.findIndex()

​     The findIndex() method returns the index of the first element in the array that satisfies the provided testing function.
     Otherwise -1 is returned.

```javascript
let arr = Array.of(1, 5, 8, 9);
console.log(arr.findIndex(item => item > 5)); // 2
console.log(arr.findIndex(item => item > 9)); // -1
```



##### (5). Array.prototype.fill()

​      The fill method fills all the elements of an array from a start index to an end index with a static value.
     // The end index is not included.
     // If only the start index is given and the end index is not specified, the default ending index is the end of the array.

```javascript
let arr = Array.of(1, 5, 8, 9);
console.log(arr.fill(0, 1, 2)); // [1, 0, 8, 9]
console.log(arr.fill(0, 1)); // [1, 0, 0, 0]
```


##### (6). Array.prototype.copyWithin()

​     The copyWithin method shallow copy part of an array to another location in the same array and returns it, without modifying its size.
      //arr.copyWithin(target, start, end)

```javascript
let arr = Array.of(1, 5, 8, 9, 10);
console.log(arr.copyWithin(1, 2, 2)); // [1, 5, 8, 9, 10]
console.log(arr.copyWithin(1,3)) // [1, 9, 10, 9, 10]
```



#####(7).Array.prototype.keys()

​     The keys() method returns a new **Array Iterator** object that contains the keys for each index in the array.

```javascript
let arr = [1, 2, 3];
let iterator = arr.keys();

for (let key of iterator) {
    console.log(key); // 0, 1, 2
}
```



#####(8).  Array.prototype.values()

​       The keys() method returns a new **Array Iterator** object that contains the keys for each index in the array.

   ```javascript
let arr = [1, 2, 3];
let iterator = arr.values();

for (let values of iterator) {
    console.log(values); // 1, 2, 3
} 
   ```



#####(9).Array.prototype.entries()

​       The entries() method returns a new **Array Iterator** object that contains the key/value pairs for each index in the array.

  ```javascript
let arr = [1, 2, 3];
let iterator = arr.entries();

for (let item of iterator) {
    console.log(item); // [0, 1], [1, 2], [2, 3]
}   
  ```



####TypedArray

#####(1).array buffer

```javascript
let buffer = new ArrayBuffer(10);
console.log(buffer.byteLength); // 10
```



##### (2).TypedArray.prototype.slice()

​     The slice() method returns a shallow copy of a typed array into a new typed array object.

```javascript
let buffer = new ArrayBuffer(10);
let buf = buffer.slice(4, 8);
console.log(buf.byteLength); // 4
```

#####(3).dataview

​     The dataview provides a low-level interface of reading and writing multiple number types in an ArrayBuffer irrespective of the platform's endianness.

```javascript
let buffer = new ArrayBuffer(10);
view = new DataView(Buffer, 5, 2);
```

