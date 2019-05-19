---
title: 从源码看 Vue 中的 Mixin
date: 2019-05-19 17:54:38
categories:
    -   Vue
    -   JavaScript
---

最近在做项目的时候碰到了一个奇怪的问题，通过 `Vue.mixin` 方法注入到 Vue 实例的一个方法不起作用了，后来经过仔细排查发现这个实例自己实现了一个同名方法，导致了 `Vue.mixin` 注入方法的失效。后来查阅资料发现 `Vue.mixin` 注入到实例的 `methods` 方法会被实例中的同名方法替换，而不会依次执行。于是我就有了查看源码的想法，进而诞生了这篇文章～

> 本文所用源码版本为  2.2.6

首先从 `Vue.mixin` 这个方法入手，打开 `src` 目录不难找到 `mixin` 所在的文件：`src/core/global-api/mixin.js`，其内容如下：

![](https://www.cnblogs.com/images/cnblogs_com/DM428/1358673/o_code1.png)

<!-- more -->

可以看到这只是一层简单的封装，核心内容基本都在 `mergeOptions` 方法中，所以下面打开这个方法所在的文件：`src/core/util/options.js`。注意 `mergeOptions` 方法是通过 `src/core/util/index.js` 引入导出的，其源码在 `options.js` 中，直接看 `options.js` 就好了。

在 `options.js` 中找到 `mergeOptions` 方法，内容如下：

![](https://www.cnblogs.com/images/cnblogs_com/DM428/1358673/o_code2.png)

其主流程大致如下：

1. 如果是非生产环境下，首先调用 `checkComponents` 检查传入参数的合法性，后面再讲具体实现。
2. 调用 `normalizeProps` 方法和 `normalizeDirectives` 方法对这两个属性进行规范化。
3. 检查传入参数是否具有 `extends` 属性，这个属性表示扩展其它 Vue 实例，具体参考[官方文档](<https://cn.vuejs.org/v2/api/#extends>)。这里为什么要检查这个属性呢？因为当传入对象具有该属性时，表示所有的 Vue 实例都要扩展它所指定的实例（`Vue.mixin` 的功能即是如此），那么我们在合并之前，需要先把 `extends` 进行合并，如果 `extends` 是一个 Vue 构造函数（也可能是扩展后的 Vue 构造函数），那么合并参数变为其 `options` 选项了；否则直接合并 `extends`。
4. 检查完传入参数的 `extends` 属性之后，我们还要检查其 `mixins` 属性，这个属性的功能参考[官方文档](https://cn.vuejs.org/v2/api/#mixins)。因为如果传入的 Vue 配置对象仍然指定了 `mixins` 的话，我们需要递归的进行 merge。
5. 做完以上的工作之后，就可以开始合并单纯的 `mixin` 参数了。可以看到通过 `mergeField` 函数进行了合并，先遍历合并的目标对象，进行合并了；随后遍历要合并的对象，只对目标对象上不存在的属性进行合并操作。那么合并的重点就到了 `mergeFiled` 函数了。

继续看 `mergeField` 函数：

```js
function mergeField (key) {
    const strat = strats[key] || defaultStrat
    options[key] = strat(parent[key], child[key], vm, key)
}
```

该函数通过 `key` 值在 `strats` 中选取合并的具体函数，这是一种典型的策略模式，所以我们看 `strats`是如何定义的。

`options.js` 中关于 `strats` 的定义如下：

```js
/**
 * Option overwriting strategies are functions that handle
 * how to merge a parent option value and a child option
 * value into the final value.
 */
const strats = config.optionMergeStrategies
```

其中 `config` 对象来自于 `src/core/config.js`，它定义了 `config` 的所有类型及初始值，当然初始值都还是一些空数组之类的，所以我们要在 `options.js` 中看具体的实现。

下面根据 Vue 的配置属性分开讲解不同的合并方式。

### 一、el

`el` 的合并方式比较简单，因为它本身

源码如下：

```js
/**
 * Options with restrictions
 */
if (process.env.NODE_ENV !== 'production') {
  strats.el = strats.propsData = function (parent, child, vm, key) {
    if (!vm) {
      warn(
        `option "${key}" can only be used during instance ` +
        'creation with the `new` keyword.'
      )
    }
    return defaultStrat(parent, child)
  }
}
```

可以看到这里有个条件，只有在开发环境下才会定义 `strats.el` 方法以及 `propsData` 方法（[propsData](<https://cn.vuejs.org/v2/api/#propsData>) 文档），这是因为这两个属性比较特殊，尤其是 `propsData` 只在开发环境下才使用，方便测试而已。另外一个比较特殊的地方是这两者只能在 `new` 操作符调用 Vue 构造函数所构造的 Vue 实例中才能存在，所以当 `vm` 未传递时，会弹出一个警告。

这两个属性的合并方法都是 `defaultStrat`，其源码如下：

```js
/**
 * Default strategy.
 */
const defaultStrat = function (parentVal: any, childVal: any): any {
  return childVal === undefined
    ? parentVal
    : childVal
}
```

可以看出在 `childVal` 已定义的时候直接替代 `parentVal`。

这个方法在后边还会用到。

### 二、data

`data`选项的合并是重中之重，因为 `data` 在子组件中是一个函数，它返回的也是一个特殊的响应式对象。

其源码如下：

![](https://www.cnblogs.com/images/cnblogs_com/DM428/1358673/o_code3.png)

这里分了两种情况，一种是传递了 vm 参数，一种是没传递。

当没传递 vm 参数的时候，需要校验 `childVal` 是否是函数，而 `parentVal` 不需要校验，因为它必须是函数才能通过之前的 merge 校验，到达现在这一步。确定都是函数之后，就调用这两个函数，再然后对返回的两个 data 对象通过 `mergeData` 做处理，这里后面再讲。

当传递了 vm 参数的时候，需要用其他方式处理，当是函数的时候，使用返回值做下一步合并；当是其他值的时候，直接使用其值进行下一步合并。

这一步要校验 `childVal `和 `parentVal` 是否为函数。正是因为这一步校验了，所以前面所讲的情况就不再需要校验，为什么呢？

我们可以回头看 `mergeOptions` 的源码，发现其第三个参数 vm 是可选的，在递归的时候它会把 vm 传递给自身，这就导致当我们一开始调用 `mergeOptions` 的时候传递了 vm，则其后所有递归都会传递 vm；当我们一开始未传递 vm 值的时候，其后所有的递归也不会传递 vm 参数。那么是否有 vm 就取决于我们最开始调用该函数时所传递的参数是否包含 vm 了。

全局查找 `mergeOptions` 函数的调用，可以看到有两处：

1. 第一处位于 `src/core/instance/init.js`，该文件也定义了 `initMixin` 方法，用于初始化 Vue 把传递给 Vue 构造函数的配置对象合并到 vm.$options 中。这种情况下会传递 vm，其值为当前正在构造的 Vue 实例。
2. 第二处位于之前一直在讲的 `src/core/global-api/mixin.js`，这处才是定义的全局 API。

简而言之，Vue 构造函数构造 Vue 实例时，会调用 `mergeOptions` 并且传递 vm 实例作为第三个参数；当我们调用 `Vue.mixin` 进行全局混淆时是不会传递 vm 的。前者对应第二种情况，后者对应第一种情况。

当我们先构造 Vue 实例的时候，vm 被传递进而执行第二种情况，`parentVal` 会被校验，所以之后再调用 `Vue.mixin` 时第一种情况不再需要校验。

当我们先不实例化 Vue 而先调用 `Vue.mixin` 时，会先执行第一种情况的代码，那么会导致 bug 出现吗？答案肯定是不会，因为此时 `parentVal` 为 `undefined`，因为 `Vue.mixin` 调用时 `parentVal` 的初始值为 `Vue.options`，这个对象根本不包含 data 属性。

那么 data 合并的任务主要在 `mergeData` 函数中了，查看其源码：

![](https://www.cnblogs.com/images/cnblogs_com/DM428/1358673/o_code4.png)

可以看到这里遍历了要合并的 data 的所有属性，然后根据不同情况进行合并：

1. **当目标 data 对象不包含当前属性时**，调用 `set` 方法进行合并，后面讲 `set`。
2. **当目标 data 对象包含当前属性并且当前值为纯对象时**，递归合并当前对象值，这样做是为了防止对象存在新增属性。

继续看 `set` 函数：

![](https://www.cnblogs.com/images/cnblogs_com/DM428/1358673/o_code5.png)

可以看到 `set` 也对 target 分了两种情况进行处理。首先判断了 target 是数组的情况，然后如果 target 包含当前属性，那么就直接赋值。接下来判断了 target 是否是响应式对象，如果是的话就会在开发环境下弹出警告，最好不要让 data 函数返回一个响应式对象，因为会造成性能浪费。如果不是响应式对象也可以直接赋值返回，其他情况下就会进一步转化 target 为响应式对象，并收集依赖。

以上大概就是 data 的合并方式，可以看出来如果实例指定了与 mixins 相同名称的 data 值，那么以实例中的为准，mixin 中执行的 data 会失效，如果都是对象但是 mixin 中新增了属性的话，还是会被添加到实例 data 中去的。

### 三、生命周期钩子（Hooks）

Hooks 的合并函数定义为 `mergeHook` 钩子，其源码如下：

```js
/**
 * Hooks and props are merged as arrays.
 */
function mergeHook (
  parentVal: ?Array<Function>,
  childVal: ?Function | ?Array<Function>
): ?Array<Function> {
  return childVal
    ? parentVal
      ? parentVal.concat(childVal)
      : Array.isArray(childVal)
        ? childVal
        : [childVal]
    : parentVal
}
```

这个比较简单，代码注释也写得很清楚了，Vue 实例的生命周期钩子被合并为一个数组。具体有哪些钩子可以被合并被写在 `src/core/config.js` 中：

```js
/**
   * List of lifecycle hooks.
   */
_lifecycleHooks: [
    'beforeCreate',
    'created',
    'beforeMount',
    'mounted',
    'beforeUpdate',
    'updated',
    'beforeDestroy',
    'destroyed',
    'activated',
    'deactivated'
],
```

合并 assets （components、filters、directives）的方法也比较简单，下面跳过了。

### 四、watch

合并 `watch` 的函数源码如下：

![](https://www.cnblogs.com/images/cnblogs_com/DM428/1358673/o_code6.png)

这一段源码也很简单，注释也很明了，跟生命周期的钩子一样，`Vue.mixin` 会把所有同名的 watch 合并到一个数组中去，在触发的时候依次执行就好了。

### 五、props、methods、computed

这三项的合并都使用了相同的策略，源代码如下：

![](https://www.cnblogs.com/images/cnblogs_com/DM428/1358673/o_code7.png)

这里的处理也比较简单，可以看出来当多次调用 Vue.mixin 混淆时，同名的 props、methods、computed 会被后来者替代；但是当 Vue 构造函数传递了同名的属性时，会以构造函数所接受的配置对象为准。因为 Vue 实例化时也会调用 mergeOptions 第二个参数即为 Vue 构造函数所接受的配置对象，正如前文所述。

### 六、一些辅助函数

前文有讲到几个辅助函数，比如：`checkComponents`、`normalizeProps`、`normalizeDirectives`。这里简单贴一下源码：

#### checkComponents

![](https://www.cnblogs.com/images/cnblogs_com/DM428/1358673/o_code8.png)

这个函数是为了检查 components 属性是否符合要求的，主要是防止自定义组件使用 HTML 内置标签。

#### normalizeProps

![](https://www.cnblogs.com/images/cnblogs_com/DM428/1358673/o_code9.png)

这个函数主要是对 props 属性进行整理。包括把字符串数组形式的 props 转换为对象形式，对所有形式的 props 进行格式化整理。

#### normalizeDirectives

![](https://www.cnblogs.com/images/cnblogs_com/DM428/1358673/o_code10.png)

这个函数也主要是对 directives 属性进行格式化整理的，把原来的对象整理成一个新的符合标准格式的对象。

### 七、自定义合并策略

看到 Vue 的官方文档：[自定义选项合并策略](https://cn.vuejs.org/v2/guide/mixins.html#%E8%87%AA%E5%AE%9A%E4%B9%89%E9%80%89%E9%A1%B9%E5%90%88%E5%B9%B6%E7%AD%96%E7%95%A5)，它允许我们自定义合并策略，具体方式就是替换 `Vue.config.optionsMergeStrategies`，也就是前文所提到的那个定义在 `src/core/config.js` 中的属性。我们也可以看一下源代码，这一功能在 `src/core/global-api/index.js` 文件中的 `initGlobalAPI` 定义。

```js
const configDef = {}
configDef.get = () => config
if (process.env.NODE_ENV !== 'production') {
    configDef.set = () => {
        warn(
            'Do not replace the Vue.config object, set individual fields instead.'
        )
    }
}
Object.defineProperty(Vue, 'config', configDef)
```

可以看到最后一句给 Vue 函数定义了一个 `config` 属性，其 property 定义为 `configDef`。在生产环境下不允许设置其值，但是在开发环境下，我们可以直接设置 `Vue.config`。那么通过设置 `Vue.config.optionsMergeStrategies`，我们可以改变合并策略，在后面再进行合并操作时，都会读取 config 对象中的属性，这时就可以使用我们自定义的合并策略进行合并了。

### 八、总结

看了这些属性的合并方式以后，对 `Vue.mixin` 的工作方式也有了一定的了解了。个人认为基本上可以把 `Vue.mixin` 合并属性的方式分为三类，一类是替换式、一类是合并式、还有一类是队列式。

替换式的有 `el`、`props`、`methods` 和 `computed`，这一类的行为是新的参数替代旧的参数。

合并式的有 `data`，这一类的行为是新传入的参数会被合并到旧的参数中。

队列式合并的有 `watch`、所有的生命周期钩子（`hooks`），这一类的行为是所有的参数会被合并到一个数组中，必要时再依次取出。

所以对于 `Vue.mixin` 的使用我们也需要小心，尤其是替换式合并的属性，当你在 mixins 里面指定了以后，就不要再实例中再指定同名属性了，那样的话你的 mixins 中的属性会被替代导致失效。

作者水平有限，文章难免存在纰漏，敬请大家指正。