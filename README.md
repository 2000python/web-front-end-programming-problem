---
noteId: "597a90205bdc11ec8096653657d5f244"
tags: []
---

#  前端手写题

必须自己动手写，不然啥也记不住。

## 1.类型判断

###  1.1 各种方法的优缺点

####  1.1.1 typeOf方法

```js
console.log(typeof 2000);            // number
console.log(typeof true);            // boolean
console.log(typeof 'string');        // string
console.log(typeof undefined);       // undefined
console.log(typeof null);            // object
console.log(typeof []);              // object 
console.log(typeof {});              // object
console.log(typeof function(){});    // function
```

> 优点：可以快速判断基本数据类型，`null`除外。
>
> 缺点：不太擅长判断引用类型，除了`function`。

####  1.1.2 instanceof方法

这个方法会遍历左侧对象的原型链判断右侧的构造函数是否在其上面出现过。

```js
console.log(2000 instanceof Number);                 // false
console.log(true instanceof Boolean);                // false 
console.log('string' instanceof String);             // false  
console.log([] instanceof Array);                    // true
console.log(function(){} instanceof Function);       // true
console.log({} instanceof Object);                   // true
```

> 优点：可以判断一部分引用类型，适合判断自定义类的实例对象。
>
> 缺点：不能判断基本类型，基本类型必须装箱才能判断。

####  1.1.3 Object.toString方法

这个方法返回对象或装箱对象的`[[Class]]`的属性值。

```js
const toString = Object.prototype.toString;

console.log(toString.call(NaN));                    //[object Number]
console.log(toString.call(document));               //[object HTMLDocument]
console.log(toString.call(new Set()));              //[object Set]
console.log(toString.call(new RegExp()));           //[object RegExp]
console.log(toString.call(new Date()));             //[object Date]
console.log(toString.call(2000));                   //[object Number]
console.log(toString.call(true));                   //[object Boolean]
console.log(toString.call('我是一个字符串。'));        //[object String]
console.log(toString.call([]));                     //[object Array]
console.log(toString.call(function(){}));           //[object Function]
console.log(toString.call({}));                     //[object Object]
console.log(toString.call(undefined));              //[object Undefined]
console.log(toString.call(null));                   //[object Null]
```

> 优点：可以精确的判断类型，无论是引用类型还是基本类型。
>
> 缺点：写法比较繁琐，需要用`Call`方法来调用，所以需要对他进行封装。

###  1.2具体实现

综合上述几种方法，我们可以自己封装一个类型判断函数。

> 基本类型(除去`null`)+`function`：可以用`typeof`方法，因为这个方法速度快。
>
> `null`和`promise`：单独判断，单独返回。
>
> 其他引用类型：使用toString判断。
>
> 提取类型有好多方法，但类型判断的原理都一样。

**代码**

```js
//判断promise
const isPromise = (value) => (typeof value.then === 'function' && typeof value.catch === 'function')
//提取类型
const extractReference = (value) => (
  /(?<=\[object\s).*(?=\])/.exec(Object.prototype.toString.call(value))[0].toLowerCase()
)
//另外一种提取方法
/**
const extractReference = (value) => (
  Object.prototype.toString.call(value).replace('[object ','').replace(']','').toLowerCase()
)
**/
function type(value){
  if(value == null)return String(value);
  if(isPromise(value)){
    return 'promise';
  }
  return typeof value === 'object' ? extractReference(value) || 'object' :  typeof value;
}

//test
console.log(type(2000))                                 //number
console.log(type('string'))                             //string
console.log(type(null))                                 //null
console.log(type(undefined))                            //undefined
console.log(type(NaN))                                  //number
console.log(type(document))                             //htmldocument
console.log(type(new Set()))                            //set
console.log(type(new Map()))                            //map
console.log(type(function(){}))                         //function
console.log(type(()=>{}))                               //function
console.log(type(new Promise(()=>{})))                  //promise
```

##  2.new

根据new的过程，一步一步写出代码，逻辑挺清晰的。

* 先创建一个空对象
* 将这个对象的`__proto__`指向构造函数的`prototype`
* 运行构造函数的方法，改变`this`的指向
* 如果构造函数返回一个对象则返回这个对象，否则返回新对象 

**代码**

