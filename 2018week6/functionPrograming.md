#### 函数式编程（FP）

一种编程范式， 类似于面向对象编程和面向过程编程。

一 基本的函数式编程

- 基本的函数式编程
- 纯函数（Pure functions）
- 函数复合（Function composition）
- 避免共享状态（Avoid shared state）
- 函数柯里化
- 高阶函数
- 对比声明式与命令式
- 避免改变状态（Avoid mutating state）
- 避免副作用（Avoid side effects）

二 应用 
- 高阶函数的应用
- 函数式编程程的应用



##### 一 基本的函数式编程
```
// 数组中每个单词，首字母大写


// 一般写法
const arr = ['apple', 'pen', 'apple-pen'];
for(const i in arr){
  const c = arr[i][0];
  arr[i] = c.toUpperCase() + arr[i].slice(1);
}

console.log(arr);


// 函数式写法一
function upperFirst(word) {
  return word[0].toUpperCase() + word.slice(1);
}

function wordToUpperCase(arr) {
  return arr.map(upperFirst);
}

console.log(wordToUpperCase(['apple', 'pen', 'apple-pen']));



// 函数式写法二
console.log(arr.map(['apple', 'pen', 'apple-pen'], word => word[0].toUpperCase() + word.slice(1)));
```
优点：
- 函数分装组合调用 表意清晰 易于扩展
- 利用高阶函数`map`,减小中间变量

==>>>>尽量使函数符合纯函数的标准

#####  纯函数

两个特性：
- 没有任何副作用。 函数不会更改函数以外的任何变量或任何类型的数据。
- 具有一致性。 在提供同一组输入数据的情况下，它将始终返回相同的输出值。

不依赖外界变量
```
var a = 5;
function A(b) {
  return a + b;  //读取全局变量,a很容易被改变
}
A(5);  

const a = 5;
function A(b) {
  return a + b;   //依赖外部变量,不纯
}
A(5);
```

不产生副作用
```
const a = 1
const foo = (obj, b) => {
  obj.x = 2    // 对外部counter产生了影响
  return obj.x + b
}
const counter = { x: 1 }
foo(counter, 2) // => 4
counter.x // => 2
```

######  避免共享状态
共享状态 的意思是任意变量、对象或者内存空间存在于共享作用域下，或者作为对象的属性在各个作用域之间被传递。
在OOP中，对象以添加属性到其他对象上的方式在作用域之间共享。

FP 依赖于不可变数据结构和纯粹的计算过程来从已存在的数据中出新的数据。
```
const x = {
  val: 2
};
const x1 = () => x.val += 1;
const x2 = () => x.val *= 2;
x1(); // -> 3
x2(); // -> 6
```
下面的代码和上面的一样，除了函数的调用顺序
```
const x = {
  val: 2
};
const x1 = () => x.val += 1;
const x2 = () => x.val *= 2;
x2(); // -> 4
x1(); // -> 5
```
如果避免共享状态，就不会改变函数内容，或者改变函数调用的时序不会波及和破坏程序的其他部分：
```
const x = {
  val: 2
};
const x1 = x => Object.assign({}, x, { val: x.val + 1});
const x2 = x => Object.assign({}, x, { val: x.val * 2});

x1(x); // -> 3
x2(x); // -> 4

/**
x2(x); // -> 4
x1(x); // -> 3
*/
```
在上面的例子里，我们使用了 Object.assign() 并传入一个空的 object 作为第一个参数来拷贝 x 的属性，以防止 x 在函数内部被改变。(浅深拷贝)



我们如果不了解函数使用或操作的每个变量的完整历史，就不可能完全理解它做了什么。  （纯函数）


##### 函数柯里化
> 把接受多个参数的函数变换成接受一个单一参数(最初函数的第一个参数)的函数，并且返回接受余下参数且返回结果的新函数

**概念摘要**
即： 传入一个（或很少量的）参数调用父函数，父函数返回一个可接受多个参数的子函数。
```
const add = (x) => {
  return (y, z) => {
    return x + y + z
  }
}

let increase = add(1);
console.log(increase(2, 3)); // 6
```
函数式编程+柯里化，将提取成柯里化的函数部分配置好之后，可作为参数传入，简化操作流程。
```
// 给list中每个元素先加1，再加5，再减1
let list = [1, 2, 3, 4, 5];

//正常做法
let list1 = list.map((value) => {
  return value + 1;
});
let list2 = list1.map((value) => {
  return value + 5;
});
let list3 = list2.map((value) => {
  return value - 1;
});
console.log(list3); // [6, 7, 8, 9, 10]

// 柯里化
const changeList = (num) => {
  return (data) => {
    return data + num
  }
};
let list1 = list.map(changeList(1)).map(changeList(5)).map(changeList(-1));
console.log(list1); // [6, 7, 8, 9, 10]
```
#####  高阶函数
> 接受或者返回一个函数的函数称为高阶函数

