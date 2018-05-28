---
layout:       post
title:        "理解vue.js的数据绑定"
subtitle:     "JavaScript, 前端开发, 基础知识, vue.js"
date:         2018-05-28
author:       "Chou"
header-img:   "img/post-bg-vue-data-binding.jpg"
header-mask:  0.3
catalog:      true
multilingual: false
tags:
    - 前端开发
    - JavaScript
    - vue.js
    - 知识总结

---

### MVVM的数据绑定

现在主流的MVVM框架都实现了单向或双向数据绑定。

简单理解就是`视图（View）`的变化能实时反应在数据层`（Model）`，`数据层（Model）`的变化能实时反应在`视图（View）`上。

对于View的变化，我们可以设置一些监听事件，如 ```keyup```，```click```，```change```等。然后在回调函数中更新Model的值。当Model变化时，对Model的监听就比较麻烦了。各大框架的实现主要的就是以下几种：



> backbone.js => 发布者-订阅模式
>
> Angular.js => 脏数据检测
>
> vue.js => 数据劫持结合发布者-订阅模式，下个版本中可能用Proxy代替。



#### 发布者-订阅模式

这个方法是最简单也最有效的。思路就是使用自定义的```data```属性在HTML代码中指明绑定，所有绑定起来的JavaScript对象和DOM元素都将订阅一个发布者对象。任何时候如果JavaScript对象或者一个HTML输入字段被侦测发生了变化，我们将代理事件到发布-订阅者模式，这会反过来将变化广播并传播到所有绑定的元素和对象。

