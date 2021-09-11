---
title: 五分钟理解什么是 Monad
date: 2021-09-11 15:20:37
tags:
  - Haskell
  - 范畴论
  - 数学
  - 函数式编程
  - Monad
categories:
  - 学习笔记
---

## 引言

对于很多想要了解函数式编程（Functional Programming）或者是 Haskell 的 OIer 而言，Monad 是一个非常不友好的概念，但当你理解了它之后你就会不理解为什么你之前不理解它（

> 一个单子（Monad）说白了不过就是自函子范畴上的一个幺半群而已，这有什么难以理解的？

（上面这句话其实并不是 Monad 的严格定义，因为是“自函子范畴上的一个幺半群”的东西不一定是 Monad，比如 Applicative 也符合上述定义。）

由于 Monad 这一概念本身对新手并不友好，大众 Monad 的误解有一箩筐，包括但不限于：


- Monads are impure.
- Monads are about effects.
- Monads are about state.
- Monads are about imperative sequencing.
- Monads are about IO.
- Monads are dependent on laziness.
- Monads are a “back-door” in the language to perform side-effects.
- Monads are an embedded imperative language inside Haskell.
- Monads require knowing abstract mathematics.
- Monads are unique to Haskell.

虽然在实际的开发中，Monad 的应用确实很复杂，但如果只是要理解 Monad 的概念的话其实很简单。（？）

## 解释

### Functor

要理解 Monad 是什么，首先要理解 Functor 是什么。

Functor 可以理解为一个装着函数或值的盒子（定义不切确，等会会提到）。

用 Maybe 类型来举例就是：

```haskell
data Maybe a = Just a | Nothing
```

这种数据类型无论在哪里都很常用，在 C++ 等语言里我们也经常处理除了 int，double 之外的各种复杂数据类型。但 Functor 并不是一个盒子那么简单，它还有一个重要定义是 `fmap`：

```haskell
class Functor f where
    fmap :: (a -> b) -> f a -> f b
```

具体来说，就是一个函数，输入一个 输入一个值输出一个值的函数 f(x) 和一个 Functor x，返回一个 Functor f(x)，简单来说就是对盒子里的值进行操作。

举个栗子：


```haskell
Prelude> fmap (*3) Just 2
Just 6
```

**总而言之，Functor 就是一个装着值的盒子，并且可以用 `fmap` 函数来用一个普通函数生成另一个 Functor。**

### Applicative

**Applicative 是一种特殊的 Functor**，它除了 `fmap` 外还实现了另一个函数 `<*>`


```haskell
class (Functor f) => Applicative f where  
    pure :: a -> f a  
    (<*>) :: f (a -> b) -> f a -> f b
```

（了解面对对象的同学可以理解为 Applicative 继承了 Functor）

`<*>` 函数（熟悉 C++ 的同学可以理解为一个运算符）用来给给一个装在盒子里的值施加一个装在盒子里的函数，举个栗子：


```haskell
Prelude> Just (*3) <*> Just 2
Just 6
```

准确的说，`<*>` 函数接受一个`f (a -> b)`，返回一个`f a -> f b`。即输入一个含有函数的 Applicative 和一个含有值的 Applicative，输出一个含有结果的 Applicative。

《Haskell 趣学指南》中有一个典型的示例：

```haskell
myAction :: IO String
myAction = (++) `fmap` getLine <*> getLine
```

这段代码输入两行字符串并拼接，返回对应的 `IO Action`，它具体做了什么？首先，`geiLine`的返回值是一个 `IO Action`，它也属于一种 Applicative，所以也实现了`<*>` 函数。Applicative  作为 Functor 的子集当然也实现了 `fmap`，换句话说 `(++) `fmap` getLine`实际上是一个`IO ([Char] -> [Char])`，即一个含有函数的 Applicative，所以还要使用`<*>` 函数将其应用到另一个`IO Action`即`getLine`，最后的结果也是一个 Applicative（`IO Action`）。

下面这个例子可能更好理解：

```haskell
Prelude> (+) `fmap` Just 2 <*> Just 8
Just 10
```

前面的 `fmap (+) Just 2`的值实际上等于`Just (+2)`，所以整个表达式就等于`Just (+2) <*> Just 8`，即`Just 10`。

Applicative的定义：一个实现了`<*>`函数的 Functor

总而言之，Applicative 就是一个装着值的盒子，并且可以用 `fmap` 函数来用一个普通函数生成另一个 Applicative，并且还可以用`<*>`函数来用一个装在盒子（Functor）里的函数生成另一个 Applicative。

### Monad

现在你已经了解了所有前置知识，可以了解 Monad 了。（大雾）

