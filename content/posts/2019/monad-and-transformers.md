---
title: Monad and Transformers
date: 2019-04-06 20:26:43
categories: 
- Things I Learned
tags:
- Haskell
---

Monad是Haskell中讨论最多的结构, 需要更详细的探讨其相关内容, 即使它对于Haskell而言不是必须的:(.

<!-- more -->

参考: [All about Monads](https://wiki.haskell.org/All_About_Monads)

## Monad Support

除了之前介绍过的一些基本函数, Haskell本身定义了一些辅助函数配合Monad一起使用:

- `sequence`

  ```haskell
  -- 任意一个fail会导致整个fail
  sequence :: Monad m => [m a] -> m [a]
  sequence = foldr mcons (return [])
  						where mcons p q = p >>= \x -> q >>= \y -> return (x : y)
  ```

- `sequence_`和`sequence`类似但其不返回值, 在只关心序列的副作用时其非常有用.

  ```haskell
  sequence_ :: Monad m => [m a] -> m ()
  sequence_ = flodr (>>) (return ())
  ```

- `mapM`其由`sequence`和`map`定义

  ```haskell
  mapM :: Monad m => (a -> m b) -> [a] -> m [b]
  mapM f as = sequence $ map f as
  ```

- `mapM_`, 类似的使用`sequence_`定义

  ```haskell
  mapM_ :: Monad m => (a -> m b) -> [a] -> m ()
  mapM_ f as = sequence_ $ map f as
  ```

- `=<<`是`>>=`调换参数位置的版本

  ```haskell
  (=<<) :: Monad m => (a -> m b) -> m a -> m b
  f =<< x = x >>= f
  ```

上面提到的函数都是standard prelude中定义的, Haskell在Control.Monad模块中定义了更多函数.

下面是一些列表函数的Monad版本:

- `foldM`

  ```haskell
  foldM :: (Monad m) => (a -> b -> m a) -> a -> [b] -> m a
  foldM f a [] = return a
  foldM f a (x:xs) = f a x >>= \y -> foldM f y xs
  ```

- `filterM`

  ```haskell
  filterM :: Monad m => (a -> m Bool) -> [a] -> m [a]
  filterM p [] = return []
  filterM p (x:xs) = do b  <- p x
  										 ys <- filterM p xs
                        return (if b then (x:ys) else ys)
  ```

- `zipWithM`和`zipWithM_`

  ```haskell
  zipWithM :: (Monad m) => (a -> b -> m c) -> [a] -> [b] -> m [c]
  zipWithM f xs ys = sequence (zipWith f xs ys)
  
  zipWithM_ :: (Monad m) => (a -> b -> m c) -> [a] -> [b] -> m ()
  zipWithM_ f xs ys = sequence_ (zipWith f xs ys)
  ```

Monad模块中还包含了一些流程控制函数, `when`和`unless`:

```haskell
when :: (Monad m) => Bool -> m () -> m ()
when p s = if p then s else return ()

unless :: (Monad m) => Bool -> m () -> m ()
unless p s = when (not p) s
```

_Lifting_将一个non-monadic函数转换为在Monad上操作的等价函数. 最简单的lift函数是`liftM`:

```haskell
liftM :: (Monad m) => (a -> b) -> (m a -> m b)
liftM f = \a -> do {a' <- a; return (f a')}
```

Control.Monad模块中定义了`liftM`, `liftM2`到`liftM5`分别将不同参数个数的函数lift成monadic. 另外还定义了`$`的monadic版本:

```haskell
ap :: (Monad m) => m (a -> b) -> m a -> m b
ap = liftM2 ($)
```

## Monads ##

### The Identity Monad ###

Identity Monad(Data.Functor.Identity)不包含任何计算:

```haskell
newtype Identity a = Identity {runIdentity :: a}

instance Monad Identity where
  return a = Identity a
  (Identity x) >>= f = f x
```

Identity Monad是Monad转换的基石, 任意一个Monad transformer作用在Identity Monad上返回一个非转换器版本的Monad.

### The Maybe Monad ###

Maybe Monad表示有可能不返回值(`Nothing`)的计算:

```haskell
data Maybe a = Nothing | Just a

instance Monad Maybe where
  return = Just
  fail = Nothing
  Nothing >>= f = Nothing
  (Just x) >>= f = f x
  
instance MonadPlus Maybe where
  mzero = Nothing
  Nothing `mplus` x = x
  x `mplus` _ = x
```

### The Error Monad ###

Error Monad(或Exception Monad)表示可能出错或抛出异常的计算, 例如Either Monad. Haskell中的`MonadError`是由错误的类型和相应Monad构造器参数化的:

```haskell
class Error a where
  noMsg :: a
  strMsg :: String -> a
  
class (Monad m) => MonadError e m | m -> e where
  throwError :: e -> m a
  catchError :: m a -> (e -> m a) -> m a
```

`catchError`一种常见的使用是```do {action1; action2; action3} `catchError` handler ```, 其中action可以调用`throwError`, 且`handler`和do-block必须有相同的返回类型. `Either e`则实例化了`MonadError`:

```haskell
instance MonadError (Either e) where
  throwError = Left
  (Left e) `catchError` handler = handler e
  a `catchError` _ = a
```

### The List Monad ###

List monad表示可能返回0, 1, 或多个值的计算:

```haskell
data [] a = [] | a : [a]

instance Monad [] where
  m >>= f = concatMap f m
  return x = [x]
  fail s = []
  
 instance MonadPlus [] where
   mzero = []
   mplus = (++)
```

### The IO Monad

> The IO Monad is just an instance of the ST monad, where
> the state is the real world.
>
> The wonderful feature of a one-way monad is that it can support side-effects in its monadic operations but prevent them from destroying the functional properties of the non-monadic portions of the program.*

需要注意的是, IO Monad并不是IO, 而仅仅是IO类型的Monad实例.GHC经常会为了提升性能而优化代码, 诸如调整运算顺序, 共享变量, 内联函数. IO类型的最主要工作就是禁止其中的大部分工作. 显然调整运算顺序就在IO(以及ST)中被禁止, IO操作被包含在嵌套的lambdas中以保证运算顺序的不变. 之所以需要IO Monad是因为这是一种将嵌套lambdas的噪声剥离的抽象. IO Monad是一种One-way Monad, One-way Monad意味着你无法设计一个函数在IO Monad中完成计算并返回一个没有IO Monad类型的值. IO monad的定义是平台相关的, 且没有任何构造器可以使用, 也没有任何函数能够从IO Monad中获得值.

### The State Monad ###

State Monad表示带有状态的计算:

```haskell
newtype State s a = State {runState :: s -> (a, s)}

instance Monad (State s) where
  return a = State $ \s -> (a, s)
  (State x) >>= f = State $ \s -> let (v, s') = x s in runState (f v) s'
  
-- 提供了State Monad的一些接口
class MonadState m s | m -> s where
  get :: m s
  put :: s -> m ()
  
instance MonadState (State s) s where
  get = State $ \s -> (s, s)  -- 通过将值设置为状态来获取状态
  put s = State $ \_ -> ((), s)  -- 设置状态且没有值
```

### The Reader Monad ###

Reader Monad表示从共享环境中读取值的计算:

```haskell
-- Monad instance of `(->) r`
newtype Reader e a = Reader {runReader :: (e -> a)}

instance Monad (Reader e) where
  return a = Reader $ \e -> a
  (Reader r) >>= f = Reader $ \e -> runReader (f $ r e) e
  
class MonadReader e m | m -> e where
  ask :: m e
  local :: (e -> e) -> m a -> m a
  
instance MonadReader (Reader e) where
  ask = Reader id
  local f c = Reader $ \e -> runReader c (f e)
  
asks :: (MonadReader e m) => (e -> a) -> m
asks sel = ask >>= return . sel
```

### The Writer Monad ###

Write Monad表示除了计算值以外还产生数据流的计算:

```haskell
newtype Writer w a = Writer { runWriter :: (a,w) } 
 
instance (Monoid w) => Monad (Writer w) where 
    return a             = Writer (a,mempty) 
    (Writer (a,w)) >>= f = let (a',w') = runWriter $ f a in Writer (a',w `mappend` w')
    
class (Monoid w, Monad m) => MonadWriter w m | m -> w where 
    pass   :: m (a,w -> w) -> m a 
    listen :: m a -> m (a,w) 
    tell   :: w -> m () 
 
instance (Monoid w) => MonadWriter w (Writer w) where 
    pass   (Writer ((a,f),w)) = Writer (a,f w) 
    listen (Writer (a,w))     = Writer ((a,w),w) 
    tell   s                  = Writer ((),s) 
 
listens :: (MonadWriter w m) => (w -> b) -> m a -> m (a,b)
listens f m = do (a,w) <- listen m; return (a,f w)
 
censor :: (MonadWriter w m) => (w -> w) -> m a -> m a 
censor f m = pass $ do a <- m; return (a,f)
```

### The Continuation monad

Continuation Monad表示可以被打断和恢复的计算(没错, 就是continuation):

```haskell
-- r is the final result type of the whole computation
newtype Cont r a = Cont { runCont :: ((a -> r) -> r) }
  
instance Monad (Cont r) where 
    return a       = Cont $ \k -> k a
    (Cont c) >>= f = Cont $ \k -> c (\a -> runCont (f a) k)
    
class (Monad m) => MonadCont m where 
    callCC :: ((a -> m b) -> m a) -> m a 
 
instance MonadCont (Cont r) where 
    callCC f = Cont $ \k -> runCont (f (\a -> Cont $ \_ -> k a)) k
```

## Combining monads

Functors和applicatives对于composition都是封闭的, 但两个monad结合却不一定是另一个monad. 一个monad transformer是一个接受monad作为monad作为参数的类型构造器, 类似与一个wrapper(因此很多都是用newtype定义的).

### Compose

Compose类型代表着函数结合:

```haskell
newtype Compose f g a = Compose {getCompose :: f (g a)} deriving (Eq, Show)
```

这里的f, g不在是普遍意义上的函数, 而是类型构造器. 我们很容易实现Compose的Functor实例:

```haskell
instance (Functor f, Functor g) => Functor (Compose f g) where
  fmap f (Compose fga) = Compose $ (fmap . fmap) f fga
```

容易看到两个Functor结合之后依然可以是一个Functor. 同样的, 我们也可以定义Applicative实例:

```haskell
instance (Applicative f, Applicative g) => Applicative (Compose f g) where
	pure = Compose . pure . pure
	Compose fgf <*> Compose fga = Compose $ (<*>) <$> fgf <*> fga
```

如果我们要定义Compose的Monad实例, 就需要实现`(>>=)`或:

```haskell
(>>=) :: Compose f g a -> (a -> Compose f g b) -> Compose f g b
-- 或者通过join来实现>>=
join :: Compose f g (Compose f g a) -> Compose f g a 
```

我们总是可以忽略掉外层的Compose, 此时对于join而言, 我们就得打了`f g (f g a) -> f g a`, 如果存在一个函数`aux :: (Monad f, Monad g) => f (g a) -> g (f a)`, 那么我们就可以从`f g (f g a)`中获得`f g a`, 因为对于f和g而言都有各自的`join`, 使得 `f (f a) -> f a`. 然而并不是对于每个monad都存在这样的`aux`, 例如`IO (Maybe a) -> Maybe (IO a)`就不存在这样的`aux`. 因而"Monad do not compose", 此时我们就需要monad transformer.

### IdentityT

我们已经看到对于任意的两个monad的compose, `join`并不总是可以的. 因此我们需要限制其中一个monad, 希望它对于另外一个任意的monad的compose总是可行的. 我们从IdentityT开始介绍monad transformer:

```haskell
newtype Identity a = Identity {runIdentity :: a} deriving (Eq, Show)

instance Functor Identity where
  fmap f (Identity) = Identity $ f a

instance Applicative Identity where
  pure = Identity
  Identity f <*> Identity a = Identity $ f a
  
instance Monad Identity where
  Identity a >>= f = f a

newtype IdentityT m a = IdentityT {runIdentityT :: m a} deriving (Eq, Show)

instance Functor (IdentityT m) where
  fmap f (IdentityT ma) = IdentityT $ fmap f ma

instance Applicative (IdentityT m) where
	pure a = IdentityT $ pure a
	IdentityT mab <*> IdentityT ma = IdentityT $ mab <*> fa
  
instance Monad (IdentityT m) where
	IdentityT ma >>= f = IdentityT $ ma >>= runIdentityT . f
```

在这里我们限制了其中一个monad为Identity, 此时可以实现Monad实例, 因为我们有`runIdentityT`来获取额外的信息. 更一般的, 对于两个有Monad实例的类型f, g, 组合它们最终会止步与`f(g (f b))`, 而transformer的作用就是限制了g, 使得能够得到`f (f b)`.

### MaybeT

```haskell
newtype MaybeT m a = MaybeT {runMaybeT :: m (Maybe a)}
```

Functor实例和Applicative实例都可以直接从之前的Compose拿过来

```haskell
instance (Functor m) => Functor (MaybeT m) where
  fmap f (Maybe ma) = MaybeT $ (fmap . fmap) f ma
  
instance (Applicative m) => Applicative (MaybeT m) where
	pure = MaybeT . pure . pure
	MaybeT fab <*> MaybeT mma = MaybeT $ (<*>) <$> fab <*> mma
```

终于到了Monad实例:

```haskell
instance (Monad m) => Monad (MaybeT m) where
	(MaybeT ma) >>= f = MaybeT $ do
		v <- ma
		case v of
			Nothing -> return Nothing
			Just y -> runMaybeT (f y)
```

### EitherT

EitherT和MaybeT的处理是类似的.

```haskell
newtype EitherT e m a = EitherT {runEitherT :: m (Either e a)}

instance Functor m => Functor (EitherT e m) where
  fmap f (EitherT ema) = EitherT $ (fmap . fmap) f ema

instance Applicative m => Applicative (EitherT e m) where
  pure = EitherT . pure . pure
  EitherT emab <*> EitherT ema = EitherT $ (<*>) <$> emab <*> ema

instance Monad m => Monad (EitherT e m) where
  EitherT ema >>= f = EitherT $ do
    v <- ema
    case v of
      Right y -> runEitherT $ f y
      Left x -> return $ Left x
```

### ReaderT

ReaderT是常规的Haskell应用中用到最多的transformer

```haskell
newtype ReaderT r m a = ReaderT {runReaderT :: r -> m a}
```

不过其处理和MaybeT以及EitherT处理依然是类似的, 注意ReaderT的参数是一个函数.

```haskell
instance Functor m => Functor (ReaderT r m) where
  fmap f (ReaderT rma) = ReaderT $ (fmap . fmap) f rma

instance Applicative m => Applicative (ReaderT r m) where
  pure = ReaderT . pure . pure
  ReaderT fmab <*> ReaderT rma = ReaderT $ (<*>) <$> fmab <*> rma

instance Monad m => Monad (ReaderT r m) where
  ReaderT rma >>= f = ReaderT $ \r -> do
    a <- rma r
    runReaderT (f a) r
```

### StateT

StateT和ReaderT是类似的不过StateT还需要额外处理状态

```haskell
newtype StateT s m a = StateT {runStateT :: s -> m (a, s)}

instance Functor m => Functor (StateT s m) where
  fmap f (StateT sma) = StateT $ fmap (\(a, s1) -> (f a, s1)) . sma

-- http://stackoverflow.com/questions/18673525/
instance Monad m => Applicative (StateT s m) where
  pure x = StateT $ \s -> return (x, s)
  StateT smab <*> StateT sma = StateT $ \s -> do
    (fab, s1) <- smab s
    (a, s2) <- sma s1
    return (fab a, s2)

instance Monad m => Monad (StateT s m) where
  StateT sma >>= f = StateT $ \s -> do
    (a, s1) <- sma s
    runStateT (f a) s1
```

### WriterT和ListT

关于WriterT, 由于State总是能够代替Writer(State既能读又能写), 因此我们并不总是需要Writer. 实际上还有一个RWST将Reader, Writer, State结合起来的更大的类型.

```haskell
newtype RWST r w s m a = RWST {runRWST :: r -> s -> m (a, s, w)}
```

> It’s a bit too easy to get into a situation where Writer is either too
> lazy or too strict for the problem you’re solving, and then it’ll use
> more memory than you’d like. Writer can accumulate unevaluated thunks, causing memory leaks. It’s also inappropriate for logging long-running or ongoing programs due to the fact that you can’t retrieve any of the logged values until the computation is complete.

ListT也并不是总需要的, 其实现并不是很快. 而且Streaming库中的`pipes`和`conduit`总是能够很好的胜任大部分情况.

对于任何一个transformer, 我们总是能够从中恢复对应的monad类型, 只需要传入一个`Identity`类型,例如`type Maybe a = MaybeT Indentity a`

### Lifting

```haskell
fmap :: Functor f => (a -> b) -> f a -> f b
liftA :: Applicative f => (a -> b) -> f a -> f b
liftM :: Monad m => (a -> b) -> m a -> m b
```

Monad transformer也同样有lift, 其将一个monadic计算放到一个combined monad中.

```haskell
class MonadTrans t where
	lift :: (Monad m) => m a -> t m a
	
class (Monad m) => MonadIO m where
	liftIO :: IO a -> m a
```

