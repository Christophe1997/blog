---
title: "functional data structure 1"
date: 2018-07-19T17:00:11
categories:
- notes
tags:
- OCaml
- Functional
---

## introduction ##

To implement data structure in a functional way, there are two basic problems. First, from the point of view of designing
and implementing efficient data structures, functional programming's stricture against destructive updates(i.e. assignments)
is a staggering handicap, tantamount to confiscating a master chef's knives.
<!-- more -->
Imperative data structures often rely on
assignments in crucial ways, and so different solutions must be found for functional programs. The second difficulty is
that functional data strcutures are expected to be more flexible than their imperative counterparts. In particular, when
we update an imperative data structure we typically accept that the old version of the data strcuture will no longer
be available, but when we update a functional data structure, we expect that both the old and the new version of the data
structure will be available for further processing, this is called _persistent_, while the other is called _ephemeral_.
And we are not surprised if the persistent version is more complicated and even less efficient that the ephemeral one.

The term _data structure_ has at least four distinct, but related, meanings:
1. An _abstract data type_, that is, a type and a collection of functions on that type, we can refer to this as an _abstraction_.
2. A _concrete realization of an abstract data type_, we can refer to this as an _implementation_, but note that an 
implentation need not be actualized as code -- a concrete design is sufficient.
3. An _instance of a data type, such as a particular list or tree_. We can refer to such an instance generically as an
_object_ or a _version_. However, particular data types often have their own nomenclature, for example, we simply refer 
to stack or queue objects as stacks or queues.
4. A _unique identity that is invariant under updates_. For example, in a stack-based interpreter, we often speak informally
about "the stack" as if there were only one stack, rather than different versions at different times. We can refer to this
identity as a _persistent identity_.

## persistence ##
A distinctive property of functional data structures is that they are always persistent, updating a functional data 
structure does not destory the existing version, but rather creates a new version that coexists with the old one. Persistence
is achieved by coping the affected nodes of a data structure and making all changes in the copy of rather than in the 
original. Because nodes are never modified directly, all nodes that are unaffected by an update can be shared between
the old and new version of the data structure without worrying that a change in one version will inadvertently be visible
to the other.

### lists ###
Linked lists are common in imperative programming and ubiquitous in functional programming. And we can define its abstraction
as follow:
```ocaml
(** userList.mli *)
type 'a t

val empty : 'a t
val isEmpty : 'a t -> bool

exception Empty (* the exception does not matter *)

val cons : 'a -> 'a t -> 'a t
val head : 'a t -> 'a
val tail : 'a t -> 'a t
```
It can be implemented trivially using either the built-in type of lists or a custom datatype:
```ocaml
(** userList.ml *)
type 'a t = 
  | Nil 
  | Cons of 'a * ('a t) 

let empty = Nil

exception Empty

let isEmpty = function 
  | Nil -> true
  | _ -> false

let cons(hd, tl) = Cons (hd, tl)

let head = function
  | Nil -> raise Empty
  | Cons (hd, tl) -> hd

let tail = function
  | Nil -> raise Empty
  | Cons (hd, tl) -> tl
```

Another common function on the lists is append(`@` in ocaml), we denote it as `++`, and it's easy to implement it in a
O(n) way, while we can implement in a O(1) way in an imperative setting:
```ocaml
let rec (++) = function
  | Nil, ys -> ys
  | Cons (hd, tl), ys -> tl ++ Cons (hd, ys)
```
In functional setting, we have to copy the entire list to keep the persistence so that we are free to continue using the
old lists as well as the new list. Although this is undeniably a lot of copying, notice that the second list ys shares
the nodes wit the new list. Another function that illustrates these twin concepts of copying and sharing is update, which
changes the value of a node at a given index in the list:
```ocaml
let rec update xs idx x = match (xs, idx) with
| Nil, _ -> raise Subscript
| Cons (hd, tl), 0 -> Cons (x, tl)
| Cons (hd, tl), (_ as n) -> Cons (hd, update tl (n - 1) x)
```
Notice that this version of update is not tail call, so we can transform it via CPS:
```ocaml
let rec update xs idx x = 
  let rec updateK xs idx x k = match xs, idx with
    | Nil, _ -> raise Subscript
    | Cons (hd, tl), 0 -> k (Cons (x, tl))
    | Cons (hd, tl), (_ as n) -> updateK tl (n - 1) x (fun var1 -> Cons (x, var1))
  in
  updateK xs idx x (fun var1 -> var1)
```
so that the tail optimization can be applied.

