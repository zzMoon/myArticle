> 对ES6的部分知识点的简要概述与分析  
[查看其他文章](https://github.com/hangyangws/myArticles#文章列表)

# 简谈Promise

> 写在前面:  
在Promise之前，实现异步可使用回调，也可以自己实现类似Promise的结构，比如jQuery的ajax  
回调是很好的异步解决方案，不过“嵌套多了”就惹得人心烦，且代码难以阅读  
Promises并非解决具体问题的算法，而已代码组织更好的模式

### 一个Promise列子

```javascript
// 注意：为什么要用函数把“new Promise”包起来，因为new Promise时，也会执行函数内部代码
//      所以通常用一个函数把“new Promise”包起来，在函数内部return Promise的实例

function imageLoad(_url) {
    return new Promise(function(resolve, reject) {
        var _image = new Image(); // 新建Image对象
        _image.onload = function() { // 定义Image对象的加载事件
            resolve('加载成功'); // 如果图片加载成功调用resolve方法
        };
        _image.onerror = function() {
            reject(new Error('加载失败'));
        };
        _image.src = _url;
    });
}

imageLoad('这是图片地址') // 执行imageLoad方法，会返回一个Promise实例
    .then( // 注册Promise的状态改变时的回调事件
        _success => console.log(_success), // 加载成功调用的方法
        _error => console.log(_error) // 加载失败时调用的方法（比如当图片地址不存在的时候）
    );

// 仔细观察上面的代码，开发者会发现
// 创建Promise实例时传入函数的第一个参数指向的就是“Promise实例的then方法的第一个参数”
// 同理，
// 创建Promise实例时传入函数的第二个参数指向的就是“Promise实例的then方法的第二个参数”
// 所以，
// resolve函数执行时等同执行then方法第一个函数参数；reject函数执行时等同执行then方法第二个函数参数
```

认真看了上面的典型的简单的Promise例子，开发者应该对Promise不陌生了，至少对then方法不陌生^_^  
下面，进一步揭开Promise的面纱

### Promise实例的状态

每个Promise实例都有一个状态，初始为`Pending`  
resolve方法可以将Pending改为`Resolved`  
reject方法可以将Pending改为`Rejected`  
注意：没有其他方式可以修改Promise实例的状态，且状态不可逆

### Promise原型链方法

> 原型链方法又称`实例方法`

**Promise.prototype.then()**

then方法接受2个函数参数，状态变为*Resolved*调用第一个函数参数，状态变为*Rejected*调用第二个函数参数  
then方法内部必须返回全新的`Promise`对象  
如果then方法内部return的不是一个`Promise`对象，或者没有显示return语句  
那么会自动返回一个全新的`Promise`对象  
所以then后面可以继续调用其他实例方法，实现链式调用

**Promise.prototype.catch()**

catch与then一样，返回值是**新的Promise 对象**
我们知道，then的第二个函数参数，可以看做捕获错误的方法  
我们还知道then可以链式调用  
试想一下，当链式调用多个then方法时，难道要写多个错误处理方法，不会显得臃肿么  
那么，catch方法就是为此而生  
所以，catch方法可以充当then方法的第二个函数参数，并且建议使用catch方法  
请看下面的详细分析

```javascript
new Promise(resolve => {
        resolve(msg); // 抛出错误：msg is not defined
    })
    .then(_data => console.log(_data)) // 不会执行，因为then之前的错误没有捕获
    .catch(_error => console.log(_error)); // 捕获错误： msg is not defined

// 上面的then方法未执行，是因为错误没有被捕获，如果把catch放在then之前（如下代码）

new Promise(resolve => {
        resolve(msg); // 抛出错误：msg is not defined
    })
    .catch(_error => console.log(_error)) // 捕获错误： msg is not defined。如果没有错误，直接跳过catch方法
    .then(_data => console.log(_data)); // 执行，打印：undefined


// catch可以还可以捕获then方法中的错误（如下代码）

new Promise(resolve => {
        resolve('OK');
    })
    .then(_data => {
        var test = unnamed; // 抛出错误：unnamed is not defined
    })
    .catch(_error => console.log(_error)); // 捕获错误： unnamed is not defined

// catch内部依旧可以抛出错误，但是需要另外一个catch来监听了（如下代码）

new Promise(resolve => {
        resolve('OK');
    })
    .then(_data => {
        console.log(_data); // 执行，打印：Ok
        var test = unnamed_one; // 抛出错误：unnamed_one is not defined
    })
    .catch(_error => {
        console.log(_error); // 捕获错误：unnamed_one is not defined
        var test = unnamed_two; // 抛出错误：unnamed is not defined
    })
    .catch(_error => console.log(_error)); // 捕获错误：unnamed is not defined
```

看过上面的代码分析后，至少可以总结出：  
Promise错误具有冒泡性质，错误会不断的向后传递，直到 .catch() 捕获  
如果then方法遇到没有捕获的储物，就不会执行  

还有：catch方法是`then(null, rejection)`的别名（如下代码）

```javascript
Promise.resolve()
    .then(_success => console.log(_success))
    .catch(_error => console.log(_error));
// 等同于
Promise.resolve()
    .then(_success => console.log(_success))
    .then(null, _error => console.log(_error));

// 建议第一种方式，实现更简洁，代码更具语义性
```


### Promise的静态方法

- Promise.resolve()

- Promise.reject()

- Promise.all()

- Promise.race()

> [参考](http://es6.ruanyifeng.com/#docs/promise#Promise-的含义)  
[参考2](http://liubin.org/promises-book/#introduction)  
[参考3](http://coderlt.coding.me/2016/12/03/promise-in-depth-an-introduction-1/#comments)