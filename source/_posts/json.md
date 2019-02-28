---
title: javascript API JSON
date: 2019-02-28 15:37:09
tags: javascript
---

所有 javascript 内所有 JSON-safe 的值都可以被 `JSON.stringify`。

```javascript
JSON.stringify(42);
// "42"

JSON.stringify("42");
// ""42"" (a string with a quoted string value in it)

JSON.stringify(null);
// "null"

JSON.stringify(true);
// "true"

JSON.stringify({ name: "Allen", hobby: "painting" });
// "{"name":"Allen","hobby":"painting"}"
```

## JSON-safe 的值

什么是 JSON-safe 的值呢？我的理解是不方便跨语言的值都不是 JSON-safe 的。例如 `undefined` `function` `symbol` 还有循环引用的 `object`。

```javascript
// 非JSON-safe的值会被略掉
JSON.stringify(undefined);

// 数组内非JSON-safe的值会被替换为null
JSON.stringify([1, undefined, "a"]); // "[1,null,"a"]"

// 对象内非JSON-safe的值会被排除掉
JSON.stringify({ name: "Allen", age: undefined }); // "{"name":"Allen"}"

// 循环引用的对象会报错
var a = { name: "outer" };
a.b = { name: "inner", outer: a };
JSON.stringify(a);
```

## toJSON 方法

当对一个对象直接 stringify 的结果不是我们想要的，或者那个对象含循环引用时，我们可以在该对象上定义一个`toJSON`方法来自定义它需要返回的结果。返回的结果得是 JSON-safe 的。

```javascript
var o = {};

var a = {
  b: 42,
  c: o,
  d: function() {}
};

// create a circular reference inside `a`
o.e = a;

// would throw an error on the circular reference
// JSON.stringify( a );

// define a custom JSON value serialization
a.toJSON = function() {
  // only include the `b` property for serialization
  return { b: this.b };
};

JSON.stringify(a); // "{"b":42}"
JSON.stringify(o); // "{"e":{"b":42}}"
```

### stringify 的第二个参数 replacer

```javascript
var a = {
  b: 42,
  c: "42",
  d: [1, 2, 3]
};

JSON.stringify(a, ["b", "c"]); // "{"b":42,"c":"42"}"

JSON.stringify(a, function(k, v) {
  if (k !== "c") return v;
});
// "{"b":42,"d":[1,2,3]}"
```

如上所示，如果在第二个参数传入了一个字符串数组，那么 stringify 的过程中就会只保留对应的键值对。

如果在第二个参数传入一个函数，那么可以自定义需要保留的键值对。

### stringify 的第三个参数 space

```javascript
var a = {
  b: 42,
  c: "42",
  d: [1, 2, 3]
};

JSON.stringify(a, null, 3);
// "{
//    "b": 42,
//    "c": "42",
//    "d": [
//       1,
//       2,
//       3
//    ]
// }"

JSON.stringify(a, null, "-----");
// "{
// -----"b": 42,
// -----"c": "42",
// -----"d": [
// ----------1,
// ----------2,
// ----------3
// -----]
// }"
```

如上所示，第三个参数可以是数字，表示缩进的空格个数；也可以是字符串，表示占位的符号。
