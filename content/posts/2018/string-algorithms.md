---
title: 字符串处理算法
date: 2018-05-07 21:18:59
categories:
- Things I Learned
tags:
- Java
- Algorithm
---

字符串处理算法具有很高的重要性以及应用领域的多样性. 以下讨论默认的是扩展的ASCII字符集(R=256).

## 字符串排序算法 ##
对于许多排序应用而言, 决定顺序的键都是字符串. 利用字符串的特殊性质来将其排序通常能够获得更高的效率.
<!-- more -->

### 键索引计数法 ###
我们首先介绍一种适用于小整数键的简单排序算法.
```java
public class KeyIndexedCount {

    public static void sort(char[] chr, int R) {
        int[] count = new int[R + 1];
        char[] temp = new char[chr.length];
        for (char aChr : chr) { count[aChr + 1]++; }
        for (int i = 0; i < R; i++) { count[i + 1] += count[i]; }
        for (char aChr : chr) { temp[count[aChr]] = aChr; count[aChr]++; }
        System.arraycopy(temp, 0, chr, 0, chr.length);
    }

    public static void sort(char chr[]) { sort(chr, 256); }
    
}
```
1. 进行频率统计,
2. 将频率转换为索引, 任意一个键的起始位置总是在比它小的键的频率之和的后面,
3. 用一个辅助数组进行排序,
4. 将排序好的内容回写到原来的数组.

很容易发现, 算法仅仅使用了四个循环, 所以其时间复杂度是线性的(四次循环共访问数组10N+3R次). 上述算法的另一个显而易见的特性是排序的稳定性(因为该算法
按顺序遍历数组来排序).

### 低位优先排序算法 ###
低位优先排序算法很容易处理那些长度相等的键的排序.
```java
public class LSD {

    public static void sort(String[] a, int W) {
        int N = a.length;
        final int R = 256;
        int[] count = new int[R + 1];
        String[] aux = new String[N];
        for (int i = W - 1; i >= 0; i--) {
            for (String str : a) { count[str.charAt(i) + 1]++; }
            for (int k = 0; k < R; k++) { count[k + 1] += count[k]; }
            for (int k = 0; k < N; k++) {
                aux[count[a[i].charAt(i)]] = a[k];
                count[a[i].charAt(i)]++;
            }
            System.arraycopy(aux, 0, a, 0, a.length);
        }
    }
}
```
利用键索引计数法就能够得到线性时间复杂度的该算法. 低位优先排序算法依赖于键索引计数法排序的稳定性, 如果没有该特性, 那么该算法在某一轮中打破了稳定性可
能打破上一轮的排序. 对于基于R个字符的字母表的N个以长为W的字符串为键的元素, 该算法需要访问数组~7WN+3WR次, 使用的空间与N+R成正比. 因此在R远小于N
的情形下, 算法总是与WN成正比.

### 高位优先排序算法 ###
虽然我们可以修正上述低位优先的排序算法以适应字符串长度不等的情况, 但通常从左向右遍历字符更符合这种情况. 对于某些语言来说(例如C/C++), 已经约定了字符
串末尾的表示方法(\0). 而对于没有约定结尾的可以实现一个专门的内置方法, 对于指定位置超过字符串长度时取-1, 这样对于频率统计数组count而言, 则需要R+2
的位置来存储.
```java
public class MSD {
    private static final int R = 256;
    private static final int M = 15;
    private static String[] aux;

    private static int charAt(String s, int d) {
        if (d >= s.length()) { return -1; }
        else { return s.charAt(d); }
    }

    public static void sort(String[] a) {
        aux = new String[a.length];
        sort(a, 0, a.length - 1, 0);
    }

    private static void sort(String[] a, int lo, int hi, int d) {
        if (hi <= lo + M) {
            InsertionX.sort(a, lo, hi, d); // 将lo到hi的部分从按d位开始的字符串进行排序
            return;
        }
        int[] count = new int[R + 2];
        for (int i = lo; i <= hi; i++) { count[charAt(a[i], d) + 2]++; }
        for (int i = 0; i < R + 1; i++) { count[i + 1] += count[i]; }
        for (int i = lo; i <= hi; i++) {
            aux[count[charAt(a[i], d) + 1]] = a[i];
            count[charAt(a[i], d) + 1]++;
        }
        System.arraycopy(aux, lo, a, lo, hi - lo + 1);
        for (int i = 0; i < R; i++) { sort(a, lo + count[i], lo + count[i + 1] - 1, d + 1); }
    }
}
```

