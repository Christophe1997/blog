---
title: 排序算法
date: 2018-03-30 12:07:09
categories:
- Things I Learned
tags:
- Java
- Algorithm
---
排序算法的目标就是将所有元素的主键按照某种方式排列. 成本模型: 计算比较和交换的次数, 对于不交换元素的算法, 计算访问数组的次数. 排序算法的额外内存开销
和运行时间是同等重要的, 排序算法可以分为原地排序算法以及需要额外内存空间来存储另一份数组副本的其他排序算法.
<!-- more -->

我们使用的算法模板如下
```java
public interface Sort {
    static void sort(Comparable[] a) {
    }

    static boolean less(Comparable v, Comparable w) {
        return v.compareTo(w) < 0;
    }

    static void exch(Comparable[] a, int i, int j) {
        Comparable t = a[i];
        a[i] = a[j];
        a[j] = t;
    }

    static void show(Comparable[] a) {
        for (Comparable x : a) {
            StdOut.print(x + " ");
            StdOut.println();
        }
    }

    static boolean isSorted(Comparable[] a) {
        for (int i = 1; i < a.length; i++) {
            if (less(a[i], a[i - 1])) return false;
        }
        return true;
    }

    static void main(String[] args) {
        String[] a = StdIn.readAllStrings();
        sort(a);
        assert isSorted(a);
        show(a);
    }
}
```

## 选择排序 ##
最简单的算法是选择排序, 不断寻找数组剩下元素的最小值, 并将其放在适当的位置. 其特点在于:
- 运行时间与输入无关.
- 数据移动最少, 交换次数和数组大小是线性关系, 其他算法大都是线性对数或者平方级别的.
```java
public static void sort(Comparable[] a) {
    int N = a.length;
    for (int i = 0; i < N; i++) {
        int min = i;
        for (int j = i + 1; j < N; j++) {
            if (Sort.less(a[j], a[min])) min = j;
            Sort.exch(a, i, min);
        }
    }
}
```

对于长度为N的数组, 选择排序大约需要$ N^2/2 $次比较和N次交换.

## 插入排序 ##
对于随机排列长度为N且主键不重复的数组, 平均情况下插入排序需要$ \sim N^2/4 $次比较以及$ \sim N^2/4 $次交换. 最坏情况下, 比较和交换的次数
都为$ \sim N^2/2 $, 最好的情况下需要N-1次比较和0次交换.
```java
public static void sort(Comparable[] a) {
    int N = a.length;
    for (int i = 1; i < N; i++) {
        for (int j = i; j > 0 && Sort.less(a[j], a[j-1]); j--) {
            Sort.exch(a, j, j-1);
        }
    }
}
```

倒置是指数组中两个顺序颠倒的元素, 例如"E A"中E-A就是一对倒置, 如果数组中倒置的数量小于数组大小的某个倍数, 则称该数组为部分有序. 部分有序有以下
几种典型情况:
- 数组中每个元素距离它的最终位置都不远.
- 一个有序的大数组接一个小数组.
- 数组中只有几个元素的位置不正确.

插入排序对这样的数组很有效, 当倒置数量很少时, 插入排序可能比其他任何排序都快. 事实上, 设长度为N的数组的倒置数量为m, 则插入排序需要的交换操作等于m,
需要的比较次数大于等于m, 小于等于m+N-1.

对于随机排序的无重复主键的数组, 插入排序和选择排序的运行时间是平方级别的, 两者之比应该是一个较小的常数.

## 希尔排序 ##
希尔排序简单的改进了插入排序, 其思想是使数组中任意间隔为h的元素都是有序的, 这样的数组称为h有序数组. 一个h有序数组指的是h个互相独立的数组交错在一起
的数组, 当h很大时, 就能把元素移到较远的位置. 对于任意以1结尾的h序列, 数组都能够排序.
```java
public static void sort(Comparable[] a) {
    int N = a.length;
    int h = 1;
    // 使用序列1/2 (3^k - 1)
    while (h < N/3) h = 3 * h + 1;
    while (h >= 1) {
        for (int i = h; i < N; i++) {
            for (int j = i; j >= h && Sort.less(a[j], a[j - h]); j -= h) {
                Sort.exch(a, j, j - h);
            }
        }
        h /= 3;
    }
}
```
算法的性能取决于h序列的选择, 我们选择的序列在最坏的情况下与$ N^{3/2} $成正比. 其他算法在N不是特别大的情况下, 大约只比希尔排序快2倍左右.


## 归并排序 ##
归并排序保证任意长度为N的数组排序所需的时间和NlgN成正比, 它的主要缺点是所需的额外空间也和N成正比.
```java
// 自顶向下
public static void sort(Comparable[] a) {
    int N = a.length;
    // 辅助数组
    aux = new Comparable[N];
    sort(a, 0, N - 1);
}

public static void sort(Comparable[] a, int lo, int hi) {
    if (hi <= lo) return;
    int mid = lo + (hi - lo)/2;
    sort(a, lo, mid);
    sort(a, mid + 1, hi);
    merge(a, lo, mid, hi);
}

public static void merge(Comparable[] a, int lo, int mid, int hi) {
    int i = lo;
    int j = mid + 1;
    for (int k = lo; k <= hi; k++) aux[k] = a[k];

    for (int k = lo; k <= hi; k++) {
        if (i > mid) a[k] = aux[j++];
        else if (j > hi) a[k] = aux[i++];
        else if (Sort.less(aux[j], aux[i])) a[k] = aux[j++];
        else a[k] = aux[i++];
    }
}
```
对于长度为N的数组, 自顶向下的归并排序需要1/2NlgN到NlgN次比较.

