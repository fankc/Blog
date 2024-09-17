# Vue2 与 Vue3 的区别

## 1. API 类型不同

### 选项式 API

Vue2 使用选项式 API，在代码中分割了不同的属性：`data`，`computed`，`methods`等。

**选项式 API 特点**：理解容易，好上手。因为各个选项都有固定的书写位置（比如数据就写到 `data` 中，操作方法就写到 `methods` 中，等等），应用大了之后，相信大家都遇到过来回上下找代码的困境。主要的**逻辑复用机制**是 `mixins`（混入）。

#### mixins

`mixins` 选项接受一个 mixin 对象数组。这些 mixin 对象可以像普通的实例对象一样包含实例选项，它们将使用一定的选项合并逻辑与最终的选项进行合并：

1. 值为对象（`components`、`methods`、`computed`、`data`）的选项，混入当前组件时选项会被合并。键冲突时，当前组件中的键会覆盖混入对象的。
2. 值为函数（`created`、`mounted`）的选项，混入当前组件时选项会被合并调用，混合对象里的钩子函数会在当前组件的钩子函数之前调用。

mixins 的方法和参数在各组件中不共享。

mixins 有三个主要的短板：

1. **不清晰的数据来源**：当使用了多个 mixin 时，实例上的数据属性来自哪个 mixin 变得不清晰，这使追溯实现和理解组件行为变得困难。
2. **命名空间冲突**：多个来自不同作者的 mixin 可能会注册相同的属性名，造成命名冲突。
3. **隐式的跨 mixin 交流**：多个 mixin 需要依赖共享的属性名来进行相互作用，这使得它们隐性地耦合在一起。

### 组合式 API

Vue3 除了选项式 API，还支持组合式 API。组合式 API 能够通过**组合式函数**来实现更加简洁高效的**逻辑复用**。

**组合式 API 特点**：特定功能相关的所有东西都放到一起维护，比如功能 A 相关的响应式数据，操作数据的方法等放到一起，这样不管应用多大，都可以快读定位到某个功能的所有相关代码，维护方便。如果功能复杂，代码量大，还可以通过组合函数进行逻辑拆分处理。

#### 组合式函数

组合式函数是一个利用 Vue 的组合式 API 来封装和复用**有状态逻辑**的函数。

当构建前端应用时，我们常常需要复用公共任务的逻辑。例如为了在不同地方格式化时间，我们可能会抽取一个可复用的日期格式化函数。这个函数封装了**无状态的逻辑**：它在接收一些输入后立刻返回所期望的输出。相比之下，**有状态逻辑**负责管理会随时间而变化的状态。

鼠标跟踪器示例：

如果我们要直接在组件中使用组合式 API 实现鼠标跟踪功能，它会是这样的：

```javascript
<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const x = ref(0)
const y = ref(0)

function update(event) {
  x.value = event.pageX
  y.value = event.pageY
}

onMounted(() => window.addEventListener('mousemove', update))
onUnmounted(() => window.removeEventListener('mousemove', update))
</script>

<template>Mouse position is at: {{ x }}, {{ y }}</template>
```

但是，如果我们想在多个组件中复用这个相同的逻辑呢？我们可以把这个逻辑以一个组合式函数的形式提取到外部文件中：

```javascript
// mouse.js
import { ref, onMounted, onUnmounted } from 'vue'

export function useMouse() {
  // 被组合式函数封装和管理的状态
  const x = ref(0)
  const y = ref(0)

  // 组合式函数可以随时更改其状态。
  function update(event) {
    x.value = event.pageX
    y.value = event.pageY
  }

  // 一个组合式函数也可以挂靠在所属组件的生命周期上来启动和卸载副作用
  onMounted(() => window.addEventListener('mousemove', update))
  onUnmounted(() => window.removeEventListener('mousemove', update))

  // 通过返回值暴露所管理的状态
  return { x, y }
}
```

下面是它在组件中使用的方式：

```javascript
<script setup>
import { useMouse } from './mouse.js'

const { x, y } = useMouse()
</script>

<template>Mouse position is at: {{ x }}, {{ y }}</template>
```

## 2. 更好的 TypeScript 支持

Vue3 是基于 TypeScipt 编写的，可以享受到自动的类型定义提示