小数组处理对于高位优先的字符串排序算法非常重要, 否则使用Unicode(65536)字符集时, 排序速度会下降非常明显. 因为如果不做任何处理, 最后算法会递归到
对长度为1的数组进行排序. 因此在子数组长度较小时, 切换到插入排序是非常有必要的(例如在长度小于或等于10时使用插入排序可以将运行时间降低为原来的1/10).
高位优先排序对于含有大量的等值键或者有很多前缀相同的键排序会较慢, 因为算法无法跳过等值键的排序(无法对等值键进行分组). 当递归深度较深的时候, 由于在
每一层递归不得不创建一个和Alphabet大小相当的count数组, 算法会造成非常大的空间开销.对于随机输入, 算法的运行时间是亚线性的; 而在最坏的情况下, 即所
有键都相等的情况下, 算法的时间复杂度是线性的.

将基于大小为R的Alphabet和N个字符串排序, 该算法平均需要检查$N\log_RN$, 访问数组在8N+3R到~7wN+3wR之间, 其中w是字符串平均长度, 所需空间在最坏
情况下与R乘以最长的字符串的长度之积成正比. 可见该算法的主要限制在于有较长公共部分的数据.

### 三向字符串快速排序 ###
该算法只是对原来的三向快速排序改成了字符串的版本. 相较与高位优先算法可能会创建大量的子数组, 三向快速排序总是创建3个, 因此该算法能够很好的处理等值键,
有较长公共前缀的键, 取值范围较小的键和小数组. 而且该算法不需要额外的空间.
```java
public class Quick3string {
    
    public static void sort(String[] a) {
        sort(a, 0, a.length - 1, 0);
    }

    private static int charAt(String s, int d) {
        if (d >= s.length()) { return -1; }
        else { return s.charAt(d); }
    }
    
    private static void exch(String[] a, int i, int j) {
        String temp = a[i];
        a[i] = a[j];
        a[j] = temp;
    }
    
    public static void sort(String[] a, int lo, int hi, int d) {
        if (hi <= lo) { return; }
        int lt = lo, gt = hi;
        int i = lo + 1;
        int v = charAt(a[lo], d);
        while (i < gt) {
            int t = charAt(a[i], d);
            if (t > v) { exch(a, i, gt); gt--; }
            else if (t < v) { exch(a, i, lt); lt++; i++; }
            else { i++; }
        }
        sort(a, lo, lt - 1, d);
        if (v > 0) { sort(a, lt, gt, d + 1); }
        sort(a, gt + 1, hi, d);
    }
}
```
当然, 也可以在数组长度较小时切换到插入排序以提高效率, 以及可以在排序前将数组随机打乱(比较简单的做法是将第一个元素与一个随机位置的元素交换)以防止出现
最坏的情况.


## Trie(单词查找树) ##
在一般情况下, Trie可以到达以下性能:
- 查找命中所需要的时间与被查找的键的长度成正比.
- 查找未命中的键只需要检查若干字符.

### R向Trie ###

```java
// 一种简单实现
public class TrieST<Value> {

    private static final int R = 256;
    private Node root;

    private static class Node {
        private int size;
        private Object val;
        private Node[] next = new Node[R];
    }

    @SuppressWarnings("unchecked")
    public Value get(String key) {
        Node x = get(root, key, 0);
        if (x == null) { return null; }
        else { return (Value) x.val; }
    }

    private Node get(Node x, String key, int d) {
        if (x == null) { return null; }
        if (d == key.length()) { return x; }
        char c = key.charAt(d);
        return get(x.next[c], key, d + 1);
    }

    public void put(String key, Value val) { root = put(root, key, val, 0); }

    private Node put(Node x, String key, Value val, int d) {
        if (x == null) { x = new Node(); }
        if (d == key.length()) {
            x.val = val;
            return x;
        }
        char c = key.charAt(d);
        x.size += 1;
        x.next[c] = put(x.next[c], key, val, d + 1);
        return x;
    }

    public int size() { return root.size; }

    private int size(Node x) {
        if (x == null) { return 0; }
        int sum = 0;
        if (x.val != null) { sum += 1; }
        for (char c = 0; c < R; c++) { sum += size(x.next[c]); }
        return sum;
    }

    public Iterable<String> keys() { return keysWithPrefix(""); }

    public Iterable<String> keysWithPrefix(String pre) {
        Queue<String> queue = new Queue<>();
        collect(get(root, pre, 0), pre, queue);
        return queue;
    }

    private void collect(Node x, String pre, Queue<String> q) {
        if (x == null) { return; }
        if (x.val != null) { q.enqueue(pre); }
        for (char c = 0; c < R; c++) {
            collect(x.next[c], pre + c, q);
        }
    }

    public String longestPrefixOf(String s) {
        int length = search(root, s, 0, 0);
        return s.substring(0, length);
    }

    private int search(Node x, String s, int d, int length) {
        if (x == null) { return length; }
        if (x.val != null) { length = d; }
        if (d == s.length()) { return length; }
        char c = s.charAt(d);
        return search(x.next[c], s, d + 1, length);
    }

    public void delete(String key) { root = delete(root, key, 0); }

    private Node delete(Node x, String key, int d) {
        if (x == null) { return null; }
        if (d == key.length()) {
            x.val = null;
        } else {
            char c = key.charAt(d);
            x.next[c] = delete(x.next[c], key, d + 1);
            x.size = size(x);
        }
        if (x.val != null) {
            return x;
        }
        for (char c = 0; c < R; c++) {
            if (x.next[c] != null) { return x; }
        }
        return null;
    }
}
```

