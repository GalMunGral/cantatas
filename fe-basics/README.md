- Template compiles to (`{ render: function(createElement) { with(this) { ... } } }`), at runtime or AoT using webpack + `vue-loader`
- **`a {{expression}} b`** -> `_v("a " + _s(expression) + " b")`
- **`v-show="boolExpr"`** -> CSS `display: none` if false.
- **`v-if="aExpr"`, `v-else-if="bExpr"`, `v-else`** -> `(aExpr) ? _c(...) : (bExpr) ? _c(...) : _c(...)`
- **`v-for` + `v-bind:key`**
  - **`v-for="(item, index) in list"`** -> `_l((list), function(item, index) { ... })`
  - **`v-for="(value, name, index) in obj"`** -> `_l((obj), function(value, name, index) { ... })`
  - **Caveat**: Don't use `v-if` on the same tag as `v-for`: `v-for` has higher precedence, `v-if` will be repeated
- Props: **`v-bind:`, `:`**
  - Prop declaration, validation
    - **`{ props: ['aProp', ... ] }`**
    - **`{ props: { aProp: SomeType, ... } }`**
    - **`{ props: { aProp: { type, required, default }, ... } }`**
  - **`:name="expression"`** -> `{ attrs: { "name": expression }}`
  - **`:class="[aClass, bClass]"`** or **`:class="{aClass: aBool}"`**
  - **`:style="{key: val}"`**
- Events: **`v-on:`, `@`**
  - **`@event="method"`** -> `{ on: { "event": method } }`
  - **`@event="[...statements]"`** -> `{ on: { "event": function($event) { [...statements] } }}`
  - Native events: _attached to each DOM node, no delegation_
    - **`@event.modifier="method"`** -> `{ on: { "event": function($event) { if (...) return null; return method($event) } }}`
    - Event modifiers: `.stop`, `.prevent`, `.capture`, `.self` (only when `event.target == event.currentTarget`)
    - Key modifiers: `.ctrl`, `.shift`, `.enter`, `.space`, `.up`, ..., `.exact`
  - Custom events: `@a-event="..."` + **`this.$emit('a-event', ...args)`**
  - Use Vue instance (e.g. `new Vue()`) as _event bus_: **`vm.[$on|$off](eventName, callback)`, `vm.$emit(eventName, ...args)`**
- **`v-model="name"`** -> `{ domProps: { "value": (name) }, on: { "input": function($event) { ... name = $event.target.value } } }`
  - Modifiers: `.lazy` (sync on `change`), `.trim` -> `$event.target.value.trim()`, `.number` -> `_n($event.target.value)`
  - Custom `v-model`: **`{ model: { prop, event }}`**
- **`<slot>`** - compare React _children, render prop_
  - Named (具名插槽): **`<slot name="aName">` + `<template v-slot:aName>`**
  - Scoped (作用域插槽): **`<slot :aProp="aVal">` + `<template v-slot:default="allProps">`**
  - `<template v-slot:name="slotProps">` -> `{ scopedSlots: _u([{ key: "name", fn: function(slotProps) { ... } }]) }`
- Dynamic component: **`<component :is="a-comp"/>`** -> `_c(a-comp, { tag: "component" })`
- Async component: **`{ components: { aComp: () => import('path/to/module') } }`**
- **`ref="a"`, `this.$refs.a`**
- **`v-html`**
- **`{ mixin: [mixinA, mixinB] }`**

#### Reactivity

- **`data: function() { return {...} }`**
- **computed properties (计算属性)**: **`computed: { a() {}, b: { get() {}, set(){} } }`** - values are cached.
- **watchers (侦听器)**: **`watch: { a(val, oldVal) {}, 'aExpr': ['aMethod', { handler(val, oldVal) {}, deep: true }, ...] }`**
  - **Caveat**: For updates in nested objects, `oldVal == newVal`