### binary search trees ###
Binary search trees provide a good example of the kind of sharing the node with more than one pointer field. And a binary
search tree can implement sets or finite maps, whose minimal interfaces as:
```ocaml
module type Set = sig
  type t
  type elem 
  
  val empty : t 
  val insert : elem -> t -> t
  val member : elem -> t -> bool
end
```
```
module type finiteMap = sig
  type 'a t
  type key
  
  exception NotFound
  
  val empty : 'a t
  val bind : key -> 'a -> 'a t -> 'a t
  val lookup : key -> 'a t -> 'a (* raise NotFound if key is not found *)
end
```
where the elem is some fixed type of totally-ordered elements. A more realistic implementation would probably include 
many additional functions, such as deleting an element or enumerating all elements. An unblanced set via binary search
tree can be implemented as:
```ocaml
open Core

module UnbalancedSet(Elem : Comparable) : (Set with type elem = Elem.t) = 
  struct
    type elem = Elem.t
    type t = 
      | Leaf 
      | Node of t * elem * t

    let empty = Leaf

    exception AlreadyInSet

    let rec insert x tree = 
      let aux = function
        | Leaf -> Node (Leaf, x, Leaf)
        | Node (left, v, right) ->
          if Elem.compare x v < 0 
          then Node (insert x left, v, right)
          else if Elem.compare x v > 0
          then Node (left, v, insert x right)
          else raise AlreadyInSet 
      in
      try 
        aux tree
      with AlreadyInSet -> tree
      
    
    let rec member x tree = 
      let rec aux candidate = function
        | Leaf -> 
          match candidate with
          | None -> false
          | Some v -> Elem.compare x v = 0
        | Node (left, v, right) ->
          if Elem.compare x v < 0
          then memberHelp candidate left
          else member (Some v) right
      in
      aux None tree
  end
```
where the Comparable is:
```
module type Comparable = sig
type t
val compare : t -> t -> int
end
```
Notice that in the `member` function, we defined a auxiliary function that take a candidate that less than or equal to 
the search value such that it take no more than `d + 1` comparisons, where `d` is the depth of the tree, while in a traditional
way, we need `2d` comparisons in the worst case. And in the `insert` function, we introduce a `AlreadyInSet` exception
so that it return the tree itself while inserting an existing value to the tree to avoid the extra copying in the situation.

To sum up, we keep the persistence via sharing and copying.

## some familiar data structures in a functional setting ##
Although many imperative data structures are difficult or impossible to adapt to a functional setting, some can be adapted
quite easily.

### leftist heaps ###
Sometimes we need efficient access only to the minimum element, a data structure supporting this kind of access is called
a _priority queue_ or a _heap_:
```ocaml
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
```
Heap can be implemented as _heap-ordered_ trees, in which the element at each node is no large than the elements at its
children. Under this ordering, the minimum element in a tree is always at the root. Leftist heaps are heap-ordered binary
trees that satisfy the leftist property: the rank of any left child is at least as large as the rank of its right sibling.
The rank of node is defined to be the length of its _right spine_(the rightmost path from the node in question to an empty
node). A simple consequence of the leftist property is that the right spine of any node is always the shortest path to 
an empty node.

We define the leftist heap as functor:
```ocaml
module LeftistHeap (Elem : Comparable) : (Heap with type elem = Elem.t) = 
  struct
    type elem = Elem.t
    type t = 
      | Leaf
      | Node of int * elem * t * t  (* rank * value * left * right *)
                
    let empty = Leaf
    let isEmpty = function
      | Leaf -> true
      | _ -> false
  end
```