> 证明: 令C(N)表示将一个长度为N的数组排序所需要的比较次数, 我们容易得到$ C(0)=C(1)=0 $, 对于N>1的情况有:
> $$ C(N) \le C(\lceil N/2 \rceil) + C(\lfloor N/2 \rfloor) + N $$
> 以及:
> $$ C(N) \ge C(\lceil N/2 \rceil) + C(\lfloor N/2 \rfloor) + N/2 $$
> 当$ N=2^{n} $时, 有解:
> $$ C(2^n)/2^n = C(2^0)/2^0 + n = n $$
> 两边同时乘以$ 2^n $并代回N,有:
> $$ C(N) = C(2^n) = n \times 2^n = NlgN $$

同理, 自顶向下的归并排序最多需要访问数组6NlgN.

用不同的方法处理小规模问题能改进大多数递归算法的性能, 因为递归会使小规模问题中方法的调用过于频繁, 从而改进对它们的处理方法能够改进整个算法. 在归并
算法中, 使用插入排序来处理小规模的子数组一般能将运行时间缩短10%~15%. 而通过测试数组是否有序可以将任何有序的子数组运行时间变为线性.

## 排序算法的复杂度 ##
我们研究的排序算法是基于比较的(忽略了访问数组的开销), 一个基于比较的算法在两次比较之间可能会进行任意规模的计算, 但它们只能通过主键之间的比较来得到关
于某个主键的信息.

没有任何基于比较的算法能够保证使用少于~NlgN(lgN!)次比较将长度为N的数组排序.
> 证明: 假设数组没有重复的主键, 用二叉树来表示所有可能的情况. 那么叶子数最少是N!, 因为对于N个不同的主键有N!种不同的排列, 于是任何基于比较的
> 排序算法都对应着一颗高为h的二叉树, 其中:
> $$ N! \le 叶子数 \le 2^h $$
> 而算法的复杂度对应与根节点到叶子的路径长, 即 $ T \ge lgN! $, 根据斯特灵公式有lgN! ~NlgN.

上述论证意味着, 没有任何排序算法能够用少于~NlgN次比较将数组排序, 归并排序是一种渐进最优的基于比较排序的算法.
但在实际情况中:
- 归并排序的空间复杂度不是最优的.
- 在实践中不一定是最坏的情况.
- 算法的其他操作也可能成为关键所在.
- 不进行比较也能够将某些数组排序.

## 快速排序 ##
快速排序的特点包括原地排序(只需要一个很小的辅助栈), 且将长度为N的数组排序的时间和NlgN成正比. 另外, 快速排序的内循环比大多数排序算法都小. 它的主要
缺点在于非常脆弱, 实现时要避免低劣的性能. 快速排序的效率依赖于切分数组的效果, 即依赖于切分数组的值.
```java
//简单的实现
public static void sort(Comparable[] a) {
    StdRandom.shuffle(a);
    sort(a, 0, a.length);
}

private static void sort(Comparable[] a, int lo, int hi) {
    if (hi < lo) return;
    int j = partition(a, lo, hi);
    sort(a, lo, j);
    sort(a, j + 1, hi);
}

private static int partition(Comparable[] a, int lo, int hi) {
    if (lo == hi) return lo;
    int i = lo, j = hi + 1;
    Comparable v = a[lo];
    while (true) {
        while (Sort.less(a[++i], v)) if (i == hi) break;
        while (Sort.less(v, a[--j])) if (j == lo) break;
        if (i > j) break;
        Sort.exch(a, i, j);
    }
    Sort.exch(a, lo, j);
    return j;

}
```

快速排序的最好情况是每次都正好能将数组对半分, 此时满足分治递归的$ C_N = 2C_{N/2} + N $公式, 这样得到的解是NlgN. 平均而言, 切分元素都能够落在
数组的中间.

将长度为N的无重复数组排序, 快速排序平均需要~2NlgN次比较.
> 证明: 令$ C_k $为将N个不同元素排序平均需要的比较次数, 显然有$ C_0=C_1=0 $, 对于N>1的情况, 可得到以下递归关系:
> $$ C_N = N + 1 + (C_0 + C_1 + \cdots + C_{N-2} + C_{N-1})/N + (C_{N-1} + C_{N-2} + \cdots + C_0)/N $$
> 其中第一项是切分成本, 总是N+1, 后面两项是左右子数组排序的平均成本, 等式两边同乘以N得到:
> $$ NC_N = N(N+1) + 2(C_0 + C_1 + \cdots + C_{N-1}) $$
> 即:
> $$ NC_N - (N-1)C_{N-1} = 2N + 2C_{N-1} $$
> 进而得到:
> $$ C_N/(N+1) = C_{N-1}/N + 2/(N+1) $$
> 于是有$ C_N/(N+1) - C_1/2 = 2(1/3 + 1/4 + \cdots + 1/(N+1)) $, 即$ C_N = 2(N+1)(1/3 + 1/4 + \cdots + 1/(N+1)) $
> 近似的得到$ C_N \approx 2NlnN \approx 1.39NlgN $, 即平均的比较次数比最好的情况多39%.