- **Implementation (v2)**:
  - `Observer` associates each property to a `Dep` instance to manage subscription
  - `Watcher` (one for each component) runs render function after setting `Dep.target` to itself.
  - Issues with `Object.defineProperty` -> Vue 3 uses `Proxy`/`Reflect` API
    - Traversal of properties is expensive
    - Cannot detect property addition/deletion
      - Use `Vue.set(target, key, value)`/`Vue.delete(target, key)`
    - Disabled for arrays (setting items by index, setting `length`) due to performance concerns [[GitHub]](https://github.com/vuejs/vue/issues/8562#issuecomment-408292597)
      - **Implementation**: Array methods are overriden by extending `Array.prototype`
      - Use **`arr.splice(i, 1, newValue)`** to set item
- **Lifecycle**: `constructor`, `beforeCreate`, `created`, `beforeMount`, `mounted`, `beforeUpdate`, `updated`, `beforeDestroy`, `destroyed`
  - `<keep-alive>` (caching) -> additional lifecycles :`activated`/`deactivated`

### Vuex

- `new Vuex.Store({ state, getters, mutations, actions, modules })`
- `store.commit(mutationType, payload)`, `store.dispatch(actionType, payload)`
- component binding helpers: [`mapState`](https://vuex.vuejs.org/guide/state.html#the-mapstate-helper), [`mapGetters`](https://vuex.vuejs.org/guide/getters.html#the-mapgetters-helper), [`mapMutations`](https://vuex.vuejs.org/guide/mutations.html#committing-mutations-in-components), [`mapAction`](https://vuex.vuejs.org/guide/actions.html#dispatching-actions-in-components)

### Absolute positioning

- `position: fixed` - relative to initial containing block or the first ancestor that has `transform` (or `perspective`, `filter`) set
- `left|right|top|bottom: auto` - positioned as if it is static [[MDN]](https://developer.mozilla.org/en-US/docs/Web/CSS/top)
- `width|height: auto` will be set to _shrink-to-fit_
- `display` will be implicitly set to `block`

- Abosolute positioning (`position: absolute`, parent `position: relative`)
  - **`left|right: 0`** (any fixed number) + **`top|bottom: 0`** (any fixed number) + **`margin: auto`** [[W3C spec]](https://drafts.csswg.org/css-position/#abspos-margins)
  - **`left: 50%; top: 50%`** + **`transform: translate(-50%, -50%)`**
  - **`left: 50%; top: 50%`** + **`margin-left: -<width>/2; margin-top: -<height>/2`** (_negative margins: T/L pulls self, B/R pulls neighbor_)
- Flexbox: **`display: flex; align-items: center; justify-content: center;`** (parent)
- Table layout: `display: table-cell; text-align: center; vertical-align: middle` (parent) + `display: inline-block` (self)

### FLow Layout

- `<display-outside>`[[MDN]](https://developer.mozilla.org/en-US/docs/Web/CSS/display-outside) (`block`, `inline`) [[MDN]](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Flow_Layout/Block_and_Inline_Layout_in_Normal_Flow), `<display-inside>`[[MDN]](https://developer.mozilla.org/en-US/docs/Web/CSS/display-inside) (`flow`, `flex`, `grid`, etc.)
  - `display: inline-block` is equivalent to `display: inline flow-root`
- **[Block formatting context (BFC)](https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Block_formatting_context)**
  - Created by setting _`float`,`position` (`absolute`, `fixed`), `overflow` (not `visible`), `display` (`flow-root`, `inline-block`)_
  - Usage: _contain internal floats (**"clearfix"**), exclude external floats, suppress [margin collapsing](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Box_Model/Mastering_margin_collapsing)_
    ```css
    /* 1 - Create new BFC */
    .clearfix {
      overflow: hidden;
    }
    .clearfix {
      display: flow-root;
    }
    /* 2 - Pseudo-element */
    .clearfix::after {
      content: "";
      display: block;
      clear: both;
    }
    ```

`initial` (`0 1 auto`), `auto` (`1 1 auto`), `none` (`0 0 auto`)

### Float 3-Column Layout

```html
<!-- 圣杯布局 -->
<div id="header"></div>
<div id="container">
  <div id="center" class="column"></div>
  <div id="left" class="column"></div>
  <div id="right" class="column"></div>
</div>
<div id="footer"></div>
```

```css
#container {
  width: 100%;
  padding: 0 200px;
}
.column {
  float: left;
}
#center {
  width: 100%;
}
#left {
  margin-left: -100%;
  position: relative;
  right: 200px;
  width: 200px;
}
#right {
  width: 200px;
  margin-right: 200px;
}
#footer {
  clear: both;
}
```

```html
<!-- 双飞翼布局 -->
<div id="container" class="column">
  <div id="main"></div>
</div>
<div id="left" class="column"></div>
<div id="right" class="column"></div>
```

```css
#container {
  width: 100%;
}
.column {
  float: left;
}
#main {
  margin: 0 200px;
}
#left {
  margin-left: 100%;
  width: 200px;
}
#right {
  margin-left: 200px;
  width: 200px;
}
```

### Graph, Tree Traversal

```js
function traverse(root) {
  const visited = new Set();
  function dfs(node) {
    visited.add(node);
    // process node
    for (let n of /* neighbors(node) */) {
      if (!visited.has(n)) {
        dfs(n);
      }
      // process edge node -> n
    }
  }
  dfs(root);
}

function traverse(root) {
  const visited = new Set(); // including those in queue
  function bfs(root) {
    const queue = [root];
    visited.add(root);
    while (queue.length) {
      const node = queue.shift();
      /* if node is the goal, return node */
      for (let n of /* neighbors(node) */) {
        if (!visited.has(n)) {
          visited.add(n);
          queue.push(n);
          /* n.parent = node -> if path is needed */
        }
      }
    }
  }
}
```

```js
/**
 * stack: nodes to return to
 * cur: node to enter
 */
function postOrder(root) {
  let res = [],
    stack = [],
    cur = root;
  while (cur || stack.length) {
    if (cur) {
      stack.push([cur, false]);
      cur = cur.left;
    } else {
      const [node, done] = stack.pop();
      if (done) {
        res.push(node.val);
      } else {
        stack.push([node, true]);
        cur = node.right;
      }
    }
  }
  return res;
}

function inorder(root) {
  let res = [],
    stack = [],
    cur = root;
  while (cur || stack.length) {
    if (cur) {
      stack.push(cur);
      cur = cur.left;
    } else {
      const node = stack.pop();
      res.push(node.val);
      cur = node.right;
    }
  }
  return res;
}

function preorder(root) {
  let res = [],
    stack = [],
    cur = root;
  while (cur || stack.length) {
    if (cur) {
      res.push(cur.val);
      stack.push(cur);
      cur = cur.left;
    } else {
      cur = stack.pop().right;
    }
  }
  return res;
}
```

### Sorting

```js
/**
 * Hoare partition scheme
 * - Invariant: A[L...l-1] <= pivot, A[r+1...R] >= pivot
 * - Terminating Condition: l == r + 1 or l == r
 *     +----+----+        +-----+-----+-----+
 *     | <p | >p |   or   | <=p | ==p | >=p |    partition point: l or r+1
 *     +----+----+        +-----+-----+-----+
 *     r   l,r+1               l,r   r+1
 *          ^                   ^     ^
 *   EDGE CASE I: pivot on the left, loop terminates after first iteration
 *     +---+---+      +---+
 *     | p |   |  ... |   |    l will cause infinite recursion
 *     +---+---+      +---+
 *     l  r+1
 *
 *   EDGE CASE II: pivot on the right, loop terminates after first iteration
 *     +---+---+      +---+
 *     |   |   |  ... | p |    r+1 will cause infinite recursion
 *     +---+---+      +---+
 *                    l  r+1
 */
function quickSort(nums) {
  function partition(l, r) {
    const pivot = nums[l];
    // const pivot = nums[Math.floor((l + r) / 2)];
    // const pivot = nums[r];
    // const pivot = nums[Math.ceil((l + r) / 2)];
    l--;
    r++;
    while (true) {
      do {
        l++;
      } while (nums[l] < pivot);
      do {
        r--;
      } while (nums[r] > pivot);
      if (l >= r) return r + 1;
      // if (l >= r) return l;
      [nums[l], nums[r]] = [nums[r], nums[l]];
    }
  }
  function sort(l, r) {
    if (l < r) {
      const mid = partition(l, r);
      sort(l, mid - 1);
      sort(mid, r);
    }
  }
  sort(0, nums.length - 1);
  return nums;
}

/**
 * Lomuto partition scheme
 * - Invariant: A[l...i-1] < pivot, A[i...j-1] >= pivot
 * - Terminating condition: j == r
 */
function quickSort(nums) {
  function partition(l, r) {
    const pivot = nums[r];
    let i = l;
    for (let j = l; j < r; j++) {
      if (nums[j] < pivot) {
        [nums[j], nums[i]] = [nums[i], nums[j]];
        i++;
      }
    }
    [nums[i], nums[r]] = [nums[r], nums[i]];
    return i;
  }
  function sort(l, r) {
    if (l < r) {
      const mid = partition(l, r);
      sort(l, mid - 1);
      sort(mid + 1, r);
    }
  }
  sort(0, nums.length - 1);
  return nums;
}

// Out-of-place version
function quickSort(nums) {
  function merge(left, pivot, right, arr) {
    let i = 0;
    for (let n of left) arr[i++] = n;
    arr[i++] = pivot;
    for (let n of right) arr[i++] = n;
  }
  function sort(arr) {
    if (arr.length < 2) return;
    const pivot = arr[0],
      left = [],
      right = [];
    for (let i = 1; i < arr.length; i++) {
      if (arr[i] < pivot) {
        left.push(arr[i]);
      } else {
        right.push(arr[i]);
      }
    }
    sort(left);
    sort(right);
    merge(left, pivot, right, arr);
  }
  sort(nums);
  return nums;
}

function mergeSort(nums) {
  function merge(left, right, arr) {
    let i = 0,
      j = 0,
      k = 0;
    while (i < left.length && j < right.length) {
      if (left[i] < right[j]) {
        arr[k++] = left[i++];
      } else {
        arr[k++] = right[j++];
      }
    }
    while (i < left.length) arr[k++] = left[i++];
    while (j < right.length) arr[k++] = right[j++];
  }
  function sort(arr) {
    if (arr.length < 2) return;
    const mid = Math.floor(arr.length / 2);
    const left = arr.slice(0, mid);
    const right = arr.slice(mid);
    sort(left);
    sort(right);
    merge(left, right, arr);
  }
  sort(nums);
  return nums;
}

// Invariant: A[0...i-1] is sorted
function insertionSort(nums) {
  for (let i = 1; i < nums.length; i++) {
    let cur = nums[i],
      j = i;
    while (j > 0 && nums[j - 1] > cur) {
      nums[j] = nums[j - 1];
      j--;
    }
    nums[j] = cur;
  }
  return nums;
}

// Invariant: A[0...i-1] <= A[i...n-1] and A[0...i-1] is sorted
function selectionSort(nums) {
  for (let i = 0; i < nums.length - 1; i++) {
    let min = i;
    for (let j = i; j < nums.length; j++) {
      if (nums[j] < nums[min]) min = j;
    }
    [nums[min], nums[i]] = [nums[i], nums[min]];
  }
  return nums;
}

// Invariant: A[0...n-i-1] <= a[n-i...n-1], A[n-i...n-1] is sorted
function bubbleSort(nums) {
  for (let i = 0; i < nums.length - 1; i++) {
    for (let j = 0; j < nums.length - i - 1; j++) {
      if (nums[j] > nums[j + 1]) {
        [nums[j], nums[j + 1]] = [nums[j + 1], nums[j]];
      }
    }
  }
  return nums;
}
```
