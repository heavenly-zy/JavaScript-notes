数组的键名其实也是字符串。之所以可以用数值读取，是因为非字符串的键名会被转为字符串。
```js
var arr = ['a', 'b', 'c'];

arr['0'] // 'a'
arr[0] // 'a'
```
对象有两种读取成员的方法：点结构（`object.key`）和方括号结构（`object[key]`）。但是，对于数值的键名，不能使用点结构。本质原因其实还是因为单独的数值不能作为标识符（identifier）
```js
var arr = [1, 2, 3];
arr.0 // SyntaxError
```

# length 属性
`length` 属性是一个**动态**的值，等于键名中的最大整数加上1。且数组的数字键不需要连续，`length`属性的值总是比最大的那个整数键大1。
```js
var arr = ['a', 'b'];
arr.length // 2

arr[2] = 'c';
arr.length // 3

arr[9] = 'd';
arr.length // 10

arr[1000] = 'e';
arr.length // 1001
```
清空数组的一个有效方法，就是将`length`属性设为0。
```js
var arr = [ 'a', 'b', 'c' ];

arr.length = 0;
arr // []
```
如果人为设置`length`大于当前元素个数，则数组的成员数量会增加到这个值，新增的位置都是空位。
```js
var a = ['a'];

a.length = 3;
a[1] // undefined
```

# for...in 循环和数组的遍历
`for...in`不仅会遍历数组所有的数字键，还会遍历非数字键。因此不推荐使用`for...in`遍历数组。
```js
var a = [1, 2, 3];
a.foo = true;

for (var key in a) {
  console.log(key);
}
// 0
// 1
// 2
// foo
```
数组的遍历可以考虑使用`for`循环或`while`循环。
```js
var a = [1, 2, 3];

// for循环
for(var i = 0; i < a.length; i++) {
  console.log(a[i]);
}

// while循环
var i = 0;
while (i < a.length) {
  console.log(a[i]);
  i++;
}

var l = a.length;
while (l--) {
  console.log(a[l]);
}
```
数组的`forEach`方法，也可以用来遍历数组
```js
var colors = ['red', 'green', 'blue'];
colors.forEach(function (color) {
  console.log(color);
});
// red
// green
// blue
```
# 数组的空位
数组的空位不影响`length`属性
```js
var a = [1, , 1];
a.length // 3
```
数组的空位是可以读取的，返回`undefined`。
```js
var a = [, , ,];
a[1] // undefined
```
使用`delete`命令删除一个数组成员，会形成空位，并且不会影响`length`属性。`length`属性不过滤空位。所以，使用`length`属性进行数组遍历，一定要非常小心。
```js
var a = [1, 2, 3];
delete a[1];

a[1] // undefined
a.length // 3
```
数组的某个位置是空位，与某个位置是undefined，是不一样的。空位就是数组没有这个元素，所以不会被遍历到，而`undefined`则表示数组有这个元素，值是`undefined`，所以遍历不会跳过。

如果是空位，使用数组的`forEach`方法、`for...in`结构、以及`Object.keys`方法进行遍历，空位都会被跳过。
```js
var a = [, , ,];

a.forEach(function (x, i) {
  console.log(i + '. ' + x);
})
// 不产生任何输出

for (var i in a) {
  console.log(i);
}
// 不产生任何输出

Object.keys(a)
// []
```
如果某个位置是`undefined`，遍历的时候就不会被跳过。
```js
var a = [undefined, undefined, undefined];

a.forEach(function (x, i) {
  console.log(i + '. ' + x);
});
// 0. undefined
// 1. undefined
// 2. undefined

for (var i in a) {
  console.log(i);
}
// 0
// 1
// 2

Object.keys(a)
// ['0', '1', '2']
```

# 伪数组
如果一个对象的所有键名都是正整数或零，并且有`length`属性，那么这个对象就很像数组，语法上称为“类似数组的对象”（array-like object），即“伪数组”。
```js
var obj = {
  0: 'a',
  1: 'b',
  2: 'c',
  length: 3
};

obj[0] // 'a'
obj[1] // 'b'
obj.length // 3
obj.push('d') // TypeError: obj.push is not a function
```
“类似数组的对象”的根本特征，就是具有length属性。只要有length属性，就可以认为这个对象类似于数组。但是有一个问题，这种`length`属性**不是动态值**，不会随着成员的变化而变化。
```js
var obj = {
  length: 0
};
obj[3] = 'd';
obj.length // 0
```
典型的伪数组有三类：
1. 函数的`arguments`对象
```js
function args() { return arguments }
var arrayLike = args('a', 'b');

arrayLike[0] // 'a'
arrayLike.length // 2
arrayLike instanceof Array // false
```
2. 大多数 DOM 元素集
```js
var elts = document.getElementsByTagName('h3');
elts.length // 3
elts instanceof Array // false
```
3. 字符串
```js
'abc'[1] // 'b'
'abc'.length // 3
'abc' instanceof Array // false
```
让伪数组也能使用真正数组方法的方法：
1. 数组的`slice`方法可以直接将伪数组变成真正的数组。
```js
var arr = Array.prototype.slice.call(arrayLike);
// 或者
var arr = [].slice.call(arrayLike);
```
用数组的`slice`方法就可把伪数组变为真正数组的原因是：`slice`方法返回的是一个Array类型的对象，可以理解为`arguments.toArray().slice()`，同理用`Array.prototype.splice.call(arrayLike,0)`也可以做到

[如何理解Array.prototype.slice.call()](https://www.cnblogs.com/wphl-27/p/10336591.html)

2. 通过`call()`把数组的方法放到对象上面
```js
function print(value, index) {
  console.log(index + ' : ' + value);
}

Array.prototype.forEach.call(arrayLike, print);
```
注意：此方法并未将伪数组变为真正的数组，只是让伪数组也可以使用数组的方法

字符串也是伪数组，所以也可以用`Array.prototype.forEach.call`遍历。
```js
Array.prototype.forEach.call('abc', function (chr) {
  console.log(chr);
});
// a
// b
// c
```
注意，这种方法比直接使用数组原生的`forEach`要慢，所以最好还是先将“类似数组的对象”转为真正的数组，然后再直接调用数组的`forEach`方法。
```js
var arr = Array.prototype.slice.call('abc');
arr.forEach(function (chr) {
  console.log(chr);
});
// a
// b
// c
```
