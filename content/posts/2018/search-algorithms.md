---
title: 查找算法
date: 2018-04-05 15:21:41
categories:
- Algorithm
tags:
- Java
---

符号表是一种存储键值对的数据结构, 支持插入和查找操作. 要确定一个给定的键是否存在于符号表中, 首先要建立对象等价性的概念. 为了保证一致性, 最好选择不
可变的数据类型作为键. 成本模型: 统计比较的次数, 在内循环不进行比较的情况下, 转为统计数组的访问次数.
<!-- more -->

## 二叉查找树 ##

每个结点的键都大于其左子树的任意结点的键而小于右子树的任一结点的键. 一棵二叉查找树代表了一组键(及其相应的值)的集合, 同一个集合可以用多棵不同的二叉树
表示, 如果将所有键都投影到一条直线上, 则可以保证得到一个有序的键列.

查找的惯用做法是: 如果含有该键的结点存在于表中, 则返回相应的值或者返回null. 我们将采用Java 8的Optional类型使得查找的结果总是返回Optional, 将
显示的调用交给使用者.

使用二叉查找树的算法的运行时间取决于树的形状. 在最好的情况下, 一棵含有N个结点的树是完全平衡的, 每条空链接和根结点的距离都为~lgN; 而在最坏的情况下
树的高度为N. 当键的分布是随机的情况下, 二叉查找树一般具有较好的平衡性, 此时查找或者插入操作平均所需的次数为~2lnN(1.39lgN). 虽然二叉查找树的查找
成本比二分查找高约39%, 但插入操作是对数级别的.
```java
public class BST<Key extends Comparable<Key>, Value> {
    private Node root;
    private class Node {
        private Node left, right;
        private Key key;
        private Value val;
        private int N;

        Node(Key key, Value val, int N) {
            this.key = key; this.val = val; this.N = N;
        }       
    }
    
    public int size() {
        return size(root);
    }

    private int size(Node node) {
        if (node == null) return 0;
        return node.N;
    }

    public Optional<Value> get(Key key) {
        return Optional.of(get(root, key));
    }

    private Value get(Node node, Key key) {
        if (node == null) return null;
        int cmp = key.compareTo(node.key);
        if (cmp > 0) get(node.right, key);
        else if (cmp < 0) get(node.left, key);
        return node.val;
    }

    public void put(Key key, Value val) {
        root = put(root, key, val);
    }

    private Node put(Node node, Key key, Value val) {
        if (node == null) return new Node(key, val, 1);
        int cmp = key.compareTo(node.key);
        if (cmp > 0) node.right = put(node.right, key, val);
        else if (cmp < 0) node.left = put(node.left, key, val);
        else node.val = val;
        node.N = size(node.left) + size(node.right) + 1;
        return node;
    }
}
```

## 平衡查找树 ##

### 2-3查找树 ###
一棵2-3查找树或为一棵空树, 或者由以下结点组成:
- 2-结点, 含有一个键和两条链接.
- 3-结点, 含有两个键和三条链接.

一棵完美平衡的2-3查找树中所有空链接到根结点的距离都应该是相同的.

### 红黑二叉查找树 ###
红黑二叉查找树是用标准的二叉查找树和一些额外的信息来表示2-3树, 树中的链接分为两者情形:
- 红链接将两个2-结点连接起来构成一个3-结点.
- 黑链接是2-3树中普通的链接.

另一种定义是含有红黑链接并满足下列条件的二叉查找树:
- 红链接均为左链接.
- 没有任何一个结点同时和两条红链接相连.
- 该树是完美黑色平衡的, 即任意空链接到根结点的路径上的黑链接数量相同.

