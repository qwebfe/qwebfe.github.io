---
layout: post
title: 解密Vue3.0的新特性
date: 2020-05-18
categories: article
---
![](https://p1.ssl.qhimg.com/t01803ad9bd07851fc7.png)

# 前言

尤大大早在2019年10月5日就开放了vue3.0的源码，时隔半年，于今年的4月17日发布了vue3.0的Beta版本，4月21日B站开直播，对vue3.0做分享总结，这一连串的操作，也算是兢兢业业，大写的优秀了。这篇文章重点来解密一下他在视频里所提及的一些新特性，其中非常重磅的Composition API，虽然有那么点像React Hooks，但是本质上还是有很多不同的。先附上Beta版源码地址：[https://github.com/vuejs/vue-next#status-beta](https://github.com/vuejs/vue-next#status-beta)

# Performance

## Virtual Dom

相信使用过前端框架React的人，对虚拟Dom应该很熟悉了。它的核心思想就是抽象原生Dom，封装对应操作的API。其中的Diff算法算是难点。vue3.0这次优化主要就是在Diff算法上下了功夫。它是怎么来实现的呢？通过patchflag来标记不同类型的Dom结构数据，来实现静态数据的一次加载，动态数据的动态更新，从而使其更快的编译渲染。upadate的性能提升了1.3~2倍，开启ssr可以提升2~3倍，性能方面的提升还是很可观的。我们看下一个最简单的例子：

### 代码

#### html
```
<div id="app">
  <span>{{ vue3 }}</span>
  <span>vue</span>
</div>
```

#### vue2的编译结果
```
function anonymous() {
  with(this) {
    return _c('div', {
      attrs: {
        "id": "app"
      }
    }, [_ssrNode("<span>" + _ssrEscape(_s(vue3)) +
      "</span> <span>vue</span>")])
  }
}
```
每次vue3值变化的时候，所有的节点都会重新创建，显然性能是额外损耗的。

#### vue3的编译结果
```
import { toDisplayString as _toDisplayString, createVNode as _createVNode, openBlock as _openBlock, createBlock as _createBlock } from "vue"

const _hoisted_1 = { id: "app" }
const _hoisted_2 = /*#__PURE__*/_createVNode("span", null, "vue", -1 /* HOISTED */)

export function render(_ctx, _cache) {
  return (_openBlock(), _createBlock("div", _hoisted_1, [
    _createVNode("span", null, _toDisplayString(_ctx.vue3), 1 /* TEXT */),
    _hoisted_2
  ]))
}
```
我们注意到，vue3的动态节点，被标记了1，这个数字就是patchflag值，这里表示text类型。它的作用是在之后的动态变化过程中，判断有没有此标记，如果有，就update该节点，如果没有，就忽略不更新。这样，就能把性能损耗降到最低。尤其在有大量静态节点的Dom里，性能提升会更明显。

### patchflag标记
patchflag的值有很多，我们常用到的有text、props，组合类型和事件监听等。下面是代码里列举的：
```
// 这里注释仅供参考。
export const enum PatchFlags {
  TEXT = 1,// 表示具有动态textContent的元素
  CLASS = 1 << 1,  // 表示有动态Class的元素
  STYLE = 1 << 2,  // 表示动态样式（静态如style="color: red"，也会提升至动态）
  PROPS = 1 << 3,  // 表示具有非类/样式动态道具的元素。
  FULL_PROPS = 1 << 4,  // 表示带有动态键的道具的元素，与上面三种相斥
  HYDRATE_EVENTS = 1 << 5,  // 表示带有事件监听器的元素
  STABLE_FRAGMENT = 1 << 6,   // 表示其子顺序不变的片段（没懂）。 
  KEYED_FRAGMENT = 1 << 7, // 表示带有键控或部分键控子元素的片段。
  UNKEYED_FRAGMENT = 1 << 8, // 表示带有无key绑定的片段
  NEED_PATCH = 1 << 9,   // 表示只需要非属性补丁的元素，例如ref或hooks
  DYNAMIC_SLOTS = 1 << 10,  // 表示具有动态插槽的元素
  // 特殊 FLAGS -------------------------------------------------------------
  HOISTED = -1,  // 特殊标志是负整数表示永远不会用作diff,只需检查 patchFlag === FLAG.
  BAIL = -2 // 一个特殊的标志，指代差异算法（没懂）
}
```
patchflag有两个含义：
1. 用来标记当前节点需要更新，静态节点没有这个参数，更新的时候直接略过
2. 组合更新，比如：1-只更新显示文本，8-只更新属性，9-更新显示文本和属性

在每次Dom更新的时候，只更新带有patchflag值的节点，没有的直接略过。而且，无论Dom节点层级有多深，都同样适用。这样更新只用关注变化的内容，节省更多的内存，让渲染更加的快。这点确实很实用。

其中，有一种组合patchflag值，这里说明下。这种类型用的是位运算原理。举个例子：

![](https://p1.ssl.qhimg.com/t0154eed47f7ff06ff4.png)

图中，我们可以看到文本和属性的组合值，在按位与之后，就得到了patchflag值为9。其他同理。

## 静态提升

### 代码
```
<div id="app">
  <span>移动</span>
  <span>端</span>
  <span>流量</span>
  <span>入口</span>
  <span>{{ vue3 }}</span>
  <span>5G</span>
</div>
```
编译结果
```
import { createVNode as _createVNode, toDisplayString as _toDisplayString, openBlock as _openBlock, createBlock as _createBlock } from "vue"

const _hoisted_1 = { id: "app" }
const _hoisted_2 = /*#__PURE__*/_createVNode("span", null, "移动", -1 /* HOISTED */)
const _hoisted_3 = /*#__PURE__*/_createVNode("span", null, "端", -1 /* HOISTED */)
const _hoisted_4 = /*#__PURE__*/_createVNode("span", null, "流量", -1 /* HOISTED */)
const _hoisted_5 = /*#__PURE__*/_createVNode("span", null, "入口", -1 /* HOISTED */)
const _hoisted_6 = /*#__PURE__*/_createVNode("span", null, "5G", -1 /* HOISTED */)

export function render(_ctx, _cache) {
  return (_openBlock(), _createBlock("div", _hoisted_1, [
    _hoisted_2,
    _hoisted_3,
    _hoisted_4,
    _hoisted_5,
    _createVNode("span", null, _toDisplayString(_ctx.vue3), 1 /* TEXT */),
    _hoisted_6
  ]))
}
```
我们看到所有的静态节点都被拿到了渲染函数之外，这样做的好处就是只用被创建一次，在以后的渲染的时候拿来复用就行。可以极大的优化大型项目的内存占用，不用每次渲染都要重新创建。

## 事件监听缓存cacheHandlers

我们知道在vue2中，针对节点绑定的事件，每次触发都要重新生成全新的function去更新，而React目前也没有去缓存事件，具体为什么不去做，不得而知。在vue3.0中，就做了这么一件事情。当cacheHandlers开启的时候，编译会自动生成一个内联函数，将其变成一个静态节点，相当于React中的useCallback。并且本身就支持手写内联函数，这点就比较方便。我们来看下代码示例：

### 代码

#### html
```
<div id="app">
  <button @click="confirmHandler">确认</button>
  <span>{{ vue3 }}</span>
</div>
```

#### 开启cacheHandlers

```
import { createVNode as _createVNode, toDisplayString as _toDisplayString, openBlock as _openBlock, createBlock as _createBlock } from "vue"

const _hoisted_1 = { id: "app" }

export function render(_ctx, _cache) {
  return (_openBlock(), _createBlock("div", _hoisted_1, [
    _createVNode("button", {
      onClick: _cache[1] || (_cache[1] = $event => (_ctx.confirmHandler($event)))
    }, "确认"),
    _createVNode("span", null, _toDisplayString(_ctx.vue3), 1 /* TEXT */)
  ]))
}
```

#### 关闭cacheHandlers
```
import { createVNode as _createVNode, toDisplayString as _toDisplayString, openBlock as _openBlock, createBlock as _createBlock } from "vue"

const _hoisted_1 = { id: "app" }

export function render(_ctx, _cache) {
  return (_openBlock(), _createBlock("div", _hoisted_1, [
    _createVNode("button", { onClick: _ctx.confirmHandler }, "确认", 8 /* PROPS */, ["onClick"]),
    _createVNode("span", null, _toDisplayString(_ctx.vue3), 1 /* TEXT */)
  ]))
}
```
可以很清楚的看到其中的差别，开启cacheHandlers事件缓存之后，生成内联函数_cache，这样每次就不用重复渲染了，在事件更新频繁或者绑定事件过多的情况下，性能优化非常明显。

## SSR

这个原理其实很简单，就是把所有的静态内容编译成一个字符串push到一个buffer里。

### 代码

#### html
```
<div id="app">
  <button @click="confirmHandler">确认</button>
  <span>{{ vue3 }}</span>
  <span>vue</span>
  <span>vue</span>
  <span>vue</span>
  <span>vue</span>
  <span>vue</span>
  <span>vue</span>
  <span>vue</span>
  <span>vue</span>
  <span>vue</span>
  <span>vue</span>
  <span>vue</span>
  <span>vue</span>
  <span>vue</span>
  <span>vue</span>
  <span>vue</span>
  <span>vue</span>
  <span>vue</span>
  <span>vue</span>
  <span>vue</span>
  <span>vue</span>
  <span>vue</span>
</div>
```

#### 编译结果

```
import { ssrInterpolate as _ssrInterpolate } from "@vue/server-renderer"

export function ssrRender(_ctx, _push, _parent) {
  _push(`<div id="app"><button>确认</button><span>${_ssrInterpolate(_ctx.vue3)}</span><span>vue</span><span>vue</span><span>vue</span><span>vue</span><span>vue</span><span>vue</span><span>vue</span><span>vue</span><span>vue</span><span>vue</span><span>vue</span><span>vue</span><span>vue</span><span>vue</span><span>vue</span><span>vue</span><span>vue</span><span>vue</span><span>vue</span><span>vue</span><span>vue</span></div>`)
}
```

是不是很神奇，我们看到所有的静态节点span全部编译成了一个字符串，甚至里面的动态节点也被尽可能的字符化。这样做，至少节省内存一半以上，总体速度加快一倍以上。ssr服务端渲染在性能上，比单纯的抽成Virtual Dom去渲染，快的不是一个量级，两者完全不能在一个层面上。

这里要注意一点，如果静态节点数量很多，超过一个阈值，vue3会单独创建一个div，将静态节点innerHTML到该div里去。

# Tree-shaking

这里有很多解释，我更偏向于理解成，把没用的树叶”抖掉“这个比喻。

换句话说，就是按需引入我们需要的模块。对于用不到的就不会打包。

### 代码

#### html
```
<div id="app">
  <input type="checkbox" v-model="vue3" />
</div>
```
#### 编译结果
```
import { vModelCheckbox as _vModelCheckbox, createVNode as _createVNode, withDirectives as _withDirectives, openBlock as _openBlock, createBlock as _createBlock } from "vue"

const _hoisted_1 = { id: "app" }

export function render(_ctx, _cache) {
  return (_openBlock(), _createBlock("div", _hoisted_1, [
    _withDirectives(_createVNode("input", {
      type: "checkbox",
      "onUpdate:modelValue": $event => (_ctx.vue3 = $event)
    }, null, 8 /* PROPS */, ["onUpdate:modelValue"]), [
      [_vModelCheckbox, _ctx.vue3]
    ])
  ]))
}
```
这里在编译阶段，仅仅引入了vModelCheckbox一个Dom相关模块，其他模块都是通用的必须引入的。这样打出的包会体积更小，性能更好。这一点在vue2里是没有做排除的。

# Composition API

这里是vue3.0升级里最重磅的内容。与React Hooks类似，只是实现方式不同。

有以下特性
* 兼容现有的Options API，无缝衔接
* 更加灵活，组合更方便，复用性更强
* 可以跟其他框架搭配使用

那么，为什么vue3.0必须要有Composition API这样的东西呢？vue3.0采用RFC来讨论所有的API的实现，并且把所有讨论的细节都呈现给使用者，我觉得这点是非常棒的事情。可以更友好的让开发者理解和学习API的使用，附上文档地址：[https://composition-api.vuejs.org/#summary](https://composition-api.vuejs.org/#summary)。下面我们看下几个场景，你就能理解Composition API的强大了。

## 场景

### Mixin
![](https://p2.ssl.qhimg.com/t01ea41ae7d0a1b93c5.png)

图中我们可以看到在解决API复用上，Mixin是可以的。但是它最要命的就是命名空间冲突的问题，这个缺陷可以导致代码的不可维护，这点对于开发者是不能接受的。

### Scoped Slots
![](https://p0.ssl.qhimg.com/t01adf14597d4f2bd21.png)

Scoped Slots可以解决Mixin的问题，当时它的问题是配置太多，逻辑也太多，这样就导致它的复用性和灵活性极差，还不如Mixin。

我们来看看Composition API怎么实现

### Composition Functions
![](https://p2.ssl.qhimg.com/t01b41cd8e633003da5.png)

是不是很棒，使用了我们熟悉的函数式编程，结构非常清晰，代码量也减少了很多，最重要的是灵活性和复用性很友好。也不用担心会命名冲突的问题。

## 组件中的使用
![](https://p1.ssl.qhimg.com/t01c66ed230f8a7ef3f.png)

图中不同颜色代表不同逻辑的业务代码，相同颜色表示同一业务需要的代码。我们在写一个比较复杂的组件的时候，有没有这种体会，里面包含很多个小的功能模块，这些功能可能在其他页面也有用到，但是数据强依赖这个组件的接口。所以只能分开，单独写到这个组件里，导致组件代码又臭又长，还没法复用。遇到这种情况，我们也只能望洋兴叹。在vue3.0出来之后，我们就可以完美的解决这种问题。可以任意在同一组件里，写不同的业务逻辑，不仅层次分明，逻辑清楚，而且神奇的是，你可以信手拈来，拿去复用在其他组件里。你可能也不再需要为了一个页面，去封装十几个小组件了。费神费力，还容易出错。

使用Composition API写出的代码，后期查找问题和维护升级上，体验也是极度舒适的。

我们也不用担心之前的Options API不能兼容，vue3.0中可以一起使用的。建议有了它就不要再考虑使用其他方案了。

具体使用方法可以查看文档：[https://composition-api.vuejs.org/#summary](https://composition-api.vuejs.org/#summary)

# Fragments

碎片，这个我理解vue3.0中，不再限制组件中必须提供根节点。你可以这么写

```
<div>vue3</div>
text
```

也可以这么写
```
vue3
```

其实看编译的代码，会发现，vue3只是帮你生成了一个根节点来包裹。这么做的好处就是你省事了。

# Teleport

这个特性，我理解相当于占位。看下代码
```
<!-- 预留一块空地，专门用来显示特性内容 -->
<div id="modal-container"></div>

<!-- app -->
<div id="app"></div>


<template>
  <div>
    ...
    
    <!-- 注意这一块代码 -->
    <Teleport to="#modal-container">
        <div v-show="isPopUpOpen">
          <p>确定删除？</p>
          <button @click="removeUser">确定</button>
          <button @click="isPopUpOpen = false">取消</button>
        </div>
    </Teleport>
    
  </div>
</template>
```

id是modal-container的模块会在根页面里占位，在调用的时候插入<Teleport></Teleport>模块。这么做可以避免由于z-index层级问题导致的显示异常。
这个特性不仅仅这么简单，其他应用场景有待挖掘。

# Suspense

翻译为，”悬念“。我理解就是异步加载依赖。

### 用法

* 可在嵌套层级中等待嵌套的异步依赖项
* 支持 async setup()
* 支持异步组件

### 代码示例

```
// vue2实现
<template>
    <div>
        <div v-if="!loading">
            ...
        </div>
        <div v-if="loading">Loading...</div>
    </div>
</template>

// vue3实现
<Suspense>
  <template>
    <Suspended-component />
  </template>
  <template #fallback>
    Loading...
  </template>
</Suspense>
```

这里实现一个过渡效果，使用Suspense更直接简单一些。

这个特性的应用场景，可能是图片或者模块的顺序加载。业务逻辑比较复杂，需要异步加载静态节点的时候，可以考虑使用。

# Custom Renderer API

自定义渲染API这块，意味着以后我们可以通过 vue ，实现用 dom 编程的方式来进行 webgl 编程。

其他的一些变化
```
// 周期函数变化
被替换

beforeCreate -> setup()
created -> setup()

重命名

beforeMount -> onBeforeMount
mounted -> onMounted
beforeUpdate -> onBeforeUpdate
updated -> onUpdated
beforeDestroy -> onBeforeUnmount
destroyed -> onUnmounted
errorCaptured -> onErrorCaptured

新增的
新增的以下2个方便调试 debug 的回调钩子：

onRenderTracked
onRenderTriggered

// 对应的自定义指令变化
// vue2
const MyDirective = {
  bind(el, binding, vnode, prevVnode) {},
  inserted() {},
  update() {},
  componentUpdated() {},
  unbind() {}
}

// vue3
const MyDirective = {
  beforeMount(el, binding, vnode, prevVnode) {},
  mounted() {},
  beforeUpdate() {},
  updated() {},
  beforeUnmount() {}, // new
  unmounted() {}
}

```

# 更好的TypeScript支持

vue3采用了TypeScript重写，意味着之后会全面拥抱TypeScript。类型提示、检测都更加的友好强大。废弃了Class，如果你还是喜欢Class的用法，可以安装单独库vue-class-component支持。

# 结语

vue3主要的新特性以上做了简单的总结，也加入了一些自己的理解，目前还不支持深入使用，只能尝鲜阶段。提前熟悉下，方便后期入手vue3，不至于太突然。vue3正式版本发布还有一段路要走，尤大大表示年终会发版，后续内容主要的内容包括，文档继续编写中，一些开发者工具持续完善。同时也在做自动迁移工具，主要是为了vue2升级vue3做准备，尤大大在直播中也提到，会发布一个过渡版本也是最后一个vue2版本vue2.7，计划在不损坏兼容性的前提下，加入一些好的vue3的功能提前合并入2.7版本，在之后18个月后只剩下安全性的描述，不在维护。

关于vue2升级到vue3，作者也友情提示，对于业务很重的项目，请酌情考虑升级，规避风险，言下之意就是说全面拥抱vue3之前可能需要做取舍。

另外，我猜想，Composition API是有极大可能提前加入到2.7版本的，让我们敬请期待吧。