The key insight behind leftist heaps is that two heaps can be merged by merging their right spines as you would merge 
two sorted lists, and then swapping the children of nodes along this path as necessary to restore the leftist proerty,
this can beimplemented as follow:
```ocaml
let rec merge h1 h2 = match h1, h2 with
      | (_ as h), Leaf | Leaf, (_ as h) -> h
      | Node (_, x1, left1, right1), Node (_, x2, left2, right2) ->
        if Elem.compare x1 x2 <= 0
        then makeTree x1 left1 (merge right1 h2)
        else makeTree x2 left2 (merge h1 right2)
```
where the makeTree function make a new tree from two tree and a key:
```ocaml
let rank = function
  | Leaf -> 0
  | Node (r, _, _, _) -> r

let makeTree x h1 h2 = 
  if rank left >= rank right
  then Node (rank b + 1, x, h1, h2)
  else Node (rank a + 1, x, h2, h1)
```
Because the length of each right spine is at most logarithmic, merge runs in O(logN) time. And remaining functions are
trivial via `merge`. To sum up, we have:
```ocaml
(** leftistHeap.ml *)
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

module Make (Elem : Comparable) : (Heap with type elem = Elem.t) = 
  struct
    type elem = Elem.t
    type t = 
      | Leaf
      | Node of int * elem * t * t  (* rank * value * left * right *)
                
    let empty = Leaf

    let isEmpty = function
      | Leaf -> true
      | _ -> false

    let rank = function
      | Leaf -> 0
      | Node (r, _, _, _) -> r

    let makeTree x h1 h2 = 
      if rank h1 >= rank h2
      then Node (rank h2 + 1, x, h1, h2)
      else Node (rank h1 + 1, x, h2, h1)

    let rec merge h1 h2 = match h1, h2 with
      | (_ as h), Leaf | Leaf, (_ as h) -> h
      | Node (_, x1, left1, right1), Node (_, x2, left2, right2) ->
        if Elem.compare x1 x2 <= 0
        then makeTree x1 left1 (merge right1 h2)
        else makeTree x2 left2 (merge h1 right2)

    let insert x h = merge (Node (1, x, Leaf, Leaf)) h 
    
    exception Empty
    
    let findMin = function
      | Node (_, x, _, _) -> x
      | _ -> raise Empty

    let deleteMin = function 
      | Node (_, x, left, right) -> merge left right
      | _ -> raise Empty
      
  end
```

### binomial heaps ###
Another common implementation of heaps is binomial queues, which we call _binomial heaps_ to avoid confusion with FIFO
queues. Binomial heaps are composed of more primitive objects known as binomial trees. Binomial trees are inductively 
defined as follows:
1. A binomial tree of rank 0 is a singleton node
2. A binomial tree of rank k + 1 is formed by linking two binomial trees of rank k, making one tree the leftmost child 
of the other.

So, it is easy to see that a binomial tree of rank r contains exactly $ 2^r $ nodes. Another equivalent definition of 
binomial trees is : a binomial tree of rank r in a node with r children $ t_1, \cdots, t_r $, where each $ t_i $ is a
binomial tree of rank $ r - i $, so we can represent a node in binomial tree as an element and a list of children as:
```ocaml
type tree = Node of int * elem * (tree list)
```
Each list of children is maintained in decreasing order of rank, and elements are sorted in heap order. Now a binomial
heap is a collection of heap-orderd binomial trees in which no two trees have the same rank, this collection is represented
as a list trees in increasing order of rank.
```ocaml
type heap = tree list
```
Because each binomial tree contains $ 2^r $ elements and no two trees have the same rank, the trees in a binomial heap
of size n correspond exactly to ones in the binary representation of n. For example, the binary representation of 21
is 10101 so a binomial heap of size 21 would contain one tree of rank 0, one of rank 2 and one of rank 4. Note that, just
as the binary representation of n contains at most log(n+1) ones, a binomial heap of size n contains at most log(n+1) trees.

