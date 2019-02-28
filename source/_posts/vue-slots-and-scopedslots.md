---
title: vm.$slots和vm.$scopedSlots 在渲染函数内的使用
date: 2019-02-28 15:50:21
tags: javascript
---

以下例子是在 Vue@2.6.2 中运行的。

`vm.$slots` 与 `vm.$scopedSlots` 是两个与插槽相关的属性。在使用渲染函数替代 vue 的模板功能时我们可能需要用到这两个属性来实现插槽的功能。

下面利用一个例子来研究 `vm.$slots` `vm.$scopedSlots` 的区别。

```html
<!-- App.vue -->

<template>
  <div id="app">
    <HelloWorld>
      <h1 slot="head" slot-scope="{author}">this is head slot -- {{author}}</h1>
      <div>this is default slot</div>
      <div slot="foot">this is foot slot</div>
    </HelloWorld>
  </div>
</template>

<!-- 后面代码 略 -->
```

```html
<!-- HelloWorld.vue -->

<template>
  <div class="hello">
    <slot name="head" author="Allen"></slot>
    <slot name="aside"></slot>
    <slot></slot>
    <slot name="foot"></slot>
  </div>
</template>

<!-- 后面代码 略 -->
```

我们在 `HelloWorld.vue` 内定义三个具名插槽和一个默认插槽。在 `App.vue` 内实例化 HelloWorld 组件时，只使用了 `head` `default` `foot` 插槽，其中 `head` 插槽是一个作用域插槽。

然后我们将 `vm.$slots` `vm.$scopedSlots`打印的到控制台上，chrome 浏览器上打印的结果如下：

#### vm.\$slots

