
## 基本知识

内存泄漏, 一般存在两个层面:
1. 语言, 即 JS 本省,有没有内存泄漏的问题
2. 编码习惯,你在编写代码的时候,是否有意识的去避免这件事情!
3. 
### 内存泄漏

内存泄漏（Memory Leak）是指不再用到的内存，没有及时释放。

当这样的情况越来越多, 内存占用越来越高的时候, 会影响系统性能,甚至系统崩溃.

### 垃圾回收机制

javaScript是有自己的内存回收机制，可以确定那些变量不再需要，并将其清除.


## 常见内存泄漏场景

### 1. 全局变量

在非严格模式下当引用未声明的变量时，会在全局对象中创建一个新变量。在浏览器中，全局对象将是window，这意味着

```js
function foo（arg）{
  bar =“some text”; // bar将泄漏到全局. 
}
```


**原因** 
全局变量是根据定义, 无法被垃圾回收机制收集.
需要特别注意用于临时存储和处理大量信息的全局变量。
如果必须使用全局变量来存储数据，请确保将其指定为null或在完成后重新分配它。 

**解决办法**
严格模式

### 2. 被遗忘的定时器和回调函数

```js
var someResource = getData(); 

var intervalID = setInterval(function() {
  var node = document.getElementById('Node');     
  if(node) {         
    node.innerHTML = JSON.stringify(someResource));        
    // 定时器也没有清除    
  }    
  // node、someResource 存储了大量数据 无法回收 
}, 1000);

clearInterval(intervalID); // 手动清除

```

**原因**:与节点或数据关联的计时器不再需要，node 对象可以删除，整个回调函数也不需要了。可是，计时器回调函数仍然没被回收（计时器停止才会被回收）。同时，someResource 如果存储了大量的数据，也是无法被回收的。

**解决方法**： 在定时器完成工作的时候，手动清除定时器

https://developer.mozilla.org/zh-CN/docs/Web/API/setInterval#example
[`clearInterval()`](https://developer.mozilla.org/zh-CN/docs/Web/API/clearInterval)

#### 3. DOM引用



```js
var refA = document.getElementById('refA'); 
document.body.removeChild(refA); // dom删除了 
console.log(refA, "refA");  // 但是还存在引用 能console出整个div 没有被回收
refA = null;//清空
```


**原因**: 保留了DOM节点的引用,导致GC没有回收

**解决办法**：refA = null;

### 4. 闭包

注意

注意

注意: 闭包本身没有错,不会引起内存泄漏.而是使用错误导致.

arcade

复制代码
```js
var theThing = null; 
var replaceThing = function () {
  var originalThing = theThing;   
  var unused = function () {     
    if (originalThing)       
      console.log("hi");   
  };   
  theThing = {     
    longStr: new Array(1000000).join('*'),     
    someMethod: function () {       
      console.log(someMessage);     
    }   
  };
  originlThing = null; 
}; 
setInterval(replaceThing, 1000);
```

这是一段糟糕的代码,每次调用 replaceThing ，theThing 得到一个包含一个大数组和一个新闭包（someMethod）的新对象。同时，变量 unused 是一个引用 originalThing 的闭包（先前的 replaceThing 又调用了theThing）。思绪混乱了吗？最重要的事情是，闭包的作用域一旦创建，它们有同样的父级作用域，作用域是共享的。someMethod 可以通过 theThing 使用，someMethod 与 unused 分享闭包作用域，尽管 unused 从未使用，它引用的 originalThing 迫使它保留在内存中（防止被回收）。当这段代码反复运行，就会看到内存占用不断上升，垃圾回收器（GC）并无法降低内存占用。本质上，闭包的链表已经创建，每一个闭包作用域携带一个指向大数组的间接的引用，造成严重的内存泄漏。

**解决**: 去除unuserd函数或者在replaceThing函数最后一行加上 originlThing = null.

## 使用 chrome memory profiling tools 来测试
  

## 资源

链接：https://juejin.cn/post/6844903917986267143  
https://auth0.com/blog/four-types-of-leaks-in-your-javascript-code-and-how-to-get-rid-of-them/

https://medium.com/sessionstack-blog/how-javascript-works-memory-management-how-to-handle-4-common-memory-leaks-3f28b94cfbec