## 3. 生命周期钩子函数不同

| Vue2          | Vue3          | Vue3 组合式 API |
| ------------- | ------------- | --------------- |
|               | setup         |                 |
| beforeCreate  | beforeCreate  |                 |
| created       | created       |                 |
| beforeMount   | beforeMount   | onBeforeMount   |
| mounted       | mounted       | onMounted       |
| beforeUpdate  | beforeUpdate  | onBeforeUpdate  |
| updated       | updated       | onUpdated       |
| beforeDestroy | beforeUnmount | onBeforeUnmount |
| destroyed     | unmounted     | onUnmounted     |
| errorCaptured | errorCaptured | onErrorCaptured |
| activated     | activated     | onActivated     |
| deactivated   | deactivated   | onDeactivated   |

## 4. 双向数据绑定原理不同

**Vue2**：Vue2 的双向数据绑定是利用 ES5 的 `Object.defineProperty` 对数据进行劫持（遍历对象所有属性，监听各属性的 `getter`、`setter`），结合发布订阅模式的方式来实现的。

使用 `Object.defineProperty` 劫持对象和数组存在的问题：

1. **对于对象**，Vue2 无法检测 property 的添加或移除。对于已经创建的实例，Vue2 不允许动态添加根级别的响应式 property。但是，可以使用 `Vue.set(object, propertyName, value)` 方法向嵌套对象添加响应式 property，其原理也是利用 `Object.defineProperty`。

    ```javascript
    var vm = new Vue({
    data:{
        a:1 // `vm.a` 是响应式的
    }
    })
    vm.b = 2 // `vm.b` 是非响应式的
    Vue.set(vm.someObject, 'b', 2) // `vm.someObject.b` 是响应式的
    ```

2. **对于数组**，Vue2 不能检测以下数组的变动：
   1. 利用索引直接设置一个数组项时，例如：`vm.items[indexOfItem] = newValue`。可以通过 `Vue.set` 或 `Array.prototype.splice` 解决。
        ```javascript
        // Vue.set
        Vue.set(vm.items, indexOfItem, newValue)
        // Array.prototype.splice
        vm.items.splice(indexOfItem, 1, newValue)
        ```
   2. 修改数组的长度时，例如：`vm.items.length = newLength`。可以通过 `Array.prototype.splice` 解决。
        ```javascript
        vm.items.splice(newLength)
        ```

**Vue3**：Vue3 中使用了 ES6 的 `Proxy` 对数据代理。`Proxy` 对象用于创建一个对象的代理，从而实现基本操作的拦截和自定义（如属性查找、赋值、枚举、函数调用等）。语法：

```javascript
const p = new Proxy(target, handler)
```

## 5. 是否支持碎片

Vue2 不支持碎片（Fragments）。每个组件只能有一个根，所以，写每个组件模板时都要套一个父元素。

```html
<template>
  <div>
    <h1>标题</h1>
    <p>正文内容</p>
  </div>
</template>
```

Vue3 支持碎片，可以拥有多个根节点。

```html
<template>
  <h1>标题</h1>
  <p>正文内容</p>
</template>
```

## 6. Diff 算法不同

### Vue2 —— 双端 diff 算法

1. 首先进行首尾对比，这样找到的可复用节点一定是性能最优（即原地复用 DOM 节点，不需要移动）。
2. 首尾对比完交叉对比，这一步即寻找移动后可复用的节点。
3. 然后在剩余结点中对比寻找可复用 DOM，为了快速对比，会创建一个 map 记录 key，然后通过 key 查找旧的 DOM。
4. 最后进行善后工作，即移除多余节点或者新增节点。