**Monad 是一种特殊的 Applicative**，它除了 `<*>` 外还实现了另一个函数 `>>=`


```haskell
class (Applicative m) => Monad m where
    (>>=) :: m a -> (a -> m b) -> m b
    return  :: a -> m a
```

`>>=` 函数用把一个装在盒子里的值直接扔进一个“处理数据并自动打包盒子的函数”，举个栗子：


```haskell
threeTimes x = Just (x * 3)
```

```haskell
Prelude> threeTimes 2
Just 6
```

但是盒子里的值是无法被直接取出的，所以我们不能直接用这个函数处理装在盒子里的值，所以此时要用到 Monad：


```haskell
Prelude> Just 2 >>= threeTimes
Just 6
```


总而言之，Monad 就是一个装着值的盒子，并且可以用 `fmap` 函数来用一个普通函数生成另一个 Monad，并且还可以用`<*>`函数来用一个装在盒子（Functor）里的函数生成另一个 Monad，并且还可以用`>>=`函数来用一个“处理数据并自动打包盒子的函数”生成另一个 Monad。

这只是一个通俗的解释，更形式化的，一个 Monad 必须符合以下三条规则才能被称为 Monad:


```haskell
(return x) >>= f == f x          -- left unit law
m >>= return == m          -- right unit law
(m >>= f) >>= g == m >>= (\x -> f x >>= g)   -- associativity law
```

Haskell 中的 `return` 是指把一个值打包进 Monad 里，和其他语言中的 `return` 没什么关系。

如果你要实现自己的 Monad，则必须符合上述三条规则。

**在数学中，符合上述三条定律的东西被称为幺半群。**敏锐的读者可以立即察觉到，Monad 就是一个幺半群。

#### Left Unit Law

这条规则指的是，将一个值打包进 Monad 然后再`>>=`到一个函数，等同于直接将值应用于这个函数。

```haskell
Prelude> (return 3) >>= (\x -> Just (x * 2))
Just 6
Prelude> (\x -> Just (x * 2)) 3
Just 6
```

考虑到`>>=`的定义，这是比较显然的。

#### Right Unit Law

这条规则指的是，将一个 Monad `>>=`到`return`函数，得到的是原来的 Monad。用形式化的语言来说就是“Monad 是一个自相似的几何结构”。

```haskell
Prelude> Just 3 >>= return
Just 3
```

我们可以参照 `>>=`的实现来理解：

```haskell
instance Monad Maybe where
    return x = Just x
    (>>=) Nothing f = Nothing
    (>>=) (Just x) f = f x
```

注意第四行代码，这里的`Just`中的值被拆出来放进函数`f`里，而这里的`f`就是`return`，所以又产生了原先的 Monad。

#### Associativity Law

`>>=`函数符和结合律。

```haskell
Prelude> f x = Just (x + 3)
Prelude> g x = Just (x * 2)
Prelude> (Just 1 >>= f) >>= g
Just 8
Prelude> Just 1 >>= (\x -> f x >>= g)
Just 8
```

这里的结合律体现的不是很明显，我们考虑形如`a -> v`普通函数之间的运算`.`

```haskell
(.) :: (b -> c) -> (a -> b) -> (a -> c)
f . g = (\x -> f (g x))
```

则有

```haskell
(f . g). h == f . (g . h)
```

那么对于形如`Monad m => a -> m b`的函数，有

```haskell
(<=<) :: (Monad m) => (b -> m c) -> (a -> m b) -> (a -> m c)
f <=< g = (\x -> g x >>= f)
```

易得

```haskell
(f <=< g) <=< h == f <=< (g <=< h)
```

示例如下：

```haskell
Prelude> f x = Just (x + 3)
Prelude> g x = Just (x * 2)
Prelude> h x = Just (x + 5)
Prelude> ((f <=< g) <=< h) 7
Just 27
Prelude> (f <=< (g <=< h)) 7
Just 27
```

如果读者了解幺半群的概念，就会意识到这里`return`就是`<=<`运算的幺元（有时也称为单位元）：

```haskell
Prelude> (f <=< return) 7
Just 10
Prelude> (return <=< f) 7
Just 10
```

## Monad 辟谣

谣言 1：Monad 不纯

回答：你才不纯！你全家都不纯！

$\lambda$ 演算与图灵机等价，Haskell 显然也与 C++ 等价，纯函数本身就能解决所有问题。

谣言 2：Monad 能实现状态

回答：$\lambda$ 演算中状态可以用映射的方式模拟，Monad 只是一个更方便的语法用来处理状态。

谣言 3：Monad 是 Haskell 中的嵌入式命令式语言

回答：Haskell 是纯粹的函数式编程语言，`do` 语句只是长得像命令式而已。下面的例子会解释 `do` 其实只是 `>==`的语法糖。