![vm.$slots](https://note.youdao.com/yws/res/4098/EB147A0005BE49C4BE7E572B7767079F)

观察发现

- 只有被使用到的插槽才会出现在 `vm.$slots` 内。
- 默认插槽和具名插槽的值是虚拟 dom 数组（`VNode[]`）,作用域插槽对应的值是一个 `get` 方法且属性修饰为不可枚举。

#### vm.\$scopedSlots

![vm.$scopedSlots](https://note.youdao.com/yws/res/4100/F15683549E22485EBEED1246DCB44F0E)

观察发现

- 只有被使用到的插槽才会出现在 `vm.$scopedSlots` 内。
- 这里面多了 `$stable` `_normalized` 属性。具体什么作用，未知。
- 无论是具名插槽、默认插槽、作用域插槽都在，而且值是一个工厂函数，调用后会返回虚拟 dom 数组（`VNode[]`）。
- 作用域插槽对应的函数内多个参数 `scope`，可以通过它将参数传给插槽。
- 所有值都是可枚举的。

## vm.\$slots 在开发中的使用

以下例子来自 Vue 官方文档，仅做了些许的改动。

### 简单的例子

```html
<!-- MyHeadline.vue -->

<script>
  export default {
    name: "MyHeadline",
    props: {
      level: {
        type: Number,
        required: true
      }
    },
    render(createElement) {
      return createElement("h" + this.level, this.$slots.default);
    }
  };
</script>
```

```html
<!-- App.vue -->

<template>
  <div id="app">
    <MyHeadline :level="1">这是level 1</MyHeadline>
    <MyHeadline :level="2">这是level 2</MyHeadline>
    <MyHeadline :level="3">这是level 3</MyHeadline>
    <MyHeadline :level="4">这是level 4</MyHeadline>
    <MyHeadline :level="5">这是level 5</MyHeadline>
    <MyHeadline :level="6">这是level 6</MyHeadline>
  </div>
</template>

<!-- 后面代码 略 -->
```

页面渲染的结果：

![渲染结果](https://note.youdao.com/yws/res/4101/0E93C9D17EF543CAAD51705E67B781ED)

页面的 html 结构：

![html结构](https://note.youdao.com/yws/res/4099/CCAF184EDE8348D7A9047FED4DE328E1)

`MyHeadline` 组件的目的很明显，就是根据传入的 `level` 属性生成对应级别的 h 标签。使用 vue 的模板是很难实现这样的功能的，所以这里用到了 vue 的渲染函数（其实 vue 模板最终也是编译成渲染函数）。

### 进阶例子

```html
<!-- AnchoredHeading.vue -->

<script>
  var getChildrenTextContent = function(children) {
    return children
      .map(function(node) {
        return node.children
          ? getChildrenTextContent(node.children)
          : node.text;
      })
      .join("");
  };

  export default {
    name: "AnchoredHeading",
    props: {
      level: {
        type: Number,
        required: true
      }
    },
    render(createElement) {
      // 创建 kebab-case 风格的ID
      var headingId = getChildrenTextContent(this.$slots.default)
        .toLowerCase()
        .replace(/\W+/g, "-")
        .replace(/(^\-|\-$)/g, "");

      return createElement("h" + this.level, [
        createElement(
          "a",
          {
            attrs: {
              name: headingId,
              href: "#" + headingId
            }
          },
          this.$slots.default
        )
      ]);
    }
  };
</script>
```

```html
<!-- App.vue -->

<template>
  <div id="app">
    <AnchoredHeading :level="1">chapter 1 love</AnchoredHeading>
    <AnchoredHeading :level="2">chapter 2 marry</AnchoredHeading>
    <AnchoredHeading :level="3">chapter 3 rival</AnchoredHeading>
    <AnchoredHeading :level="4">chapter 4 hate</AnchoredHeading>
    <AnchoredHeading :level="5">chapter 5 divorce</AnchoredHeading>
    <AnchoredHeading :level="6">chapter 6 真香</AnchoredHeading>
  </div>
</template>

<!-- 后面代码 略 -->
```

页面渲染结果：

![渲染结果2](https://note.youdao.com/yws/res/4137/D580782D95F743839C502CDDFFC17897)

页面 html 结构：

![html结构2](https://note.youdao.com/yws/res/4136/19E72031A54E4B6DAD7A7B8B21D9C002)

## vm.\$scopedSlots 在开发中的使用

### 简单例子

```html
<!-- Menu.vue -->

<script>
  export default {
    name: "Menu",
    render(createElement) {
      return createElement(
        "div",
        { class: "menuBox" },
        this.$scopedSlots.default({ title: "🌎 菜单栏目 🌎" })
      );
    }
  };
</script>
```

```html
<!-- MenuItem.vue -->

<script>
  export default {
    name: "MenuItem",
    render(createElement) {
      return createElement("div", { class: "MenuItem" }, this.$slots.default);
    }
  };
</script>
```

```html
<!-- App.vue -->

<template>
  <div id="app">
    <menu>
      <template slot-scope="{title}">
        <h4>{{title}}</h4>
        <menuitem>首页</menuitem>
        <menuitem>产品</menuitem>
        <menuitem>公司简介</menuitem>
        <menuitem>联系我们</menuitem>
      </template>
    </menu>
  </div>
</template>

<!-- 后面代码 略 -->
```

在`Menu.vue`内我们通过`this.$scopedSlots.default({ title: "🌎 菜单栏目 🌎" })`向作用域插槽传了参数（`{ title: "🌎 菜单栏目 🌎" }`）。然后我们在 `App.vue` 内使用 `<template slot-scope="{title}">` 获得传过来的参数（这里用了解构赋值）。

上面只是为了展示`vm.$scopedSlots`在渲染函数内的使用，这个例子内的行为有点多此一举。

页面渲染结果：

![scopedslots渲染结果.png](https://note.youdao.com/yws/res/4468/WEBRESOURCE882982210635beeed57bdfff7154dfb8)

页面 html 结构：

![scopedslots_html结构.png](https://note.youdao.com/yws/res/4470/WEBRESOURCE178ed95b3ed30d5d2c691c508c683530)

### 在渲染函数内插入作用域插槽

如果我们把上面那个例子里`App.vue`的模板写法改成渲染函数的话，我们该如何插入作用域插槽呢？

在模板写法内插入插槽很简单，只要在组件标签内插入即可。在渲染函数内需要用到`createElement`方法内的`scopedSlots`选项。

```html
<!-- App.vue -->

<script>
  import Menu from "./components/Menu.vue";
  import MenuItem from "./components/MenuItem.vue";

  export default {
    name: "app",
    components: {
      Menu,
      MenuItem
    },
    render(createElement) {
      return createElement("div", { attrs: { id: "app" } }, [
        createElement("Menu", {
          scopedSlots: {
            default: ({ title }) => {
              return [
                createElement("h4", title),
                createElement("MenuItem", "首页"),
                createElement("MenuItem", "产品"),
                createElement("MenuItem", "公司简介"),
                createElement("MenuItem", "联系我们")
              ];
            }
          }
        })
      ]);
    }
  };
</script>
```

这个写法和上面的模板写法的效果是等同的。