满足这样定义的红黑树和相应的2-3树是一一对应的. (注: 标准的红黑树是允许红链接为右链接的).
一种标准红黑树的实现(具体删除插入调整细节网上很多):
```java
public class RBST<Key extends Comparable<Key>, Value> {
    private final Node nil = new Node(null, null, BLACK);
    private Node root = nil;
    private static final boolean RED = true;
    private static final boolean BLACK = false;

    private class Node {
        private Node left, right, parent;
        private Key key;
        private Value val;
        private boolean color;
//        private int N;

        Node(Key key, Value val, boolean color) {
            this.key = key;
            this.val = val;
//            this.N = N;
            this.color = color;
        }

        boolean nonLeft() {
            return left == nil;
        }

        boolean nonRight() { return right == nil; }

        boolean isNil() { return this == nil; }

        void flipColors() { color = !color; }

        @Override
        public String toString() {
            StringBuilder result = new StringBuilder();
            result.append("<key: ").append(key);
            if (parent != null) result.append(", parent: ").append(parent.key);
            result.append(">");
            return result.toString();
        }
    }

    public Iterable<Key> keys() {
        Queue<Key> queue = new Queue<>();
        keys(root, queue);
        return queue;
    }

    private void keys(Node node, Queue<Key> queue) {
        if (node.isNil()) return;
        keys(node.left, queue);
        queue.enqueue(node.key);
        keys(node.right, queue);
    }

    private boolean isRed(Node node) {
        if (node == null) return BLACK;
        return node.color == RED;
    }


    private void rotateLeft(Node node) {
        Node newNode = node.right;
        node.right = newNode.left;
        if (!newNode.left.isNil()) newNode.left.parent = node;
        newNode.parent = node.parent;
        if (node.parent.isNil()) this.root = newNode;
        else if (node == node.parent.left) node.parent.left = newNode;
        else node.parent.right = newNode;
        newNode.left = node;
        node.parent = newNode;
    }

    private void rotateRight(Node node) {
        Node newNode = node.left;
        node.left = newNode.right;
        if (!newNode.right.isNil()) newNode.right.parent = node;
        newNode.parent = node.parent;
        if (node.parent.isNil()) this.root = newNode;
        else if (node == node.parent.left) node.parent.left = newNode;
        else node.parent.right = newNode;
        newNode.right = node;
        node.parent = newNode;
    }

    private void flipColors(Node node) {
        node.flipColors();
        node.left.flipColors();
        node.right.flipColors();
    }

    public void insert(Key key, Value val) {
        Node parent = nil;
        Node current = root;
        Node insertNode = new Node(key, val, RED);
        while (!current.isNil()) {
            parent = current;
            int cmp = key.compareTo(current.key);
            if (cmp < 0) current = current.left;
            else if (cmp > 0) current = current.right;
            else {
                current.val = val;
                return;
            }
        }
        insertNode.parent = parent;
        if (parent.isNil()) this.root = insertNode;
        else if (key.compareTo(parent.key) < 0) parent.left = insertNode;
        else parent.right = insertNode;

        insertNode.left = nil;
        insertNode.right = nil;

        insertFixUp(insertNode);
    }

    private void insertFixUp(Node current) {
        Node parent = current.parent;
        Node grandparent;
        while (isRed(parent)) {
            parent = current.parent;
            grandparent = parent.parent;
            if (parent == grandparent.left) {
                Node uncle = grandparent.right;
                if (isRed(uncle)) {
                    flipColors(grandparent);
                    current = grandparent;
                } else {
                    if (current == parent.right) {
                        current = parent;
                        rotateLeft(current);
                    }
                    parent.color = BLACK;
                    grandparent.color = RED;
                    rotateRight(grandparent);
                }
            } else {
                Node uncle = grandparent.left;
                if (isRed(uncle)) {
                    flipColors(grandparent);
                    current = grandparent;
                } else {
                    if (current == parent.left) {
                        current = parent;
                        rotateRight(current);
                    }
                    parent.color = BLACK;
                    grandparent.color = RED;
                    rotateLeft(grandparent);
                }
            }
        }
        root.color = BLACK;
    }

    public Optional<Value> get(Key key) {
        return get(root, key).map(x -> x.val);
    }

    private Optional<Node> get(Node root, Key key) {
        Node current = root;
        int cmp;
        while (!current.isNil()) {
            cmp = key.compareTo(current.key);
            if (cmp > 0) current = current.right;
            else if (cmp < 0) current = current.left;
            else{
//                System.out.println("hit");
                return Optional.of(current);
            }
        }
        return Optional.empty();
    }

    public Optional<Key> min() {
        if (root.isNil()) return Optional.empty();
        return min(root).map(x -> x.key);
    }

    private Optional<Node> min(Node root) {
        if (root == nil) return Optional.empty();
        Node current = root;
        while (!current.left.isNil()) current = current.left;
        return Optional.of(current);
    }

    public Optional<Key> deleteMin() {
        return deleteMin(root).map(x -> x.key);
    }

    private Optional<Node> deleteMin(Node root) {
        Optional<Node> optionalNode = min(root);
        if (optionalNode.isPresent()) {
            Node minNode = optionalNode.get();
            Node current;
            if (minNode.nonRight()) current = nil;
            else current = minNode.right;
            current.parent = minNode.parent;
            if (minNode == minNode.parent.left) { minNode.parent.left = current; }
            else { minNode.parent.right = current; }
            if (this.root == minNode) { this.root = current; }
            deleteFixUp(current);
            return Optional.of(minNode);
        }
        return optionalNode;
    }

    private void deleteFixUp(Node current) {
        while (current != this.root && !isRed(current)) {
            if (current == current.parent.left) {
                Node sibling = current.parent.right;
                if (isRed(sibling)) {
                    sibling.color = BLACK;
                    current.parent.color = RED;
                    rotateLeft(current.parent);
                    sibling = current.parent.right;
                }
                if (!isRed(sibling.left) && !isRed(sibling.right)) {
                    sibling.color = RED;
                    current = current.parent;
                } else {
                    if (!isRed(sibling.right)) {
                        sibling.left.color = BLACK;
                        sibling.color = RED;
                        rotateRight(sibling);
                        sibling = current.parent.right;
                    }
                    sibling.color = current.parent.color;
                    current.parent.color = BLACK;
                    sibling.right.color = BLACK;
                    rotateLeft(current.parent);
                    current = this.root;
                }
            } else {
                Node sibling = current.parent.left;
                if (isRed(sibling)) {
                    sibling.color = BLACK;
                    current.parent.color = RED;
                    rotateRight(current.parent);
                    sibling = current.parent.left;
                }
                if (!isRed(sibling.left) && !isRed(sibling.right)) {
                    sibling.color = RED;
                    current = current.parent;
                } else {
                    if (!isRed(sibling.left)) {
                        sibling.right.color = BLACK;
                        sibling.color = RED;
                        rotateLeft(sibling);
                        sibling = current.parent.left;
                    }
                    sibling.color = current.parent.color;
                    current.parent.color = BLACK;
                    sibling.left.color = BLACK;
                    rotateRight(current.parent);
                    current = this.root;
                }
            }
        }
        current.color = BLACK;
        nil.parent = null;
    }

    public void delete(Key key) {
        Optional<Node> optionalNode = get(root, key);
        if (optionalNode.isPresent()) {
            Node deleteNode = optionalNode.get();
            Node parent = deleteNode.parent;
            Optional<Node> optionalTempNode = min(deleteNode.right);
            if (optionalTempNode.isPresent()) {
                Node tempNode = optionalTempNode.get();
                if (this.root == deleteNode) { this.root = tempNode; }
                flipNode(deleteNode, tempNode);
                deleteMin(tempNode.right);
            } else {
                Node current = deleteNode.left;
                if (this.root == deleteNode) { this.root = current; }
                current.parent = parent;
                if (deleteNode == parent.left) parent.left = current;
                else parent.right = current;
                deleteFixUp(current);
            }
        }
    }

    private void resetNil() {
        nil.parent = null;
        nil.left = null;
        nil.right = null;
    }

    private void flipNode(Node lnode, Node rnode) {
        Node tep;
        if (lnode == lnode.parent.left) { lnode.parent.left = rnode; }
        else { lnode.parent.right = rnode; }
        if (rnode == rnode.parent.left) { rnode.parent.left = lnode; }
        else { rnode.parent.right = lnode; }
        tep = lnode.parent;
        lnode.parent = rnode.parent;
        rnode.parent = tep;

        tep = lnode.left;
        lnode.left = rnode.left;
        rnode.left.parent = lnode;
        rnode.left = tep;
        tep.parent = rnode;

        tep = lnode.right;
        lnode.right = rnode.right;
        rnode.right.parent = lnode;
        rnode.right = tep;
        tep.parent = rnode;

        resetNil();
    }
}
```

