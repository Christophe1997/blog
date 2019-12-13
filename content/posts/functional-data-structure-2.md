---
title: "functional data structure 2"
date: 2018-07-24T15:54:24
categories: 
- notes
tags:
- OCaml
- Functional
---

## amortization ##
Implementations with good amortized bounds are often simpler and faster than implementations with comparable worst-case
bounds. Given a sequence of operations, we may wish to know the running time of the entire sequence, but not care about
the running time of any individual operation.
<!-- more -->
For instance, given a sequence of n operations, we may wish to bound the
total running time of the sequence by O(n) without insisting than every individual operation run in O(1) time. We might
be satisfied if a few operations run in O(log n) or even O(n) time, provided the total cost of the sequence is only O(n).
This freedom opens up a wide design space of possible solutions, and often yields new solutions that are simpler and
faster than worst-case solutions with equivalent bounds.

To provide an amortized bound, one defines the amortized cost of each operation and then proves that, for any sequence
of operations, the total amortized cost of the operations in an upper bound on the total actual cost,
$$ \sum_{i=1}^m a_i \geq \sum_{i=1}^m t_i $$
where $ a_i $ is the amortized cost of operation i, $t_i$ is the actual cost of operation i, and m is the total number
of operations. Usually, in fact, one proves a slightly strong result: that at any intermediate stage in a sequence of 
operations, the accumulated amortized cost is an upper bound on the accumulated actual cost,
$$ \sum_{i=1}^j a_i \geq \sum_{i=1}^j t_i $$
for any j. The difference between the accumulated amortized costs and the accumulated actual costs is called the _accumulated
 savings_. Thus, the accumulated amortized costs are an upper bound on the accumulated actual costs whenever the accumulated
 saving is non-negative.
 
Amortization allows for occasional operations to have actual costs that exceed their amortized costs, such operations are
called _expensive_, while the operations whose actual costs are less than their amortized costs are called _cheap_. It's
easy to see that expensive operations decrease the accumulated saving and cheap operations increase it. The key to proving
amortized bounds is to show that expensive operations occur only when the accumulated saving are sufficient to cover the
remaining cost.

There are two techniques for analyzing amortized data structures: the _banker's method_ and the _physicist's method_. In
the banker's method, the accumulated saving are represented as _credits_ that are associated with individual locations
in the data structure. These credits are used to pay for future accesses to these locations. The amortized cost of any 
operation is defined to be the actual cost of the operation plus the credits allocated by the operation minus the credits
spent by the operation:
$$ a_i = t_i + c_i - \bar{c_i} $$
where $ c_i $ is the number of credits allocated by operation i and $\bar{c_i}$ is the number of credits spent by operation
i. Every credit must be allocated before it is spent, and no credit may be spent more than once. Therefore, $\sum c_i \geq
\sum \bar{c_i}$ which in turn guarantees that $\sum a_i \geq \sum t_i$, as desired. Proofs using the banker's method 
typically define a _credit invariant_ that regulates the distribution of credits in such a way that, whenever an expensive
operation might occur, sufficient credits have been allocated in the right locations to cover its cost.

In the physicist's method, one describes a function $\Phi$ that maps each object d to a real number called the _potential_
of d. The function $\Phi$ is typically chosen so that the potential is initially zero and is always non-negative. Then the
potential represents a lower bound on the accumulated savings.

Let d_i be the output of operation i and input of operation i + 1. Then the amortized cost of operation i is defined to
be the actual cost plus the change in potential between $d_{i-1}$ and $d_i$:
$$ a_i = t_i + \Phi(d_i) + \Phi(d_{i-1}) $$
so, the accumulated actual costs of the sequence of operations are:
$$ \sum_{i=1}^j t_i = \sum_{i=1}^j a_i + \Phi(d_0) - \Phi(d_j) $$
Provided $\Phi$ is chosen in such a way that $\Phi(d_0)$ is zero and $\Phi(d_j)$ is non-negative, then we conclude that
the accumulated amoritized costs are an upper bound on the accumulated actual costs.