上述简单的快速排序有一个潜在的缺点, 即在切分不平衡时, 可能会极为低效, 在最坏的情况下快速排序需要$ N^2/2 $次比较, 我们在排序前shuffle数组就是为了
减少这种情况的发生. 

### 改进 ###
1. 适时切换到插入排序
    和大多数递归算法一样, 对于小数组快速排序比插入排序更慢, 在数组长度为5~15之间选择插入排序通常能取得较好的效果.
2. 三取样切分
    使用子数组的一小部分元素的中位数来切分数组, 这样能得到更好的切分, 但需要额外计算中位数的代价, 当取样大小为3时切分效果最好, 还可以放置哨兵来取消
    边界检测.
3. 熵最优排序
    当数组中含有大量重复元素的时候, 对于元素全部重复的元素可以不再排序. 一个简单的办法是将数组切分为三部分(荷兰国旗问题).
    
### 三向切分的快速排序 ###
E. W. Dijkstra提出的三向切分解法:
```java
public static void sort(Comparable[] a) {
    StdRandom.shuffle(a);
    sort(a, 0, a.length - 1);
}

@SuppressWarnings("unchecked")
private static void sort(Comparable[] a, int lo, int hi) {
    if (hi <= lo) return;
    int lt = lo, i = lo + 1, gt = hi;
    Comparable v = a[lo];
    while (i <= gt) {
        int cmp = a[i].compareTo(v);
        if (cmp < 0) Sort.exch(a, lt++, i++);
        else if (cmp > 0) Sort.exch(a, gt--, i);
        else i++;
    }
    sort(a, lo, lt - 1);
    sort(a, gt + 1, hi);
}
```
对于只有若干不同主键的随机数组, 三向切分快速排序能够达到线性级别. 给定包含k个不同值的N个主键, 定义$ f_i $为第i个主键出现的次数, $ p_i = f_i/N $
即第i个主键出现的概率, 那么香农熵定义为:
$$ H = -(p_1lgp_1 + p_2lgp_2 + \cdots + p_klgp_k) $$
可以得到:
- 不存在任何基于比较的算法能够保证在NH-N次比较内将N个元素排序.
- 对于大小为N的数组, 三向切分的快速排序需要~(2ln2)NH次比较.

当不存在重复主键时, $ H=lgN $, 而有重复主键时, 其具有更好的性能. 这两个性质说明了三向切分的快速排序是信息最优的.

## 堆排序 ##
堆排序首先构造一个堆, 然后使用下沉操作将数组按升序排序.
```java
public static void sort(Comparable[] a) {
    int N = a.length;
    for (int k = N / 2; k >= 1; k--) sink(a, k, N);
    while (N > 1) {
        exch(a, 1, N--);
        sink(a, 1, N);
    }
}

private static boolean less(Comparable[] a, int i, int j) {
    return a[i - 1].compareTo(a[j - 1]) < 0;
}

private static void exch(Comparable[] a, int i, int j) {
    Comparable t = a[i - 1];
    a[i - 1] = a[j - 1];
    a[j - 1] = t;
}

private static void sink(Comparable[] a, int k, int N) {
    while (2 * k <= N) {
        int j = 2 * k;
        if (j < N && less(a, j, j + 1)) j++;
        if (!less(a, k, j)) break;
        exch(a, k, j);
        k = j;
    }
}
```
将N个元素排序, 堆排序需要(2NlgN+2N)次比较(2N来源于堆的构造). 堆排序在排序复杂性的研究中具有重要地位, 因为它是唯一一个能够同时最优地利用空间和时间
的方法, 在最坏的情况下它也能保证~2NlgN次比较和恒定的空间. 在空间紧张的场景下具有很好的效果, 但却很少被现代操作系统的其他应用采用, 因为其无法利用缓
存.

## 常用排序算法的比较 ##
| 算法 | 稳定性 | 原地 | 时间复杂度 | 空间复杂度 |
| :-: | :-:   | :-: | :------: | :------: |
| 选择排序 | 否 | 是 | $ N^2 $ | 1 |
| 插入排序 | 是 | 是 | N到$ N^2 $ | 1 |
| 希尔排序 | 否 | 是 | $ N^{3/2} $ | 1 |
| 快速排序 | 否 | 是 | NlgN | lgN |
| 三向快速排序 | 否 | 是 | N到lgN | lgN |
| 归并排序 | 是 | 否 | NlgN | N |
| 堆排序 | 否 | 是 | NlgN | 1|

大多数情况下快速排序是最佳选择; 如果需要稳定性且空间充足的情况下, 归并排序是很好的选择; 对于原始的数据, 直接操作数据排序更合适.
