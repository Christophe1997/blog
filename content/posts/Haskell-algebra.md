---
title: Haskell algebra
date: 2019-03-24 14:32:05
categories: 
- language concepts 
tags:
- Haskell
---

## Semigroup ##
半群即一个集合以及在之上定义的一个满足结合律的二元运算, 在Haskell中定义为(部分):
<!-- more -->
```haskell
class Semigroup where
  (<>) :: a -> a -> a
  {-# MINIMAL (<>) #-}
```
### Laws ###
```haskell
-- associativity
a <> (b <> c) = (a <> b) <> c
```

## Monoid ##
"A monoid is a binary associative operation with an identity", 在Haskell中定义为(部分):
```haskell
class Monoid m where
  mempty :: m
  mappend :: m -> m -> m
  mconcat :: [m] -> m
  mconcat = foldr mappend mempty
  {-# MINIMAL mempty #-}
```
### Laws ###
```haskell
-- left identity
mappend menpty x = x

-- right identity
mappend x mempty = x

-- associativity
mappend x (mappend y z) = mappend (mappend x y) z
```
这和数学上带幺半群所满足的性质是一致的, 带幺半群即一个包含单位元的半群.

## Functor ##
Functor最初由逻辑学家Rudolf Carnap在1930s引入, 其接受一个_sentence_(_phrase_)作为输入, 并生成一个_sentence_(_phrase_)作为输出. Functor在Haskell中定义为(部分):
```haskell
-- 易见一个Functor的kind必为`* -> *`
class Functor f where
  fmap :: (a -> b) -> f a -> f b
  {-# MINIMAL fmap #-}
```
其中`fmap`的中缀运算符为`<$>`.

### Laws ###
```haskell
-- identity
fmap id = id

-- Composition
fmap (f . g) == fmap f . fmap g
```
Functor是可堆叠的(_stacked_), 对于多个Functor嵌套的类型，可以通过多次复合fmap来获得对应不同层级的fmap:
```haskell
Prelude> :t (fmap . fmap)
(fmap . fmap)
  :: (Functor f1, Functor f2) => (a -> b) -> f1 (f2 a) -> f1 (f2 b)
  
Prelude> :t (fmap . fmap . fmap)
(fmap . fmap . fmap)
  :: (Functor f1, Functor f2, Functor f3) =>
     (a -> b) -> f1 (f2 (f3 a)) -> f1 (f2 (f3 b))
```
简单的推导:
```shell
(.) :: (b -> c) -> (a -> b) -> a -> c
fmap :: (m -> n) -> f m -> f n
fmap :: (x -> y) -> g x -> g y
=> 
(m -> n) <=> b
(f m -> f n) <=> c
(x -> y) <=> a
(g x -> g y) <=> b
=>
(m -> n) <=> (g x -> g y)
=> 
m <=> g x and n <=> g y
=>
(fmap . fmap) <=> a -> c
			  <=> (x -> y) -> (f m -> f n)
              <=> (x -> y) -> (f g x -> f g y
```

### IO Functor ###
Haskell的IO是Haskell关键设计之一, 由于其没有构造器, 因此只能使用IO typeclass所提供的来处理`IO a`, 其中最简单的处理之一是Functor.
例如:
```haskell
-- read :: Read a => String -> a
-- getLine :: IO String
getInt :: IO Int
getInt = fmap read getLine
```
`getLine`应当看成是获取String的方法(_a way to obtain a string_), IO不确保副作用会被执行, 而是确保副作用可以被执行. 我们可以用`fmap`对输入做任何处理:
```haskell
Prelude> fmap (+1) getInt
1
2
```
这和do notation是一致的:
```haskell
incIt :: IO int
incIt = do
  input <- getInt
  return (input + 1)
```

## Applicative ##
Applicative是monoidal functor, 其在Haskell中定义为(部分):
```haskell
class Functor f => Applicative f where
pure :: a -> f a
(<*>) :: f (a -> b) -> f a -> f b  -- apply
{-# MINIMAL pure, ((<*>) | liftA2) #-}

-- 定义fmap
fmap f x = pure f <*> x
```
### Laws ###
```haskell
-- idenitity
pure id <*> v = v

-- Composition
pure (.) <*> u <*> v <*> w = u <*> (v <*> w)

-- Homomorphism
pure f <*> pure x = pure (f x)

-- Interchange
u <*> pure y = pure ($ y) <*> u
```

## Monad ##
Monad是Haskell中讨论最多的结构, 但严格来讲Monad对于Haskell并不是必须的(_Haskell Programming From First Principle_ p745). 当前的Haskell确实使用Monad来构成和转换IO动作, 但更早的Haskell并不是.

Monad是applicative functor, 但有一些唯一的特性使得其比applicative或functor更强大, 其在haskell中定义为(部分):
```haskell
class Applicative m => Monad m where
(>>=) :: m a -> (a -> m b) -> m b  -- bind 
(>>) :: m a -> m b -> m b
return :: a -> m a
{-# MINIMAL (>>=) #-}

-- 定义fmap
fmap f xs = xs >>= return . f
```
因此, 任何Monad的实例都必须是Applicative和Functor的实例.

### Laws ###
```haskell
-- left identity
return x >>= f = f x

-- right identity
m >>= return = m

-- associativity
(m >>= f) >>= g = m >>= (\x -> f x >>= g)
```

### Monad与计算 ###