```javascript
function updateChildren (parentElm, oldCh, newCh, insertedVnodeQueue, removeOnly) {
  let oldStartIdx = 0
  let newStartIdx = 0
  let oldEndIdx = oldCh.length - 1
  let oldStartVnode = oldCh[0]
  let oldEndVnode = oldCh[oldEndIdx]
  let newEndIdx = newCh.length - 1
  let newStartVnode = newCh[0]
  let newEndVnode = newCh[newEndIdx]
  let oldKeyToIdx, idxInOld, vnodeToMove, refElm
​
  // removeOnly is a special flag used only by <transition-group>
  // to ensure removed elements stay in correct relative positions
  // during leaving transitions
  const canMove = !removeOnly
​
  if (process.env.NODE_ENV !== 'production') {
    checkDuplicateKeys(newCh)
  }
​
  while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
    if (isUndef(oldStartVnode)) {
      oldStartVnode = oldCh[++oldStartIdx] // Vnode has been moved left
    } else if (isUndef(oldEndVnode)) {
      oldEndVnode = oldCh[--oldEndIdx]
    } else if (sameVnode(oldStartVnode, newStartVnode)) {
      patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue)
      oldStartVnode = oldCh[++oldStartIdx]
      newStartVnode = newCh[++newStartIdx]
    } else if (sameVnode(oldEndVnode, newEndVnode)) {
      patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue)
      oldEndVnode = oldCh[--oldEndIdx]
      newEndVnode = newCh[--newEndIdx]
    } else if (sameVnode(oldStartVnode, newEndVnode)) { // Vnode moved right
      patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue)
      canMove && nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm))
      oldStartVnode = oldCh[++oldStartIdx]
      newEndVnode = newCh[--newEndIdx]
    } else if (sameVnode(oldEndVnode, newStartVnode)) { // Vnode moved left
      patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue)
      canMove && nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm)
      oldEndVnode = oldCh[--oldEndIdx]
      newStartVnode = newCh[++newStartIdx]
    } else {
      if (isUndef(oldKeyToIdx)) oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx)
      idxInOld = isDef(newStartVnode.key)
        ? oldKeyToIdx[newStartVnode.key]
        : findIdxInOld(newStartVnode, oldCh, oldStartIdx, oldEndIdx)
      if (isUndef(idxInOld)) { // New element
        createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
      } else {
        vnodeToMove = oldCh[idxInOld]
        if (sameVnode(vnodeToMove, newStartVnode)) {
          patchVnode(vnodeToMove, newStartVnode, insertedVnodeQueue)
          oldCh[idxInOld] = undefined
          canMove && nodeOps.insertBefore(parentElm, vnodeToMove.elm, oldStartVnode.elm)
        } else {
          // same key but different element. treat as new element
          createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
        }
      }
      newStartVnode = newCh[++newStartIdx]
    }
  }
  if (oldStartIdx > oldEndIdx) {
    refElm = isUndef(newCh[newEndIdx + 1]) ? null : newCh[newEndIdx + 1].elm
    addVnodes(parentElm, refElm, newCh, newStartIdx, newEndIdx, insertedVnodeQueue)
  } else if (newStartIdx > newEndIdx) {
    removeVnodes(parentElm, oldCh, oldStartIdx, oldEndIdx)
  }
}

```

### Vue3 —— 快速 diff 算法

1. 从头部开始遍历，若新旧节点相同直接复用，否则跳出循环。
2. 然后从尾部开始遍历，若新旧节点相同直接复用，否则跳出循环。
3. 如果旧节点遍历完了，依然有新的节点，那么剩下的新节点就是要新增的
4. 如果新节点遍历完了, 还有旧的节点, 那么剩下的旧节点就是要移除的
5. 如果都没遍历完：
   1. 创建一个新节点与旧节点的位置映射表，无法与新节点映射的旧节点直接被移除。
   2. 然后根据这个映射表计算出最长递增子序列，这个序列中的结点代表可以原地复用，不需要移动。之后移动（或新增不在映射表中的新节点）剩下的新结点到正确的位置即递增序列的间隙中。

