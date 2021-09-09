---
theme: default
layout: center
highlighter: shiki
---
# Haskell 與代數視角
第一性原理：從算術到函數式設計模式 
---
layout: center
---
# Number
- 閉合： $+ : Number \times Number \rightarrow Number$
- 單位元：$0 + a = a + 0 = a$
- 結合律：$(a + b) + c = a + (b + c)$
---
layout: center
---

# Number
- 閉合： $\times : Number \times Number \rightarrow Number$
- 單位元：$1 \times a = a \times 1 = a$
- 結合律：$(a \times b) \times c = a \times (b \times c)$

---
layout: center
---

# Number 
- 閉合： $max : Number \times Number \rightarrow Number$
- 單位元：$max(-\infty, a) = max(a, -\infty) = a$
- 結合律：$max(max(a, b), c) = max(a, max(b, c))$

---
layout: center
---

# Boolean
- 閉合： $\wedge : Boolean \times Boolean \rightarrow Boolean$
- 單位元：$T \wedge a = a \wedge T = a$
- 結合律：$(a \wedge b) \wedge c = a \wedge (b \wedge c)$

---
layout: center
---

# String
- 閉合: $+ : String \times String \rightarrow String$
- 單位元：$\epsilon + a = a + \epsilon = a$
- 結合律：$(a + b) + c = a + (b + c)$

---
layout: center
---

# List
- 閉合： $+ : List \times List \rightarrow List$
- 單位元：$[] + a = a + [] = a$
- 結合律：$(a + b) + c = a + (b + c)$

---
layout: center
---

# List
- 閉合： $merge(a, b) := sorted(a + b)$ 
- 單位元：$merge([], a) = merge(a, []) = a$
- 結合律：$merge(merge(a, b), c) = merge(a, merge(b, c))$

---
layout: center
---

# Function
$\cdot \rightarrow \cdot$ (泛型)
```ts
type Func<A, B> = (_: A) => B;
```
- 閉合： $\circ : (Y \rightarrow Z) \times (X \rightarrow Y) \rightarrow (X \rightarrow Z)$
- 單位元：$(x \rightarrow x) \circ f = f \circ (x \rightarrow x)$
- 結合律：$(f \circ g) \circ h = f \circ (g \circ h)$

---
layout: center
---

# Function
函數調用不是一個閉合的二元運算
- $\cdot(\cdot): (X \rightarrow Y)\times X \rightarrow Y$
- 單位元：$(x \rightarrow x)(y) = y$
- 結合律：$(f \circ g)(x) = f(g(x))$

---
layout: center
---
# 應用式函子 Applicative Functor
函數調用 2.0（代數化）：把函數裝進盒子（泛型），把調用變成加法
```ts
// 數學（類型）意義上的“盒子”，不（只）是物理意義上的“盒子”
class BlackBox<T> { magic(): Map<string, () => BlackBox<T> | T> {} }
type Applicative<T> = () => BlackBox<BlackBox<T>>>;
```
- 閉合： $ap: A[X \rightarrow Y]\times A[X] \rightarrow A[Y]$
- 單位元：$ap(A[x \rightarrow x], Ay) = Ay$
- 結合律：$ap(Af \circ Ag,Ax) = ap(Af, ap(Ag, Ax))$

--- 
layout: center
---

# 解析器組合子 Parser Combinator
概念由 Haskell 的 parsec 庫首創

```ts
const jsonArray = T.leftBracket
  .apr(sepBy(T.comma, jsonValue))
  .apl(T.rightBracket);

const jsonProperty = pure(makeKeyValuePair)
  .ap(jsonString)
  .apl(T.colon)
  .ap(jsonValue);

const jsonObject = T.leftBrace
  .apr(sepBy(T.comma, jsonProperty))
  .apl(T.rightBrace)
  .map(makeObject);

const jsonValue = jsonNull
  .or(jsonBoolean)
  .or(jsonNumber)
  .or(jsonString)
  .or(jsonArray)
  .or(jsonObject);
```


---
layout: center
---
# 黑洞
吞噬一切

- $0 \times a = 0 \times a = 0$（Number）
- $NaN \cdot a = a \cdot NaN = NaN$（對於任何數字運算）
- $null(x) = f(null) = null$ （純屬妄想)

--- 
layout: center
---

定義 $ap\Bigg(
  \begin{pmatrix}
    f_0 \\ f_1 \\ \vdots
  \end{pmatrix},
  \begin{pmatrix}
    x_0 \\ x_1 \\ \vdots
  \end{pmatrix}
\Bigg) := 
\begin{pmatrix}
  f_0(x_0) \\ f_0(x_1) \\ \vdots \\
  f_1(x_0) \\ f_1(x_1) \\ \vdots
\end{pmatrix}$，顯然 $ap(Af \circ Ag, Ax) = ap(Af, ap(Ag, Ax))$