```js
function myNew(fun,...args){
  //第一步：先创建一个空对象
  let newObj = {};
  //第二步：将这个对象的__proto__指向构造函数的prototype
  newObj.__proto__ = fun.prototype;
  //第三步：调用构造函数，改变this的指向
  let res = fun.apply(newObj,args);
  //第四步：如果构造函数返回一个对象则返回这个对象，否则返回新对象
  return typeof res === 'object' ? res : newObj; 
}
```

##  3.call/apply/bind

先分清他们的异同点，可以更好的理思路。

利用了`this`的隐式绑定，`this`记录函数调用的上下文对象，

### 3.1 call的实现

* call的参数有两部分：`this`要指向的新对象和其他参数
* 将上下文对象变成传入对象的一个属性
* 执行这个属性并删除
* 返回执行结果

```js
function myCall(context = window, ...args){
  //判断上下文对象是否为函数
  if(typeof this !== 'function'){
    throw new TypeError('error')
  }
  let key = Symbol('key');
  //将上下文对象赋给传入对象的一个属性
  context[key] = this;
  //这里使用隐式绑定，将函数的this绑定为传入对象
  let res = context[key](...args);
  //删除属性
  delete context[key];
  
  return res;
}
```

### 3.2 apply的实现

apply与call的不同在于第二个传入参数。

apply的第二个参数是一个数组，所以apply与call的区别就是调用`context[key]`函数时传入参数不同，其他都一样。

```js
function myApply(context = window, ...args){
  //判断上下文对象是否为函数
  if(typeof this !== 'function'){
    throw new TypeError('error')
  }
  let key = Symbol('key');
  //将上下文对象赋给传入对象的一个属性
  context[key] = this;
  //这里使用隐式绑定，将函数的this绑定为传入对象，传入参数与call不同
  let res = context[key](args);
  //删除属性
  delete context[key];
  
  return res;
}
```

### 3.3 bind的实现

bind的实现稍微比前面两位复杂一点，因为他要返回绑定了`this`的函数。

返回的函数可能会用new，也可以普通调用，无论怎么调用，函数的`this`的指向不会变，所以bind绑定也被称为硬绑定。

> 可以用bind实现函数柯里化

```js
function myBind(context,...firstargs){
  //判断上下文对象是否为函数
  if(typeof this !== 'function'){
    throw new TypeError('error')
  }
  //保存上下文
  let self = this;
  return function func(...secondargs){
    //这里如果函数被new，则将this重新指向self
    if(self instanceof func){
      return new self(...firstargs,...secondargs);
    }
    //执行上下文函数，并改变this指向
    return self.apply(context,...firstargs,...secondargs);
  }
}
```

## 4.函数柯里化

**函数柯里化**：将使用多个参数的一个函数转换成一系列使用一个参数的函数。

#### 4.1 普通实现

先将传入的参数拼接进`args list`

判断现在`args list`的`lenght`是否满足传入函数所需的参数长度

如果小于则返回函数，等待后续的传入

如果大于等于则将`args list`传入函数，直接执行，返回执行结果

```js
function curry(func,...args){
  const length = func.length;
  const argList = args ? [...args]:[];
  return function newFun(...lastargs){
    let newArglist = [...argList,...lastargs];
    if(newArglist.length < length){
      return curry(func,...newArglist);
    }else{
      //test-statement
      //console.log(func.apply(null,newArglist));
      return func.apply(null,newArglist)
    }
  }
}

//test
let ss = (a,b,c) => a + b + c;
curry(ss,1,2)(3);                    //6
curry(ss,1,2,3);                     //6
curry(ss)(1,2,3);                    //6
curry(ss)(1)(2)(3);                  //6
curry(ss)(1,2)(3);                   //6
let ma = curry(ss)(1,2);
ma(3,4);                             //6
ma(4,3);                             //7
```

#### 4.2 ES6实现

代码（**使用bind实现**）

bind函数有天然的拼接参数的功能。

```js
function curry(func,...args){
  return func.length <= args.length ? func(...args):curry.bind(null,func,...args);
}

//test
let ss = (a,b,c) => a + b + c;    
console.log(curry(ss,1,2)(3));
console.log(curry(ss,1,2,3));
console.log(curry(ss)(1,2,3));
console.log(curry(ss)(1)(2)(3));
console.log(curry(ss)(1,2)(3));                   
let ma = curry(ss)(1,2);
ma(3,4); 
console.log(ma(3,4)); 
ma(4,3); 
console.log(ma(4,3));                            
```