谣言 4：了解 Monad 必须理解范畴论

回答：不不不，在不了解范畴论的情况下并不是不能写出包含 Monad 的程序，只是要更进一步的话需要一些范畴论的知识。

谣言 5：Monad 是 Haskell 独有的

回答：Monad 一开始是范畴论的概念，后来被函数式编程（FP）借用。任何编程语言都可以实现 Monad，包括 C++ 也可以，这只是必要性的问题。

谣言 5：Monad 用来进行 IO 操作

回答：Monad 的特性确实使它很适合用来管理 IO，但 Monad 本身在定义上和 IO 半毛钱关系没有！

谣言 6：Monad 用来进行有副作用的操作

回答：同上！！！

**谣言 7**：JavaScript 中的 `Promise`是 Monad

**回答**：这个谣言流传的非常广泛，在网上很多地方都能看到“Promise 是一种 Monad”的观点。`Promise`确实看起来像一个 Monad，也有类似 Monad 的操作，但它并不符合上述三条定律，即不符合 Monad 的定义。如果你在 Haskell 中弄出类似的东西并将其作为 Monad 使用，肯定会在 `do`语句等地方出大问题！

让我们仔细审视一下 Promise 是否符合上述三条定律：

```javascript
Promise.resolve(x).then(f) === f(x)
m.then(x => Promise.resolve(x)) === m
m.then(f).then(g) === m.then(x => f(x).then(g))
```

仿佛是符合的，但如果`x`是一个 Promise，`Promise.resolve(x)`就会把`a`给`resolve`掉，也就不是一个 Promise 了，即不符合第一条Left Unit Law。

## Monad 的具体应用

### Maybe 类型

Haskell 中的 Maybe 类型在概念上与 Haskell 的 `Option<T>`类似，由一个值或者是`Nothing`组成，用来表示返回无意义值。

举个例子，用 Maybe 类型来写除法：


```haskell
safeDiv :: Int -> Int -> Maybe Int
safeDiv _ 0 = Nothing
safeDiv a b = Just (a / b)
```


假设我们要计算 `a / b / c / d / e`，在知道 Haskell 的模式匹配用法后，我们可以写出这样的代码：


```haskell
calc :: Int -> Int -> Int -> Int -> Int -> Maybe Int
calc a b c d e = case safeDiv a b of
    Just x -> case safeDiv x c of
        Just y -> safeDiv y d
            Just z -> safeDiv z e
            Nothing -> Nothing
        Nothing -> Nothing
    Nothing -> Nothing
```

令人头大，但利用 Monad 的 `>>=` 运算可以很方便地完成“模式匹配拆包再打包的过程”，具体代码如下：

```haskell
calc :: Int -> Int -> Int -> Int -> Int -> Maybe Int
calc a b c d e = 
    safeDiv a b >>= (\x ->
    safeDiv x c >>= (\y ->
    safeDiv y d >>= (\z ->
    safeDiv z e)))
```

简单明了多了，但是嵌套的闭包第一眼看上去并不直观。所以 Haskell 还提供了 `do` 语句作为 `>==` 的语法糖：

```haskell
calc :: Int -> Int -> Int -> Int -> Int -> Maybe Int
calc a b c d e = do
    x <- safeDiv a b
    y <- safeDiv x c
    z <- safeDiv z d
    safeDiv z e
```

这就是常见的 `do` 语句的实际含义。注意，这并不是命令式编程，只是看起来像，它的本质是嵌套的闭包。

Maybe 的 Monad 实际上是这样实现的：

```haskell
instance Monad Maybe where
    return x = Just x
    (>>=) Nothing f = Nothing
    (>>=) (Just x) f = f x
```

第三行语句帮助我们略去了不断检查是否为`Nothing`的过程。

### IO Monad

可能您还是不明白 Monad 到底有什么用，这里就再举一个例子： IO

```haskell
main = do
    putStrLn "What is your name?"
    name <- getLine
    putStrLn ("Hello, " ++ name ++ "!")
```

它等价于下面的代码：

```haskell
main = putStrLn "What is your name?" >== (\_ ->
       getLine >== (\name ->
       putStrLn ("Hello, " ++ name ++ "!")))
```

（`do` 语句实质上是 `>==` 的语法糖）

这样写的原因是 Haskell 中对于语句的执行顺序并没有严格规定（惰性求值），但 IO 操作必须以顺序的方式执行。使用 Monad 嵌套后，第一行、第二行、第三行就肯定会按照严格的顺序执行。（被调用前会先被求值。）

## 自函子范畴上的幺半群

这里解释一句老话“一个单子（Monad）说白了不过就是自函子范畴上的一个幺半群而已，这有什么难以理解的？”

