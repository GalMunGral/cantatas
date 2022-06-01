## Type, Coercion

- `typeof`: `undefined`, `boolean`, `number`, `bigint`, `string`, `symbol`, `function`, `object`
- `==`: [[MDN]](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness#Loose_equality_using), `+` [[article]](https://2ality.com/2019/10/type-coercion.html#addition-operator-(%2B)), truthy/falsy [[MDN]](https://developer.mozilla.org/en-US/docs/Glossary/Falsy)

```js
const clone = new Map();
function deepClone(obj) {
  if (typeof obj != "object" || obj === null) return obj;
  const res = Array.isArray(obj) ? [] : {};
  clone.set(obj, res);
  for (let key of Object.keys(obj)) {
    res[key] = clone.get(obj[key]) || deepClone(obj[key]);
  }
  return res;
}
```

## OO, Prototypal Inheritance

- Meta programming - `Proxy`, `Reflect`

  - `apply`, `construct`
  - `has`, `ownKeys`, `get`, `set`, `isExtensible`, `preventExtensions`, `getOwnPropertyDescriptor`, `defineProperty`, `deleteProperty`
  - `getPrototypeOf`, `setPrototypeOf`

  ```js
  // Without using `Reflect`
  function _new(Ctor, ...args) {
    const instance = Object.create(Ctor.prototype);
    return Ctor.call(instance, ...args) || instance;
  }

  function _instanceof(obj, Ctor) {
    let cur = Object.getPrototypeOf(obj);
    while (cur) {
      if (cur === Ctor.prototype) return true;
      cur = Object.getPrototypeOf(cur);
    }
    return false;
  }
  ```

### Array [[MDN]](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String#Instance_methods), Set

- ```js
  Array.prototype._reduce = function (callback, initialValue) {
    if (initialValue == null && this.length == 0) {
      throw new TypeError("Reduce of empty array with no initial value");
    }
    let acc = initialValue == null ? this[0] : initialValue;
    for (let i = initialValue == null ? 1 : 0; i < this.length; i++) {
      acc = callback(acc, this[i], i, this);
    }
    return acc;
  };
  ```
- Array deduplication: `[...new Set(arr)]`
- Set operations [[article]](https://exploringjs.com/impatient-js/ch_sets.html#missing-set-operations)
  ```js
  const union = new Set([...a, ...b]);
  const intersection = new Set([...a].filter((x) => b.has(x)));
  const difference = new Set([...a].filter((x) => !b.has(x)));
  ```
- Convert array-like (has `length` property) and iterable objects to arrays
  > `arguments` is both array-like and iterable
  ```js
  const arr = Array.from(obj); // array-like or iterable
  const arr = Array.prototype.slice.call(obj); // array-like
  const arr = [...obj]; // iterable
  ```

### String [[MDN]](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array#Instance_methods), Regular Expression

- Regex: [character classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions/Character_Classes), [assertions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions/Assertions), [groups](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions/Groups_and_Ranges), [quantifiers](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions/Quantifiers)
  ```js
  "221234567".replace(/\B(?=(\d{3})+$)/g, ","); // -> 221,234,567
  ```
- `String.prototype.replace` backreference: `$n`, `$<name>`, `$&` (entire match) [[MDN]](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/replace#Specifying_a_string_as_a_parameter)
- `String.prototype.match(regexp)` returns:
  - Without `y`, `g` flags: `[firstMatch, [...capturingGroups], index, input, groups: { ... } ]`
  - Otherwise `[...allMatches]`
- `String.prototype.matchAll(regexp)` return an iterator of all matches.
- `RegExp.prototype.exec(str)` returns:

  - Without `y`, `g` flags: same as `String.prototype.match`
  - Otherwise is stateful (`lastIndex`), can be called repeatedly to iterate over all matches.

- Reference counting (引用计数法), Tracing (mark-and-sweep (标记清除法), stop-and-copy, etc.)
- V8 GC: _most objects die young (generational hypothesis)_. [[official blog]](https://v8.dev/blog/free-garbage-collection#deep-dive-into-v8%E2%80%99s-garbage-collection-engine)
  - Young generation (新生代): **Scavenge - [semi-space (stop-and-copy)](https://www.memorymanagement.org/glossary/t.html#term-two-space-collector)**
  - Old generation (老生代): **[mark-and-sweep](https://www.memorymanagement.org/glossary/m.html#term-mark-sweep)** - incremental marking steps (~5ms), concurrent sweeper threads, [compaction (压缩)](https://www.memorymanagement.org/glossary/c.html#term-compaction)
  - Scheduled as idle tasks (e.g. after rendering and before vsync)
- Causes of memory leaks

  - Reference counting + reference cycles.
  - Global variables (accidental or intentional) - always reachable (GC root is the global object)
  - Internal references: event listeners, `setInterval`, recursive `setTimeout` [[article]](https://javascript.info/settimeout-setinterval)
    - Use `clearTimeout(id)`, `clearInterval(id)`, `removeEventListener(fn)`,
  - Closure do not cause memory leak, unless deoptimized (e.g. by using `eval`) [[article]](https://www.smartik.net/2017/03/browsers-closures-optimization-avoiding-memory-leaks.html)
  - Problematic closure implemenation by JS engines - e.g. multiple closures sharing a parent scope [[article]](https://auth0.com/blog/four-types-of-leaks-in-your-javascript-code-and-how-to-get-rid-of-them)

  ```js
  var f = (function () {
    var a = "some potentially very large data";
    return function () {};
  })(); // <- The entire function scope is optimized away

  var g = (function () {
    var a = "some potentially very large data";
    function b() {
      console.log(a);
    }
    return function () {};
  })(); // <- `b` is optimized away, but `a` survives because it's referenced by `b`!
  ```

#### Node.js Event Loop - libuv

- Uses kernel API for I/O event notifications (e.g. [`epoll`](https://en.wikipedia.org/wiki/Epoll) on Linux) [[Youtube]](https://www.youtube.com/watch?v=P9csgxBgaZ8)
  - Directly epoll-able: sockets, pipes, etc.
  - Thread pool (epoll on a pipe and write to pipe when data is ready): `fs` module, etc.
- Initialize the event loop, processes input script, then enters the event loop:
  - Microtasks: flush `process.nextTick` queue, flush `Promise.then` queue, repeat
    - Flushed after each macrotask phase
    - **UPDATE**: As of v11, flushed after each `setTimeout`/`setInterval`/`setImmediate` callback.
  - Macrotasks: Phases
    - Timers (`setTimeout`, `setInterval`) **NOTE**: delay set to 1 if outside [1, 2147483647]. [[doc]](https://nodejs.org/api/timers.html#timers_settimeout_callback_delay_args)
    - Poll (I/O) [[official blog]](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/#poll) `int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);`
    - Check (`setImmediate`)

### Promise - eventual result of an asynchronous operation [[Promise/A+ spec]](https://promisesaplus.com/)

- `Promise.all`, `Promise.race`, `Promise.resolve` (flattens nested thenables), `Promise.reject`
- `Promise.prototype.finally(onFinally)`, receives no args, returns a promise that resolves/rejects witht the same state

```js
class _Promise {
  constructor(executor) {
    this.status = "pending";
    this.value = null;
    this.reason = null;
    this.callbacks = [];
    executor(this.resolve.bind(this), this.reject.bind(this));
  }

  resolve(value) {
    if (this.status !== "pending") return;
    this.status = "fulfilled";
    this.value = value;
    this.callbacks.forEach(queueMicrotask);
  }

  reject(reason) {
    if (this.status !== "pending") return;
    this.status = "rejected";
    this.reason = reason;
    this.callbacks.forEach(queueMicrotask);
  }

  then(onFulfilled, onRejected) {
    if (typeof onFulfilled != "function") {
      onFulfilled = (value) => value;
    }
    if (typeof onRejected != "function") {
      onRejected = (reason) => {
        throw reason;
      };
    }
    const promise = new _Promise(() => {});
    const processResult = () => {
      try {
        this.status === "fulfilled"
          ? resolvePromise(promise, onFulfilled(this.value))
          : resolvePromise(promise, onRejected(this.reason));
      } catch (e) {
        promise.reject(e);
      }
    };
    this.status === "pending"
      ? this.callbacks.push(processResult)
      : queueMicrotask(processResult);
    return promise;
  }
}
```

```js
function resolvePromise(promise, x) {
  if (x === promise) {
    promise.reject(new TypeError());
  } else if (x instanceof _Promise) {
    if (x.status === "fulfilled") {
      resolvePromise(promise, x.value);
    } else if (x.status === "rejected") {
      promise.reject(x.reason);
    } else {
      x.then(
        (value) => resolvePromise(promise, value),
        (reason) => promise.reject(reason)
      );
    }
  } else if ((typeof x == "object" && x) || typeof x == "function") {
    let then;
    try {
      then = x.then;
    } catch (e) {
      promise.reject(e);
      return;
    }
    if (typeof then == "function") {
      let hasCalled = false;
      try {
        then.call(
          x,
          (value) => {
            if (hasCalled) return;
            hasCalled = true;
            resolvePromise(promise, value);
          },
          (reason) => {
            if (hasCalled) return;
            hasCalled = true;
            promise.reject(reason);
          }
        );
      } catch (e) {
        if (hasCalled) return;
        promise.reject(e);
      }
    } else {
      promise.resolve(x);
    }
  } else {
    promise.resolve(x);
  }
}
```

```js
function runGenerator(generator) {
  const it = generator();
  function iterate(val) {
    let ret = it.next(val);
    if (!ret.done && ret.value instanceof Promise) {
      ret.value.then(iterate);
    }
  }
  iterate();
}
```

- ```js
  function debounce(fn, delay) {
    let timer;
    return function () {
      clearTimeout(timer);
      timer = setTimeout(() => {
        fn.apply(this, arguments);
      }, delay);
    };
  }

  function throttle(fn, interval) {
    let timer;
    return function () {
      if (!timer) {
        fn.apply(this, arguments);
        timer = setTimeout(() => {
          timer = null;
        }, interval);
      }
    };
  }

  function throttle(fn, interval) {
    let last = 0;
    return function () {
      let now = +new Date(); // Date.now(), performance.now()
      if (now - last >= interval) {
        fn.apply(this, arguments);
        last = now;
      }
    };
  }
  ```

### Routing - locating views via URL

- URL hash - meant for same-page navigation (jump to `id`)
  - set: `location.hash` (other `location` properties: `href`, `protocol`, `host`, `port`, `pathname`, `search`)
  - get: `hashchange`(`window.onhashchange`) event, `e.newURL|oldURL`
  - Hash is never sent to server. Updating hash does not reload the page.
  - **DISADVANTAGE**: Search engine crawlers ignore hash to prevent reindexing the same page
- HTML5 History API
  - set: `history.pushState|replaceState(state, title, url)|forward|back|go(delta)`
  - get: `popstate` (`window.onpopstate`) event, `e.state`
  - Normal URL, better for SEO, but requires server to always return `index.html`

### Storage

- Cookies, at least 4KB max each (name, value, attributes included), 50 cookies per domain
  ```js
  document.cookie = "key=value"; // Set one cookie
  const cookies = new Map(
    document.cookie.split(/\s*;\s*/).map((s) => s.split("="))
  );
  const value = cookies.get("key");
  ```
- `localStorage|sessionStorage`, at least 5MB/10MB max per domain

#### Basic Usage

```js
function Node({ data, children }) {
  const entries = Object.entries(data).filter(
    ([, value]) => typeof value != "object"
  );
  return (
    <div style={{ border: "1px solid #DDD" }}>
      {entries.length ? (
        <section style={{ background: "#EEE", margin: 10, padding: 10 }}>
          {entries.map(([key, value]) => (
            <p key={key}>
              {key} - {JSON.stringify(value)}
            </p>
          ))}
        </section>
      ) : null}
      {children}
    </div>
  );
}

function Tree({ data }) {
  const children = Object.entries(data).filter(
    ([, value]) => typeof value == "object"
  );
  return (
    <Node data={data}>
      {children.map(([key, value]) => (
        <section key={key} style={{ margin: 30 }}>
          <h5>{key}</h5>
          <Tree data={value} />
        </section>
      ))}
    </Node>
  );
}
```

- JSX (`{JS expression}`) -> `React.createElement(type, [props], [...children])` [[docs]](https://reactjs.org/blog/2015/12/18/react-components-elements-and-instances.html)
- `SyntheticEvent` - cross-browser compatibility
  - **Event delegation**: Native events bubble to `document` -> event pooling -> `dispatchEvent`
    > **UPDATE**: no more pooling in v17 [[official blog]](https://reactjs.org/blog/2020/08/10/react-v17-rc.html#no-event-pooling)
  - `e.nativeEvent.currentTarget === document`, `this === undefined`
  - **IMPORTANT**: `SyntheticEvent`s are nullified after callbacks are invoked, therefore can't be accessed asynchronously.
- DOM elements, forms: [[docs]](https://reactjs.org/docs/dom-elements.html), [[docs]](https://reactjs.org/docs/forms.html)
  - `htmlFor`, `className`, `style` (object), `dangerouslySetInnerHTML` (`{ __html }`)
  - Controlled components
    - `value`(input/textarea/select) or `checked`(radio/checkbox) (DOM properties instead of attributes)
    - `onChange` (`input` instead of `change` event)
  - Uncontrolled components (e.g. file input)
    - `defaultValue`, `defaultChecked` (correspond to `value`, `checked` attributes)
    - `React.createRef()` -> `ref={this.someRef}` -> `this.someRef.current`
- Props: _lift shared state up_, _HOC_, _render prop_, typecheck with `prop-types` [[docs]](https://github.com/facebook/prop-types).
- **`setState(updater, callback)`** [[docs]](https://reactjs.org/docs/react-component.html#setstate)
  - Enqueue updates to be merged during reconcilliation [[source code]](https://github.com/facebook/react/blob/15-stable/src/renderers/shared/stack/reconciler/ReactCompositeComponent.js#L894)
  - [Why is `setState` asynchronous](https://github.com/facebook/react/issues/11527)
  - [When is `setState` batched (asynchronous)](https://stackoverflow.com/questions/48563650/does-react-keep-the-order-for-state-updates/48610973#48610973)
    - Batched in React event handlers, lifecycle hooks
      > Transaction (Implementation Detail): `isBatchingUpdates = true` -> `someMethod()` -> `isBatchingUpdates = false`
    - Otherwise (`setTimeout`, `addEventListener`, `Promise.then` etc.) flushed immediately
    - **NOTE**: This is an implementation detail and will change in later versions [[GitHub]](https://github.com/facebook/react/issues/10231#issuecomment-316644950) [[official blog]](https://reactjs.org/docs/concurrent-mode-adoption.html#feature-comparison)
      > Legacy mode has automatic batching in React-managed events but it’s limited to one browser task. Non-React events must opt-in using unstable_batchedUpdates. In Blocking Mode and Concurrent Mode, all setStates are batched by default.
      > chingUpdates == false`
- `React.createContext` -> `Context.Provider` -> (`Context.Consumer` + render prop) or (`static contextType` + `this.context`)
- `ReactDOM.createPortal(child, container)`
- Async components - `React.lazy`, `React.Suspense`

#### Reconciliation [[docs]](https://reactjs.org/docs/reconciliation.html), Optimizations

- **React Fiber** - incremental rendering [[article]](https://github.com/acdlite/react-fiber-architecture)
  > - A fiber corresponds to a stack frame, but it also corresponds to an instance of a component.
  > - Conceptually, the type (of a React element or fiber) is the function (as in v = f(d)) whose execution is being tracked by the stack frame.
  > - Conceptually, props are the arguments of a function.
- **Lifecycle Hooks** [[diagram]](https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/)
  - If using `createReactClass`, `getDefaultProps`/`getInitialState` [[docs]](https://reactjs.org/docs/react-without-es6.html#declaring-default-props) (With ES6 classes, use `static defaultProps` and set initial state in constructor instead)
  - `static getDerivedStateFromProps`
  - ~~`componentWillMount`~~
  - `componentDidMount`
  - ~~`componentWillReceiveProps(nextProps)`~~
  - `shouldComponentUpdate(nextProps, nextState)`
  - ~~`componentWillUpdate(nextProps, nextState`)~~
  - `getSnapshotBeforeUpdate(prevProps, prevState)` - Return value will be passed to `componentDidUpdate`
  - `componentDidUpdate(prevProps, prevState, snapshot)`
  - `componentWillUnmount` - Clean up timers, ongoing requests, subscriptions here
- `React.memo(functionalComponent[, areEqual])` shallow-compares props.
- **`shouldComponentUpdate(nextProps[, nextState])`** returns `true` by default.
- `PureComponent` implements SCU with a _shallow_ (deep is expensive) comparison of props and state.

#### Redux [[docs]](https://redux.js.org/api/api-reference)

- **Unidirectional data flow (单向数据流)** [[diagrams]](https://github.com/reduxjs/redux/issues/653#issuecomment-216844781)
- `createStore(reducer, applyMiddleware(thunk))`, `store.dispatch(action)`, `store.subscribe(() => { store.getState() })`
- Redux Thunk: `store.dispatch((dispatch) => { })`;
- React Redux: `<Provider store={store}>`, `connect(mapStateToProps, mapDispatchToProps)(Component)` [[Gist]](https://gist.github.com/gaearon/1d19088790e70ac32ea636c025ba424e)

#### React Router

- `HashRouter`, `BrowserRouter` (HTML5 history API)
- `<Route path="..."><A/></Router>`, `Switch` renders only the first matching, `<Link to="..."></Link>`
- Hooks (functional components): `useParams()`, `useHistory()`, etc.
- Route props: `match`, `location`, `history` [[docs]](https://reactrouter.com/web/api/Route/route-props)
- Route-based code splitting (lazy loading) [[React docs]](https://reactjs.org/docs/code-splitting.html#route-based-code-splitting)

### Vue

#### Template, Render Function

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
- **`this.$nextTick`/ `Vue.nextTick`**
  ```js
  function nextTick(cb, ctx) {
    let _resolve;
    callbacks.push(() => {
      if (cb) {
        /* ... */
        cb.call(ctx);
        /* ... */
      } else if (_resolve) {
        _resolve(ctx);
      }
    });
    if (!pending) {
      pending = true;
      // Schedule `flushCallbacks` using `Promise.then`, `MutationObserver`, `setImmediate`, or `setTimeout`
      timerFunc();
    }
    if (!cb && typeof Promise !== "undefined") {
      return new Promise((resolve) => {
        _resolve = resolve;
      });
    }
  }
  ```

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
