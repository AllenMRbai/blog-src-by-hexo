---
title: generator内部抛出异常时的执行机制
date: 2019-02-28 15:42:22
tags: javascript
---

执行 generator 函数会返回一个 iterator,iterator 调用`next`方法会返回一个值。如果`next`执行过程中出现异常是否还会返回值呢？如果返回值的话，会返回那个值呢？

#### 情况一：捕获的异常

`main` generator 内出现了一个被捕获的异常。

```javascript
function* main() {
  yield 1;
  try {
    undefinedValue.sayHello(); // 这里会出现ReferenceError
    yield 2;
  } catch (e) {
    console.log("捕获了异常");
  }
  yield 3;
  return 4;
}
var it = main();
it.next();
// {value: 1, done: false}
it.next();
// 捕获了异常
// {value: 3, done: false}
it.next();
// {value: 4, done: true}
```

第二个`next`方法被调用时，打印了“捕获了异常”的字样，并返回了 `{value: 3, done: false}`。说明`yield 2`被跳过，并执行`yield 3`。

#### 情况二：未捕获的异常

`main` generator 内出现了一个未被捕获的异常。

```javascript
function *main(){
	yield 1;

    undefinedValue.sayHello();// 这里会出现ReferenceError
	yield 2;

	yield 3;
	return 4;
}
var it=main();
it.next()；
// {value: 1, done: false}
it.next()
console.log('测试是否会打印')
// Uncaught ReferenceError: undefinedValue is not defined
//     at main (<anonymous>:4:5)
//     at main.next (<anonymous>)
//     at <anonymous>:1:4
it
// main {<closed>}
```

第二个`next`方法被调用时直接抛出了异常，并且后面的`console.log('测试是否会打印')`没被执行。再次查看`it`时，发现它的状态变成了`closed`。

#### 情况三：通过 iterator 的`throw`方法产生的异常

我们在第二个`next`方法之后调用 iterator 的`throw`方法产生一个异常，该异常刚好被`try..catch`捕获。

```javascript
function* main() {
  yield 1;
  try {
    yield 2;
    yield 3;
  } catch (e) {
    console.log(e);
  }
  yield 4;
  return 5;
}
var it = main();
it.next();
// {value: 1, done: false}
it.next();
// {value: 2, done: false}
it.throw("异常");
// 异常
// {value: 4, done: false}
it.next();
// {value: 5, done: true}
```

我们在第二个`next`方法后面调用 iterator 的`throw`方法产生一个异常，此时的异常出现在`yield 2`的位置，并被`try..catch`捕获处理，所以程序跳过了`yield 3`执行了`yield 4`。

#### 总结

如果 generator's iterator 调用`next`的过程中遇到被捕获的异常，它处理完异常后会继续运行，直到碰到`yield` `return`或函数的结尾。如果遇到的是未被捕获的异常，它将抛出异常，并且 iterator 的状态变为`closed`。通过 iterator 的`throw`方法产生的异常和普通的异常是一样的执行逻辑。
