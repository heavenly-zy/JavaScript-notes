# 函数的定义

>函数是一段可以反复调用的代码块。函数还能接受输入的参数，不同的参数会返回不同的值。

JavaScript 语言将函数看作一种值，与其它值（数值、字符串、布尔值等等）地位相同。凡是可以使用值的地方，就能使用函数。比如，可以把函数赋值给变量和对象的属性，也可以当作参数传入其他函数，或者作为函数的结果返回。函数只是一个可以执行的值，此外并无特殊之处。

由于函数与其他数据类型地位平等，所以在 JavaScript 语言中又称函数为**第一等公民**。

# 三种声明函数的方法


1. 函数声明
```js
function print(s) {
  console.log(s);
}
```
2. 函数表达式
```js
var print = function(s) {
  console.log(s);
};
```
注意语句的结尾要加分号
3. Function 构造函数
```js
var add = new Function(
  'x',
  'y',
  'return x + y'
);

// 等同于
function add(x, y) {
  return x + y;
}
```
Function构造函数接受三个参数，除了最后一个参数是add函数的“函数体”，其他参数都是add函数的参数。这种声明函数的方式非常不直观，几乎无人使用。

# 用递归求斐波那契数列
```js
function fib(num) {
  if (num === 0) return 0;
  if (num === 1) return 1;
  return fib(num - 2) + fib(num - 1);
}

fib(6) // 8
```

# 函数名的提升
JavaScript 引擎将函数名视同变量名，所以采用function命令声明函数时，整个函数会像变量声明一样，被提升到代码头部。所以，下面的代码不会报错。
```js
f();

function f() {}
```
但是，如果采用赋值语句定义函数，JavaScript 就会报错。
```js
f();
var f = function (){};
// TypeError: undefined is not a function
```
因为上面的代码等价于
```js
var f;
f();
f = function () {};
```
如果同时采用function命令和赋值语句声明同一个函数，因为函数名的提升和函数的重复声明导致最后总是采用赋值语句的定义。
```js
var f = function () {
  console.log('1');
}

function f() {
  console.log('2');
}

f() // 1
```

# 函数的 name 属性
如果是通过变量赋值定义的函数，且变量的值是一个具名函数，那么name属性返回function关键字之后的那个函数名。
```js
var f3 = function myName() {};
f3.name // 'myName'
```
上面代码中，`f3.name`返回函数表达式的名字。注意，真正的函数名还是`f3`，而`myName`这个名字只在函数体内部可用。

# length 属性
函数的`length`属性返回函数预期传入的参数个数，即函数定义之中的参数个数。
```js
function f(a, b) {}
f.length // 2
```
上面代码定义了空函数`f`，它的`length`属性就是定义时的参数个数。不管调用时输入了多少个参数，`length`属性始终等于2。

# toString()
对于那些原生的函数，`toString()`方法返回`function (){[native code]}`。
```js
Math.sqrt.toString()
// "function sqrt() { [native code] }"
```
实现多行字符串
```js
var multiline = function (fn) {
  var arr = fn.toString().split('\n');
  return arr.slice(1, arr.length - 1).join('\n');
};

function f() {/*
  这是一个
  多行注释
*/}

multiline(f);
// " 这是一个
//   多行注释"
```

# 函数作用域

作用域（scope）指的是变量存在的范围。

在 ES5 的规范中，JavaScript 只有两种作用域：
- 全局作用域：变量在整个程序中一直存在，所有地方都可以读取
- 函数作用域：变量只在函数内部存在

ES6 新增了块级作用域

##### 全局变量（global variable）
对于顶层函数来说，函数外部声明的变量就是全局变量，它可以在函数内部读取。
```js
var v = 1;

function f() {
  console.log(v);
}

f()
// 1
```
##### 局部变量（local variable）
在函数内部定义的变量，外部无法读取，称为“局部变量”
```js
function f(){
  var v = 1;
}

v // ReferenceError: v is not defined
```
注意，对于var命令来说，局部变量只能在函数内部声明，在其他区块中声明，一律都是全局变量。
```js
if (true) {
  var x = 5;
}
console.log(x);  // 5
```

# 函数内部的变量提升
与全局作用域一样，函数作用域内部也会产生“变量提升”现象。var命令声明的变量，不管在什么位置，变量声明都会被提升到函数体的头部。
```js
function foo(x) {
  if (x > 100) {
    var tmp = x - 100;
  }
}

// 等同于
function foo(x) {
  var tmp;
  if (x > 100) {
    tmp = x - 100;
  };
}
```

# 函数本身的作用域
函数执行时所在的作用域，是定义时的作用域，而不是调用时所在的作用域。
```js
var a = 1;
var x = function () { // 定义时
  console.log(a);
};

function f() {
  var a = 2;
  x(); // 调用时
}

f() // 1
```
同样的，函数体内部声明的函数，作用域绑定函数体内部。
```js
function foo() {
  var x = 1;
  function bar() {
    console.log(x);
  }
  return bar;
}

var x = 2;
var f = foo();
f() // 1
```
函数`foo`内部声明了一个函数`bar`，`bar`的作用域绑定`foo`。当我们在`foo`外部取出`bar`执行时，变量`x`指向的是`foo`内部的`x`，而不是`foo`外部的`x`。正是这种机制，“闭包”现象。