定義 $NULL := \begin{pmatrix}\end{pmatrix}$，則自然得到
$$
ap\Bigg(
  NULL,
  \begin{pmatrix}
    x_0 \\ x_1 \\ \vdots
  \end{pmatrix}
\Bigg)
= ap\Bigg(
  \begin{pmatrix}
    f_0 \\ f_1 \\ \vdots
  \end{pmatrix},
  NULL
\Bigg) = NULL
$$

---
layout: center
---
# 函子 Functor
函數調用 1.5：$map: (X \rightarrow Y) \times A[X] \rightarrow A[Y]$

對於應用式函子：$map(f, Ax) = ap(A[f], Ax)$

$$
map \Bigg(f,
  \begin{pmatrix}
    x_0 \\ x_1 \\ x_2
  \end{pmatrix}
\Bigg) := ap \Bigg(
  \begin{pmatrix} f \end{pmatrix},
  \begin{pmatrix}
    x_0 \\ x_1 \\ x_2
  \end{pmatrix}
\Bigg) = 
\begin{pmatrix}
  f(x_0) \\ f(x_1) \\ f(x_2)
\end{pmatrix}
$$

---
layout: center
---
# Map + 坍縮
### 結構坍縮
$$
\begin{aligned}
  & bind(A[x], f) & f: X \rightarrow A[Y]\\
  := & flatMap(f, A[x])\\
  := & flat(map(f, A[x])) & flat: A[A[X]] \rightarrow A[X]\\
\end{aligned}
$$
### 內容坍縮
> 這裏 M 表示定義了某種加法的集合/類型，數學上叫 Monoid 幺半群

$$
\begin{aligned}
  & foldMap(f, A[x])     & f: X \rightarrow M\\
  := & fold(map(f, A[x])) & fold: A[M] \rightarrow M\\
  := & reduce((m, x) \rightarrow m \cdot_M f(x), e, A[x])
\end{aligned}
$$


比如求最大年齡：$A[X] \mapsto List[Person],\quad f \mapsto getAge,\quad \cdot_M \mapsto max$

--- 
layout: center
---
# 新代數：單子 Monad
函數調用3.0：拍平也要講基本法

- 定義 $f \circ_K g := x \rightarrow flatMap(f, g(x))$， 要求 $flat$ 必須滿足：
- 單位元：$flatMap(x \rightarrow A[x], f(y)) = flatMap(f, (x \rightarrow A[x])(y)) = f(y)$
- 結合律： $flatMap(f \circ_K g, x) = flatMap(f, flatMap(g, x))$

---
layout: center
---
$$
\begin{aligned}
  ap \Bigg(
    \begin{pmatrix}
      f_0 \\ f_1
    \end{pmatrix},
    \begin{pmatrix}
      x_0 \\ x_1
    \end{pmatrix}
  \Bigg) :=
  & bind \Bigg(
    \begin{pmatrix}
      f_0 \\ f_1
    \end{pmatrix}, f_i \rightarrow
      bind \Bigg(
        \begin{pmatrix}
          x_0 \\ x_1
        \end{pmatrix},
        x_j \rightarrow
          \begin{pmatrix}
            f_i(x_j)
          \end{pmatrix}
      \Bigg)
  \Bigg) \\
  = & flat \begin{pmatrix}
    bind \Bigg(
      \begin{pmatrix}
        x_0 \\ x_1
      \end{pmatrix}, x_j \rightarrow 
        \begin{pmatrix}
          f_0(x_j)
        \end{pmatrix}
    \Bigg) \\
    bind \Bigg(
      \begin{pmatrix}
        x_0 \\ x_1
      \end{pmatrix}, x_j \rightarrow 
        \begin{pmatrix}
        f_1(x_j)
      \end{pmatrix}
    \Bigg)
  \end{pmatrix}\\
  = & flat \begin{pmatrix}
    flat \begin{pmatrix}
      \begin{pmatrix} f_0(x_0)\end{pmatrix} \\
      \begin{pmatrix}f_0(x_1)\end{pmatrix}
    \end{pmatrix} \\
    flat\begin{pmatrix}
      \begin{pmatrix}f_1(x_0)\end{pmatrix} \\
      \begin{pmatrix}f_1(x_1)\end{pmatrix}
    \end{pmatrix}
  \end{pmatrix}
  = \begin{pmatrix}
    f_0(x_0) \\ f_0(x_1) \\ f_1(x_0) \\ f_1(x_1)
  \end{pmatrix}
\end{aligned}
$$

--- 
layout: center
---
# 新代數： Kleisli 組合
函數組合3.0：單子的第二種表述