### queue ###
We illustrate the banker's and physicist's methods by analyzing a simple functional implementation of the FIFO queue 
abstraction as:
```ocaml
module type Queue = sig
  type 'a t
  
  val empty : 'a t
  
  val isEmpty : 'a t -> bool
  
  (** snoc means "cons on the right" *)
  val snoc : 'a -> 'a t -> 'a t
  
  val head : 'a t -> 'a
  
  val tail : 'a t -> 'a t
end
```
The most common implementation of queues in a purely functional setting is a pair of lists, _front_ contains the front
elements of the queue in the correct order and _rear_ contains the rear elements of the queue in reverse order:
```
type 'a t = 'a list * 'a list
```
in this representation, the head of the queue is the first element of _front_, and the last element of the queue is the
first element of _rear_, so the remains can implement straightforward:
```ocaml
let head = function
  | [], _ -> raise Empty
  | x :: tl, _ -> x

let checkf = function
  | [], rear -> (List.rev rear, [])
  | queue -> queue

let tail = function
  | [], _ -> raise Empty
  | x :: tl, rear -> checkf (tl, rear)

let snoc x = function
  | (front, rear) -> checkf (front, x :: rear)
```
Here we use the auxiliary function `checkf` so that we check if the front is empty and then reverse the rear as the new 
front. Notice that, if _front_ were empty when _rear_ was not, then the first element of the queue would be the last 
element of rear, which would take O(n) time to access. By maintaining this invariant, we guarantee that head can always
find the first element in O(1) time, and we also know that if the _front_ is empty, then so is _rear_. 

Now we show that snoc and tail both take O(1) amortized time using either the banker's method or the physicist's method,
though `tail` takes O(n) time in the worse-case. Using the banker's method, we maintain a credit invariant that every element
in the real list is assiciated with a single credit. Every snoc into a non-empty queue takes one actual step and allocates
a credit to the new element of the rear list, for an amortized cost of two. Every tail that does not reverse the rear list,
takes one actual step and neither allocates nor spends any credits, for an amortized cost of one. Finally, every tail that
does reverse the rear list takes $ m+1 $ actual steps, where m is the length of the rear list, and spends the m credits 
contained by that list, for an amortized cost of $ m + 1 - m = 1 $.

Using the physicist's method, we define the potential function $\Phi$ to be the length of the rear list. Then every snoc
into a non-empty queue takes one actual step and increases the potential by one, for an amortized cost of two. Every tail
that does not reverse the rear list takes one actual step and leaves the potential unchanged, for an amortized cost of
one. Finally, every tail that does reverse the rear list takes $m + 1$ actual steps and sets the new rear list to [],
decreasing the potential by m, for an amortized cost of $ m+1-m = 1 $

### splay heaps ###
Splay trees are perhaps the most famous and successful of all amortized data structure. Splay trees are a close relative
of balanced binary search trees, but they maintain no explicit balance information. Instead, every operation blindly
restructures the tree using some simple transformations that tend to increase balance. Although any individual operation
can take as much as O(n) time, but every operation runs in O(log n) amortized time. A major difference between splay trees
and balanced binary search trees such as the red-black trees is that splay trees are restructured even during queries instead
of only during updates. This property makes it awkward to use splay trees to implement abstractions such as sets or finite
maps in a purely functional setting, because the query would have to return the new tree along with the answer. For some
abstractions, however, the queries are limited enough to avoid these problems. A good example is the heap abstraction,
where the only interesting query is findMin. In fact, splay tree make an excellent implementation of heaps.

The representation of splay tree in identical to that of unbalanced binary search trees:
```ocaml
type t = Leaf | Node of t * elem * t
```
unlike the unbalanced binary search trees, we allow duplicate elements within a single tree. Unlike the insertion into
ordinary binary search trees, we just partition the existing tree into two subtrees, one containing all the elements smaller
than or equal to the new element and one in contrast, and then construct a new tree where the root is the new element
and other two as its children:
```ocaml
let insert x t = Node (smaller x t, x, bigger x t)
```
and bigger implement as bellow(smaller implement same way):
```ocaml
let bigger pivot = function
    | Leaf -> Leaf
    | Node (left, v, right) ->
      if Elem.compare v pivot <= 0
      then bigger pivot right
      else match left with 
        | Leaf -> Node (Leaf, v, right)
        | Node (left1, v1, right1) ->
          if Elem.compare v1 pivot <= 0
          then Node (bigger pivot right1, v, right)
          else Node (bigger pivot left1, v1, Node (right1, v, right))
```
Notice that we restructure the tree to make in more balanced: every time we follow two left branches in a row, we rotate
those two nodes. Although, the tree may be still not balance in the usual sense, the new tree will be much more balanced
than the original tree. In fact, this is the guiding principle of splay trees: search paths should be restructured to
reduce the depth of every node in the path by about half.