一种Monad定义了一种计算类型, `return`用来构造该计算类型的值, `bind`用来组合这些计算以构建更为复杂的计算. 例如, `Maybe`Monad定义了可能不返回任何值的计算, 而`List`Monad则定义了模糊计算, 即返回结果包含多个值的计算.

Haskell的do-notation允许使用命令式的风格来写monadic计算, Monad计算的值可以通过`<-`来"绑定"(do-notation中才能使用), 例如`x:xs <- Just [1, 2, 3]`, `x:xs`会匹配`[1, 2, 3]`. 一个do-notation的块也可以使用分号和大括号, 例如`do {a <- Just 2; b <- Just 3; ...}`. 因此, do-notation很像命令式编程, 虽然其只是语法糖, 例如你可以将`x <- expr1`写成`expr1 >>= \x ->`, 没有绑定的`expr2`写成`expr2 >>= \_ ->`, 可见相比与不使用do-notation, 其要方便很多, 可以说是很甜了.

Haskell的Monad的定义中还包含了两个函数, `fail`和`>>`:

```haskell
fail :: String -> m a
fail s = error s

(>>) :: m a -> m b -> m b
m >> k = m >>= \_ -> k
```

`fail`函数在do-block中模式匹配失败时被调用:

```haskell
-- fn 1会调用fail直接返回Nothing
-- fail _ = Nothing
fn :: Int -> Maybe [Int]
fn idx = do let l = [Just [1,2,3], Nothing]
            (x:xs) <- l!!idx   -- a pattern match failure will call "fail"
            return xs
```

### MonadPlus ###

除了上述三条基本的规则以外, 一些Monad还符合额外的规则, 这些Monad有一个`mzero`和`mplus`:

```haskell
-- mzero类似0
-- mplus类似加法
-- >>=类似乘法
mzero >>= f == mzero
m >>= \x -> mzero == mzero
mzero `mplus` m == m
m `mplus` mzero == m
```

Haskell中这样的Monad可以实例化`MonadPlus`:

```haskell
-- 构成了一个加法群
class (Monad m) => MonadPlus m where
  mzero :: m a
  mplus :: m a -> m a -> m a
```

## Foldable

对Foldable最恰当的描述应该是其官方文档的描述: "class of data structures that can be folded to a summary value", 其在Haskell中定义为(部分):
```haskell
class Foldable (t :: * -> *) where
  fold :: Monoid m => t m -> m  -- Data.Foldable.fold
  foldMap :: Monoid m => (a -> m) -> t a -> m
  foldr :: (a -> b -> b) -> b -> t a -> b
  foldl :: (b -> a -> b) -> b -> t a -> b
  {-# MINIMAL foldMap | foldr #-}
```

另外还有一些基本的操作:

```haskell
-- List element of a structure from left to right
-- import Data.Foldable
toList :: Foldable t => t a -> [a]

-- Test whether the structure is empty
null :: Foldable t => t a -> Bool

-- Return the size of a finite structure
length :: Foldable t => t a -> Int

-- Does the element occur in the structure
elem :: (Foldable t, Eq a) => a -> t a -> Bool

-- The lagest element of a non-empty structure
maximum :: (Foldable t, Ord a) => t a -> a

-- The least element of a non-empty structure
minimum :: (Foldable t, Ord a) => t a -> a

sum :: (Foldable t, Num a) => t a -> a
product :: (Foldable t, Num a) => t a -> a
```

一个不明显的例子:

```haskell
Prelude> fmap length Just [1, 2, 4]
1
```

此处`fmap`作用在`Just :: a -> Maybe a`上, 即作用在`(->) a Maybe a`上, 而`(->)`的`fmap = (.)`([source](https://hackage.haskell.org/package/base-4.10.1.0/docs/src/GHC.Base.html#line-693)), 因此`fmap length Just == (length . Just)`.

## Traversable

Traversable依赖于Applicative, 因此也依赖与Functor, 并且是Foldable的升级版:

```haskell
class (Functor t, Foldable t) => Traversable t where
  -- mapM is traverse
  traverse :: Applicative f => (a -> f b) -> t a -> f (t b)
  traverse f = sequenceA . fmap f
  
  -- Evaluate each action in the structure from left to right
  -- and collect the results
  -- sequence is sequenceA
  sequenceA :: Applicative f => t (f a) -> f (t a)
  sequenceA = traverse id
  {-# MINIMAL traverse | sequenceA #-}
```

Traversable可以用来翻转两个类型构造器, 或者先map在翻转.

### Laws

```haskell
-- traverse function
-- Naturality
t . traverse f = traverse (t . f)

-- Identity
traverse Identity = Identity

-- Composition
traverse (Compose . fmap g . f) = Compose . fmap (traverse g) . traverse f

-- sequenceA function
-- Naturality
t . sequenceA = sequenceA . fmap t

-- Identity
sequenceA . fmap Identity = Identity

-- Composition
sequence . fmap Compose = Compose . fmap sequenceA . sequenceA
```

## `(->) r` as Functor, Applicative and Monad

`(->)`通常称为函数箭头或者函数类型构造器, 因此`(->) r`的kind为`* -> *`, 于是其也可以实例化Functor, Applicative以及Monad(Reader):

```haskell
instance Functor ((->) r) where
  fmap = (.)
  
instance Applicative ((->) a) where
  pure = const
  (<*>) f g x = f x (g x)
  liftA2 q f g c = q (f x) (g x)
  
-- Reader also
instance Monad ((->) r) where
  f >>= k = \r -> k (f r) r
```