红黑树可以看成是一种折中的平衡策略, 很多操作和二叉查找树一致, 但因为更好的平衡性而具有更好的性能.

## 散列表 ##
散列最主要的目的在于均匀地将键散布开来, 因此在散列之后会丢失键的顺序信息. 散列表实现的重点在于散列函数的选择以及处理碰撞冲突的策略.
我们假设所使用的散列函数能够均匀并独立地将所有的键散布于0到M-1之间.

### 散列函数 ###
一个散列函数应该易于计算并且能够均匀分布所有键. 一个优秀的散列方法需要满足以下条件:
- 一致性, 等价的键必须产生相等的散列值.
- 高效性, 计算简便.
- 均匀性, 均匀的散列所有值而避免大量的碰撞.

### 拉链法 ###
一种处理碰撞的方法是拉链法, 该方法将大小为M的数组中的每个元素指向一条链表, 链表中的每个结点都存储了散列值为该元素的索引的键值对. 这种方法需要选择
足够大的M, 使所有链表都尽可能短以保证高效的查找.

```java

public class SequentialSearchST<Key, Value> {
    private Node first;
    private class Node {
        Key key;
        Value val;
        Node next;

        Node(Key key, Value val, Node next) {
            this.key = key;
            this.val = val;
            this.next = next;
        }
    }

    public Optional<Value> get(Key key) {
        for (Node current = first; current != null; current = current.next) {
            if (key.equals(current.key)) return Optional.of(current.val);
        }
        return Optional.empty();
    }

    public void put(Key key, Value val) {
        for (Node current = first; current != null; current = current.next) {
            if (key.equals(current.key)) {
                current.val = val;
                return;
            }
        }
        first = new Node(key, val, first);
    }
    public void delete(Key key) {
        Node parent = first;
        for (Node current = first; current != null; current = current.next) {
            if (key.equals(current.key)) {
                if (current == first) {
                    first = current.next;
                } else {
                    parent.next = current.next;
                }
                return;
            }
            parent = current;
        }
    }
}

public class SeparateChainingHashST<Key, Value> {
    private int N;
    private int M;
    private SequentialSearchST<Key, Value> [] st;
    public SeparateChainingHashST() { this(996); }

    @SuppressWarnings("unchecked")
    public SeparateChainingHashST(int M) {
        this.M = M;
        st = (SequentialSearchST<Key, Value>[]) new SequentialSearchST[M];
        for (int i = 0; i < M; i++) {
            st[i] = new SequentialSearchST<>();
        }
    }

    private int hash(Key key) { return key.hashCode() & 0x7fffffff % M; }

    public Optional<Value> get(Key key) { return st[hash(key)].get(key); }

    public SeparateChainingHashST<Key, Value> put(Key key, Value val) {
        st[hash(key)].put(key, val);
        return this;
    }
    public void delete(Key key) {
        st[hash(key)].delete(key);
    }
}
```

