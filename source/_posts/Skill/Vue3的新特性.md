---
title: Vue3的新特性
categories: skill
tags: [Vue3]
---

## 新的特性、代码升级

vue3 组件模板标签可以不写 div 根标签

setup是所有 Composition API（组合API）“表演的舞台”

组件中用到的数据、方法等等，均要配置在 setup 中。
setup的两种返回值

1. 若返回一个对象，则对象中的属性、方法在模板中均可以使用（重点）
2. 若返回一个渲染函数。则可以自定义渲染内容

如果按照配置项（vue2.x）去写， setup 的执行时机在 beforeCreate() 之前。

> **注意：**
> 1、尽量不要与 Vue2.x 配置混用
> 2、在setup中不能访问到 Vue2.x 配置，vue2.x的配置可以获取setup中的数据。
> 3、如果有重名，setup优先（以setup为主）。
> 配置项：即按照 vue2.x 的语法去写，例如 data(){}, methods() {}

### reactive 和 ref

从定义数据角度对比：
ref 用来定义基本数据类型；reactive 用来定义：对象，数组类型数据
ref 也可以用来定义对象或数组类型数据，它内部会自动通过 reactive 转为代理对象。

从原理角度对比：
ref 通过 Object.defineProperty() 的 get 与 set 来实现响应式（数据劫持）
reactive 通过使用 Proxy 来实现响应式（数据劫持），并通过 Reflect 操作源对象内部的数据。
从使用角度对比：
ref 定义的数据，操作数据需要 `.value`； 读取数据模板中的数据直接读取，不需要 `.value`。
reactive 定义的数据，操作与读取数据时均不需要 `.value`

### watch与watchEffect

watch 既需要指明监视的属性，也需要指明监视的回调
watchEffect：不需要指明监视的属性，监视的回调中用到哪个属性，那就监视哪个属性。
watchEffect 有点像 computed：
但 computed 注重计算出来的值（回调函数的返回值），所以需要返回值；
而 watchEffect 更注重的是过程（回调函数的函数体），所以不用写返回值。

### 自定义hook

自定义 hook 函数，类似于 vue2.x 的 mixin
优势：使用自定义函数，复用代码，让 setup 中的逻辑更加清晰易懂。

### toRef、toRefs

toRef toRefs:
相当于返回对象的引用：创建一个 ref 对象，其 value 值指向另一个对象中的某个属性。
应用：要将响应式对象中的某个属性单独提供给外部使用时。
扩展：toRef 与 toRefs 功能一致，但可以批量创建多个 ref 对象，语法 `toRefs(person)`

### 其它的组合API

shallowReactive:只处理对象最外层的属性的响应式（浅响应式）
shallowRef：只处理基本数据类型的响应式，不进行对象的响应式处理。
什么时候使用？

- 如果有一个对象数据，结构比较深，但变化时只是外层数据变化，shallowReactive
- 如果一个对象数据，后续功能不会直接修改该对象的属性，而是生成新的对象来替换，shallowRef

纸上得来终觉浅，绝知此事要躬行。
