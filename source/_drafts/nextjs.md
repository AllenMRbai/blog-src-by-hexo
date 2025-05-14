---
title: nextjs
tags:
---

## 服务端渲染优点

1. **减少白屏时间：**纯客户端渲染的React和Vue的项目，首先需要加载JS资源，解析并执行，如果存在异步接口渲染的内容，还需要等待接口返回，才能渲染页面。使用服务端渲染的页面，HTML内包括接口数据填充的内容都已经在服务端渲染完成，浏览器接收到的就是一个完整的HTML，可以直接渲染，减少了白屏时间。
2. **SEO优化：**服务端渲染的页面，搜索引擎爬虫可以直接抓取，不需要等待JS资源加载和执行，可以更快的收录页面。
3. **减轻客户端负担：**服务端渲染将部分渲染工作放在服务器上完成，从而减轻了客户端浏览器的负担。这对于性能有限的设备（如移动设备）和低版本的浏览器来说比较友好。
4. **避免客户端渲染的闪烁问题：**在客户端渲染中，当 JavaScript 文件加载并执行时，可能会出现页面内容闪烁或重新排列的情况。服务端渲染可以避免这种问题，因为它在服务器端就已经生成了完整的页面布局。

## 什么样的项目要使用服务端渲染

对SEO优化有要求的项目，比如电商网站、信息咨询类网站等，可以考虑使用服务端渲染。同时对页面首屏加载速度有要求的项目，提高首屏时间能带来收益增长或用户粘性增长的项目，也可以考虑使用服务端渲染。

因为服务端渲染相当于用服务器资源来置换部分客户端资源，需要投入额外的服务器成本。体量较大的场景还需要考虑服务器的负载能力，对开发者的技术要求也更高。需要根据具体的应用场景和需求进行权衡。

## 什么是 Next.js，它与 React 有何不同？

Next.js 是一个基于 React 的开源框架，可帮助开发人员构建服务器端呈现的 React 应用程序。

React和 Next.js 之间的主要区别在于它们处理路由的方式。 React 使用客户端路由，这意味着页面转换完全在客户端使用 JavaScript 处理。

相比之下，Next.js 提供服务器端路由，这意味着服务器处理路由并将预渲染的页面发送给客户端，从而实现更快的页面加载和更好的 SEO。

Next.js 还提供其他功能，如自动代码拆分、静态站点生成和动态导入。

## 常见问题

### 怎么判断是否服务端组件（RSC）?

在next.js内，默认所有组件为服务端组件，服务端组件只会在服务端执行，不会在客户端执行。如果要在组件内使用可交互的hooks，需要在组件上添加`use client`，否则会报错。如果你希望组件可以在客户端和服务端都使用，可以使用`use client`。

### 什么是服务端函数？

在服务端组件内部的函数体声明'use server'，标识该函数在服务端执行。可通过props的形式传递给客户端组件，客户端组件调用该函数，会触发服务端函数执行，并将结果返回给前端。

```typescript
// Server Component
import Button from './Button';

function EmptyNote () {
  async function createNoteAction() {
    // Server Function
    'use server';
    
    await db.notes.create();
  }

  return <Button onClick={createNoteAction}/>;
}
```

服务端函数另一种用法，单独在文件内声明，然后客户端组件使用`import`方法导入使用。

```typescript
"use server";

export async function createNote() {
  await db.notes.create();
}
```

```tsx
"use client";
import {createNote} from './actions';

function EmptyNote() {
  console.log(createNote);
  // {$$typeof: Symbol.for("react.server.reference"), $$id: 'createNote'}
  <button onClick={() => createNote()} />
}
```