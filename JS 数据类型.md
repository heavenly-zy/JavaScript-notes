# typeof 运算符
JavaScript 有三种方法，可以确定一个值到底是什么类型。
- `typeof`运算符
- `instanceof`运算符
- `Object.prototype.toString`方法

undefined返回undefined。
```js
typeof undefined
// "undefined"
```
利用这一点，typeof可以用来检查一个没有声明的变量，而不报错。
```js
v
// ReferenceError: v is not defined

typeof v
// "undefined"
```
实际编程中，这个特点通常用在判断语句。
```js
// 错误的写法
if (v) {
  // ...
}
// ReferenceError: v is not defined

// 正确的写法
if (typeof v === "undefined") {
  // ...
}
```
instanceof运算符可以区分数组和对象
```js
var o = {};
var a = [];

o instanceof Array // false
a instanceof Array // true
```
特殊：null的类型是object
`typeof null // "object"`

# null, undefined 和布尔值
null与undefined都可以表示“没有”，含义非常相似。
```js
if (!undefined) {
  console.log('undefined is false');
}
// undefined is false

if (!null) {
  console.log('null is false');
}
// null is false

undefined == null
// true
```
null和undefined的区别：
null是一个表示“空”的对象，转为数值时为0；
```js
Number(null) // 0
5 + null // 5
```
undefined是一个表示"此处无定义"的原始值，转为数值时为NaN。
```js
Number(undefined) // NaN
5 + undefined // NaN
```
五个falsy值：
- undefined
- null
- 0
- NaN
- ""或''（空字符串）
注意，空数组（[]）和空对象（{}）对应的布尔值，都是true。

# 字符串
多行字符串转换为单行
```js
var longString = 'Long \
long \
long \
string';

longString
// "Long long long string"
```
```js
var longString = 'Long '
  + 'long '
  + 'long '
  + 'string';

longString
// "Long long long string"
```
输出多行字符串
```js
(function () { /*
line 1
line 2
line 3
*/}).toString().split('\n').slice(1, -1).join('\n')
// "line 1
// line 2
// line 3"
```
### 转义
- `\0` ：null（\u0000）
- `\b` ：后退键（\u0008）
- `\f` ：换页符（\u000C）
- `\n` ：换行符（\u000A）
- `\r` ：回车键（\u000D）
- `\t` ：制表符（\u0009）
- `\v` ：垂直制表符（\u000B）
- `\'` ：单引号（\u0027）
- `\"` ：双引号（\u0022）
- `\\` ：反斜杠（\u005C）

# 字符串和数组
字符串可以被视为字符数组，因此可以使用数组的方括号运算符，用来返回某个位置的字符（位置编号从0开始）。
```js
var s = 'hello';
s[0] // "h"
s[1] // "e"
s[4] // "o"

// 直接对字符串使用方括号运算符
'hello'[1] // "e"
```
字符串内部的单个字符无法改变和增删，这些操作会默默地失败。
```js
var s = 'hello';

delete s[0];
s // "hello"

s[1] = 'a';
s // "hello"

s[5] = '!';
s // "hello"
```
字符串的length属性无法改变，但是不会报错。
```js
var s = 'hello';
s.length // 5

s.length = 3;
s.length // 5
```
### 字符集
对于码点在U+10000到U+10FFFF之间的字符，JavaScript 总是认为它们是两个字符（length属性为2）。JavaScript 返回的字符串长度可能是不正确的。
`'��'.length // 2`

### Base64 转码
所谓 Base64 就是一种编码方法，可以将任意值转成 0～9、A～Z、a-z、`+`和`/`这64个字符组成的可打印字符。使用它的主要目的，不是为了加密，而是为了不出现特殊字符，简化程序的处理。

JavaScript 原生提供两个 Base64 相关的方法。
   - btoa()：任意值转为 Base64 编码
   - atob()：Base64 编码转为原来的值
```js
var string = 'Hello World!';
btoa(string) // "SGVsbG8gV29ybGQh"
atob('SGVsbG8gV29ybGQh') // "Hello World!"
```
注意，这两个方法不适合非 ASCII 码的字符，会报错。
```js
btoa('你好') // 报错
```
要将非 ASCII 码字符转为 Base64 编码，必须中间插入一个转码环节，再使用这两个方法。
```js
function b64Encode(str) {
  return btoa(encodeURIComponent(str));
}

function b64Decode(str) {
  return decodeURIComponent(atob(str));
}

b64Encode('你好') // "JUU0JUJEJUEwJUU1JUE1JUJE"
b64Decode('JUU0JUJEJUEwJUU1JUE1JUJE') // "你好"
```

