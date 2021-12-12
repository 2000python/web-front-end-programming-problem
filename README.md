---
Author:尉旭胜(rainsin)
Editer:Typora

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

#### 4.2 使用bind实现