To insert a new element into a heap, we first create a new binomial tree with rank 0, we then step through the existing
trees in increasing order of rank until we find a missing rank, linking trees of equal rank as we go:
```ocaml
let rank = function
  | Node (r, _, _) -> r

let rec insertTree t = function
  | [] -> [t]
  | hd :: tl as h ->  
    if rank t < rank hd
    then t :: h
    else insertTree (link t hd) tl

let insert x h = insertTree (Node (0, x, [])) h
```
The worst case is insertion into a heap of size $ n = 2^k - 1 $, requiring a total of k links and $ O(k)=O(log n) $. To
merge two heaps, we step through both lists of trees in increasing order of rank, linking trees of equal rank as we go:
```ocaml
let rec merge ts1 ts2 = match ts1, ts2 with
  | [], (_ as ts) | (_ as ts), [] -> ts
  | hd1 :: tl1, hd2 :: tl2 -> 
    if rank hd1 < rank hd2
    then hd1 :: (merge tl1 ts2)
    else if rank hd2 < rank hd1
    then hd2 :: (merge tl2 ts1)
    else insertTree (link hd1 hd2) (merge tl1 tl2)
```
And the deleteMin and findMin can implement simply by calling an auxiliary function removeMinTree, which find the 
tree with minimum root and remove it from the list, return both the tree and remaining list, thus we have complete 
implementation as follow:
```ocaml
(** binomialHeap.ml *)
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
  type tree = Node of int * elem * tree list
  type t = tree list
    
  exception Empty

  let empty = []
  let isEmpty = function 
    | [] -> true
    | _ -> false
    
  let link t1 t2 = match t1, t2 with
    | Node (r, v1, ts1), Node (_, v2, ts2) -> 
      if Elem.compare v1 v2 <= 0
      then Node (r + 1, v1, t2 :: ts1)
      else Node (r + 1, v2, t1 :: ts2)

  let rank = function
    | Node (r, _, _) -> r

  let root = function
    | Node (_, v, _) -> v

  let rec insertTree t = function
    | [] -> [t]
    | hd :: tl as ts ->  
      if rank t < rank hd
      then t :: ts
      else insertTree (link t hd) tl

  let insert x ts = insertTree (Node (0, x, [])) ts

  let rec merge ts1 ts2 = match ts1, ts2 with
    | [], (_ as ts) | (_ as ts), [] -> ts
    | hd1 :: tl1, hd2 :: tl2 -> 
      if rank hd1 < rank hd2
      then hd1 :: (merge tl1 ts2)
      else if rank hd2 < rank hd1
      then hd2 :: (merge tl2 ts1)
      else insertTree (link hd1 hd2) (merge tl1 tl2)

  let rec removeMinTree = function
    | [] -> raise Empty
    | [t] -> t, []
    | hd :: tl -> match (removeMinTree tl) with
      | t, ts -> 
        if Elem.compare (root hd) (root t) <= 0 
        then hd, tl
        else t, hd :: ts

  let findMin ts = match (removeMinTree ts) with
    | t, _ -> root t

  let deleteMin ts = match (removeMinTree ts) with
    | Node (_, _, ts1), ts2 -> merge (List.rev ts1) ts2
 
end
```
And each major operation require O(log n) time in the worst case.

### red black tree ###
As we know, a simply binary search tree perform very poorly on ordered data, for which any individual operation might
take up to O(n) times. The solution to this problem is to keep each tree approximately balanced, which is knwon as 
balanced binary search tree, and red-black trees are one of the most popular families of balanced binary search tree. A
red-black tree is a binary search tree in which every node is colored either red or black. every red-black tree satisfy
the following invariants:
1. all empty nodes are black
2. no red node has a red child
3. every path from root to an empty node contains the same number of black nodes.
Take together, these invariants guarantee that the longest possible path in a red-black tree, one with alternating black
and red nodes, is no more than twice as long as the shortest possible path, one with black nodes only.

