## 1.事件模型解释
> ` DOM0级事件模型 `
直接在HTML的标签中加入 onclick=fun() ，没有事件流的概念，事件不会传播

> `IE事件模型`
IE事件模型共有两个过程:
事件处理阶段(target phase)。事件到达目标元素, 触发目标元素的监听函数。
事件冒泡阶段(bubbling phase)。事件从目标元素冒泡到document, 依次检查经过的节点是否绑定了事件监听函数，如果有则执行.

> `DOM2级事件模型`
属于W3C标准模型，现代浏览器(除IE6-8之外的浏览器)都支持该模型。在该事件模型中，一次事件共有三个过程:
事件捕获阶段(capturing phase)。事件从document一直向下传播到目标元素, 依次检查经过的节点是否绑定了事件监听函数，如果有则执行。
事件处理阶段(target phase)。事件到达目标元素, 触发目标元素的监听函数。
事件冒泡阶段(bubbling phase)。事件从目标元素冒泡到document, 依次检查经过的节点是否绑定了事件监听函数，如果有则执行。

## 2.如何使用事件，以及IE和DOM事件模型之间存在的差别
###DOM
事件绑定监听函数的方式如下:
```
addEventListener(eventType, handler, useCapture)
```
事件移除监听函数的方式如下:
```
removeEventListener(eventType, handler, useCapture)
```
### IE9 以下
不支持addEventListener()
使用 attachEvent('onclick',function)  第一个参数与在前面加'on'  ,第二个参数是触发事件执行函数，与addEventListener()不同，没有第三个参数。
detachEvent() 取消事件监听
### 不同 
1. IE只支持冒泡，而Dom支持捕获和冒泡
2. 函数参数问题
![](http://upload-images.jianshu.io/upload_images/3185709-0ef6b91d5648454e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3. 事件处理函数作用域的区别: IE中事件处理程序处于全局作用域，其内的this会指向window;而用DOM（0或2）级事件的事件处理程序的作用域是元素作用域，其内的this指向监听所绑定的元素
    例: document.addEventListener("click", function(){ 
            if(this == document){
              alert("此时this指向document");
            }
          }, false);
4. 事件对象event的属性方法的差别( 获取所点击的元素)
   IE  :event.srcElement         
DOM:   event.target
参考：
[IE和标准DOM事件模型之间存在的差别](https://www.jianshu.com/p/94965b55489a)

## 3.兼容ie的事件封装
```
function delegateEvent(element, tag, event, listener) {
    // 判断是否支持addEventlistener
    if(element.addEventListener){
        // 给父元素添加事件
        element.addEventListener(event,function(e){
            // 获取当前触发的元素
            var target = e.target;
            // 判断当前元素是否是我需要的
            if(target.nodeName.toLowerCase()===tag){
                listener(target);
            }
            })
    }else{
        // 兼容IE
        element.attachEvent("on"+event,function(){
            var target = window.event.srcElement;
            if(target.nodeName.toLowerCase()===tag){
                listener(target);
            }
            })
    }
    
    
}
var ul = document.getElementById("ul");

delegateEvent(ul,"li","mouseover",function(target){
    target.style.backgroundColor = "red";
})
```
## 4.事件冒泡与事件捕获应用:事件代理 
详见：[事件冒泡和事件捕获](https://segmentfault.com/a/1190000005654451)