# 传参方式
##### 简单类型传参------传值传递（深拷贝）
函数参数如果是原始类型的值（数值、字符串、布尔值），传递方式是**传值传递**（passes by value）。这意味着，在函数体内修改参数值，不会影响到函数外部。
```js
var p = 2;

function f(p) {
  p = 3;
}
f(p);

p // 2
```
##### 复杂类型传参------传址传递（浅拷贝）
如果函数参数是复合类型的值（数组、对象、其他函数），传递方式是**传址传递**（pass by reference）。也就是说，传入函数的原始值的地址，因此在函数内部修改参数，将会影响到原始值。
```js
var obj = { p: 1 };

function f(o) {
  o.p = 2;
}
f(obj);

obj.p // 2
```
注意，如果函数内部修改的，不是参数对象的某个属性，而是替换掉整个参数，这时不会影响到原始值。
```
var obj = [1, 2, 3];

function f(o) {
  o = [2, 3, 4];
}
f(obj);

obj // [1, 2, 3]
```
# arguments 对象
`arguments`对象包含了函数运行时的所有参数，`arguments[0]`就是第一个参数，`arguments[1]`就是第二个参数，以此类推。
注意：这个对象只有在函数体内部，才可以使用。
```js
var f = function (one) {
  console.log(arguments[0]);
  console.log(arguments[1]);
  console.log(arguments[2]);
}

f(1, 2, 3)
// 1
// 2
// 3
```
正常模式下，`arguments`对象可以在运行时修改。
```js
var f = function(a, b) {
  arguments[0] = 3;
  arguments[1] = 2;
  return a + b;
}

f(1, 1) // 5
```
严格模式下，arguments对象与函数参数不具有联动关系。也就是说，修改arguments对象不会影响到实际的函数参数。
```js
var f = function(a, b) {
  'use strict'; // 开启严格模式
  arguments[0] = 3;
  arguments[1] = 2;
  return a + b;
}

f(1, 1) // 2
```
通过`arguments`对象的`length`属性，可以判断函数调用时到底带几个参数。
```js
function f() {
  return arguments.length;
}

f(1, 2, 3) // 3
f(1) // 1
f() // 0
```
`arguments`本质上是一个伪数组，不能直接使用数组专有的方法（比如`slice`和`forEach`）

将`arguments`转为真正的数组
```js
var args = Array.prototype.slice.call(arguments);

// 或者
var args = [];
for (var i = 0; i < arguments.length; i++) {
  args.push(arguments[i]);
}
```

# 闭包
JavaScript 语言特有的"链式作用域"结构（chain scope），子对象会一级一级地向上寻找所有父对象的变量。所以，父对象的所有变量，对子对象都是可见的，反之则不成立。

![image.png](https://upload-images.jianshu.io/upload_images/18574809-58ed2cd63378ee87.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

既然`f2`可以读取`f1`的局部变量，那么只要把`f2`作为返回值，我们不就可以在`f1`外部读取它的内部变量了吗！
```js
function f1() {
  var n = 999;
  function f2() {
    console.log(n);
  }
  return f2;
}

var result = f1();
result(); // 999
```
闭包最大的特点，就是它**可以“记住”诞生的环境**，比如`f2`记住了它诞生的环境`f1`，所以从`f2`可以得到`f1`的内部变量。在本质上，**闭包就是将函数内部和函数外部连接起来的一座桥梁**。

闭包的作用：
1. 可以读取函数内部的变量
2. 封装对象的私有属性和私有方法

# 立即执行函数
`function`这个关键字即可以当作语句，也可以当作表达式。语句直接加括号会报错，而表达式不会报错
```js
// 语句
function f() {}() // SyntaxError: Unexpected token (

// 表达式
var f = function f() {}()
```
立即执行函数的写法
```js
(function(){ /* code */ }());
// 或者
(function(){ /* code */ })();
// 或者
!function () { /* code */ }();
...
```
使用立即执行函数的两个目的：
1. 不必为函数命名，避免了污染全局变量。
2. 内部形成了一个单独的作用域，可以封装一些外部无法读取的私有变量。

# eval 命令
eval命令接受一个字符串作为参数，并将这个字符串当作语句执行。
```js
eval('var a = 1;');
a // 1
```
`eval`没有自己的作用域，都在当前作用域内执行，因此可能会修改当前作用域的变量的值，造成安全问题。
```js
var a = 1;
eval('a = 2');

a // 2
```
为了防止这种风险，JavaScript 规定，如果使用严格模式，`eval`内部声明的变量，不会影响到外部作用域。
```js
(function f() {
  'use strict';
  eval('var foo = 123');
  console.log(foo);  // ReferenceError: foo is not defined
})()
```
不过，即使在严格模式下，eval依然可以读写当前作用域的变量。
```js
(function f() {
  'use strict';
  var foo = 1;
  eval('foo = 2'); // 改写了外部变量 foo 的值
  console.log(foo);  // 2
})()
```
总之，`eval`的本质是在当前作用域之中，注入代码。由于安全风险和不利于 JavaScript 引擎优化执行速度，所以一般不推荐使用。通常情况下，`eval`最常见的场合是解析 JSON 数据的字符串，不过正确的做法应该是使用原生的`JSON.parse`方法。

# eval 的别名调用
为了保证`eval`的别名不影响代码优化，JavaScript 的标准规定，凡是使用**别名**执行`eval`，`eval`内部一律是**全局作用域**。
```js
var a = 1;

function f() {
  var a = 2;
  var e = eval;
  e('console.log(a)');
}

f() // 1
```
