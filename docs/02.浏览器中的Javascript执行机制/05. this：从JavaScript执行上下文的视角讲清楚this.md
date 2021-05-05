# this：从 JavaScript 执行上下文的视角讲清楚 this

## this 机制诞生的原因?

```js
let bar = {
  myName: "I'm string",
  printName: function () {
    console.log(myName);
  }
};
function foo() {
  let myName = "极客时间";
  return bar.printName;
}
let myName = "极客邦";
let _printName = foo();
_printName(); // 极客邦
bar.printName(); // 极客邦
```

对象内部的方法中使用对象内部的属性是非常普遍的需求,但 JavaScript 的作用域机制并不支持这一点,基于这个需求,便诞生了 this 机制

JavaScript 作用域链和 this 机制是两套不同的系统,没有太多联系;

## JavaScript 中 this 是什么?

执行上下文中包含变量环境(var 变量和函数),词法环境(块级作用域),外部环境(变量查找链,词法作用域),this

执行上下文类型

1. 全局执行上下文
2. 函数执行上下文
3. eval 执行上下文

### 全局执行上下文中 this

```js
function test() {
  console.log(this);
}
test(); // globalThis => window
```

```js
"use strict";
function test() {
  console.log(this); // undefined
}
test();
```

全局执行上下文中`this`在严格模式下指向`undefined`,非严格模式下指向`window(globalThis)`

### 函数执行上下文中的 this

1. 通过 call,apply,bind 绑定 this(显式绑定)
2. 当函数由一个对象引导调用时,this 指向该对象(隐式绑定)
3. 当函数被当做构造函数使用,并使用`new`关键字引导调用时,`this`指向`new`创建出来的对象(`new`关键字绑定)
4. 箭头函数`this`指向函数定义时的上下文

## this 机制的设计缺陷

1. 嵌套函数中的 this 不会被外层函数继承

   ```js
   var myObj = {
     name: "浏览器工作原理与实践",
     showThis: function () {
       console.log("showThis:", this); // myObj
       function bar() {
         console.log("bar:", this); // window
       }
       bar();
     }
   };
   myObj.showThis();
   ```

   ```js
   var myObj = {
     name: "浏览器工作原理与实践",
     showThis: function () {
       console.log("showThis:", this); // myObj
       function bar() {
         console.log("bar:", this); // window
       }
       // 绑定this
       bar.call(this);
     }
   };
   myObj.showThis();
   ```

2. 普通函数 this 默认指向全局对象

### 总结

1. 当函数在没有任何修饰的情况下调用，非严格模式下，this 指向全局对象，严格模式下 this 指向 undefined。（默认绑定）
2. 当函数由一个对象引导调用时，this 指向该对象。（隐式绑定）
3. 函数通过 apply,call,bind 绑定时，this 指向绑定的对象。（显式绑定）
4. 当函数被当做构造函数使用，用 new 引导调用时，this 指向 new 创建出来的对象。（new 绑定）；
5. 箭头函数其 this 取决于函数定义时所在的上下文。

其优先级为：new 绑定 > 显示绑定 > 隐式绑定 > 默认绑定；