Also we can combine bigger and smaller as one function `partition`, which return a pair:
```ocaml
let rec partition pivot = function
  | Leaf -> Leaf, Leaf
  | Node (left, v, right) as tree ->
    if Elem.compare v pivot <= 0
    then match right with
      | Leaf -> tree, Leaf
      | Node (left1, v1, right1) ->
        if Elem.compare v1 pivot <= 0
        then let small, big = partition pivot right1 in
          Node (Node (left, v, left1), v1, small), big
        else let small, big = partition pivot left1 in
          Node (left, v, small), Node (big, v1, right1)
    else match left with
      | Leaf -> Leaf, tree
      | Node (left1, v1, right1) ->
        if Elem.compare v1 pivot <= 0
        then let small, big = partition pivot right1 in
          Node (left1, v1, small), Node (big, v, right)
        else let small, big = partition pivot left1 in
          small, Node (big, v1, Node (right1, v, right))
```
The remains are straightforward:
```ocaml
(** splayHeap.ml *)
open Core

module type Heap = sig
  type t
  type elem

  exception Empty

  val empty : t
  val isEmpty : t -> bool

  val insert : elem -> t -> t
  val merge : t -> t -> t

  val findMin : t -> elem
  val deleteMin : t -> t
end

module Make (Elem : Comparable) : (Heap with type elem = Elem.t) = struct 
  type elem = Elem.t
  type t = Leaf | Node of t * elem * t

  exception Empty

  let empty = Leaf

  let isEmpty = function
    | Leaf -> true
    | _ -> false

  let rec partition pivot = function
    | Leaf -> Leaf, Leaf
    | Node (left, v, right) as tree ->
      if Elem.compare v pivot <= 0
      then match right with
        | Leaf -> tree, Leaf
        | Node (left1, v1, right1) ->
          if Elem.compare v1 pivot <= 0
          then let small, big = partition pivot right1 in
            Node (Node (left, v, left1), v1, small), big
          else let small, big = partition pivot left1 in
            Node (left, v, small), Node (big, v1, right1)
      else match left with
        | Leaf -> Leaf, tree
        | Node (left1, v1, right1) ->
          if Elem.compare v1 pivot <= 0
          then let small, big = partition pivot right1 in
            Node (left1, v1, small), Node (big, v, right)
          else let small, big = partition pivot left1 in
            small, Node (big, v1, Node (right1, v, right))

  let insert x t =
    let left, right = partition x t in
    Node (left, x, right)

  let rec merge t1 t2 = match t1, t2 with
    | Leaf, _ -> t2
    | Node (left, v, right), _ -> 
      let small, big = partition v t2 in
      Node (merge small left, v, merge big right)

  let rec findMin = function
    | Leaf -> raise Empty
    | Node (Leaf, v, _) -> v
    | Node (left, v, _) -> findMin left

  let rec deleteMin = function
    | Leaf -> raise Empty
    | Node (Leaf, v, right) -> right
    | Node (Node (Leaf, v1, right1), v2, right2) -> Node (right1, v2, right2)
    | Node (Node (left1, v1, right1), v2, right2) -> 
      Node (deleteMin left1, v1, Node (right1, v2, right2))

end
```

And the amortized cost of insert and deleteMin run in O(log n) time, proofs are omited.

A particularly pleasant feature of splay trees is that they naturally adapt to any order that happens to be present in
the input data. For example, using splay heaps to sort an already sorted list takes only O(n) time rather than O(nlog n)
time. Leftist heaps also share this property, but only for decreasing sequences. Splay heaps excel on both increasing and
decreasing sequences, as well as on sequence that are only partially sorted.