We can simple implement a red-black tree as:
```ocaml
type color = Red | Black
type t = Empty | Node of (color * t * elem * t)  (* for some convenience *)
```
The member function is the same as the function in binary search tree, because it does not care the color of node:
```ocaml
let rec member x = function
  | Empty -> false
  | Node (_, left, v, right) ->
    if Elem.compare x v < 0 
    then member x left
    else if Elem.compare x v > 0
    then member x right
    else true
```
The insert function need to mantain the two balanced invariants:
```ocaml
let insert x tree = 
  let rec aux = function
    | Empty -> Node (Red, Empty, x, Empty)
    | Node (color, left, v, right) ->
      if Elem.compare x v < 0
      then balance color (aux left) v right
      else if Elem.compare x v > 0
      then balance color left v (aux right)
      else tree
  in match (aux tree) with
  | Node (_, left, v, right) -> Node (Black, left, v, right)
  | Empty -> Empty  (* have no sense except avoid warning *)
```
First, when we create a new node with `Empty`, we initially color it red. Second we force the final root to be black, 
regardless of the color. Finally, we replace the calls to the `Node` constructor in the with the balance function, which
acts just as the `Node` constructor except that it massages its arguments as necessary to enforce the balance invariants.

Coloring the new node red maintains invariant 3, but violate invariant 2 whenever the parent of the new node is red. We
allow a single red-red violation at a time, and percolate this violation up the search path toward the root during rebalancing.
The balance function detects and repairs each red-red violation when it processes the black parent of the red node with
a red child. This black-red-red path can occur in any of four configurations, depending on whether each red node is a
left or right child. However, the solution is the same in every case: rewrite the black-red-red path as a red node with
two black children:
```ocaml
let balance color left v right = match color, left, v, right with
  | Black, (Node (Red, Node (Red, a, x, b), y, c)), z, d       (* left-left case   *)
  | Black, (Node (Red, a, x, Node (Red, b, y, c))), z, d       (* left-right case  *)
  | Black, a, x, (Node (Red, Node (Red, b, y, c), z, d))       (* right-left case  *)
  | Black, a, x, (Node (Red, b, y, Node (Red, c, z, d))) ->    (* right-right case *)
    Node (Red, Node (Black, a, x, b), y, Node (Black, c, z, d))
  | _ as tuple -> Node tuple
```
And you can see it's much elegant than any version written in imperative setting. After balancing a given subtree, the
red root of that subtree might now be the child of another red node. Thus we continue balancing all the way to the top
of the tree. At the very top of the tree, we might end up with a red node with a red child, but with no black parent, we
handle this case by always recoloring the root to be black. Now we sum it all:
```ocaml
(** redBlackSet.ml *)
open Core

module type Set = sig
  type t
  type elem

  val empty : t
  val insert : elem -> t -> t
  val member : elem -> t -> bool
end

module Make (Elem : Comparable) : (Set with type elem = Elem.t) = struct
  type elem = Elem.t
  type color = Red | Black
  type t = Empty | Node of (color * t * elem * t)

  let empty = Empty

  let rec member x = function
    | Empty -> false
    | Node (_, left, v, right) ->
      if Elem.compare x v < 0 
      then member x left
      else if Elem.compare x v > 0
      then member x right
      else true

  let balance color left v right = match color, left, v, right with
    | Black, (Node (Red, Node (Red, a, x, b), y, c)), z, d       (* left-left case   *)
    | Black, (Node (Red, a, x, Node (Red, b, y, c))), z, d       (* left-right case  *)
    | Black, a, x, (Node (Red, Node (Red, b, y, c), z, d))       (* right-left case  *)
    | Black, a, x, (Node (Red, b, y, Node (Red, c, z, d))) ->    (* right-right case *)
      Node (Red, Node (Black, a, x, b), y, Node (Black, c, z, d))
    | _ as tuple -> Node tuple

  let insert x tree = 
    let rec aux = function
      | Empty -> Node (Red, Empty, x, Empty)
      | Node (color, left, v, right) ->
        if Elem.compare x v < 0
        then balance color (aux left) v right
        else if Elem.compare x v > 0
        then balance color left v (aux right)
        else tree
    in match (aux tree) with
    | Node (_, left, v, right) -> Node (Black, left, v, right)
    | Empty -> Empty  (* have no sense except avoid warning *)
end
```
Even without optimization, this implementation of balanced binary search trees is one of the fastest around. With appropriate
optimizations, such as eliminating comparison (we have done this before) and eliminating redundant testing(while recursing
on the left child their is no need for red-red violations involving the right child)