### 基于线性探测法的散列表 ###
实现散列表的另一种方式就是用大小为M的数组保存N个键值对(M > N), 依靠数组中的空位解决碰撞冲突. 这类方法统称为开放地址散列表, 其中最简单的方法是
线性探测法: 当碰撞发生时, 我们检查散列表中的下一个位置:
- 命中, 该位置的键和被查找的键相同,
- 为命中, 该位置没有键, 以及
- 继续查找, 该位置的键和被查找的键不同.

```java
public class LinearProbingHashST<Key, Value> {
    private int N;
    private int M;
    private Key[] keys;
    private Value[] vals;

    @SuppressWarnings("unchecked")
    public LinearProbingHashST(int size) {
        M = size;
        keys = (Key[]) new Object[M];
        vals = (Value[]) new Object[M];
    }

    private int moduloPlusOne(int i) {return (i + 1) % M; }
    private void remove(int i) {
        keys[i] = null;
        vals[i] = null;
        N--;
    }

    private int hash(Key key) { return (key.hashCode() & 0x7fffffff) % M; }

    private void resize(int size) {
        LinearProbingHashST<Key, Value> t = new LinearProbingHashST<>(size);
        for (int i = 0; i < M; i++) {
            if (keys[i] != null) {
                t.put(keys[i], vals[i]);
            }
        }
        keys = t.keys;
        vals = t.vals;
        M = t.M;
    }

    public void put(Key key, Value val) {
        if (N > M/2) { resize(2 * M); }
        int i;
        for (i = hash(key); keys[i] != null; i = moduloPlusOne(i)) {
            if (key.equals(keys[i])) {
                vals[i] = val;
                return;
            }
        }
        keys[i] = key;
        vals[i] = val;
        N++;
    }

    public Optional<Value> get(Key key) {
        for (int i = hash(key); keys[i] != null; i = moduloPlusOne(i)) {
            if (key.equals(keys[i])) {
                return Optional.of(vals[i]);
            }
        }
        return Optional.empty();
    }

    public void delete(Key key) {
        get(key).ifPresent(value -> {
            int i = hash(key);
            while (!key.equals(keys[i])) { i = moduloPlusOne(i); }
            remove(i);
            i = moduloPlusOne(i);
            while (keys[i] != null) {
                Key keyRedo = keys[i];
                Value valRedo = vals[i];
                remove(i);
                put(keyRedo, valRedo);
                i = moduloPlusOne(i);
            }
            if (N > 0 && N < M/8) resize(M / 2);
        });
    }
}
```

线性探测的平均水平取决于元素在插入数组后聚集成的一组连续的条目, 也叫键簇. 显然, 插入的效率依赖于键簇的长度. 在一张大小为M并含有$ N=\alpha M $个
键的基于线性探测的散列表中, 在均匀散列假设的条件下, 命中和未命中查找所需的次数分别为:
$$ \sim \frac{1}{2}(1 + \frac{1}{1 - \alpha}) 以及 \sim \frac{1}{2}(1 + \frac{1}{(1 - \alpha)^2}) $$

可见当散列表在快满的时候查找所需的探测次数非常多, 因此需要动态调整数组以保证使用率不超过1/2(或其他值).

注: 关于Java的散列函数:
- Integer类型会直接返回该整数的32位值, Double和Long会返回机器表示的前32位和后32位异或的结果.
- String的hash函数为:
  $$ h(s) = \sum_{i=0}^{n-1} s[i] \times 31^{n-1-i} $$
  "polygenelubricants"的散列值为$ -2^{31} $
  

相对于二叉查找树, 散列表的实现简单, 且查找时间最优. 红黑树能够保证最差情况的性能且支持更多操作.
