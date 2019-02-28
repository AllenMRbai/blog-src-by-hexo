---
title: yield-delegation
date: 2019-02-28 15:47:39
tags: javascript
---

在 generator 函数内部，通过`yield*`语句，可以将`yield`委托给其他任何实现`iterable`的对象。

### 委托给 generator 函数生成的`iterable`对象

调用 generator 函数会返回一个实现`iterable`的对象(该对象同时也是一个`iterator`)。

通过`yield* otherGenerator()`可以将 generator 内的`yield`委托给其他 generator 生成的`iterable`对象。

```javascript
function* foo() {
  console.log("*foo() starting");
  yield "foo 1";
  yield "foo 2";
  console.log("*foo() finished");
}

function* bar() {
  yield "bar 1";
  yield "bar 2";
  yield* foo(); // `yield`-delegation!
  yield "bar 3";
}

var it = bar();

it.next().value;
// "bar 1"
it.next().value;
// "bar 2"
it.next().value;
// *foo() starting
// "foo 1"
it.next().value;
// "foo 2"
it.next().value;
// *foo() finished
// "bar 3"
```

可以看到上面的代码中，在调用第 3 个`next`方法时返回的是`foo`里面`yield`的`"foo 1"`；在调用第 5 个`next`时，并没有返回`foo` generator 隐式`return`的`undefined`,而是返回了`"bar 3"`。

如果`foo`内有显式的`return`语句，那么在进行 yield-delegation 时是否会返回`foo` `return`的值吗？

下面的代码在`foo`中添加了一个`return`语句。在`bar`内将`yield* foo()`表达式的值赋值给`tmp`变量并打印。

```javascript
function* foo() {
  console.log("*foo() starting");
  yield "foo 1";
  yield "foo 2";
  console.log("*foo() finished");
  return "foo 3";
}

function* bar() {
  yield "bar 1";
  yield "bar 2";
  var tmp = yield* foo(); // `yield`-delegation!
  console.log("在bar内打印", tmp);
  yield "bar 3";
}

var it = bar();

it.next().value;
// "bar 1"
it.next().value;
// "bar 2"
it.next().value;
// *foo() starting
// "foo 1"
it.next().value;
// "foo 2"
it.next().value;
// *foo() finished
// 在bar内打印 foo 3
// "bar 3"
```

在第 5 次调用`next`方法时，可以发现`foo` `return`的`"foo 3"`变成了`yield* foo()`表达式的值，并被打印为`"在bar内打印 foo 3"`;`"bar 3"`成为`next`方法的返回值。

### 委托给其他任何实现`iterable`的对象

generator 内部的`yield*`语句也能将`yield`委托给其他非 generator 生成的`iterable`对象。例如 数组就是一个实现了`iterable`的对象。

```javascript
function* bar() {
  console.log("inside `*bar()`:", yield "A");

  // `yield`-delegation to a non-generator!
  console.log("inside `*bar()`:", yield* ["B", "C", "D"]);

  console.log("inside `*bar()`:", yield "E");

  return "F";
}

var it = bar();

console.log("outside:", it.next().value);
// outside: A

console.log("outside:", it.next(1).value);
// inside `*bar()`: 1
// outside: B

console.log("outside:", it.next(2).value);
// outside: C

console.log("outside:", it.next(3).value);
// outside: D

console.log("outside:", it.next(4).value);
// inside `*bar()`: undefined
// outside: E

console.log("outside:", it.next(5).value);
// inside `*bar()`: 5
// outside: F
```

也可以委托给自己手写的`iterable`对象。由于 javascript 不是强类型语言，如果对象上含有`Symbol.iterator`方法，那么就可以将该对象当做一个`iterable`对象；如果对象上含有`next`方法，那就可以将该对象当做一个`iterator`对象。下面的`myIterable`对象即实现了`Symbol.iterator`方法也实现了`next`方法，所以它即是一个`iterable`又是一个`iterator`。

```javascript
var myIterable = {
  [Symbol.iterator]: function() {
    return this;
  },
  num: 98,
  next: function() {
    var self = this;
    if (self.num < 101) {
      return { value: String.fromCharCode(self.num++), done: false };
    } else {
      return { value: String.fromCharCode(101), done: true };
    }
  }
};

function* bar() {
  console.log("inside `*bar()`:", yield "A");

  // `yield`-delegation to a non-generator!
  console.log("inside `*bar()`:", yield* myIterable);

  console.log("inside `*bar()`:", yield "E");

  return "F";
}

var it = bar();

console.log("outside:", it.next().value);
// outside: A
console.log("outside:", it.next(1).value);
// inside `*bar()`: 1
// outside: b
console.log("outside:", it.next(2).value);
// outside: c
console.log("outside:", it.next(3).value);
// outside: d
console.log("outside:", it.next(4).value);
// inside `*bar()`: e
// outside: E
console.log("outside:", it.next(5).value);
// inside `*bar()`: 5
// outside: F
```

### 异常委托

被委托的`iterator`内部执行过程发生异常，如果异常被捕获，那么捕获处理完异常后还按原来的逻辑运行；如果异常未被捕获，那么异常会被抛出，异常会被抛到`yield*`语句那。下面是《YOU DON'T KNOW JS》内的例子。

```javascript
function* foo() {
  try {
    yield "B";
  } catch (err) {
    console.log("error caught inside `*foo()`:", err);
  }

  yield "C";

  throw "D";
}

function* bar() {
  yield "A";

  try {
    yield* foo();
  } catch (err) {
    console.log("error caught inside `*bar()`:", err);
  }

  yield "E";

  yield* baz();

  // note: can't get here!
  yield "G";
}

function* baz() {
  throw "F";
}

var it = bar();

console.log("outside:", it.next().value);
// outside: A

console.log("outside:", it.next(1).value);
// outside: B

console.log("outside:", it.throw(2).value);
// error caught inside `*foo()`: 2
// outside: C

console.log("outside:", it.next(3).value);
// error caught inside `*bar()`: D
// outside: E

try {
  console.log("outside:", it.next(4).value);
} catch (err) {
  console.log("error caught outside:", err);
}
// error caught outside: F
```

### 递归委托

下面是《YOU DON'T KNOW JS》里的例子。

```javascript
function* foo(val) {
  if (val > 1) {
    // generator recursion
    val = yield* foo(val - 1);
  }

  return yield request("http://some.url/?v=" + val);
}

function* bar() {
  var r1 = yield* foo(3);
  console.log(r1);
}

run(bar);
```
