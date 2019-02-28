---
title: javascript类型转换
date: 2019-02-28 15:36:15
tags: javascript
---

以下内容是读《YOU DON'T KNOW JS》系列的《types & grammar》第 4 章节的笔记。

## 抽象值运算（Abstract Value Operations）

“abstract operations” 通俗的理解就是“仅内部使用的操作”。在 ES5 specification 内定义了 9 个 abstract value operations ,其中作者重点介绍了 `ToString` `ToNumber` 和 `ToBoolean`。

### ToString

任何非字符串类型的值被转为 string 时，都会经过 `ToString` 的处理。

#### 基本类型 `ToString`

基本类型`ToString`运算的规则很简单，基本上原来什么样，转换后就是什么样的。但是需要注意特大或特小的数字转成字符串时会变成指数的形式。

```javascript
String(false); // "false"
String(undefined); // "undefined"
String(null); // "null"
String(10); // "10"
String(Symbol("hello")); // "Symbol(hello)"

var a = 1.07 * 1000 * 1000 * 1000 * 1000 * 1000 * 1000 * 1000;
String(a); // "1.07e21"
```

#### 对象 `ToString`

一般对象的`ToString`运算会调用默认的`toString`方法（因为大部分对象原型链顶部是`Object.prototype`，它内部有`toString`方法;它会返回内部的`[[Class]]`,例如 `"[object Object]"`）。

如果重写了对象的`toString`方法，那么就调用重写后的`toString`。注意重写的`toString`方法只能返回基本数据类型，返回对象会抛出异常。

还有一种特殊情况就是用`Object.create(null)`创建以`null`为原型的对象。它的原型链上没有`toString`,执行`ToString`运算时会抛出异常。

### ToNumber

任何非 number 类型的值被转为 number 时，都会经过 `ToNumber` 的处理。

#### 基本类型 `ToNumber`

基本数据类型转 number 的规则如下。注意`null` 转成数字后是 0 而不`NaN`，空字符串转成数字后是 0，`symbol` 不能转数字，否则报错。

```javascript
+null; // 0
+undefined; // NaN
+true; // 1
+false; // 0
+"5"; // 5
+"5allen"; // NaN
+""; // 0
+Symbol(10); // 抛出异常
```

#### 对象 `ToNumber`

对象转 number 时，会先执行一个`ToPrimitive`的抽象值运算。执行`ToPrimitive`后返回的基本数据类型再按前面提到的基本数据类型转 number 的规则转换。

`ToPrimitive`运算会先尝试调用`valueOf`方法，如果返回的结果是基本数据类型就返回运算结果；如果对象上没有`valueOf`方法或`valueOf`返回的结果不是基本数据类型就尝试调用`toString`方法；如果`toString`方法不存在或`toString`方法返回对象，会抛出异常。

`ToPrimitive`抽象值运算的流程图如下：

```
graph TD
    A(start) -->B{valueOf存在吗}
    B -->|存在| C[执行valueOf]
    B -->|不存在| F[执行valueOf]
    C -->D{返回结果是基本类型吗}
    D -->|是| E(返回运算结果)
    D -->|否| F{toString存在吗}
    F -->|存在| H[执行toString]
    F -->|不存在| G(抛出异常)
    H -->I{返回结果是基本类型吗}
    I -->|是| J(返回运算结果)
    I -->|否| K(抛出异常)
```

## ToBoolean

强转为`boolean`后的结果为`false`的值，称为`falsy value`。以下是所有的`falsy value`。

- `undefined`
- `null`
- `false`
- `+0` `-0` `NaN`
- `""`

不在以上列表中的所以其他值都是`truthy value`，也就是说其他值在强转为`boolean`后的结果都是`true`。

## 显式转换（Explicit Coercion）

我们知道很多操作会在内部对值进行转换，但是为了让自己和他人更直观地阅读代码，我们最好用显式的方式来对值进行转换。

### 显式：Strings <--> Numbers

```javascript
var a = 42;
var b = String(a);

var c = "3.14";
var d = Number(c);

b; // "42"
d; // 3.14
```

