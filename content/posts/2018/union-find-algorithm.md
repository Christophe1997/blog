---
title: union-find算法
date: 2018-03-14 22:54:16
categories: 
- Things I Learned
tags:
- Java
- Algorithm
---

union-find算法是一个解决动态连通性的经典算法, 具有广泛性的应用. 简单起见, 我们将对象称为触点, 将整数对称为连接, 将等价类称为连通分量, 并用0
到N-1的整数来表示N个触点.

<!-- more -->

## API ##
```java
public class UF {
    UF(int N)  // 以整数标识初始化N个触点
    void union(int p, int q)  // 在p和q之间添加一条连接
    int find(int p)  //p所在分量的标识符
    boolean connected(int p, int q)  // p和q之间是否存在于同一个分量中
    int count()  // 连通分量的数量
}
```
成本模型: 以数组的访问次数作为成本模型, 无论读写.

算法的关键在于`find`和`union`的设计上.

## quick-find算法 ##
```java
//quick-find算法
public int find(int p) {
    return id[p];
}
public void union(int p, int q) {
    int pID = find(p), qID = find(q);  // 两次读操作
    if (pID != qID) {
        // N次读操作, 1~(N-1)次写操作
        for (int i = 0; i < id.length; i++) if (id[i] == pID) id[i] = qID;
        count--;
    }
}
```
quick-find使用一个触点为索引的数组作为数据结构, 每次`find()`调用访问数组一次, 而`union()`调用访问数组(N+3)到(2N+1)次. 假设我们最后只得到
了一个连通分量, 那么至少需要调用N-1次`union()`, 即至少$ (N+3)(N-1) \sim N^2 $次数组访问. 意味着该算法是平方级别的, 也意味着该算法不试用于
规模更大(百万级)的场景.

## quick-union算法 ##
这个算法在于提高`union()`的速度, 和上面的quick-find相互补.
```scala
public int find(int p) {
    while (p != id[p]) p = id[p];  // 2*树的深度+1
    return p;
}
public void union(int p, int q) {
    int pRoot = find(p);
    int qRoot = find(q);
    if (pRoot != qRoot) {
        id[pRoot] = id[qRoot];
        count--;
    }
}
```
quick-union也使用一个触点为索引的数组作为数据结构, 但对应的元素都是同一个连通分量中另一个触点的名字(链接). 该算法将`union()`操作降低到线性级别,
但`find()`操作在最好的情况下只需访问一次资源, 而最坏的情况是2N+1(这是较为保守的估计, 因为while循环在编译后对`id[p]`的第二次引用一般不会访问数
组). 对于一般的输入数据而言, 是对quick-find算法较好的改进, 但对于有些输入并不会比quick-find快.

同样, 假设我们最后只得到了一个连通分量, 在最坏的情况下是平方级别的: 输入的整数都是有序数对, 0-1, 0-2, 0-3等, 此时find访问数组的次数为
$ 3+5+7+\cdots+(2N-1)\sim N^2 $

## 加权quick-union算法(union-find) ##
为了减少quick-union算法最坏情况出现, 将原来`union()`操作中随意将一棵树连接到另一棵树改为总是将较小的树连接到较大的树上.
```java
// 新增sz权重数组
public void union(int p, int q) {
    int i = find(p); int j = find(q);
    if (i != j) {
        if (sz[i] < sz[j]) {
            id[i] = j;
            sz[j] += sz[i];
        } else {
            id[j] = i;
            sz[i] += sz[j];
        }
        count--;
    }
}
```
改进的目的在于降低树的高度, 以减少`find()`调用的访问数组次数. 在最坏的情况下, 成本模型的增长数量级为logN. 即加权quick-union算法在处理N个触点
和M条连接时最多访问数组cMlgN.

## 最优算法 ##
理想情况下的最优算法应该能够保证在常数时间内完成`union()`和`find()`调用, 但这是非常困难的. 通常情况下, 路径压缩的加权算法quick-union算法是
最优算法:
```java
public int find(int p) {
    while (p != id[p]) {
        id[p] = id[id[p]];  //将所有节点都链接到根节点
        p = id[p];
    }
    return p;
}
```
路径压缩算法将路径上遇到的所有都链接到根节点, 其和quick-find在理想情况下得到的树非常接近, 但并非所有操作都在常数时间完成. 但实际情况下已经不太可能
对加权quick-union算法进行任何改进了(笔者在测试数据为200W条时, 所得到的提升较小), 而且不存在其他算法能够保证union-find算法的所有操作在均摊后都
是常数级别的(在非常一般的cell probe模型之下, 即只记录对随机内存的访问, 内存大小足以保存所有输入且假设其他操作均没有成本). 使用路径压缩的加权quic
k-union算法已经是我们对于这个问题能够给出的最优解.