```javascript
  const patchKeyedChildren = (
    c1: VNode[],
    c2: VNodeArrayChildren,
    container: RendererElement,
    parentAnchor: RendererNode | null,
    parentComponent: ComponentInternalInstance | null,
    parentSuspense: SuspenseBoundary | null,
    isSVG: boolean,
    slotScopeIds: string[] | null,
    optimized: boolean
  ) => {
    let i = 0
    const l2 = c2.length
    // 旧子节点数组长度
    let e1 = c1.length - 1 // prev ending index
    // 新子节点数组长度
    let e2 = l2 - 1 // next ending index
​
    // 1. sync from start
    // 从头部开始遍历
    // (a b) c
    // (a b) d e
    while (i <= e1 && i <= e2) {
      const n1 = c1[i]
      const n2 = (c2[i] = optimized
        ? cloneIfMounted(c2[i] as VNode)
        : normalizeVNode(c2[i]))
      // 如果节点相同, 那么就继续遍历
      if (isSameVNodeType(n1, n2)) {
        patch(
          n1,
          n2,
          container,
          null,
          parentComponent,
          parentSuspense,
          isSVG,
          slotScopeIds,
          optimized
        )
      } else {
        // 节点不同就直接跳出循环
        break
      }
      i++
    }
​
    // 2. sync from end
    // 从尾部开始遍历
    // a (b c)
    // d e (b c)
    while (i <= e1 && i <= e2) {
      const n1 = c1[e1]
      const n2 = (c2[e2] = optimized
        ? cloneIfMounted(c2[e2] as VNode)
        : normalizeVNode(c2[e2]))
      // 如果节点相同, 就继续遍历
      if (isSameVNodeType(n1, n2)) {
        patch(
          n1,
          n2,
          container,
          null,
          parentComponent,
          parentSuspense,
          isSVG,
          slotScopeIds,
          optimized
        )
      } else {
        // 不同就直接跳出循环
        break
      }
      e1--
      e2--
    }
​
    // 3. common sequence + mount
    // 如果旧节点遍历完了, 依然有新的节点, 那么新的节点就是添加(mount)
    // (a b)
    // (a b) c
    // i = 2, e1 = 1, e2 = 2
    // (a b)
    // c (a b)
    // i = 0, e1 = -1, e2 = 0
    if (i > e1) {
      if (i <= e2) {
        const nextPos = e2 + 1
        const anchor = nextPos < l2 ? (c2[nextPos] as VNode).el : parentAnchor
        while (i <= e2) {
          patch(
            null,
            (c2[i] = optimized
              ? cloneIfMounted(c2[i] as VNode)
              : normalizeVNode(c2[i])),
            container,
            anchor,
            parentComponent,
            parentSuspense,
            isSVG,
            slotScopeIds,
            optimized
          )
          i++
        }
      }
    }
​
    // 4. common sequence + unmount
    // 如果新的节点遍历完了, 还有旧的节点, 那么旧的节点就是移除的
    // (a b) c
    // (a b)
    // i = 2, e1 = 2, e2 = 1
    // a (b c)
    // (b c)
    // i = 0, e1 = 0, e2 = -1
    else if (i > e2) {
      while (i <= e1) {
        unmount(c1[i], parentComponent, parentSuspense, true)
        i++
      }
    }
​
    // 5. unknown sequence
    // 如果是位置的节点序列
    // [i ... e1 + 1]: a b [c d e] f g
    // [i ... e2 + 1]: a b [e d c h] f g
    // i = 2, e1 = 4, e2 = 5
    else {
      const s1 = i // prev starting index
      const s2 = i // next starting index
​
      // 5.1 build key:index map for newChildren
      const keyToNewIndexMap: Map<string | number, number> = new Map()
      for (i = s2; i <= e2; i++) {
        const nextChild = (c2[i] = optimized
          ? cloneIfMounted(c2[i] as VNode)
          : normalizeVNode(c2[i]))
        if (nextChild.key != null) {
          if (__DEV__ && keyToNewIndexMap.has(nextChild.key)) {
            warn(
              `Duplicate keys found during update:`,
              JSON.stringify(nextChild.key),
              `Make sure keys are unique.`
            )
          }
          keyToNewIndexMap.set(nextChild.key, i)
        }
      }
​
      // 5.2 loop through old children left to be patched and try to patch
      // matching nodes & remove nodes that are no longer present
      let j
      let patched = 0
      const toBePatched = e2 - s2 + 1
      let moved = false
      // used to track whether any node has moved
      let maxNewIndexSoFar = 0
      // works as Map<newIndex, oldIndex>
      // Note that oldIndex is offset by +1
      // and oldIndex = 0 is a special value indicating the new node has
      // no corresponding old node.
      // used for determining longest stable subsequence
      const newIndexToOldIndexMap = new Array(toBePatched)
      for (i = 0; i < toBePatched; i++) newIndexToOldIndexMap[i] = 0
​
      for (i = s1; i <= e1; i++) {
        const prevChild = c1[i]
        if (patched >= toBePatched) {
          // all new children have been patched so this can only be a removal
          unmount(prevChild, parentComponent, parentSuspense, true)
          continue
        }
        let newIndex
        if (prevChild.key != null) {
          newIndex = keyToNewIndexMap.get(prevChild.key)
        } else {
          // key-less node, try to locate a key-less node of the same type
          for (j = s2; j <= e2; j++) {
            if (
              newIndexToOldIndexMap[j - s2] === 0 &&
              isSameVNodeType(prevChild, c2[j] as VNode)
            ) {
              newIndex = j
              break
            }
          }
        }
        if (newIndex === undefined) {
          unmount(prevChild, parentComponent, parentSuspense, true)
        } else {
          newIndexToOldIndexMap[newIndex - s2] = i + 1
          if (newIndex >= maxNewIndexSoFar) {
            maxNewIndexSoFar = newIndex
          } else {
            moved = true
          }
          patch(
            prevChild,
            c2[newIndex] as VNode,
            container,
            null,
            parentComponent,
            parentSuspense,
            isSVG,
            slotScopeIds,
            optimized
          )
          patched++
        }
      }
​
      // 5.3 move and mount
      // generate longest stable subsequence only when nodes have moved
      const increasingNewIndexSequence = moved
        ? getSequence(newIndexToOldIndexMap)
        : EMPTY_ARR
      j = increasingNewIndexSequence.length - 1
      // looping backwards so that we can use last patched node as anchor
      for (i = toBePatched - 1; i >= 0; i--) {
        const nextIndex = s2 + i
        const nextChild = c2[nextIndex] as VNode
        const anchor =
          nextIndex + 1 < l2 ? (c2[nextIndex + 1] as VNode).el : parentAnchor
        if (newIndexToOldIndexMap[i] === 0) {
          // mount new
          patch(
            null,
            nextChild,
            container,
            anchor,
            parentComponent,
            parentSuspense,
            isSVG,
            slotScopeIds,
            optimized
          )
        } else if (moved) {
          // move if:
          // There is no stable subsequence (e.g. a reverse)
          // OR current node is not among the stable sequence
          if (j < 0 || i !== increasingNewIndexSequence[j]) {
            move(nextChild, container, anchor, MoveType.REORDER)
          } else {
            j--
          }
        }
      }
    }
  }

```

