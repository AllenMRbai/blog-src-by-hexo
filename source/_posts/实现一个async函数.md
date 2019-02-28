---
title: 实现一个async函数
date: 2019-02-28 15:45:09
tags: javascript
---

async 函数能将 promise 链式调用的写法改成同步的写法。让代码变得更简洁易懂。es6 中的 async 是语法糖，我们自己可以实现类似 async 函数的功能。

下面是`myAsync`的代码：

```javascript
/**
 *
 * @param {Function} gen 一个generator函数
 */
function myAsync(gen) {
  var args = [].slice.call(arguments, 1),
    it;
  it = gen.apply(this, args);

  function fn(nextVal) {
    if (!nextVal.done) {
      return Promise.resolve(nextVal.value).then(
        function(v) {
          return fn(it.next(v));
        },
        function(err) {
          return fn(it.throw(err));
        }
      );
    } else {
      return Promise.resolve(nextVal.value);
    }
  }

  return fn(it.next());
}
```

下面是测试用例：

```javascript
/**
 * 下面这个函数模拟异步请求返回一个promise
 * @param {String} resource 资源的路径地址
 * @param {Boolean} success true表示能请求成功,false表示请求失败
 * @returns {Promise}
 */
function request(resource, success) {
  var randomTime = Math.ceil(Math.random() * 10) * 100;
  console.log(resource + " spend:", randomTime);
  return new Promise(function(resolve, reject) {
    setTimeout(function() {
      if (success) {
        resolve(resource);
      } else {
        reject(resource);
      }
    }, randomTime);
  });
}

/**
 * 下面的generator函数写平常的业务逻辑。
 * 如果下面的console.time得出来的时间与三条请求的时间之和
 * 大约相等，说明main函数内的代码以类似同步的方式执行。
 */
function* main(returnValue) {
  console.time("count");
  var rq1 = yield request("/request1", true);
  var rq2 = yield request("/request2", true);

  try {
    var rq3 = yield request("/request3", false);
  } catch (e) {
    console.log("this is an error", e);
  }

  var rq4 = yield request("/request4", true);

  console.timeEnd("count");
  console.log(returnValue);
  return returnValue;
}

myAsync(main, "hello my async");
```

可以看出`main`函数内部写法和 es6 的 async 函数很相似，相当于将`await`替换成`yield`。

下面是 generator 中出现`yield`-delegation 中的测试。

```javascript
//这里依赖上面的 myAsync 和 request 方法

function* subTask() {
  var rq2 = yield request("/request2", true);
  var rq3 = yield request("/request3", true);
  var rq4 = yield request("/request4", false); // promise rejected
  var rq5 = yield request("/request5", true);
}

function* main(returnValue) {
  console.time("count");
  var rq1 = yield request("/request1", true);

  try {
    yield* subTask();
  } catch (err) {
    console.log("catch error from subTask.", err);
  }

  var rq5 = yield request("/request6", true);

  console.timeEnd("count");
  return returnValue;
}

myAsync(main, "hello my async");
```

上面的 main 方法内`yield*`语句将`yield`委托给了 subTask 方法，并且`subTask`内 req4 的 promise 被 rejected。此时的 myAsync 还能正常运行。

下面是 generator 中出现递归委托的测试。

```javascript
//这里依赖上面的 myAsync 和 request 方法

function* subTask(num) {
  if (num < 3) {
    yield* subTask(num + 1);
  }

  yield request("/request" + num, true);
}

function* main(returnValue) {
  console.time("count");
  var rq1 = yield request("/request1", true);

  yield* subTask(2);

  var rq4 = yield request("/request4", true);

  console.timeEnd("count");
  return returnValue;
}

myAsync(main, "hello my async");
```
