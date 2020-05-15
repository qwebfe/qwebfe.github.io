---
layout: post
title: 解密Vue3.0的新特性
date: 2020-05-18
categories: article
---
![](https://p1.ssl.qhimg.com/t01803ad9bd07851fc7.png)

# 前言

尤大大早在2019年10月5日就开放了vue3的源码，时隔半年，于今年的4月17日发布了vue3.0的Beta版本，4月21日B站开直播，对vue3.0做分享总结，这一连串的操作，也算是兢兢业业，大写的优秀了。这篇文章重点来解密一下他在视频里所提及的一些新特性，包括性能方面的大幅度提升，Tree-shaking抖动，Fragments，Teleport，Suspense，自定义渲染API等，其中最重磅的某过于Composition API了，虽然有那么点像React Hooks，但是气质上还是有很多不同的，下面会具体介绍。先附上Beta版源码地址：[https://github.com/vuejs/vue-next#status-beta](https://github.com/vuejs/vue-next#status-beta)，供大家仔细研读。

# Performance

## Virtual Dom

相信使用过前端框架React的人，对虚拟Dom应该很熟悉了。它的核心思想就是抽象原生Dom，封装对应操作的API。其中的Diff算法算是核心难点。vue3.0这次优化主要就是在Diff算法上下了很多功夫。它是怎么来实现的呢？主要通过`patchflag`来标记不同类型的数据，来实现动态数据的动态更新，区分静态数据，这样只会在最开始的时候加载一次，后续编译过程中均直接略过，从而达到提升渲染速度的目的。效果是，upadate的性能提升了1.3~2倍，开启ssr的情况下可以提升2~3倍，所以在性能方面的提升还是很可观的。我们来看下一个代码示例：

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

我们注意到，vue3的动态节点，被标记了1，这个数字就是`patchflag`值，这里表示text类型。它的作用，主要是在之后的动态更新过程中，判断当前Dom节点是否有此标记，如果有，就update该节点，如果没有，就忽略不更新。如此，就能把性能损耗降到最低。尤其是在拥有大量静态节点的Dom结构里，性能提升会更显著。

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

patchflag的用途：

1. 用来标记当前节点需要更新，静态节点没有这个参数，更新的时候直接略过
2. 组合更新，比如：1-只更新显示文本，8-只更新属性，9-更新显示文本和属性

在每次Dom更新的时候，只更新带有`patchflag`值的节点，没有的直接略过。而且，无论Dom节点层级有多深，都同样适用。这样更新只用关注变化的内容，而不用再去关心那些静态数据。这么做节省了更多的内存，提升更新渲染的速度。是不是很实用。

其中值得提的一点，叫组合patchflag值。这种类型使用的是位运算的原理。举个例子说明一下：

![](https://p1.ssl.qhimg.com/t0154eed47f7ff06ff4.png)

图中，我们可以看到文本和属性的组合值，在按位与之后，就得到了patchflag值为9。其他同理，不做赘述。

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

我们看到所有的静态节点都被拿到了渲染函数之外，这样做的好处就是，静态节点只用被创建一次，在以后的渲染的时候拿来复用就行。可以极大的优化大型项目的内存占用，不用每次渲染都要重新创建。

## 事件监听缓存cacheHandlers

我们知道在vue2中，针对节点绑定的事件，每次触发都要重新生成全新的function去更新，而React目前也没有去缓存事件，具体为什么不去做，不得而知。在vue3中，就做了这么一件事情。当cacheHandlers开启的时候，编译会自动生成一个内联函数，将其变成一个静态节点，相当于React中的useCallback。并且本身就支持手写内联函数，这点就比较方便。我们来看下代码示例：

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

可以很清楚的看到其中的差别，开启cacheHandlers事件缓存之后，生成内联函数_cache，这样每次就不用重复渲染了，在事件更新频繁或者绑定事件过多的情况下，性能优化非常显著。

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

是不是很神奇，我们看到所有的静态节点span全部被编译成了一个字符串，甚至里面的动态节点也被尽可能的字符化。这样做至少可以节省一半以上内存，总体渲染速度加快一倍以上。SSR服务端渲染在性能上，比单纯的抽成Virtual Dom去渲染，快的不是一个量级，两者完全不在一个层面上。

这里要注意一点的是，如果静态节点数量很多，超过了一个阈值，vue3会单独创建一个div，将静态节点innerHTML到该div里去。

# Tree-shaking

这里有很多解释，我更偏向于理解成，把没用的树叶”抖掉“这个比喻。

换句话说，就是按需引入我们需要的模块。对于用不到的模块就不会打包。

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

这里在编译阶段，仅仅引入了vModelCheckbox一个Dom相关模块，其他模块都是通用的必须要引入的。这样打出的包会体积更小，速度更快，性能更好。这一点在vue2里是没有做区分的。

# Composition API

终于到了vue3.0版本里最重磅的内容。与React Hooks类似，只是实现方式，整体气质不同。

拥有以下特性
* 兼容现有的Options API，无缝衔接
* 更加灵活，组合更方便，复用性更强
* 可以跟其他框架搭配使用

我们知道，vue3采用RFC来讨论所有的API的实现，并且把所有讨论的细节都呈现给使用者，我觉得这点是非常棒的事情。可以更友好的让开发者理解和学习API的使用，附上文档学习地址：[https://composition-api.vuejs.org/#summary](https://composition-api.vuejs.org/#summary)。

那么，为什么vue3必须要有Composition API这样的东西呢？下面我们看下几个场景，你就能理解Composition API的强大了。

## Mixin
![](https://p2.ssl.qhimg.com/t01ea41ae7d0a1b93c5.png)

图中我们可以看到在解决API复用上，Mixin是可以达到的。但是它最要命的就是命名空间冲突的问题，这个缺陷可以导致代码的不可维护，这点对于开发者是不能接受的，甚至毁灭性的。

## Scoped Slots
![](https://p0.ssl.qhimg.com/t01adf14597d4f2bd21.png)

Scoped Slots虽然可以解决Mixin的问题，但是它的问题就是配置太多，逻辑也太多，这样就导致它的复用性和灵活性极差，还不如Mixin。

最后，我们来看看Composition API是怎么实现的。

## Composition Functions
![](https://p2.ssl.qhimg.com/t01b41cd8e633003da5.png)

是不是很棒，使用了我们熟悉的函数式编程，结构非常清晰，代码量也减少了很多，最重要的是灵活性和复用性很友好。也不用担心会命名冲突的问题。

### 组件中的使用
![](https://p1.ssl.qhimg.com/t01c66ed230f8a7ef3f.png)

图中不同颜色代表不同逻辑的业务代码，相同颜色表示同一业务需要的代码。我们在写一个比较复杂的组件的时候，有没有这种体会。组件里面包含很多个小的功能组件，这些功能可能在其他页面也有用到，但是数据强依赖这个组件的接口。所以只能分开，单独写到这个组件里，导致组件代码又臭又长，还没法复用。遇到这种情况，我们也只能望洋兴叹。在vue3.0出来之后，我们就可以完美的解决这种问题。我们可以任意在同一组件里，写不同的业务逻辑，不仅层次分明，逻辑清楚，而且神奇的是，你可以信手拈来，拿去复用在其他组件里。你可能也不再需要为了一个页面，去封装十几个小组件了。费神费力，还容易出错。

使用Composition API写出的代码，后期筛查问题、维护和update上，体验也是极度舒适的。

当然，我们不用担心之前的Options API不能用，vue3做了兼容，可以一起使用的。建议有了它就不要再考虑使用其他方案了。

具体使用方法可以查看文档：[https://composition-api.vuejs.org/#summary](https://composition-api.vuejs.org/#summary)

# Fragments

碎片，这个我理解是不再限制组件中必须提供根节点。你可以这么写

```
<div>vue3</div>
text
```

也可以这么写
```
vue3
```

其实看编译的代码，你会发现，vue3只是帮你生成了一个根节点来包裹。这么做的好处就是你省事了，腾出手来做你喜欢做的，vue3就像你的爱人一样，默默地帮你做了许多。

# Teleport

这个特性，我理解相当于”占位“。看下代码

```
<!-- 占位，专门用来显示定制内容 -->
<div id="modal-container"></div>

<!-- 父页面 -->
<div id="app"></div>

<template>
  <div>
    ...
    
    <!-- 调用Teleport -->
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

这里，id是modal-container的模块会在根页面里占位，在调用的时候插入<Teleport></Teleport>模块。这么做可以避免由于z-index层级问题导致的显示异常。

我认为这个特性不仅仅这么简单，其他应用场景有待挖掘。

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

这里实现一个过渡效果，使用Suspense实现，更简单直接一些。

这个特性的应用场景，也可能是图片或者模块的顺序加载。对于业务逻辑比较复杂，需要异步加载静态节点的时候，可以考虑使用。

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

vue3.0版本里，采用了TypeScript重写，意味着之后会全面拥抱TypeScript。类型提示、检测都更加的友好强大。废弃了Class，如果你还是喜欢Class的用法，可以安装单独库vue-class-component支持。

# 结语

以上对vue3主要的新特性做了简单的总结，也加入了一些个人的理解，目前还不能放手使用，只能算尝鲜阶段，所以如果想更深入了解，要仔细研读下vue3的源码。尤大大在直播问答环节，表示可能需要年终才会发布正式版本，所以还有一段时间来完善框架，让vue3更饱满的呈现给众人。后续还需要做的工作主要包括，文档编写，完善开发者工具，以及自动化迁移工具等。有大量的工作要做，是为了给vue2升级vue3做准备。尤大大在直播中也提及，未来会有一个过渡版本发布，这也是最后一个vue2版本vue2.7，计划在不损坏兼容性的前提下，加入一些好的vue3的功能，把它们提前合并入2.7版本，提供给开发者使用体验。而在这之后18个月，表示只会进行安全性的描述，不在继续维护。

而关于vue2升级到vue3，作者也友情提示大家，对于业务逻辑偏重的项目，请一定酌情考虑升级，自觉规避风险，言下之意就是说在全面拥抱vue3之前，可能你需要做一些取舍。

最后，我猜想，Composition API是有极大可能提前加入到2.7版本，提前让大家体验，那就让我们敬请期待吧。