### pairing heaps ###
Pairing heaps are simple to implement and perform extremely well in parctice, but they have resisted analysis for over
ten years. Pairing heaps are heap-ordered multiway trees, as defined:
```ocaml
type t = Leaf | Node of elem * t list
```
We allow only well-formed trees, in which `Leaf` never occurs in the child list of `Node`. Pairing heaps get their name
from deleteMin operation. `deleteMin` discards the root and then merges the children in two passes. The first pass merges
children in pairs from left to right. The second pass merges the resulting trees from right to left:
```ocaml
let mergePairs = function
  | [] -> Leaf
  | [h] -> h
  | h1 :: h2 :: hs -> merge (merge h1 h2) (mergePairs hs)

let deleteMin = function
  | Leaf -> raise Empty
  | Node (x, hs) -> mergePairs hs 
```
and others are straightforward:
```ocaml
(** pairing heaps *)
open Core

module type Heap = sig
  type t
  type elem

  exception Empty

  val empty : t
  val isEmpty : t -> bool

  val insert : elem -> t -> t
  val merge : t -> t -> t

  val findMin : t -> elem
  val deleteMin : t -> t

  val fromList : elem list -> t
end

module Make (Elem : Comparable) : (Heap with type elem = Elem.t) = struct
  type elem = Elem.t
  type t = Leaf | Node of elem * t list

  exception Empty

  let empty = Leaf

  let isEmpty = function
    | Leaf -> true
    | _ -> false

  let merge h1 h2 = match h1, h2 with
    | Leaf, _ as h | _ as h, Leaf -> h
    | Node (v1, hs1), Node (v2, hs2) ->
      if Elem.compare v1 v2 <= 0
      then Node (v1, h2 :: hs1)
      else Node (v2, h1 :: hs2)

  let insert x h = merge (Node (x, [])) h

  let mergePairs = function
    | [] -> Leaf
    | [h] -> h
    | h1 :: h2 :: hs -> merge (merge h1 h2) (mergePairs hs)

  let findMin = function
    | Leaf -> raise Empty
    | Node (x, hs) -> x

  let deleteMin = function
    | Leaf -> raise Empty
    | Node (x, hs) -> mergePairs hs 

end
```
Notice that `findMin`, `insert` and `merge` all run in O(1) worst-case time, however, deleteMin can take up to O(n) time
in the worst case. And `deleteMin` run in O(log n) amortized time. 

All amortized data structures we have discussed are tremendously effective in pratice. Unfortunately, they perform a
bad in persistence. Let q be the result of inserting n elements into an initially empty queue, so that the front list
of q contains only a single element and the rear list contains n - 1 elements. So if we use q persistently by taking 
tail n times, each call takes n actual steps. And the total actual cost, including the time to build q, is $ n^2 + n $,
Thus the operation can not take O(1) amortized time, and this can be solved via lazy evaluation.

## numerical representations ##
Consider the usual representations of lists and natural numbers, along with several typical functions on each type:
```ocaml
type 'a list = Nil | Cons of 'a * 'a list

let tail = function
  | Cons (x, xs) -> xs
  | _ -> raise Empty
  
let rec append xs ys = match xs with
  | Nil -> ys
  | hd :: tl -> Cons (hd, append tl ys)
```
```ocaml
type nat = Zero | Succ of nat

let pred = function
  | Succ n -> n
  | Zero -> raise PredZero
  
let rec plus n1 n2 = match n1 with
  | Zero -> n2
  | Succ n -> Succ (plus n n2)
```
Other than the fact that list contain elements and natural numbers do not, these implementations are virtually identical.
Binomial heaps exhibit a similar relationship with binary numbers. These examples suggest a strong anology between representations
of the number n and representations container objects of size n. Functions on the container strongly resemble arithmetic
functions on the number. This analogy can be exploited to design new implementations of container abstractions -- simply
choose a representation of natural numbers with certain desired properties and define the function on the container objects
accordingly. This design fashion is called _numerical representation_.

Given a positional number system, we can implement a numerical representation based on than number system as a sequence
of trees. For example, the binary representation of 73 is 1001001, so a collection of size 73 in a binary numberical 
representation would contain three trees, of size 1, 8 and 64. Trees in numerical representations typically exhibit a 
very regular structure, for example, in binary numerical representations, all trees have size that are powers of 2. And
there are three common kinds of trees that exhibit this structure are _complete binary leaf trees_, _binomial trees_, and
_pennants_. Each tree with rank r has $ 2^r $ element.

