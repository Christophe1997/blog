---
title: priority queue
date: 2018-04-04 16:18:50
categories: 
- algorithm
tags:
- Java
---
## 优先队列 ##
当一颗二叉树的每个节点都大于等于它的两个子结点时, 它被称为堆有序, 于是从任一结点向上都能够得到一列非递减的元素. 二叉堆(以下简称堆)是一组能够用堆有序
的完全二叉树排序的元素, 并在数组中按照层级存储. 在一个堆中, 位置k的节点的父结点的位置为$ \lfloor k/2 \rfloor $, 而它的两个子节点的位置是2k和
2k+1.

<!-- more -->

对于一个含有N个元素的堆, 插入元素操作只需不超过lgN+1次比较, 删除最大元素的操作不超过2lgN次比较.

```java
public class MaxPQ<Key extends Comparable<Key>> {
    private Key[] pq;
    private int N = 0;

    @SuppressWarnings("unchecked")
    private Key[] cast(Object obj) {
        return (Key[]) obj;
    }

    private boolean less(int i, int j) {
        return pq[i].compareTo(pq[j]) < 0;
    }

    private void exch(int i, int j) {
        Key t = pq[i];
        pq[i] = pq[j];
        pq[j] = t;
    }

    private void swim(int k) {
        while (k > 1 && less(k / 2, k)) {
            exch(k, k / 2);
            k = k / 2;
        }
    }

    private void sink(int k) {
        while (2 * k <= N) {
            int j = 2 * k;
            if (j < N && less(j, j + 1)) j++;
            if (!less(k, j)) break;
            exch(k, j);
            k = j;
        }
    }

    private void resize(int newSize) {
        Key[] t = cast(new Comparable[newSize]);
        System.arraycopy(pq, 1, t, 1, N);
        pq = t;
    }

    public MaxPQ() {
        pq = cast(new Comparable[1]);
    }

    public MaxPQ(int max) {
        pq = cast(new Comparable[max + 1]);
    }

    public MaxPQ(Key[] a) {
    }

    public void insert(Key v) {
        if (N >= pq.length - 1) resize(pq.length * 2);
        pq[++N] = v;
        swim(N);
    }

    public Key max() {
        return pq[1];
    }

    public Key delMax() {
        if (N < (pq.length - 1) / 2) resize(pq.length / 2);
        Key max = pq[1];
        exch(1, N);
        pq[N--] = null;
        sink(1);
        return max;
    }

    public boolean isEmpty() {
        return N == 0;
    }

    public int size() {
        return N;
    }
}
```

## 索引优先队列 ##
索引优先队列可以看成一个能够快速访问其中最小元素的数组, 事实上, 它能够快速访问数组的一个特定子集中的最小元素.
```java
public class IndexMinPQ<Key extends Comparable<Key>> {

    private int[] pq;
    private int[] qp;
    private Key[] keys;
    private int N = 0;

    @SuppressWarnings("unchecked")
    private Key[] cast(Object obj) {
        return (Key[]) obj;
    }

    private boolean greater(int i, int j) {
        return keys[pq[i]].compareTo(keys[pq[j]]) > 0;
    }

    private void exch(int i, int j) {
        int t = pq[i];
        pq[i] = pq[j];
        pq[j] = t;
        qp[pq[i]] = i;
        qp[pq[j]] = j;
    }

    private void swim(int k) {
        while (k > 1 && greater(k / 2, k)) {
            exch(k, k / 2);
            k = k / 2;
        }
    }

    private void sink(int k) {
        while (2 * k <= N) {
            int j = 2 * k;
            if (j < N && greater(j, j + 1)) j++;
            if (!greater(k, j)) break;
            exch(k, j);
            k = j;
        }
    }

    public IndexMinPQ(int max) {
        pq = new int[max + 1];
        qp = new int[max + 1];
        keys = cast(new Comparable[max + 1]);
        for (int elem : qp) elem = -1;
    }

    public void insert(int k, Key key) {
        N++;
        qp[k] = N;
        pq[N] = k;
        keys[k] = key;
        swim(N);
    }

    public void change(int k, Key key) {
        keys[k] = key;
        swim(qp[k]);
        sink(qp[k]);
    }

    public boolean contains(int k) {
        return qp[k] != -1;
    }

    public void delete(int k) {
        int index = qp[k];
        exch(index, N--);
        swim(index);
        sink(index);
        keys[k] = null;
        qp[k] = -1;
    }

    public Key min() {
        return keys[pq[1]];
    }

    public int minIndex() {
        return pq[1];
    }

    public int delMin() {
        int idxOfMin = pq[1];
        exch(1, N);
        qp[pq[N--]] = -1;
        sink(1);
        keys[pq[N]] = null;
        return idxOfMin;
    }

    public boolean isEmpty() {
        return N == 0;
    }

    public int size() {
        return N;
    }
}
```