代码（**使用箭头函数**）

```js
//第一种实现
//不支持`curry(ss,1,2,3)`这样的调用,由于实现的原因必须再一次调用，如`curry(ss,1,2,3)()`
const curry = (func,...arr) =>(...args)=>(
  arg => arg.length < func.length ? curry(func,...arg):func(...arg)
)([...arr,...args])

//第二种实现
const curry = (fn, arr=[]) => (...args) => (
  arg => arg.length === fn.length
    ? fn(...arg)
    : curry(fn, arg)
)([...arr, ...args])

//test
let ss = (a,b,c) => a + b + c;
let sss = curry(ss,[1])
sss(2)(3)
console.log(sss(2)(3));
console.log(curry(ss,1,2)(3));
console.log(curry(ss,1,2,3));
console.log(curry(ss,1,2,3)());
console.log(curry(ss)(1,2,3));
console.log(curry(ss)(1)(2)(3));
console.log(curry(ss)(1,2)(3));                   
let ma = curry(ss)(1,2);
ma(3,4); 
console.log(ma(3,4)); 
ma(4,3); 
console.log(ma(4,3)); 
```

> 第二种实现函数柯里化时传入的第二个参数是一个数组
>
> ```js
> let sss = curry(ss,[1])
> sss(1,2)
> console.log(sss(1,2)(3));
> ```

## 5. instanceof的实现

这个函数和new的思路一样，都是明确过程，然后循着过程写代码。

> 思路：遍历实例的原型链与类的原型比较

> 过程：
>
> * 第一步：获取实例的原型链和类的原型
> * 第二步：遍历实例的原型链与类的原型比较

代码

```js
function myInstanceof(left,right){
  // 获取实例的原型链的开始
  let leftProto = Object.getPrototypeOf(left);
  // 获取类的原型
  let rightProto = Object.getPrototypeOf(right);
  // 遍历实例的原型链
  while(true){
    //原型链的尽头
    if(leftProto === null) return false;
    if(leftProto === rightProto) return true;
    leftProto = Object.getPrototypeOf(leftProto)
  }
}

//test

```

## 6.防抖和节流

> 防抖：多次执行变为最后一次执行，
>
> 节流：多次执行变为每隔一段时间执行

### 6.1 防抖动

```js
const antiShake = (func,wait = 50) => {
  let clock = 0;
  return function (...args){
    //如果设置了计时器，则清空计时器
    if(clock) clearTimeout(clock);
    //设置计时器
    clock = setTimeout(()=>{func.apply(this,args)},wait)
  }
}
```

> 场景：凡是需要防止多次运行影响性能的场景，如监听页面的滚动的高度

### 6.2 节流

设置一个单位时间，函数在这个时间段里只能执行一次

```js
const throttle = (func,timeQuantum = 50) => {
  //上次执行时间
  let lastTime = 0;
  return function (...args){
    //这次执行时间
    let nowTime =  + new Date();
    if(nowTime - lastTime > timeQuantum){
      lastTime = nowTime
      func.apply(this,args)
    }
  }
}

//test
setInterval(
  throttle(() => {
    console.log('大难临头各自飞')
  }, 500),
  1
)
```

> 场景：
>
> - 拖拽场景：固定时间内只执行一次，防止超高频次触发位置变动
> - 缩放场景：监控浏览器`resize`
> - 动画场景：避免短时间内多次触发动画引起性能问题

## 7. 拷贝

***前置知识***：

> * `Object.create(proto)`方法创建一个新对象，将这个对象的`__proto__`指向`proto`，可以用它实现继承。
> * 前面的类型判断函数在深拷贝时，会有应用。

### 7.1 浅拷贝

> `Object.assign()`可以实现对象的浅拷贝

自己动手实现：

***代码：***

```js
const shallowCopy = (obj) => {
  if(typeof obj !== 'object'|| obj === null){
    return obj
  }
  let copyRes = Array.isArray(obj) ? [] : null;
 for(let item in obj){
   if(obj.hasOwnProperty(key)){
     //简易版深拷贝这里会有一个递归过程
     copyRes[item] = obj[item]
   }
 }
}
```

