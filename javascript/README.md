# JS学习总结
### 1.[重绘和重排问题](http://www.cnblogs.com/zichi/p/4720000.html)
### 2.add(2,3)和add(2)(3)问题(再考虑他的拓展性拓展)，其实是考察js函数柯里化问题。
**curry 的概念很简单：只传递给函数一部分参数来调用它，让它返回一个函数去处理剩下的参数。**

```
function add1(a){
  var sum=a
  var tmp=function(b){
      sum= sum+ b;
      return tmp;
  }

  tmp.valueOf=function(){
    return sum;
  }

  return tmp;
}
var result=add1(2)(3)(4)(5)(6);
console.log('%d',result);  //输出14 '%d'会把result转换为10进制，从而触发tmp的valueOf函数
```
因为到最后tmp(6)之后，返回的是个tmp，如果输出tmp
的话，打印的是tmp这个函数对象，如下：
```
console.log(result) :
{ [Function: tmp] valueOf: [Function] }
```
所以我们要把result转换为10进制，console.log('%d',result); 
'%d'会把result转换为10进制,从而触发tmp的valueOf函数，输出sum

**还有一种方法是利用ES6语法中的Proxy，这个方法是我在StackOverflow上看到的**
```
function add(n) {
  sum = n;
  const proxy = new Proxy(function a() {}, {
    get (obj, key) {
      return () => sum;
    },
    apply (receiver, ...args) {
    sum += args[1][0];
    return proxy;
  }
});
  return proxy
}
console.log(add(1)(2)(3));  //6
console.log(add(1)(2)(3)(4)(5)(6));   //21
```
proxy可以理解是对目标对象进行代理，外界对该对象的访问，都必须先通过这层拦截。
ES6 原生提供 Proxy 构造函数，用来生成 Proxy 实例。
```
var proxy = new Proxy(target, handler);
```
Proxy 对象的所有用法，都是上面这种形式，不同的只是handler参数的写法。其中，new Proxy()表示生成一个Proxy实例，target参数表示所要拦截的目标对象，handler参数也是一个对象，用来定制拦截行为。上面的代码就主要用到了proxy的get 和 apply 