R向Trie具有以下性质:
- 链表结构与键的插入顺序无关
- 查找或删除操作访问数组的次数最多为键的长度加1

字母表的大小为R, 在一颗N个随构造的R向Trie中, 未命中的查找平均所需检查的结点为$ \sim \log_RN $, 空间占用在RN到RNw之间. 因此上述算法在处理大型
字母表(如Unicide)的大量长键时, 空间消耗通常是不能接受的.

### 三向Trie ###
三向单词查找树通过显示保存字符来避免R向Trie的过度空间消耗. 可以看成是R向Trie的紧凑版, 但关键的差别在于三向Trie的结构依赖与键的插入顺序.
```java
// delete方法类似与二叉树的删除结点操作
public class TST<Value> {
    private Node root;

    private class Node {
        private char c;
        private Node left, mid, right;
        private Value val;
    }

    public void put(String key, Value val) { root = put(root, key, val, 0); }

    public Value get(String key) {
        Node x = get(root, key, 0);
        if (x == null) { return null; }
        else { return x.val; }
    }

    private Node get(Node cur, String key, int d) {
        if (cur == null) { return null; }
        char c = key.charAt(d);
        if (c < cur.c) { return get(cur.left, key, d); }
        else if (c > cur.c) { return get(cur.right, key, d); }
        else if (d < key.length() - 1) { return get(cur.mid, key, d + 1); }
        else { return cur; }
    }

    private Node put(Node cur, String key, Value val, int d) {
        char c = key.charAt(d);
        if (cur == null) {
            cur = new Node();
            return cur;
        }
        if (c < cur.c) { cur.left = put(cur.left, key, val, d); }
        else if (c > cur.c) { cur.right = put(cur.right, key, val, d); }
        else if (d < key.length() - 1) { cur.mid = put(cur.mid, key, val, d + 1); }
        else { cur.val = val; }
        return cur;
    }
}
```

在一颗N个随机字符串构成的三向Trie中, 查找为命中平均需要比较字符~lnN, 空间消耗在3N到3Nw之间(w为字符串平均长度).

## 子字符串查找 ##
字符串的一个基本操作就是子字符串查找: 给定一段长度为N的文本和长度为M的模式字符串(通常N >> M), 在文本中找到和该模式相符的子字符串.

### 暴力查找算法 ###
查找字符串的一个最直接的算法就是遍历
```java
public class BruteForceSearch {

    public static int search(String pat, String txt) {
        int M = pat.length();
        int N = txt.length();
        for (int i = 0; i < N; i++) {
            int j = 0;
            while (txt.charAt(i + j) == pat.charAt(j)) {
                j++;
                if (j == M) { return i; }
            }
        }
        return N;
    }
}
```
上述算法的效率取决于文本的内容, 在最坏的情况下时间复杂度为~NM, 而在最好的情况下为~N.

### KMP(_Knuth-Morris-Pratt_)算法 ###
KMP算法的主要思想是判断如何回退指针, 以避免不必要的回退, 而这只取决于模式本身. 事实上KMP算法是对确定有限状态自动机(_Deterministic finitestate 
automaton, DFA_)的模拟. DFA从0状态开始, 每次匹配成功都会将DFA带入下一个状态, 每次匹配失败则会使DFA回退到较早的一个状态. 
```java
public class KMP {
    private String pat;
    private int M;
    private int[][] dfa;
    public KMP(String pat) {
        this.pat = pat;
        M = pat.length();
        int R = 256;
        dfa = new int[R][M];
        //构造DFA
        dfa[pat.charAt(0)][0] = 1;
        for (int X = 0, j = 1; j < M; j++) {
            for (int c = 0; c < R; c++) { dfa[c][j] = dfa[c][X]; }
            dfa[pat.charAt(j)][j] = j + 1;
            X = dfa[pat.charAt(j)][X];
        }
    }
    
    public int search(String txt) {
        int i, j;
        int N = txt.length();
        for (i = 0, j = 0; i < N && j < M; i++) { j = dfa[txt.charAt(i)][j]; }
        if (j == M) { return i - M; }
        else { return N; }
    }
}
```
KMP算法时间复杂度为~M+N. KMP算法的一个优点是不需要在输入中回退, 因此KMP算法更适合在长度不确定的输入流中进行查找(如标准输入). 当回退的成本较低时,
Boyer-Moore算法具有更好的性能.

