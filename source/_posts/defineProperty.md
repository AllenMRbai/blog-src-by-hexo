---
title: Object.defineProperty
date: 2019-02-28 15:49:00
tags: javascript
---

`Object.defineProperty` 可以给对象定义属性,以及属性的描述。

## 基本使用方法

```javascript
var person = { name: "Allen" };

// 定义属性 'gender'
// 可配置(configurable)
// 可枚举(enumerable)
// 可写(writable)
Object.defineProperty(person, "gender", {
  configurable: true,
  enumerable: true,
  writable: true,
  value: "man"
});

// 定义'_birthday'
// 我们希望生日是不可改写的，
// 也不希望枚举对象的时候出现它
Object.defineProperty(person, "_birthday", {
  configurable: false,
  enumerable: false,
  writable: false,
  value: new Date("2000-03-03")
});

// 定义'age'
// 年龄是通过生日获得的，并且不可改，
// 所以只需要设置get方法
Object.defineProperty(person, "age", {
  configurable: false,
  enumerable: true,
  get() {
    return new Date().getFullYear() - this._birthday.getFullYear();
  }
});
```

- `configurable` 设置属性描述是否可修改。如果为 false,将无法对 `configurable` `enumerable` `writable` 的值进行修改。有一点例外： 当 `writable` 为 true 时，可以将其改为 false。
- `enumerable` 设置属性是否会在枚举的过程中出现。`for..in` `Object.keys` `Object.values` `Object.entries` 只会枚举属性描述 `enumerable` 为 true 的属性。其中 `for..in` 会将原型链上所有可枚举属性都遍历一遍。
- `writable` 设置属性是否可写。以上面的 `person` 对象为例，其中的 `_birthday` 属性是不可写的，假如你对它赋值（`person._birthday = new Date('2019-03-03')`）会没有效果，如果在严格模式下还会抛出异常。
- `value` 设置属性的值。
- `get` 读取属性的值。以上面 `person` 对象为例，当使用者要获取 `age` 的值时（`person.age`），获取的值就是 `get` 方法返回的值。
- `set` 写入属性的值。如果只有 `get` 没有 `set` 那么这个属性是不可写的。

注意 `writable` `value`是一对，`get`方法 `set`方法是一对。有 `value` 就没有 `get` `set`。

## 获取一个属性的描述

如果我们想知道一个属性描述可以使用 `Object.getOwnPropertyDescriptor`。

```javascript
Object.getOwnPropertyDescriptor(person, "age");
```