```javascript
var a = 42;
var b = a.toString();

var c = "3.14";
var d = +c;

b; // "42"
d; // 3.14
```

string 和 number 间的转换，作者认为`String(..)` `Number(..)` `toString()`这几个方法是显式的转换。`+ "3.14"`这种形式的转换在 js 社区内普遍也被认为是一种显式的转换，但是该方法在某些场合会让代码变得比较难以理解，例子如下：

```javascript
var c = "3.14";
var d = 5 + +c;

d; // 8.14

1 + -+(+(+-+1)); // 2
```

总结：作者推荐使用`Sting(..)` `Number(..)` `toString()`和一元操作符`+`来显示地将其他值转为 number 和 string。避免在某些会引起误解的地方使用一元操作符`+`。

### 显式：Parsing Numeric Strings

```javascript
var a = "42";
var b = "42px";

Number(a); // 42
parseInt(a); // 42

Number(b); // NaN
parseInt(b); // 42
```

除了用`Number(..)`方法进行转换，我们还可以用`parseInt(..)`(还有它的双胞胎`parseFloat(..)`)来将其他值解析为 number。`parseInt(..)`会从左到又解析，直到碰到非数字字符;而`Number(..)`方法（或其他转换方法）转换的字符串中含非数字字符就会返回`NaN`。

在使用`parseInt(..)`时，作者推荐大家仅对 string 进行解析。不然`parseInt(..)`会将值先进行`ToString`操作（前面介绍过的抽象值操作），然后再进行解析；如果这样，这里面就带有隐式的转换，可能会带来意想不到的结果，例子如下。

```javascript
parseInt(0.000008); // 0   ("0" from "0.000008")
parseInt(0.0000008); // 8   ("8" from "8e-7")
parseInt(false, 16); // 250 ("fa" from "false")
parseInt(parseInt, 16); // 15  ("f" from "function..")

parseInt("0x10"); // 16
parseInt("103", 2); // 2
```

同时作者还提示大家，当需要解析的字符串中以`0x`开头，那么`parseInt(..)`会以十六进制的方式来解析（`0o` `0b`同理）。如果明确要以 10 进制来解析，推荐大家在`parseInt(..)`的第二个参数内传入`10`，来明确告知以 10 进制的方式解析。

总结：作者推荐大家仅对 string 使用`parseInt(..)`,在必要的情况下可以在第二个参数传`10`，来确保它以十进制的方式解析。

### 显式：\* --> Boolean

和`String(..)` `Number(..)` 一样，`Boolean(..)`是一个显式的转换方式。

```javascript
var a = "0";
var b = [];
var c = {};

var d = "";
var e = 0;
var f = null;
var g = undefined;
var h = NaN;

Boolean(a); // true
Boolean(b); // true
Boolean(c); // true

Boolean(d); // false
Boolean(e); // false
Boolean(f); // false
Boolean(g); // false
Boolean(h); // false
```

虽然`Boolean(..)`是显式的方式，但在平时的开发中并不常用。就像前面提到的用一元操作符`+`将值转为 number，我们会用`!`来将值转为 boolean。`!`会将值转为 boolean，同时也反转了 boolean 的值，所以更通用的方式是`!!`。

```javascript
var a = "0";
var b = [];
var c = {};

var d = "";
var e = 0;
var f = null;
var g = undefined;
var h = NaN;

!!a; // true
!!b; // true
!!c; // true

!!d; // false
!!e; // false
!!f; // false
!!g; // false
!!h; // false
```

总结：作者推荐使用`Boolean(..)` `!!`来显示转化为 boolean。`!!`会在开发中比较常用。

## 隐式转换（Implicit Coercion）

所有对 JavaScript 类型转换的抱怨主要来自它的隐式转换。

《JavaScript 语言精粹》的作者，Douglas Crockford 在很多会议和文字上表示应该避免使用 JavaScript 类型转换。但本书的作者认为 Douglas Crockford 想表达的确切意思应该是避免使用 JavaScript 的隐式转换。