### Boyer-Moore算法 ###
Boyer-Moore算法从右向左扫描模式字符串, 当不匹配发生时, 可能有以下两种情况:
1. 不匹配的字符串在模式串中, 此时需要将模式字符串向右移动x位直到模式串中与之相等的字符与文本串对齐(对于模式串中有重复的字符而言可以简单的移动一位).
2. 不匹配的字符串不在模式中, 此时可以将模式串向有移动M位.
```java
public class BoyerMoore {
    private int[] right;
    private int M;
    private String pat;

    public BoyerMoore(String pat) {
        this.pat = pat;
        M = pat.length();
        int R = 256;
        right = new int[R];
        for (int c = 0; c < R; c++) { right[c] = -1; }
        for (int j = 0; j < M; j++) { right[pat.charAt(j)] = j; }
    }

    public int search(String txt) {
        int N = txt.length();
        int skip;
        for (int i = 0; i <= N - M; i += skip) {
            skip = 0;
            int j = M - 1;
            while (j >= 0 && pat.charAt(j) == txt.charAt(i + j)) { j--; }
            if (j >= 0) {
                skip = j - right[txt.charAt(i + j)];
                if (skip < 1) {
                    skip = 1;
                }
            } else { return i; }
        }
        return N;
    }
}
```
在一般情况下, Boyer-Moore算法时间复杂度为~N/M.

## 正则表达式 ##
正则表达式是对非确定有限状态自动机(_Nondeterministic finite-state automata, NFA_)的模拟, Kleene定理表明对于任意正则表达式都存在一个与之
对应的NFA, 并且具有以下特定:
1. 长度为M的正则表达式中的每个字符在所对应的NFA中都有且只有一个对应的状态, NFA的起始状态为0并且含有一个(虚拟的)接受状态.
2. 字母表中的字符所对应的状态都有一条从它指出的边, 这条边指向模式中的下一个字符所对应的状态.
3. 元字符"(", ")", "|", "*"所对应的状态至少含有一条指出的边, 这些边可以指向其他的任意状态.
4. 存在匹配转换和$ \epsilon $转换, 后者相当于匹配$ \epsilon $

而NFA与DFA的最大区别在于状态的不确定性, 即NFA状态转换有多种可能.
```java
// 简单正则表达式和NFA的转换
public class NFA {
    
    private char[] re;  // 匹配转换
    private Digraph G;  // epsilon转换
    private int M;

    /**
     * NFA构造函数
     * @param regexp: 用括号包裹的正则表达式, 只识别了"|"和"*"
     */
    public NFA(String regexp) {
        Stack<Integer> ops = new Stack<>();
        re = regexp.toCharArray();
        M = re.length;
        G = new Digraph(M + 1);
        for (int i = 0; i < M; i++) {
            int lp = i;
            if (re[i] == '(' || re[i] == '|') { ops.push(i); }
            else if (re[i] == ')') {
                int or = ops.pop();
                if (re[or] == '|') {
                    lp = ops.pop();
                    G.addEdge(lp, or + 1);
                    G.addEdge(or, i);
                } else { lp = or; }
            }
            if (i < M - 1 && re[i + 1] == '*') {
                G.addEdge(lp, i + 1);
                G.addEdge(i + 1, lp);
            }
            if (re[i] == '(' || re[i] == '*' || re[i] == ')') { G.addEdge(i, i + 1); }
        }
    }
    
    public boolean recogizes(String txt) {
        Bag<Integer> pc = new Bag<>();
        DirectedDFS dfs = new DirectedDFS(G, 0);
        for (int v = 0; v < G.V(); v ++) {
            if (dfs.marked(v)) { pc.add(v); }
        }
        
        for (int i = 0; i < txt.length(); i++) {
            Bag<Integer> match = new Bag<>();
            for (int v : pc) {
                if (v < M) {
                    if (re[v] == txt.charAt(i) || re[v] == '.') { match.add(v + 1); }
                }
            }
            pc = new Bag<>();
            dfs = new DirectedDFS(G, match);
            for (int v = 0; v < G.V(); v++) {
                if (dfs.marked(v)) { pc.add(v); }
            }
        }
        for (int v : pc) {
            if (v == M) { return true; }
        }
        return false;
    }
}
```
上述算法将一个正则表达式转换为NFA, 并能够识别文本串中是否包含模式串.