# 对象
对象的每一个键名又称为“属性”（property），它的“键值”可以是任何数据类型。如果一个属性的值为函数，通常把这个属性称为“方法”，它可以像函数那样调用。
```js
var obj = {
  p: function (x) {
    return 2 * x;
  }
};

obj.p(1) // 2
```
如果属性的值还是一个对象，就形成了链式引用。
```js
var o1 = {};
var o2 = { bar: 'hello' };

o1.foo = o2;
o1.foo.bar // "hello"
```
属性可以动态创建，不必在对象声明时就指定。
```js
var obj = {};
obj.foo = 123;
obj.foo // 123
```
### 对象的引用
o1和o2指向同一个对象，因此为其中任何一个变量添加属性，另一个变量都可以读写该属性。实际上就叫做浅拷贝，我的理解是仅仅只拷贝了地址
```js
var o1 = {};
var o2 = o1;

o1.a = 1;
o2.a // 1

o2.b = 2;
o1.b // 2
```
此时，如果取消某一个变量对于原对象的引用，不会影响到另一个变量。
```js
var o1 = {};
var o2 = o1;

o1 = 1;
o2 // {}
```
对于简单类型来说，复制就是深拷贝
```js
var x = 1;
var y = x;

x = 2;
y // 1
```
### 表达式还是语句？
第一种可能是，这是一个表达式，表示一个包含foo属性的对象；第二种可能是，这是一个语句，表示一个代码区块，里面有一个标签foo，指向表达式123。
```js
{ foo: 123 }
```
为了避免这种歧义，JavaScript 引擎的做法是，如果遇到这种情况，无法确定是对象还是代码块，一律解释为代码块。
```js
{ console.log(123) } // 123
```
如果要解释为对象，最好在大括号前加上圆括号。因为**圆括号的里面，只能是表达式**，所以确保大括号只能解释为对象。
```js
({ foo: 123 }) // 正确
({ console.log(123) }) // 报错
```
如果没有圆括号，eval将其理解为一个代码块；加上圆括号以后，就理解成一个对象。
```js
eval('{foo: 123}') // 123
eval('({foo: 123})') // {foo: 123}
```
### 属性的读取
如果使用方括号运算符，键名必须放在引号里面，否则会被当作变量处理。
```js
var foo = 'bar';

var obj = {
  foo: 1,
  bar: 2
};

obj.foo  // 1
obj[foo]  // 2
```
数字键可以不加引号，因为会自动转成字符串。但是，数值键名不能使用点运算符（因为会被当成小数点），只能使用方括号运算符。
```js
var obj = {
  123: 'hello world'
};

obj.123 // 报错
obj[123] // "hello world"
```
### 属性的查看
查看一个对象本身的所有属性，可以使用Object.keys方法。
```js
var obj = {
  key1: 1,
  key2: 2
};

Object.keys(obj);
// ['key1', 'key2']
```
### 属性的删除：delete命令
delete命令用于删除对象的属性，删除成功后返回true。
```js
var obj = { p: 1 };
Object.keys(obj) // ["p"]

delete obj.p // true
obj.p // undefined
Object.keys(obj) // []
```
注意，删除一个不存在的属性，delete不报错，而且返回true。因此，不能根据delete命令的结果，认定某个属性是存在的。
```js
var obj = {};
delete obj.p // true
```
delete命令只能删除对象本身的属性，无法删除继承的属性
```js
var obj = {};
delete obj.toString // true
obj.toString // function toString() { [native code] }
```
### 属性是否存在：in 运算符
```js
var obj = { p: 1 };
'p' in obj // true
'toString' in obj // true
```
in运算符的一个问题是，它不能识别哪些属性是对象自身的，哪些属性是继承的。这时，可以使用对象的hasOwnProperty方法判断一下，是否为对象自身的属性。
```js
var obj = {};
if ('toString' in obj) {
  console.log(obj.hasOwnProperty('toString')) // false
}
```
### 属性的遍历：for...in 循环
for...in循环有两个使用注意点。
   - 它遍历的是对象所有可遍历（enumerable）的属性，会跳过不可遍历的属性。
   - 它不仅遍历对象自身的属性，还遍历继承的属性。

所以使用for...in的时候，应该结合使用hasOwnProperty方法，在循环内部判断一下，某个属性是否为对象自身的属性。
```js
var person = { name: '老张' };

for (var key in person) {
  if (person.hasOwnProperty(key)) {
    console.log(key);
  }
}
// name
```
### with 语句
with 语句的作用是操作同一个对象的多个属性时，提供一些书写的方便。
```js
var obj = {
  p1: 1,
  p2: 2,
};
with (obj) {
  p1 = 4;
  p2 = 5;
}
// 等同于
obj.p1 = 4;
obj.p2 = 5;
```
但是，with语句有一个很大的弊病，就是绑定对象不明确。对象obj并没有p1属性，对p1赋值等于创造了一个全局变量p1。
```js
var obj = {};
with (obj) {
  p1 = 4;
  p2 = 5;
}

obj.p1 // undefined
p1 // 4
```
因此，建议**不要使用with语句**，可以考虑用一个临时变量代替with。
```js
with(obj1.obj2.obj3) {
  console.log(p1 + p2);
}

// 可以写成
var temp = obj1.obj2.obj3;
console.log(temp.p1 + temp.p2);
```