然后作者提出以下几个问题：

> So, is implicit coercion evil? Is it dangerous? Is it a flaw in JavaScript's design? Should we avoid it at all costs?

大意如下：

> 所以，隐式转换是魔鬼吗？它很危险吗？它是不是 JavaScript 设计上的瑕疵？我们是否要不惜一切代价避免使用它呢？

大多数的读者会一边倒地回答"Yes!"。

作者认为我们需要从不同的角度去看隐式转换，否则就太狭隘了，这样会失去很多重要的细节。

他明确了使用隐式转换的目的是为了减少冗余的、模板化的代码，避免不必要的实现细节。

可以肯定的是，隐式转换内含有很多潜在的危险，我们需要学会避开那些会危害代码健壮的坑，而不是全盘抛弃隐式转换。

以下是隐式转换中比较好的实践。

### 隐式：Strings <--> Numbers

`+` 操作符即可作为算数运算中的加号，也可以作为字符串的拼接符号。

```javascript
var a = "42";
var b = "0";

var c = 42;
var d = 0;

a + b; // "420"
c + d; // 42
```

基本类型中做`+`操作：`+`两边其中一方是 string,那么执行的是字符串拼接，否则都是做算法运算。

含对象的`+`操作：对象都会先经过`ToPrimitive`抽象值运算得到基本类型，然后再按基本类型的规则做`+`运算。

`a + ""`是我们常用的转 string 方法，但值得注意的是它和我们前面提到的显式转 string 的方法`String(..)` `toString`并不等同,例子如下：

```javascript
var a = {
  valueOf: function() {
    return 42;
  },
  toString: function() {
    return 4;
  }
};

a + ""; // "42"

String(a); // "4"
```

上面的例子中，`a`是一个对象，并且重写了`valueOf` `toString`方法。`a + ""`时，`a`要运行`ToPrimitive`运算，返回结果 42，然后再与空字符串拼接。而`String( a )`内部运行`ToString`运算，所以直接调用`a.toString()`。

一般情况，我们不会碰到这种陷进，除非我们在对象上重写了`valueOf`和`toString`方法。

`-` `*` `/` 操作只可作为算数运算。所以可以用来将字符串转数字，例 `a - 0` `a * 1` `a / 1`。

总结：相比于 `String(a)` `a + ""`在开发中会更常用。

### 隐式：\* --> Boolean

以下表达式操作会隐式地将值转为`boolean`。

- `if ( .. )`
- `for ( .. ; .. ; .. )`
- `while ( .. )` `do..while( .. )`
- 三元运算符 `? :`
- 逻辑或 `||` 逻辑与 `&&`

#### `||` 和 `&&`

这两个运算符的结果未必一定是`boolean`，它会返回运算符两边中的一个值。

```javascript
var a = 42;
var b = "abc";
var c = null;

a || b; // 42
a && b; // "abc"

c || b; // "abc"
c && b; // null
```

### 宽松相等`==` 与 严格相等`===`

严格相等：等号两边的值和类型都会进行比较；如果等号两边是对象，比较对象的引用地址。

宽松相等：只比较等号两边的值，如果类型不同会先转化再比较；如果等号两边是对象，比较对象的引用地址。

#### 宽松相等`==`

严格相等没什么好说的，最容易让人迷惑的是宽松相等`==`。大多情况下，它是瑕疵，它让代码容易出错。

宽松等于两边值类型不同时，其中的一边或两边会进行转换然后再进行比较。转换的规则会有点复杂，而且甚至有点迷惑性。

其中要比较注意的是 falsy values 间的比较。

- `undefine` `null`两个互相宽松相等，但却不与其它 falsy values 宽松相等。
- `NaN`与所有其它值都不相等。

宽松相等的规则总结：

- 如果其中一边是对象，那么先执行`ToPrimitive`转为基本类型
- `boolean`执行`ToNumber`
- string 和 number 比较，string 要`ToNumber`
- `null` `undefined`两宽松相等，和其它不宽松相等