- 定義 $f \circ_K g := x \rightarrow flatMap(f, g(x))$，要求 $flat$ 必須滿足：
- 閉合： $\circ_K: (Y \rightarrow A[Z]) \times (X \rightarrow A[Y])  \rightarrow (X \rightarrow A[Z])$
- 單位元：$(x \rightarrow A[x]) \circ_K f = f \circ_K (x \rightarrow A[x]) = f$
- 結合律： $(f \circ_K g) \circ_K h = f \circ_K (g \circ_K h)$
---
layout: center
---
# Haskell 中的單子（IO 單子）
<br>

### (Oct. 1992) 論文 [Imperative Functional Programming](https://www.microsoft.com/en-us/research/wp-content/uploads/1993/01/imperative.pdf)
```haskell
program = tell "Greetings!" >>= \_ -> 
            ask "What is your name?" >>= \name ->
              tell "Hi " ++ name
```
### (May. 1996) Haskell 1.3 "do-notation" 語法糖:
```haskell
program = do
             _ <- tell "Greetings!" 
             name <- ask "What is your name?"
             _ <- tell "Hi " ++ name 
             return name
```
---
layout: center
---
# Scala 中的單子
<div class="grid grid-cols-2 gap-4">
<div>

```scala
val program = tell("Greetings!") flatMap { s =>
  ask("What is your name?") flatMap { name =>
    tell("Hi " ++ name) flatMap {_ =>
      name
    }
  }
}
```

</div>
<div>

```scala
val program = for { 
  _    <- tell("Greetings!")
  name <- ask("What is your name?")
  _    <- tell("Hi " ++ name)
} yield name
```
```scala
for {
  i <- 0 until n
  j <- 0 until m if i + j > v
} {
  println(s"($i, $j)")
  yield i + j
}
```

</div>
</div>

---
layout: center
---
# JavaScript 中的單子
|   |   |   |
| - | - | - |
| Monad | `>>=` | “Do-Notation” |
| `Array`| `Array.prototype.flatMap` | 不支持 |
| `Promise` | `Promise.prototype.then` |  `async`/`await` |
| `rxjs/Observable`  | `rxjs/operators/concatMap` | 不支持 |
| `rxjs/Observable`  | `rxjs/operators/mergeMap` | 不支持 |
| `rxjs/Observable`  | `rxjs/operators/switchMap` | 不支持 |

---
layout: center
---
# `Promise` 單子結合律
$flatMap(f \circ_K g,x) = flatMap(f, flatMap(g, x))$

```ts {all|5-6|8-9}
const x = Promise.resolve(1);
const f = async x => x + 1;
const g = async x => x * 2;

// flatMap(compose(f, g), x)
x.then(y => g(y).then(f))

// flatMap(f, flatMap(g, x))
x.then(g).then(f);
```

---
layout: center
---
# Free Monad（自由單子）
React 與 Redux 陣營的理論根源

- __【必讀】Scala Cats__：https://typelevel.org/cats/datatypes/freemonad.html
> 為什麼會出現這些想法：React Fiber 兩階段渲染，React Hooks，Redux，Redux Saga
- 函子可以被直接改造成一個單子，通常用於內嵌 DSL
- 與 algebraic effect 的概念關係密切（effect 系統相當於 free monad 進行 foldMap）

---
layout: center
---
<div class="grid grid-cols-2 gap-10">
<div>

# Haskell 運算符
舉世皆代數，萬物皆可加

> __FP == PPAP__

<br>

> I have a pen  
> I have a apple  
> Uh!  
> Apple-Pen! 

<br>

> I have a pen  
> I have pineapple  
> Uh!  
> Pineapple-Pen!  

<br>

> Apple-Pen  
> Pineapple-Pen  
> Uh!  
> Pen-Pineapple-Apple-Pen! 

</div>

|   |   |
| - | - |
| $a \cdot b$ | `a <> b` |
| $f \circ g$ | `f . g` |
| $f \circ_K g$ | `f <=< g` |
| $f(x)$ | `f $ x` |
| $map(f,x)$ | `f <$> x` |
| $ap(f, x)$ | `f <*> x` |
| $apl(f, x)$ | `f <* x` |
| $apr(f, x)$ | `f *> x` |
| $bind(x, f)$ | `x >>= f` |

</div>

---
layout: center
---
# 擴展閱讀
- 文章：Functor, Applicative, and Monads in Picture （[英](https://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html) / [中](http://blog.forec.cn/2017/03/02/translation-adit-faamip/)）
- 文章：Free Monad （[英](https://typelevel.org/cats/datatypes/freemonad.html)）
- 課本：Learn You a Haskell For Great Good（[英](http://learnyouahaskell.com/chapters) / [中](https://learnyouahaskell.mno2.org/zh-cn)）
- 課本：Haskell Programming from First Principles （[英](https://raw.githubusercontent.com/dylannichols/Haskell-Book/master/Chris%20Allen%20%26%20Julie%20Moronuki%20-%20Haskell%20Programming%20from%20First%20Principles.pdf)）