### binary random-access lists ###
A random-access list, also called a one-sided flexible array, is a data structure that supports array-like lookup and 
update functions:
```ocaml
module type RandomAccessList = sig
  type 'a t
  
  val empty : 'a t
  val isEmpty : 'a t -> bool
  
  val cons : 'a -> 'a t -> 'a t
  val head : 'a t -> 'a
  val tail : 'a t -> 'a t
  
  val lookup : int -> 'a t -> 'a
  val update : int -> 'a -> 'a t -> 'a t
end
```
We implement random-access lists using a binary numerical representation. A binary random-access list of size n contains
a tree for each one in the binary representation of n. We choose the simplest combination of features: complete binary
leaf trees and a dense representation:
```ocaml
type 'a tree = Leaf of 'a | Node of int * 'a tree * 'a tree
type 'a digit = Zero | One of 'a tree
type 'a t = 'a digit list
```
The integer in each `Node` is the size of the tree, this number is redundant. Trees are sorted in increasing order of size,
and the order of elements is left-to-right both within and between trees. The maximum number of trees in a list of size n
is log(n+1)(all position are one), and the maximum depth of any tree is logn. Inserting an element into a binary random-acess
list is analogous to increasing a binary number. The increment function on dense binary numbers like:
```ocaml
let rec inc = function
  | [] -> One
  | Zero :: ds -> One :: ds
  | One :: ds -> Zero :: inc ds
```
Similarly, we first convert the element into a leaf, and then insert the leaf into the list follow the rule of `inc`:
```ocaml
let size = function
  | Leaf _ -> 1
  | Node (n, _, _) -> n

let link t1 t2 = Node (size t1 + size t2, t1, t2)

let cons x ts = 
  let rec consTree t = function
    | [] -> [One t]
    | Zero :: tl -> One t :: tl
    | One t1 :: tl -> Zero :: consTree (link t t1) tl
  in
  consTree (Leaf x) ts
```
Deleting an element from a binary random-access list is analogous to decrementing a binary number:
```ocaml
let rec dec = function
  | [One] -> []
  | One :: ds -> Zero :: ds
  | Zero :: ds -> One :: dec ds
```
So the implementation also follow the `dec` rule:
```ocaml
let rec unconsTree = function
  | [] -> raise Empty
  | [One t] -> t, []
  | One t :: tl -> t, Zero :: tl
  | Zero :: tl -> match (unconsTree tl) with
    | Node (_, t1, t2), ts -> t1, One t2 :: ts
    | _ -> assert false

let head ts = match (unconsTree ts) with
  | Leaf x, _ -> x
  | _ -> assert false

let tail ts = match (unconsTree ts) with
  | _, tl -> tl
```
The lookup and update functions do not have analogous arithmetic operations. Rather, they take advantage of the organization
of binary random-access lists as logarthmic-length lists of logarthmic-depth trees. Looking up or updating an element is
a two stage process, we first search the list for the correct tree, and then search the tree for the correct element. To
sum up we have:
```ocaml
(** randomAccessList.ml *)
type 'a t = 'a digit list
and 'a digit = Zero | One of 'a tree
and 'a tree = Leaf of 'a | Node of int * 'a tree * 'a tree
                                     
exception Empty

let empty = []
let isEmpty = function 
  | [] -> true
  | _ -> false

let size = function
  | Leaf _ -> 1
  | Node (n, _, _) -> n

let link t1 t2 = Node (size t1 + size t2, t1, t2)

let cons x ts = 
  let rec consTree t = function
    | [] -> [One t]
    | Zero :: tl -> One t :: tl
    | One t1 :: tl -> Zero :: consTree (link t t1) tl
  in
  consTree (Leaf x) ts

let rec unconsTree = function
  | [] -> raise Empty
  | [One t] -> t, []
  | One t :: tl -> t, Zero :: tl
  | Zero :: tl -> match (unconsTree tl) with
    | Node (_, t1, t2), ts -> t1, One t2 :: ts
    | _ -> assert false

let head ts = match (unconsTree ts) with
  | Leaf x, _ -> x
  | _ -> assert false

let tail ts = match (unconsTree ts) with
  | _, tl -> tl

let rec lookupTree i t = match i, t with
  | 0, Leaf x -> x
  | i, Node (n, left, right) ->
    if i < n / 2
    then lookupTree i left
    else lookupTree (i - n / 2) right

let rec updateTree i x t =match i, t with
  | 0, Leaf _ -> Leaf x
  | i, Leaf _ -> raise Empty
  | i, Node (n, left, right) -> 
    if i < n / 2
    then Node (n, updateTree i x left, right)
    else Node (n, left, updateTree (i - n / 2) x right)

let rec lookup i = function
  | [] -> raise Empty
  | Zero :: tl -> lookup i tl
  | One t :: tl -> 
    if i < size t 
    then lookupTree i t
    else lookup (i - size t) tl

let rec update i x = function
  | [] -> raise Empty
  | Zero :: tl -> Zero :: update i x tl
  | One t :: tl -> 
    if i < size t
    then One (updateTree i x t) :: tl
    else One t :: update (i - size t) x tl
```
All operations run in O(logn) worst-case time. But `cons`, `head` and `tail` run in O(logn) not O(1) is one disappointing 
aspect.