### 7.2 深拷贝（重点）

#### 简简易版

```js
let deepCopy = JSON.parse(JSON.stringify(obj));
```

> `JSON.stringify()`方法先将原对象JSON序列化，`JSON.parse()`又将JSON转换为JavaScript的值或对象。
>
> **这种方法需要保证对象是JSON安全的，所以只适用于部分情况。**

**问题：**

> - 他无法实现对函数 、RegExp等特殊对象的克隆
>
> ```js
> let func = () => {return 3};
> let reg = /^.$/g;
> let date = new Date();
> 
> let copy = JSON.parse(JSON.stringify(func))  //会报错
> let copy2 = JSON.parse(JSON.stringify(reg))  //{}
> let copy3 = JSON.parse(JSON.stringify(date)) //当前时间
> 
> console.log('1',copy,'2',copy2,'3',copy3);
> ```
>
> - 会抛弃对象的constructor,所有的构造函数会指向Object
> - 对象有循环引用,会报错
>
> ```js
> let a ={'ai':12};
> a.trg = a;
> console.log(JSON.parse(JSON.stringify(a)));   //Uncaught TypeError
> ```

#### 面试版

上述简简易版其实对于大部分情况都是可以直接使用的

不过我们还是要写一个通用的API，解决上面的所有问题。

**原始实现：**在浅拷贝的基础上加上递归

```js
const deepCopy = (obj) => {
  //判断传入对象
  if(typeof obj !== 'object'|| obj === null){
    return obj
  }
  let copyRes = Array.isArray(obj) ? [] : {};
  for(let item in obj){
    if(obj.hasOwnProperty(item)){
      //递归过程
      copyRes[item] = deepCopy(obj[item])
    }
  }
  return copyRes
}

//test
let func = () => {return 3};
function method(a = 2,b = 4){
  return a + b
} 
let reg = /^.$/g;
console.log(deepCopy(func));                     // () => {return 3}
console.log(deepCopy(func)());                   // 3
console.log(deepCopy(method));                   // ƒ method(a = 2,b = 4){ return a + b }
console.log(deepCopy(method)());                 // 6
console.log(deepCopy(reg));                      // {}
console.log(deepCopy(new Date()));               // {}
//循环引用
let a ={ zxz: 12};
a.trg = a;
console.log(deepCopy(a));                        // 栈溢出
```

##### **问题**：

> 函数可以复制，但是其他的特殊对象还是无法复制。

> 循环引用没有解决：因为上述方法只是一个简单的递归，并没有判断是否已经被拷贝过了。

###### 1.解决循环引用

> 我们可以用Map的键是唯一的这个性质，来判断是否已经被拷贝过，如果是就直接返回

```js
const deepCopy = (obj,map = new Map()) => {
  //判断是否已经被拷贝过，如果是就直接返回
  if(map.get(obj)) return obj;
  //判断传入对象
  if(typeof obj !== 'object'|| obj === null){
    return obj
  }else{
    map.set(obj, true);
    let copyRes = Array.isArray(obj) ? [] : {};
    for(let item in obj){
        if(obj.hasOwnProperty(item)){
            //递归过程
            copyRes[item] = deepCopy(obj[item],map)
        }
    }
    return copyRes
  }
}

//test
let a ={zxz: 12};
a.trg = a;
console.log(deepCopy(a));                        // {zxz: 12, trg: {…}}
a = null;
console.log(a);                                  // 之后并不会被回收。
```

***注意：***

> map 上的 key 和 map 构成了强引用关系，这是不安全的编码，需要改进。

题外话：

> 关于强引用、弱引用
>
> * 强引用
>
>   强引用就是我们最常见的普通对象引用（如new 一个对象），只要还有强引用指向一个对象，就表明此对象还“活着”。在强引用面前，即使浏览器内存空间不足，也不会回收它。对于一个普通的对象，如果没有其他的引用关系，只要超过了引用的作用域或者显式地将相应（强）引用赋值为null，就意味着此对象可以被垃圾收集了。但要注意的是，并不是赋值为null后就立马被垃圾回收，具体的回收时机还是要看垃圾收集策略的。
>
> * 弱引用
>
>   