许多原生的高阶函数，例如 Array.map , Array.reduce , Array.filter
以`map`为例
```
// 数组中每一项加一，组成一个新数组
// 一般写法
const arr = [1,2,3];
const rs = [];
for(const n of arr){
  rs.push(++n);
}
console.log(rs)


// map改写
const arr = [1,2,3];
const rs = arr.map(n => ++n);
```

易读


#####  对比声明式与命令式

函数式编程关注的是：describe what to do, rather than how to do it. 于是，我们把以前的过程式的编程范式叫做 [Imperative Programming](http://en.wikipedia.org/wiki/Imperative_programming) – 指令式编程，而把函数式的这种范式叫做 [Declarative Programming](http://en.wikipedia.org/wiki/Declarative_programming) – 声明式编程。

- 命令式 代码中频繁使用语句。

- 声明式 代码更多依赖表达式。

######  总结：函数式编程的几个要点
- 把函数当成变量来用，**关注于描述问题而不是怎么实现**
- 函数之间没有共享的变量
- 函数间通过参数和返回值来传递数据
- 在函数里没有临时变量


函数式编程的准则：不依赖于外部的数据，而且也不改变外部数据的值，而是返回一个新的值给你。


#####  二 函数式编程的应用
FP 或 OOP 混用 并非一定要 OOP

#####  函数节流(高阶函数的使用)
函数频繁调用的场景归纳：
- window.onresize事件
当调整浏览器窗口大小时，这个事件会被频繁出发，而在这个时间函数中的dom操纵也会很频繁，这样就会造成浏览器卡顿现象。
- mousemove事件
当被绑定该事件的dom对象被拖动时，该事件会被频繁触发。

所以，函数节流的原理就是在不影响使用效果的情况下降低函数的触发频率。
```
var throttle = function ( fn, interval ) { 
    var __self = fn,  // 保存需要被延迟执行的函数引用
        timer,        // 定时器
    firstTime = true; // 是否是第一次调用 
 
    return function () {
        var args = arguments,
            __me = this;
        // 如果是第一次调用，不需延迟执行
        if ( firstTime ) {
            __self.apply(__me, args);
            return firstTime = false;
        } 
         // 如果定时器还在，说明前一次延迟执行还没有完成
        if ( timer ) {
            return false;
        } 
         // 延迟一段时间执行
        timer = setTimeout(function () {
            clearTimeout(timer);
            timer = null;
            __self.apply(__me, args); 
        }, interval || 500 ); 
    }; 
}; 
 
window.onresize = throttle(function(){
    console.log(1);
}, 500 ); 
```
**分时函数**
防止批量添加dom元素时出现浏览器卡顿或假死的情况
```
// 创建一个数组，用来存储添加到dom的数据
var dataList = [];
// 模拟生成500个数据
for (var i = 1; i <= 500; i++) {
    dataList.push(i);
}
// 渲染数据
var renderData = timeShareRender(dataList, function(data) {
    var oDiv = document.createElement('div');
    oDiv.innerHTML = data;
    document.body.appendChild(oDiv);
}, 6);
// 分时间段将数据渲染到页面
function timeShareRender(data, fn, num) {
    var cur, timer;
    var renderData = function() {
        for(var i = 0; i < Math.min(count, data.length); i++) {
            cur = data.shift();
            fn(cur)
        }
    };

    return function() {
        timer = setInterval(function(){
            if(data.length === 0) {
                return clearInterval(timer)
            }
            renderData()
        }, 200);
    }
}
// 将数据渲染到页面
renderData();
```
[demo演示](https://jsfiddle.net/sanlv/npsnnfdj/)

####  结论
函数式编程偏好：
- 使用表达式替代语句
- 让可变数据成为不可变的
- 用函数复合替代命令控制流
- 使用声明式而不是命令式代码
- 使用纯函数而不是使用共享状态和副作用
- 使用高阶函数来操作许多数据类型，创建通用、可复用功能取代只是操作集中的数据的方法


##### reference
- https://coolshell.cn/articles/10822.html
- https://github.com/ecmadao/Coding-Guide/blob/master/Notes/JavaScript/JavaScript%E5%87%BD%E6%95%B0%E5%BC%8F%E7%BC%96%E7%A8%8B.md#%E9%AB%98%E9%98%B6%E5%87%BD%E6%95%B0
- http://taobaofed.org/blog/2017/03/16/javascript-functional-programing/
- [JavaScript 中的 Currying(柯里化) 和 Partial Application(偏函数应用)](http://www.css88.com/archives/7781)
- [一步一步教你 JavaScript 函数式编程（第一部分）](http://www.css88.com/archives/7794)
- [一步一步教你 JavaScript 函数式编程（第二部分）](http://www.css88.com/archives/7822)
- [一步一步教你 JavaScript 函数式编程（第三部分）](http://www.css88.com/archives/7829)