### zeroless representation ###
Currently, `head` is implemented via a call to `unconsTree`, this approach yields compact code `unconsTree` supports both
`head` and `tail`, but wastes time building lists that are immediately discard by `head`. For greater efficiency, we should
implement `head` directly. As a special case, `head` can easily be made to run in O(1) time whenever the first digit is
non-zero. So we seek to arrange that the first digit is always non-zero, one more principled solution is to use a zeroless
representation, in which binary numbers are constructed from ones and twos instead of zeros and ones:
```ocaml
type nat = digit list
and digit = One | Two

let inc = function
  | [] -> [One]
  | One :: ds -> Two :: ds
  | Two :: ds -> One :: inc ds
```
And the implementation of random-access list is:
```ocaml
(** randomAccessListZeroless.ml 
  * the interface is same as RandomAccessList *)
type 'a t = 'a digit list
and 'a digit = One of 'a tree | Two of 'a tree * 'a tree
and 'a tree = Leaf of 'a | Node of int * 'a tree * 'a tree

let empty = []
let isEmpty = function
  | [] -> true
  | _ -> false

exception Empty

let size = function
  | Leaf _ -> 1
  | Node (n, _, _) -> n

let link t1 t2 = Node (size t1 + size t2, t1, t2)

let cons x ts = 
  let rec consTree t = function
    | [] -> [One t]
    | One t1 :: tl -> Two (t, t1) :: tl
    | Two (t1, t2) :: tl -> One t :: consTree (link t1 t2) tl
  in consTree (Leaf x) ts

let head = function
  | One Leaf x :: tl -> x
  | Two (Leaf x, _) :: tl -> x
  | _ -> assert false

let tail = function
  | [] -> raise Empty
  | [One _] -> []
  | One _ :: One Node (_, left, right) :: tl -> Two (left, right) :: tl
  | One _ :: Two (t, Node (_, left, right)) :: tl -> Two (left, right) :: One t :: tl
  | Two (_, t) :: tl -> One t :: tl
  | _ -> assert false

let rec lookupTree i t = match i, t with
  | 0, Leaf x -> x
  | i, Leaf _ -> raise Empty
  | i, Node (n, left, right) ->
    if i < n / 2
    then lookupTree i left
    else lookupTree (i - n / 2) right

let rec updateTree i x t =match i, t with
  | 0, Leaf _ -> Leaf x
  | i, Leaf _ -> raise Empty
  | i, Node (n, left, right) -> 
    if i < n / 2
    then Node (n, updateTree i x left, right)
    else Node (n, left, updateTree (i - n / 2) x right)

let rec lookup i = function
  | [] -> raise Empty
  | One t :: tl ->
    if i < size t
    then lookupTree i t
    else lookup (i - size t) tl
  | Two (t1, t2) :: tl ->
    if i < size t1 
    then lookupTree i t1
    else if i < size t2
    then lookupTree (i - size t1) t2
    else lookup (i - size t1 - size t2) tl

let rec update i x = function
  | [] -> raise Empty
  | (One t as hd) :: tl ->
    if i < size t 
    then One (updateTree i x t) :: tl
    else hd :: update (i - size t) x tl
  | (Two (t1, t2) as hd) :: tl ->
    if i < size t1
    then Two (updateTree i x t1, t2) :: tl
    else if i < size t2
    then Two (t1, updateTree (i - size t1) x t2) :: tl
    else hd :: update (i - size t1 - size t2) x tl
```
Now the `head` and `tail` take O(1) time.

## data-structural bootstrapping ##
The term "bootstrapping" refers to solving problems via solving simple instance of the same problem. And there are three
algorithmic design techniques of data-structural bootstrapping. The first, _structural decomposition_, involves bootstrapping
complete data structures from incomplete data structures. The second, _structural abstraction_, involves bootstrapping
efficient data structures from inefficient data structures. The last bootstrapping data structures with aggregate elements
from data structures with atomic elements.