详细参考：[谈谈JavaScript中的双向数据绑定 ](http://www.html-js.com/article/Study-of-twoway-data-binding-JavaScript-talk-about-JavaScript-every-day)



#### 脏数据检测

脏数据检测就是比较UI和后台的数据是否一致。这个比较并不是周期性的，而是当某些特定事件触发时才会进行脏检测，比如：

`#` UI事件（DOM事件）：输入文本，点击按钮等。

`#` Ajax请求。

`#` Timer事件：$timeout，$interval。

详细参考：[https://www.cnblogs.com/likeFlyingFish/p/6183630.html](AngularJS脏检查深入分析)



#### 数据劫持

用数据劫持实现双向绑定有三个方法一个是```vue.js```目前在用的```Object.defineProperty```，一个是ES6中的[```Proxy```](http://es6.ruanyifeng.com/#docs/proxy)，还有一个是已经被废弃的```Object.observe```。





### 简单的数据劫持双向绑定

利用```Object.defineProperty```进行数据劫持的原理就是通过```Object.defineProperty```劫持对象属性的```getter```和```setter```方法，在该属性发生变化时进行操作。

下面是个简单的例子：

```html
<div>
    <input type="text" id="a">
    <span id="b"></span>
</div>
```



```javascript
const obj = {};
// 将obj的text属性与视图的元素绑定
Object.defineProperty(obj, 'text', {
    set: function(newVal) {
        document.getElementById('a').value = newVal;
        document.getElementById('b').innerHTML = newVal;
    }
});

// 监听到View的变化，实时更新到Model
document.addEventListener('keyup', function(e) {
    obj.text = e.target.value;
})
```



上面的代码就简单地实现了双向数据绑定的功能。

在文本框输入内容，旁边的span会同时显示相同的内容。在控制台改变```obj.text```的值，也会更新到视图上。



![simple-data-binding](/img/in-post/vue-data-binding/simple-data-binding.png)





### 理解vue.js的双向绑定



我理解的vue.js实现双向数据绑定的思路如下：

`#` 定义监听器```observer```，利用```Object.defineProperty```对数据对象进行递归遍历，这样所有的数据发生变化时都能出发```setter```而被监听。将监听到的变化通知给订阅者```Watcher```。

`#` 实现```Watcher```。当收到属性变动通知时，调用```update()```方法，并触发```compile```中的回调函数。

`#` 定义```compile```函数解析模板指令，将模板中的变量换成数据，渲染视图。并将每个指令对应的节点绑定更新函数，添加监听数据的订阅者```Watcher```，一旦数据有变动，收到通知，更新视图 。



![vuejs-data-binding-sample-graph](/img/in-post/vue-data-binding/vuejs-data-binding-sample-graph.png) 



#### 实现```Observer```

监听器```Observer```递归遍历所有属性，通过```Object.defineProperty```进行数据劫持。



```javascript
function Observe(data) {
    if (!data || typeof data != 'object') {
        return;
    }
	// 遍历所有属性
	Object.keys(data).forEach(function(key) {
        defineReactive(data, key, data[key]);
    });
};

function defineReactive(data, key, val) {
	observe(val); // 监听子属性
	Object.defineProperty(data, key, {
		enumberable: true,
		configurable: false,
		get: function() {
			return val;
		},
		set: function(newVal) {
            if (val === newVal) return;
            console.log("监听到数据发生变化", val, "->", newVal);
		}
	})
}

const data = {
	name: 'John'
};
observe(data);

data.name = 'Bob'; // 监听到数据发生变化 John -> Bob
```



监听到数据的变化，就要通知给订阅者了。因为订阅者可能会有多个，所以要定义一个```消息订阅器Dep```来收集订阅者，然后在监听器```Observer```和```Watcher```之间进行进行统一管理。数据发生变化触发```notify```通知订阅者，调用订阅者的```update```方法。

更改一下```defineReactive```函数：

```javascript
function defineReactive(data, key, val) {
    // 实例一个消息订阅器
    var dep = new Dep();
    observe(val); // 监听子属性
    Object.defineProperty(data, key, {
		enumberable: true,
		configurable: false,
		get: function() {
			return val;
		},
		set: function(newVal) {
            if (val === newVal) return;
            console.log("监听到数据发生变化", val, "->", newVal);
            dep.notify(); // 把变化通知给所有订阅者
		}
	})
}

function Dep() {
    this.subs = []; // 定义消息订阅器数组
}
// 定义订阅器方法
Dep.prototype = {
    addSub: function(sub) {
        this.subs.push(sub);
    },
    notify: function() {
        this.subs.forEach(function(sub) {
            sub.update();  // 订阅器收到通知后，订阅者执行update方法
        });
    }
};
```

由于```Watcher```就是订阅者，要想将其添加到```dep```中，就必须在闭包内操作。

```javascript
// Observer.js
// ...省略
Object.defineProperty(data, key, {
    get: function() {
        // 由于需要在闭包内添加watcher，所以通过Dep定义一个全局target属性，暂存watcher, 添加完移除
        Dep.target && dep.addDep(Dep.target);
        return val;
    }
    // ... 省略
});

// Watcher.js
Watcher.prototype = {
    get: function(key) {
        Dep.target = this;
        this.value = data[key];    // 这里会触发属性的getter，从而添加订阅者
        Dep.target = null;
    }
}
```

这样数据监听和通知订阅者功能就完备了。



#### 实现```Compile```



`Compile`做两个事情。

`#` 解析模板中的指令，将模板中绑定的变量转化为数据，显示在页面上，也就是初始化渲染视图。

`#` 将对应的DOM节点绑定更新函数，添加监听数据的订阅者，一旦数据变化，就会更新视图。



解析指令之前要获取所有的DOM节点，对绑定指令的节点进行解析。考虑到频繁操作DOM节点产生的性能问题，先将要解析的节点放入文档碎片`fragment`中，对`fragment`进行操作，之后再放回DOM节点中。



```javascript
function Compile(el) {
    this.$el = this.isElementNode(el) ? el : document.querySelector(el);
    if (this.$el) {
        this.$fragment = node2Fragment(this.$el); // 新建fragemnt对象
        this.init();
        this.$el.appendChild(this.$fragment);
    }
}

Compile.prototype = {
    // compileElement编译fragment节点
	init: function() { this.compileElement($this.fragment); },
	node2Fragment: function(el) {
		var fragment = document.createDocumentFragment(),
		    child;
		// 把DOM节点拷贝到fragment
		while (child = el.firstChild) {
			fragment.appendChild(child);
		}
		return fragment;
	}
};
```



`compileElement`函数遍历所有的`fragment`节点，对所有的节点进行判断，如果是匹配到带有`{{ }}`这种节点就进行编译：

`#` 调用渲染函数进行视图渲染；

`#` 绑定更新函数，添加监听数据的订阅者。



```javascript
Compile.prototype = {
	//...
	compileElement: function(el) {
		var childNodes = el.childNodes,
		    me = this;
		[].slice.call(childNodes).forEach(function(node) {
			var text = node.textContent;
			var reg = /\{\{(.*)\}\}/; // 指令的正则表达式
			//
			if (me.isElementNode(node)) {
				me.compile(node);
			} else if (me.isTextNode(node) && reg.test(text)) {
				me.compileText(node, RegExp.$1):
			}
			// 递归遍历编译子节点
			if (node.childNodes && node.childNodes.length) {
				me.compileElement(node);
			}
		});
	},
	compile: function(node) {
		var nodeAttrs = node.attributes,
		var me = this;
		
		[].slice.call(nodeAttrs).forEach(function(attr) {
		    // 取到节点绑定的属性名，v-text
		    // 如<span v-text="content"></span>
		    var attrName = attr.name; // v-text
		    if (me.isDirective(attrName)) {
		    	var exp = attr.value; // content
		    	var dir = attrName.substring(2); // text
		    	if (me.isEventDirective(dir)) {
		    		// 处理事件指令
		    		compileUtil.eventHandler(node, me.$vm, exp, dir);
		    	} else {
		    		// 处理普通指令
		    		compileUtil[dir] && compileUtil[dir](node, me.$vm, exp);
		    	}
		    }
		});
	}
};
// 指令集合
var compileUtil = {
	text: function(node, vm, exp) {
		this.bind(node, vm, exp, 'text');
	},
	//...
	bind: function() {
		var updaterFn = updater[dir + 'Updater'];
		// 第一次初始视图
		updaterFn && updaterFn(node, vm[exp]);
		// 实例化订阅者
		new Watcher(vm, exp, function(value, oldValue) {
			// 一旦属性值变化，会收到通知，执行此函数更新视图
			updaterFn && updaterFn(node, value, oldValue);
		});
	}
};
```



#### 实现```Watcher```



`#` 初始化时作为订阅者将自己添加到订阅器`Dep`中。因为添加订阅者的操作是在`Observer`的`get`函数中执行的，所以只要在初始化`Watcher`时触发对应的`get`函数即可。

`#` 定义自身的`update`方法，当收到属性变动通知时，调用```update()```方法，并触发```Compile```中的回调函数。



```javascript
function Watcher(vm, cb, exp) {
	this.vm = vm;
	this.cb = cb;
	this.exp = exp;
	// 触发属性的getter，从而在Dep中添加自己
	this.value = this.get();
}
Watcher.prototype = {
	update: function() {
		this.run(); // 收到属性值变化的通知，执行run函数
	},
	run: function() {
		var value = this.get(); // 拿到最新值
		var oldValue = this.value;
		if (value !== oldValue) {
			this.value = value;
			this.cb.call(this.vm, value, oldValue); // 执行Compile中绑定的回调函数，更新视图
		}
	},
	get: function() {
		Dep.target = this; // 把当前的订阅者指向自己
		var value = this.vm[exp]; // 触发getter，把自己添加到属性订阅器中
		Dep.target = null;
		return value;
	}
};
```



实例化`Watcher`时，通过`get`函数触发`getter`方法，将当前`Watcher`实例添加到订阅器`Dep`中。这样当属性变化时就能收到通知了。



#### 实现`MVVM`构造器

MVVM作为数据绑定的入口，将`Observer`，`Compile`，`Watcher`整合在一起，实现数据的双向绑定。



```javascript
function MVVM (options) {
    this.$options = options;
    var data = this._data = this.&options.data;
    observe(data, this);
    this.$compile = new Compile(options.el || document.body, this);
}
```



但是上面监听的时`options.data`，每次改变数据都要像这样：

```javascript
var vm = new MVVM({data: {name: 'Foo'}}); vm._data.name = 'Bar' 
```

我们的预想是直接通过```vm.name = 'Bar'```来调用。

所以需要给`MVVM`添加一个属性代理的方法，使访问`vm.name`代理为访问`vm._data.name`

添加代理之后的`MVVM`构造器如下：

```javascript
function MVVM (options) {
    this.$options = options;
    var data = this._data = this.$options.data;
    var me = this;
    // 实现属性代理：vm.xxx = vm._data.xxx
    Object.keys(data).forEach(function(key) {
        me._proxy(key);
    });
    observe(data, this);
    this.$compile = new Compile(options.el || document.body, this)
}

MVVM.prototype = {
    _proxy: function(key) {
        var me = this;
        // 用defineProperty劫持vm实例的属性。
        Object.defineProperty(me, key, {
            configurable: false,
            enumberable: true,
            get: function proxyGetter() {
                return me._data[key];
            },
            set: function proxySetter(newVal) {
                me._data[key] = newVal;
            }
        });
    }
};
```







