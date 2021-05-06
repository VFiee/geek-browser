# 变量提升：JavaScript 代码是按顺序执行的吗？

当一段代码被执行时,JavaScript 引擎会对其进行编译,并创建`执行上下文`;

创建执行上下文的情况:

1. 当 JavaScript 执行全局代码时,会编译全局代码并创建全局执行上下文,在整个页面的声明周期内,全局执行上下文只有一个;
2. 当调用一个函数的时候，函数体内的代码会被编译，并创建函数执行上下文，一般情况下，函数执行结束之后，创建的函数执行上下文会被销毁。
3. 当使用 eval 函数的时候，eval 的代码也会被编译，并创建执行上下文。
4. 当使用`let`,`const`声明变量时,`let`和`const`代码块内的代码会被编译并创建执行上下文;

## 执行上下文

执行上下文: JavaScript 代码在执行期间的运行环境,包含 变量环境,词法环境,outer,和 this;

执行上下文分为`全局执行上下文`,`函数执行上下文`,`块级执行上下文(let,const)`;

变量环境:

词法环境:

outer(外部引用): 每个执行上下文的`变量环境中`,都包含一个外部引用,执行外部执行上下文;

this:

作用域链: 当 JavaScript 引擎在`当前执行上下文`找不到所需变量时,便会查找 `outer(外部引用)的执行上下文`,直到找到该变量或查找到`全局执行上下文`,这个查找的链条称为`作用域链`;

## 变量提升

变量提升: JavaScript 引擎把变量声明部分和函数声明部分提升到代码的`顶部位置`的行为被称为`变量提升`;变量被提升后,默认值为`undefined`;

## JavaScript 代码执行顺序

`JavaScript 代码` -------> `编译阶段` ---------> `执行阶段`

JavaScript 代码在执行之前需要被 JavaScript 引擎编译(变量提升便是放生在编译阶段),才会进入执行阶段

### 声明和赋值

示例

```js
var isExample = true
function isTest() {
  return true
}
```

JavaScript 编译阶段,代码`var isExample = true; function isTest() { return true }`会被分为变量声明`var isExample = undefined; function isTest() { return true }`和变量赋值`isExample = true`两部分,其中变量声明(不是函数声明)会被提升到代码的顶部,即变量提升;

示例代码

```js
isCompile()
console.log(compile)
var compile = "javascript compiling!"
function isCompile() {
  console.log("isCompile is run!")
}
```

### 编译阶段

编译阶段,变量和函数声明被存放到执行上下文的变量环境,变量的默认值为 undefined;

```js
// 变量提升部分代码
var compile
function isCompile() {
  console.log("isCompile is run!")
}
```

```js
// 执行部分代码
isCompile()
console.log(compile)
compile = "javascript compiling!"
```

编译阶段会生成执行上下文和可执行代码!

如何生成执行上下文变量环境?

1. 第一行,`isCompile()`不是声明语句,不做处理;
2. 第二行,`console.log(compile)`同上,不做处理;
3. 第三行,`var compile = "javascript compiling!"`,遇到 var 声明语句,便在变量环境对象中创建名为`compile`的变量,使用`undefined`作为其初始化值;
4. 第四行,`function isCompile() { console.log("isCompile is run!") }`,遇到`function`关键字函数定义,将函数定义存储到堆(HEAP)中,并在环境变量对象中创建 key 为`isCompile`属性,属性值指向堆中函数的位置;

如何生成可执行代码?

执行上下文变量环境对象生成结束后,JavaScript 引擎会把声明外的代码编译为`字节码`,暂时理解为一下代码

```js
isCompile()
console.log(compile)
compile = "javascript compiling!"
```

### 执行阶段

执行阶段,JavaScript 引擎按照先后顺序,开始执行`可执行代码`

如何执行?

1. 当执行`isCompile()`函数时,JavaScript 引擎开始在变量环境对象中查找该函数,变量环境对象中存在该函数的引用,所以开始执行该函数,并输出`sCompile is run!`;
2. 当执行`console.log(compile)`函数时,JavaScript 引擎开始在变量环境中查找`compile`变量,找到`compile`变量的值为`undefined`,输出`undefined`;
3. 当执行`compile = "javascript compiling!"`表达式时,JavaScript 引擎在变量环境中找到`compile`变量,并将其赋值为`javascript compiling!"`,执行结束;