### structural decomposition ###
Typically _structural decomposition_ involves taking an implementation can handle objects only up to some bound size, and
extending it to handle objects of unbounded size. Consider typical recursive datatypes such as lists and binary leaf trees:
```ocaml
type 'a list = Nil | Cons of 'a * 'a list
type 'a tree = Leaf of 'a | Node of 'a tree * 'a tree
```
In some ways, these can be regarded as instances of some bounded size(zero for lists and one fo tree) and a rule for 
recursively decomposing larger objects into smaller objects until eventually each object is small enough to be handled
by the bounded case.

However, both of these definitions are particularly simple that the recursive component in each definition is identical
to the type being defined, which called _uniformly recursive_. In general, we reserve the term _structural decomposition_
to describe recursive data structure that are _non-uniform_ for cases that the recursive component is different from its
definition, e.g. `type 'a seq = Nil | Cons of 'a * ('a * 'a) seq`. But you can not implement structural decomposition
directly in ML, althought it allows the definition of non-uniform recursive datatypes. But the type system always disallow
the functions on such types like below:
```ocaml
(type illegal)
let rec size = function
  | Nil -> 0
  | Cons (x, ps) -> 1 + 2 * size ps
```
Fortunately it is always possible to convert a non-uniform type into a uniform type by introducing a new datatype to
collapse the different instances into a single type, for example, we can rewrite the `seq` as:
```ocaml
type 'a ep = Elem of 'a | Pair of 'a ep * 'a ep
type 'a seq = Nil | Cons of 'a ep * 'a seq
```
Notice that the `'a ep` type is isomorphic to binary leaf trees, so the version of `'a seq` is equivalent to `'a tree list`,
though we would tend to think a list of trees differently that we would think of a sequence of pairs -- some algorithms
will seem simpler or more natural for one of the representations, and some for the other.

To use sequence represent positional number system, we need to represent zero, it's easily corrected by adding another
constructor of the type:
```ocaml
type 'a seq = 
   | Nil
   | Zero of ('a * 'a) seq
   | One of 'a * ('a * 'a) seq
```
Now we can represent the sequence 0...10 as `One(0, One((1, 2), Zero(One((((3, 4), (5, 6)), ((7, 8), (9, 10))), Nil))))`,
which has 11 elements, written 1101 in binary. The pairs in this type are always balanced. In fact, another way to think
of pairs of elements is as complete binary leaf trees. And then we can replement binary random-access lists use this type.

### tries ###
Binary search trees work well when comparisons on the key or element type are cheap. This is true for simple types like
integers or characters, but may not be true for aggregate types like strings. A better solution for aggregate types such
as string is to choose a representation that takes advantage of the structure of that type. One such representation is
_tries_, also known as a _digital search trees_.

A trie is a multiway tree where each edge is labelled with a character. Edges leaving the root of a trie represent the 
first character of a string, and so on. and if the node is vaild, which means it contains a value, we can mark it as 
`Some x`, otherwise `None`. The critical remaining question is how to represent the edges leaving a node, we can represent
edges as a vector, an association list, a binary search tree, or even another trie. But all of these are just finite maps
from edges labels to tries. So we can use a given structure Map implementing finite maps over base type:
```ocaml
type 'a t = Trie of 'a option * 'a Map Map.t
```
Thus we can implement Trie as a functor from finiteMap to finiteMap:
```ocaml
module type FiniteMap = sig
  type 'a t
  type key

  exception NotFound

  val empty : 'a t
  val bind : key -> 'a -> 'a t -> 'a t
  val lookup : key -> 'a t -> 'a
    
end

module Make (Map : FiniteMap) : FiniteMap = struct
  type key = Map.key list

  type 'a t = Trie of 'a option * 'a t Map.t

  exception NotFound

  let empty = Trie(None, Map.empty)

  let rec lookup key trie = match key, trie with
    | [], Trie (None, _) -> raise NotFound
    | [], Trie (Some x, _) -> x
    | k :: ks, Trie (_, m) -> lookup ks (Map.lookup k m)

  let rec bind key x trie = match key, x, trie with
    | [], x, Trie (_, m) -> Trie (Some x, m)
    | k :: ks, x, Trie (v, m) -> 
      let t = 
        try Map.lookup k m
        with NotFound -> empty in
      let t' = bind ks x t in
      Trie (v, Map.bind k t' m)

end
```