具体Proxy的使用可以学习阮一峰的[ECMAScript 6 入门](http://es6.ruanyifeng.com/#docs/proxy)

其实我在看完Proxy和他的用法之后，并不知道proxy有什么优势，只觉得它是另一种方法，而且有点难理解。可能是我还不知道proxy的优势吧。



### 3.ES6中箭头函数和普通函数的区别。
1.this的区别。普通函数如果是作为对象的方法被调用的，则其 this 指向了那个调用它的对象;但是箭头函数的则会捕获其所在上下文的 this 值，作为自己的 this 值。箭头函数需要记着这句话：“箭头函数中没有 this 绑定，必须通过查找作用域链来决定其值，如果箭头函数被非箭头函数包含，则 this 绑定的是最近一层非箭头函数的 this.
```
var obj = {
  i: 10,
  b: () => console.log(this.i, this),
  c: function() {
    console.log( this.i, this)
  }
}
obj.b(); // prints undefined, Window
obj.c(); // prints 10, Object {...}

```
2.箭头函数不绑定arguments,取而代之用rest参数 … 解决 （[rest参数介绍](https://juejin.im/entry/596c7fc15188254b6237a29c) ）
```
function consol() {
  console.log(arguments)
}
consol('haha',1)  // { '0': 'haha', '1': 1 }

var conso = (...args)=>{ console.log(args)}
conso('hahaha',1)   // [ 'hahaha', 1 ]
```
### 4.javascript闭包问题。在网上看了很多关于对于什么是闭包的解释。
看过一个答案，我觉得挺通俗易懂的。
> `闭包` 就是把函数以及其所依赖的所有外部自由变量保存在一起的结合体
> `闭包` 就是一个函数把外部的那些不属于自己的对象也包含（闭合）进来了。

参见：[如何通俗易懂的理解闭包](https://www.zhihu.com/question/34547104/answer/197642727)    
[闭包的牛逼解答](https://www.zhihu.com/question/34547104/answer/59439818)

这两者都很好理解。看一段代码：
```
var foo = function(){
    var name = "exe";
    return function inner(){
        console.log( name );
    }
}
var bar = foo();//这里虽然得到的是函数inner的引用，而不是那一坨代码
bar();//这里开始执行inner函数  "exe"  读取foo函数内部的变量
```
这里面的 name 就是相对于函数 inner( )的外部自由变量。所以inner函数就是一个闭包。

#### 闭包的作用 
它的最大用处有两个，一个是前面提到的可以读取函数内部的变量，另一个就是让这些变量的值始终保持在内存中。（因为闭包引用了上一级的环境，所以导致上一级也释放不了。拿上例来说，inner()函数引用name,而这个name是上一级的变量。这导致inner始终在内存中，而inner的存在依赖于foo，因此foo也始终在内存中，不会在调用结束后，被垃圾回收机制回收。

#### 闭包的原理（为什么能够访问外部的自由变量）
**其实就是作用域链。关于作用域对象及作用域链的讲解：**
推荐一篇博客:  [JavaScript闭包的底层运行机制](http://blog.leapoahead.com/2015/09/15/js-closure/)

最后：
闭包是什么时候被创建的？因为所有JavaScript对象都是闭包，因此，当你定义一个函数的时候，你就定义了一个闭包。
闭包是什么时候被销毁的？当它不被任何其他的对象引用的时候。

### 5.实现数组扁平化处理。
一种方法是用递归的方式。
```
var arr = [1, [2, [3, 4]]];

function flatten(arr) {
    var result = [];
    for (var i = 0, len = arr.length; i < len; i++) {
        if (Array.isArray(arr[i])) {
            result = result.concat(flatten(arr[i]))
        }
        else {
            result.push(arr[i])
        }
    }
    return result;
}


console.log(flatten(arr))
```
另一种是用ES6的扩展运算符 ...
扩展运算符 ... 可以把一个数组转换为参数序列
```
var arr = [1, [2, [3, 4]]];
console.log(...arr)   // ...运算符把数组arr 转换为  1  [2,[3,4]] 两个参数
```
再用 
```
arr = [].contact(...arr)  //  [1,2,[3,4]  
```
这样就去掉了数组的一层 顺着这个方法一直继续下去，就写出了如下代码 
```
function flatten(arr) {
   while (arr.some(item => Array.isArray(item))) { //some函数会对每一个数组元素执行一次箭头函数
    var arr = [].concat(...arr);
  }
   return arr;
}
console.log(flatten(arr))
```
中间可以输出一下函数执行的过程。
```
1 [ 2, [ 3, [ 4, 5 ] ] ]
[ 1, 2, [ 3, [ 4, 5 ] ] ]
1 2 [ 3, [ 4, 5 ] ]
[ 1, 2, 3, [ 4, 5 ] ]
1 2 3 [ 4, 5 ]
[ 1, 2, 3, 4, 5 ]
```
**奇数行是...arr 的执行结果
偶数行是[].concat(...arr); 的执行结果**

### 6.es6和es5中的继承
#### ES5继承
es5的继承实际上是用原型链实现的，将子类的原型设置为父类的实例，从而实现继承
```
function father(){
  this.flag = true;
}
function child(){
  this.childFlag = false;
}
// 将子类的prototype对象指向父类的实例,实现继承
child.prototype = new father();
var child1 = new child();

console.log(child1.__proto__ === child.prototype)  // true
```
![_proto_和protoType的区别.jpg](http://upload-images.jianshu.io/upload_images/3185709-f2e5acd007266898.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这张图好像是我在知乎上看到的。
#### ES6继承
es6继承就很容易理解了，差不多就是java的class
详细学习可以看[阮一峰的ES6入门](http://es6.ruanyifeng.com/#docs/class-extends)
### 7.数组去重
#### indexOf
 ```
var array = [1, 1, '1'];

function unique(array) {
    var res = [];
    for (var i = 0, len = array.length; i < len; i++) {
        var current = array[i];
        if (res.indexOf(current) === -1) {
            res.push(current)
        }
    }
    return res;
}

console.log(unique(array));
```
由于indexOf是一个一个进行比较，效率不高。
可以先对数组进行排序,排序后，相同的值就会被排在一起，然后我们就可以只判断当前元素与上一个元素是否相同，相同就说明重复，不相同就添加进 res。这种方法效率高于使用 indexOf。
### 8.浅拷贝和深拷贝
####浅拷贝
对于数组和对象这种属于引用类型的,如果浅拷贝(例如用 = 直接赋值，或者只遍历第一层 )那么对于复杂的，具有嵌套结构的数据来说，浅拷贝只是拷贝了同一个引用，而这个引用指向存储在堆中的一个对象。拷贝完成后，两个变量实际上将引用同一个对象，如果改变其中一个变量，就会影响另一个变量。
```
// 引用类型值复制
var object1 = {a: 1};
var object2 = object1;
object1.a = 2
console.log(object1)   //  { a: 2 }
console.log(object2)   //  { a: 2 }
```
可以看出来，改变其中一个变量，另一个也会改变
#### 深拷贝
第一种方法：使用 JSON.stringify 和 JSON.parse 方法
```
var arr = ['old', 1, true, ['old1', 'old2'], {old: 1}]
var new_arr = JSON.parse( JSON.stringify(arr));
console.log(new_arr);   //  [ 'old', 1, true, [ 'old1', 'old2' ], { old: 1 } ]
```
但是这种方法有一个问题，不能拷贝函数。

第二种方法。利用递归
```
function deepClone(source) {
  // 递归终止条件
  if (!source || typeof source !== 'object') {
    return source;
  }
  var targetObj = source instanceof Array ? [] : {};
  for (var key in source) {
    if (source.hasOwnProperty(key)){
      if (source[key] && typeof source[key] === 'object') {  //数组 typeof的结果也是object
        targetObj[key] = deepClone(source[key]);
      } else {
        targetObj[key] = source[key];
      }
    }
  }
  return targetObj;
}
var object1 = {'year':12, arr: [1, 2, 3], obj: {key: 'value' }, func: function(){return 1;}};
var object2 = [1,[2,[3,4,[5,6]]]]
var newObj= deepClone(object1);
var newObj2 = deepClone(object2)
console.log(newObj)  // { year: 12,arr: [ 1, 2, 3 ],obj: { key: 'value' }, func: [Function: func] }
console.log(newObj2) // [ 1, [ 2, [ 3, 4, [Array] ] ] ]

```