## 7. v-if 和 v-for 的优先级

Vue2 中，两者同时使用，`v-for` 的优先级高于 `v-if`。

Vue3 中，两者同时使用，`v-if` 的优先级高于 `v-for`。

## 8. Vue3 体积更小

通过 `webpack` 的 `tree-shaking` 功能，可以将无用模块“剪辑”，仅打包需要的。

具体地：

1. 第一步：使用 ESLint 和 Typescript 在编译期间进行静态分析，检测出没有被使用的代码，并删除这部分无用的代码。
2. 第二步：通过使用 ES 模块，实现按需加载的效果。ES 模块支持 Tree Shaking，能够将打包好的模块切割成更小的部分，仅加载需要的特定部分。

## 参考

1. [vue2和vue3的区别](https://juejin.cn/post/7185449117999431741)
2. [组合式API和选项式API](https://juejin.cn/post/6985493062432587813)
3. [选项 / 生命周期钩子](https://v2.cn.vuejs.org/v2/api/#%E9%80%89%E9%A1%B9-%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E9%92%A9%E5%AD%90)
4. [生命周期钩子](https://cn.vuejs.org/guide/essentials/lifecycle.html)
5. [组合式 API：setup()](https://cn.vuejs.org/api/composition-api-setup.html)
6. [组合式 API：生命周期钩子](https://cn.vuejs.org/api/composition-api-lifecycle)
7. [生命周期选项](https://cn.vuejs.org/api/options-lifecycle)
8. [深入响应式原理](https://v2.cn.vuejs.org/v2/guide/reactivity.html)
9. [Proxy (MDN)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)
10. [vue2、vue3 diff 算法源码解析](https://juejin.cn/post/7100092670566989861)