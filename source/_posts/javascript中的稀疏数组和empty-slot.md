---
title: javascript中的稀疏数组与empty slot
date: 2019-02-28 15:40:07
tags: javascript
---

```javascript
var a = [];

a[0] = 1;
// no `a[1]` slot set here
a[2] = 3;

a[1]; // undefined;

a.lenght; // 3
```

这里产生了一个稀疏数组，`a[1]`这里是一个 empty slot。在谷歌浏览器控制台内显示样子如下：

![稀疏数组.png](https://note.youdao.com/yws/res/4863/WEBRESOURCEe5262cded20050416ff8700a81554beb)

`delete` 操作符也能产生一个 empty,以上面的数组`a`为例子。

```javascript
delete a[2];

a; // [1, empty x 2]
```

在谷歌浏览器控制台内显示样子如下：

![delete一个属性.png](https://note.youdao.com/yws/res/4873/WEBRESOURCE71e831eef536015e7a107f31ecf48f0f)

用`new Array(3)`方法也会生成 empty slot。

## `empty`和刻意赋值为`undefined`有差别吗？

从第一个例子看到，`a[1]`返回的值是`undefined`。那么这和我们直接赋值`a[1]`为`undefined`有差别吗？

有差别。看下面的例子：

```javascript
var a = [1, , 3]; // [1,empty,3]
var b = [1, undefined, 3]; // [1,undefined,3]

a.map(v => v * 2); // [2,empty,6]
b.map(v => v * 2); // [2, NaN, 6]
```

我们发现，`map`方法会跳过`a`数组的 empty slot,而不会跳过被显式设为`undefined`的`b[1]`项。

不仅仅是`map`方法，还有很多与遍历数组相关的操作都为跳过 empty slot。

我测试了所有我能想到与数组相关的操作，列出了以下两个列表。

#### 会跳过 empty slot 的操作

```javascript
var a = [1, , 3];
// [1,empty,3]
var b = [1, undefined, 3];
// [1,undefined,3]

// map方法
a.map(v => v * 2);
// [2,empty,6]
b.map(v => v * 2);
// [2, NaN, 6]

// forEach方法
a.forEach((v, k) => console.log(v, k));
// 1 0
// 3 2
b.forEach((v, k) => console.log(v, k));
// 1 0
// undefined 1
// 3 2

// filter方法
a.filter(v => typeof (v + "a") === "string");
// [1,3]
b.filter(v => typeof (v + "a") === "string");
// [1, undefined, 3]

// some方法
a.some(v => typeof v === "undefined");
// false
a.some(v => typeof v === "undefined");
// true

// every方法
a.every(v => typeof v === "number");
// true
b.every(v => typeof v === "number");
// false

// reduce方法
a.reduce((ac, v) => ac + v, "");
// "13"
b.reduce((ac, v) => ac + v, "");
// "1undefined3"

// reduceRight方法
a.reduceRight((ac, v) => ac + v, "");
// "31"
b.reduceRight((ac, v) => ac + v, "");
// "3undefined1"

// indexOf方法
a.indexOf(undefined); // false
b.indexOf(undefined); // true

// for..in 循环
for (let i in a) {
  console.log(i);
}
// 0
// 2
for (let i in b) {
  console.log(i);
}
// 0
// 1
// 2

// in 操作符
1 in a;
// false
1 in b;
// true

// Object.keys方法
Object.keys(a);
// ["0", "2"]
Object.keys(b);
// ["0", "1", "2"]

// Object.values方法
Object.values(a);
// [1, 3]
Object.values(b);
// [1, undefined, 3]

// Object.entries方法
Object.entries(a);
// [["0",1],["2",3]]
Object.entries(b);
// [["0",1],["1", undefined],["2",3]]
```

#### 不会跳过 empty slot 的操作

以下列出我们可能误以为会跳过 empty slot 的操作，实际上下面的操作不会跳过 empty slot。尤其是下面的`for..of`操作需要特别注意。

```javascript
var a = [1,,3]; // [1,empty,3]
var b = [1,undefined,3]; // [1,undefined,3]

// find方法,该方法返回找到的第一个符合条件的值
// find回调方法内理应返回一个boolean值，下面的写法是没什么用的，仅仅只是为了测试
a.find(function(v){console.log(v)});
// 1
// undefined
// 3
b.find(function(v){console.log(v)});
// 1
// undefined
// 3

// findIndex方法，同上。
// 不同的是findIndex返回的是索引值
a.findIndex(function(v){console.log(v)});
// 1
// undefined
// 3
b.findIndex(function(v){console.log(v)});
// 1
// undefined
// 3

// for..of 循环
for(let v of a){console.log(v)}
// 1
// undefined
// 3
for(let v of b){console.log(v)}
// 1
// undefined
// 3

// 下面的三个方法看起来与Object构造函数上的keys values entries方法很像，
// 但是他们不会跳过empty slot,而且返回的值是Iterator而不是数组。
// 我们可以用扩展运算符将Iterator内的值取出，并放到数组内
// Array.prototype.keys方法
[...a.keys()]
// [0, 1, 2]
[...b.keys()]
// [0, 1, 2]

// Array.prototype.values方法
[...a.values()]
// [1, undefined, 3]
[...b.values()]
// [1, undefined, 3]

// Array.prototype.entries方法
[...a.entries()]
// [[0,1],[1,undefined], [2,3]]
[...b.entries()]
// [[0,1],[1,undefined], [2,3]]
```

## 总结

不给数组内的某个 slot 赋值,或使用`delete`操作数组，或用`new Array`生成数组都会生成 empty slot。大部分与遍历相关的方法和操作会跳过 empty slot；但有几个操作例外，尤其要注意`for..of`循环。