(笔者并不了解范畴论，内容可能有错漏。)

### 范畴

要真正理解 Monad 实际上只能从范畴论的角度入手。

首先，范畴论中将所有事物都看作“对象”，1 是对象，`lambda x: x * 2` 是对象，`String::from("hello")` 是对象，姚明是对象，汽车的行驶是对象，加法是对象，考试作弊的方法是对象，范畴也是对象（这个很重要），你正在读的这句话也是对象。

对象时间的关系被称为“态射”，最常见的态射就是函数或者说映射（如集合论中），态射也是一种对象。

一个范畴 $\mathcal{C}$ 由两个类给定：一个对象的类和一个态射的类。用人话来说就是一些对象和对象之间的态射。

态射之间有组合操作，组合操作的幺元是单位态射 $id$，它不改变对象，与任何态射组合都得到原态射本身。

### 函子

前面说过范畴也是对象，所以说范畴之间也存在态射。**函子就是范畴之间的态射**

以下内容摘自维基百科：

> 设 $\mathcal{C}$ 和 $\mathcal{D}$ 为范畴。从 $\mathcal{C}$ 到 $\mathcal{D}$ 的函子为一映射 $F$ ：

> 1. 将每个对象 $X \in \mathcal{C}$ 映射至一对象 $F(X) \in \mathcal{D}$ 上，
> 2. 将每个态射 $f:X \rightarrow Y \in \mathcal{C} $ 映射至一态射上，使之满足下列条件：
>  - 对任何对象 $X \in \mathcal{C}$ ，恒有 $F(id_X) = id_{F(X)}$ 。
> - 对任何态射 $f:X \rightarrow Y$, $g:Y \rightarrow Z$ ，恒有 $F(g \circ f) = F(g) \circ F(f)$ 。换言之，函子会保持单位态射与态射的复合。
>
> 由一范畴映射至其自身的函子称之为“自函子”。

上面的两个条件在 Haskell 中的表述其实并不复杂：

```haskell
fmap id = id
fmap (f . g) = fmap f . fmap g
```

显而易见的，整个 Haskell 中只有一个范畴，即数据类型。**所以在 Haskell 中，所有的函子都是自函子**。

### Cat 范畴

Cat 范畴简单来说就是范畴的范畴，它的态射就是函子。Cat 范畴中也有单位态射，也称为单位函子 $Id$，它是一个特殊的自函子。同理，单位函子不改变范畴，与任何函子组合都得到原函子本身。

### 幺半群

幺半群（Monoid）简单来说就是有幺元的半群。

半群是指一个非空集合 $S$，$S$ 上定义了一个闭合二元运算运算 $\circ$ （闭合是指 $\circ: S \times S \rightarrow S$），满足结合律：

对于任意 $a, b, c \in S$，$(a \circ b) \circ c = a \circ (b \circ c)$

则 $\langle S, \circ \rangle$ 是一个半群。

幺半群是指对于一个半群 $\langle S, \circ \rangle$，存在 $e \in S$（幺元），使得其满足单位元定律：

对于任意 $a \in S$，$a \circ e = e \circ a = a$

则 $\langle S, \circ, e \rangle$ 是一个幺半群。

例如 $\langle N, +, 0 \rangle$ 就是一个幺半群。

在 Haskell 中幺半群是这么定义的：

```haskell
class Monoid m where
    mempty :: m
    mappend :: m -> m -> m
    mconcat :: [m] -> m
    mconcat = foldr mappend mempty
```

其中 `mempty`就是半群中的幺元，`mappend`就是半群中的二元运算。

 $\langle N, +, 0 \rangle$ 可以表示为

```haskell
instance Monoid Int where
    mempty = 0
    mappend = (+)
```

同样的，$\langle N, \times, 1 \rangle$ （也是一个幺半群）可以表示为

```haskell
instance Monoid Int where
    mempty = 1
    mappend = (*)
```

### Monad

Monad 说白了不过就是自函子范畴上的一个幺半群而已。

考虑一个 Cat 范畴，它只有一个对象：范畴 $\mathcal{C}$ ，那么它的态射就是就是范畴 $\mathcal{C}$ 的自函子，这些自函子构成一个自函子范畴。

**Haskell 中只有数据类型一个范畴，所以 Haskell 中的所有函子都是自函子，构成一个自函子范畴，Monad 是 Haskell 的自函子的一个子集。而前面说过这个子集本身符合幺半群的定义，所以 Monad 本质上就是自函子范畴上的一个幺半群。**

这就解释了那句老话“一个单子（Monad）说白了不过就是自函子范畴上的一个幺半群而已，这有什么难以理解的？”