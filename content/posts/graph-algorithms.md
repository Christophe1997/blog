---
title: graph algorithms
date: 2018-04-20 19:21:30
categories:
- algorithm
tags:
- Java
---

采用哪种数据结构来表示图, 主要考虑以下两个方面:
- 必须为可能在应用中碰到的各种类型的图预留出足够的空间, 以及
- Graph的实例方法一定要高效.

<!-- more -->

常见的表示方法有以下几种:
- 邻接矩阵, 对于有N个顶点的图而言, 邻接矩阵需要$ N^2 $的空间.
- 边的数组, 通过定义边来定义图, 这种方法在寻找相邻的点时需要遍历整个数组.
- 邻接表数组, 使用一个顶点为索引的列表数组, 其中每个元素都是和该顶点相邻的顶点列表. 这种结构能够满足上述的两个条件.

常见实现的性能比较(V表示结点数, E表示边数):

| 数据结构 | 所需空间 | 添加边 | 检查顶点是否相邻 | 遍历所有相邻顶点 |
| :----: | :-----: | :---: | :-----------: | :-----------: |
| 边的列表 | E | 1 | E | E |
| 邻接矩阵 | $ V^2 $ | 1 | 1 | V |
| 邻接表 | E+V | 1 | degree(V) | degree(V) |

非稠密图的标准表示是邻接表, 这种表示具有以下特性:
- 使用的空间和V+E成正比,
- 添加一条边所需要的时间为常数, 以及
- 遍历顶点v的所有相邻顶点所需要的时间和v的度数成正比.

## 无向图 ##
无向图只定义了顶点以及顶点之间的关系.

### 深度优先搜索 ###
```java
public class DepthFirstSearch {
    private boolean[] marked;
    private int count;

    public DepthFirstSearch(Graph G, int s) {
        marked = new boolean[G.V()];
        dfs(G, s);
    }

    private void dfs(Graph G, int v) {
        marked[v] = true;
        count++;
        for (int w : G.adj(v)) {
            if (!marked[w]) { dfs(G, w); }
        }
    }

    public boolean marked(int w) { return marked[w]; }

    public int count() { return count; }
}
```
深度优先搜索能够所有与起点相连的顶点, 且所需时间和所有连通顶点的度数之和成正比.

### 广度优先搜索 ###
深度优先搜索得到的路径不仅取决于图的结构, 还取决于图的表示和递归调用的性质. 
```java
public class BreadthFirthSearch {
    private boolean[] marked;
    private int count;

    public BreadthFirthSearch(Graph G, int s) {
        marked = new boolean[G.V()];
        bfs(G, s);
    }

    public void bfs(Graph G, int v) {
        Queue<Integer> queue = new Queue<>();
        marked[v] = true;
        queue.enqueue(v);
        while (!queue.isEmpty()) {
            int x = queue.dequeue();
            for (int w : G.adj(x)) {
                if (!marked[w]) {
                    queue.enqueue(w);
                    marked[w] = true;
                    count++;
                }
            }
        }
    }

    public boolean marked(int v) { return marked[v]; }

    public int count() { return count; }
}
```


深度优先搜索与广度优先搜索的差别在于数据结构的不同, 前者使用栈而后者使用队列, 我们在搜索的时候都会将起点加入数据结构, 然后重复以下步骤直到数据结构被
清空:
- 取其中的下一个顶点并标记它
- 将v的所有相邻而又为被标记的顶点加入数据结构
两者的差别只在于数据结构如何获取下一个顶点, 而这种差异导致了图的两者不同视图, 但最终所有与顶点相连的点都会被检查到

## 有向图 ##

### 环的检测 ###
有向图常常需要检测是否存在环.
```java
/**
 * only for one cycle
 */
public class DirectedCycle {

    private boolean[] marked;
    private int[] edgeTo;
    private Stack<Integer> cycle;
    private boolean[] onStack;

    public DirectedCycle(Digraph G) {
        onStack = new boolean[G.V()];
        edgeTo = new int[G.V()];
        marked = new boolean[G.V()];
        for (int v = 0; v < G.V(); v++) { if (!marked[v]) { dfs(G, v); } }
    }

    private void dfs(Digraph G, int v) {
        onStack[v] = true;
        marked[v] = true;
        for (int w : G.adj(v)) {
            if (this.hasCycle()) {
                return;
            } else if (!marked[w]) {
                edgeTo[w] = v;
                dfs(G, w);
            } else if (onStack[w]) {
                cycle = new Stack<>();
                for (int x = v; x != w; x = edgeTo[x]) { cycle.push(x); }
                cycle.push(w);
                cycle.push(v);
            }
        }
        onStack[v] = false;
    }

    public boolean hasCycle() { return cycle != null; }

    public Iterable<Integer> cycle() { return cycle; }
}
```

### 拓扑排序 ###
拓扑排序是解决优先级限制下任务调度的常见算法, 能否进行拓扑排序的先决条件是是否是有向无环图. 进而问题变为有向无环图中基于深度优先遍历的顶点排序问题,
常见的排序方式有以下三种:
- 前序, 在递归调用前将顶点加入队列
- 后序, 在递归调用之后将顶点加入队列
- 逆后序, 在递归调用之后将顶点加入栈
```java
public class DepthFirthOrder {
    private boolean[] marked;
    private Queue<Integer> pre;
    private Queue<Integer> post;
    private Stack<Integer> reverstPost;

    public DepthFirthOrder(Digraph G) {
        marked = new boolean[G.V()];
        pre = new Queue<>();
        post = new Queue<>();
        reverstPost = new Stack<>();
        for (int v = 0; v < G.V(); v++) { if (!marked[v]) { dfs(G, v); } }
    }

    private void dfs(Digraph G, int v) {
        marked[v] = true;
        pre.enqueue(v);
        for (int w : G.adj(v)) { if (!marked[w]) { dfs(G, w); } }
        post.enqueue(v);
        reverstPost.push(v);
    }

    public Iterable<Integer> pre() { return pre; }
    
    public Iterable<Integer> post() { return post; }
    
    public Iterable<Integer> reverstPost() { return reverstPost; }
}
```
进而拓扑排序可以有如下实现:
```java
public class Topological {
    private Iterable<Integer> order;

    public Topological(Digraph G) {
        DirectedCycle cycle = new DirectedCycle(G);
        if (!cycle.hasCycle()) {
            DepthFirthOrder dfs = new DepthFirthOrder(G);
            order = dfs.reverstPost();
        }
    }

    public Iterable<Integer> order() { return order; }
    
    public boolean isDAG() { return order != null; }

    public static void main(String[] args) throws FileNotFoundException {
        String filename = args[0];
        String sp = args[1];
        SymbolDigraph sg = new SymbolDigraph(new File(filename), sp);
        Topological top = new Topological(sg.G());
        top.order.forEach(v -> System.out.println(sg.name(v)));
    }
}
```
这种实现非常简单, 但它被忽略了很多年, 比它更流行的是一种使用队列存储顶点的直观算法:
1. 初始化一条含有所有起点的队列Q
2. 如果Q为空, 停止, 否则从队列Q中删除一个起点并将其标记;
3. 遍历由被删除顶点指出的所有边, 将被指向的顶点的入度减1;
4. 如果顶点的入度变为0, 将其加入Q;
5. 跳转至2.

## 加权无向图 ##
加权无向图的一个典型问题是如何寻找最小生成树.

### prim算法 ###
1. 设N<V, {E}>是连通网, 记U={ $u_0$ }, TE={}
2. 在所有$ u \in U, v \in V-U $的边$ (u, v) \in E $中找到一条代价最小的边$ (u_0, v_0)并加入TE $
3. 将$ v_0 $加入U
4. 若U=V, 停止否则继续2
我们将使用优先队列来进行添加最小边的操作. 并且我们需要在添加新顶点之后检查队列中边的有效性. Prim的一种延时实现:
```
public class LazyPrimMST {
    private boolean[] marked;
    private Queue<Edge> mst;
    private MinPQ<Edge> pq;

    public LazyPrimMST(EdgeWeightedGraph G) {
        pq = new MinPQ<>();
        marked = new boolean[G.V()];
        mst = new Queue<>();
        visit(G, 0);
        while (!pq.isEmpty()) {
            Edge e = pq.delMin();
            int v = e.either();
            int w = e.other(v);
            if (!marked[v] || !marked[w]) {
                mst.enqueue(e);
                if (!marked[v]) { visit(G, v); }
                if (!marked[w]) { visit(G, w); }
            }
        }
    }

    private void visit(EdgeWeightedGraph G, int v) {
        marked[v] = true;
        for (Edge e : G.adj(v)) {
            if (!marked[e.other(v)]) { pq.insert(e); }
        }
    }

    public Iterable<Edge> edges() { return mst; }

    public double weight() {
        double weight = 0;
        for (Edge e : edges()) {
            weight += e.weight();
        }
        return weight;
    }
}
```
延时实现会将所有边都将入优先队列中, 并且不会删除失效的边, 而是在删除的时候检查边的有效性. 所需时间与ElgE成正比, 所需空间与E成正比.

一种即时实现是总是删除优先队列中失效的边. 我们感兴趣的其实只是连接树顶点和非树顶点中权重最小的边. 当我们将一个顶点v将入到树中时, 对与一个非树中的
顶点w, 只可能使得w到最小生成树的距离更小. 换言之, 我们只会在优先队列中保存每个非树顶点的一条边: 使它与树中顶点连接起来权重最小的一条边.
```java
public class PrimMST {
    private Edge[] edgeTo;
    private double[] distTo;
    private boolean[] marked;
    private IndexMinPQ<Double> pq;

    public PrimMST(EdgeWeightedGraph G) {
        edgeTo = new Edge[G.V()];
        distTo = new double[G.V()];
        marked = new boolean[G.V()];
        for (int v = 0; v < G.V(); v++) { distTo[v] = Double.POSITIVE_INFINITY; }
        pq = new IndexMinPQ<>();

        distTo[0] = 0.0;
        pq.insert(0, 0.0);
        while (!pq.isEmpty()) { visit(G, pq.delMin()); }
    }

    private void visit(EdgeWeightedGraph G, int v) {
        marked[v] = true;
        for (Edge e : G.adj(v)) {
            int w = e.other(v);
            if (!marked[w] && e.weight() < distTo[w]) {
                edgeTo[w] = e;
                distTo[w] = e.weight();
                if (pq.contains(w)) { pq.change(w, distTo[w]); }
                else { pq.insert(w, distTo[w]); }
            }
        }
    }

    public Iterable<Edge> edges() {
        Bag<Edge> edges = new Bag<>();
        for (int v = 1; v < edgeTo.length; v++) { edges.add(edgeTo[v]); }
        return edges;
    }

    public double weight() {
        double weight = 0;
        for (Edge e : edges()) { weight += e.weight(); }
        return weight;
    }
}
```
即时的版本所需时间与ElgV成正比, 空间和V成正比.

对于常见的稀疏图而言, 两者在时间上限上没有明显的区别(对于稀疏图而言, lgE~lgV), 但空间占用上显然即时的版本更优秀.

### Kruskal 算法 ###
相较于Prim算法根据顶点构成树, Kruskal则算法根据边来构成树. 该算法不断地将最小边加入到树中并确保不会构成环. 可以使用最小队列来保存边, 并用
union-find来判断是否有环:
```java
public class KruskalMST {
    private Queue<Edge> mst;

    public KruskalMST(EdgeWeightedGraph G) {
        mst = new Queue<>();
        MinPQ<Edge> pq = new MinPQ<>();
        for (Edge e : G.edges()) { pq.insert(e); }
        WeightedQuickUnionUF uf = new WeightedQuickUnionUF(G.V());

        while (!pq.isEmpty() && mst.size() < G.V() - 1) {
            Edge e = pq.delMin();
            int v = e.either();
            int w = e.other(v);
            if (!uf.connected(v, w)) {
                uf.union(w, v);
                mst.enqueue(e);
            }
        }
    }

    public Iterable<Edge> edges() { return mst; }

    public double weight() {
        double weight = 0;
        for (Edge e : edges()) { weight += e.weight(); }
        return weight;
    }
}
```
Kruskal算法一般情况下会比Prim算法慢, 因为union-find还需要进行一次connect操作.

最小生成树各算法比较(设V个顶点, E条边):

| 算法 | 空间 | 时间 |
| :----------: | :-: | :-: |
| 延时的Prim算法 | E | ElgE |
| 即时的Prim算法 | V | ElgV |
| Kruskal | E | ElgE |
| Fredman-Tarjan | V | E+VlgV |
| Chazelle | V | 非常接近E |

## 加权有向图 ##
加权有向图的一个典型的问题是如何寻找最短路径. 我们的重点是单点最短路径问题, 对于给定的起点s得到一颗包含s到所有可达顶点的最短路径树(SPT).

### Dijkstra算法 ###
Dijkstra算法按广度优先来寻找最短路径
```java
public class DijkstraSP {
    private DirectedEdge[] edgeTo;
    private double[] distTo;
    private IndexMinPQ<Double> pq;

    public DijkstraSP(EdgeWeightedDigraph G, int s) {
        edgeTo = new DirectedEdge[G.V()];
        distTo = new double[G.V()];
        pq = new IndexMinPQ<>();
        for (int v = 0; v < G.V(); v ++) {
            distTo[v] = Double.POSITIVE_INFINITY;
        }
        distTo[0] = 0.0;
        pq.insert(s, 0.0);
        while (!pq.isEmpty()) {
            relax(G, pq.delMin());
        }
    }

    private void relax(EdgeWeightedDigraph G, int v) {
        for (DirectedEdge e : G.adj(v)) {
            int w = e.to();
            double newWeight = distTo[v] + e.weight();
            if (distTo[w] > newWeight) {
                distTo[w] = newWeight;
                edgeTo[w] = e;
                if (pq.contains(w)) { pq.change(w, distTo[w]); }
                else { pq.insert(w, distTo[w]); }
            }
        }
    }

    public double distTo(int v) { return distTo[v]; }

    public boolean hasPathTo(int v) { return distTo[v] < Double.POSITIVE_INFINITY; }

    public Iterable<DirectedEdge> pathTo(int v) {
        if (!hasPathTo(v)) { return null; }
        else {
            Stack<DirectedEdge> path = new Stack<>();
            for (DirectedEdge e = edgeTo[v]; e != null; e = edgeTo[e.from()]) { path.push(e); }
            return path;
        }
    }
}
```
可以看到, Dijkstra算法和Prim算法两者之间非常类似. 某种程度上可以把Dijkstra看成是Prim算法的有向图版本. 类似的, Dijkstra算法所需的空间与V成正比
时间与ElgV成正比. 但是Dijkstra算法不适用于存在权重为负的有向图(因为在允许负权重的有向图中, 绕路经过更多的负权重边可能会使得权重之和更小, 而这在
Dijkstra算法中是不被允许的).

### 无环加权有向图的最短路径算法 ###
无环加权有向图可以利用拓扑排序来获得一个线性时间的最短路径算法, 并且能够处理负权重的边.
```
public class AcyclicSP {
    private DirectedEdge[] edgeTo;
    private double[] distTo;

    public AcyclicSP(EdgeWeightedDigraph G, int s) {
        edgeTo = new DirectedEdge[G.V()];
        distTo = new double[G.V()];
        for (int v = 0; v < G.V(); v++) { distTo[v] = Double.POSITIVE_INFINITY; }
        distTo[s] = 0;
        EdgeweightedTopological top = new EdgeweightedTopological(G);
        for (int v: top.order()) {
            relax(G, v);
        }
    }

    private void relax(EdgeWeightedDigraph G, int v) {
        for (DirectedEdge e : G.adj(v)) {
            int w = e.to();
            double newWeight = distTo[v] + e.weight();
            if (distTo[w] > newWeight) {
                distTo[w] = newWeight;
                edgeTo[w] = e;
            }
        }
    }

    public double distTo(int v) { return distTo[v]; }

    public boolean hasPathTo(int v) { return distTo[v] < Double.POSITIVE_INFINITY; }
    public Iterable<DirectedEdge> pathTo(int v) {
        Stack<DirectedEdge> path = new Stack<>();
        for (DirectedEdge e = edgeTo[v]; e != null; e = edgeTo[e.from()]) { path.push(e); }
        return path;
    }

    public static void main(String[] args) throws FileNotFoundException {
        File file = new File(args[0]);
        EdgeWeightedDigraph G = new EdgeWeightedDigraph(file);
        int s = Integer.parseInt(args[1]);
        AcyclicSP sp = new AcyclicSP(G, s);

        for (int t = 0; t < G.V(); t++) {
            System.out.print(s + " to " + t);
            System.out.printf(" (%4.2f): ", sp.distTo(t));
            if (sp.hasPathTo(t)) {
                for (DirectedEdge e : sp.pathTo(t)) { System.out.print(e + "   "); }
            }
            System.out.println();
        }
    }
}
```

> 按照拓扑顺序来遍历并`relax`顶点, 就能够在和E+V成正比的时间内解决单点最短路径问题
> 按照拓扑顺序可以保证每条边都被`relax`一次, 任意一个顶点在`relax`后, 算法便不在处理任何指向该顶点的边. 而每次`relax`操作只会减少
> 路径长, 因此在所有从s可达的顶点都被加入到树中后, 最短路径的最优条件成立. 而拓扑排序的时间复杂度是线性的, 因此总时间复杂度也是线性的.

因为该算法依赖于拓扑排序, 所以任何有环的图都将无法使用该算法. 另外一点是, 该算法可以解决负权重的边. 值得一提的是, 简单的修改上述算法可以在
线性时间复杂度内解决无环加权有向图的单点最长路径, 将`Double.POSITIVE_INFINITY`改为`Double.NEGATIVE_INFINITY`, 并修改`relax`的
不等号方向. 但是在一般的加权有向图中寻找最长简单路径的已知最好算法是指数级别的.

### 关键路径 ###
一种和无环加权有向图的最长路径等价的问题是优先级限制下的并行任务调度问题, 即关键路径问题. 将并行调度任务转换为无环有向图的步骤如下:
1. 创建一个包含起点s和终点t且每个任务都对应两个顶点(一个起始顶点和结束顶点);
2. 对于每个任务, 都添加一条从起始顶点指向结束顶点且权重为任务所需时间的边. 对于每个优先级限制v -> w, 添加一条从v结束顶点指向w起始顶点且
   权重为0的边.
3. 为每个任务添加一条从起点指向该任务起始顶点且权重为0的边, 以及一条从该任务结束顶点指向终点且权重为0的边.
例如, 有如下测试数据
```
10
41.0  1 7 9 // 必须在1 7 9之前完成
51.0  2
50.0
36.0
38.0
45.0
21.0  3 8
32.0  3 8
32.0  2
29.0  4 6
```
```java
public class CMP {
    public static void main(String[] args) {
        Scanner stdIn = new Scanner(System.in);
        int N = stdIn.nextInt();
        stdIn.nextLine();
        EdgeWeightedDigraph G = new EdgeWeightedDigraph(2 * N + 2);
        int s = 2 * N;
        int t = 2 * N + 1;
        for (int i = 0; i < N; i++) {
            String[] line = stdIn.nextLine().split("\\s+");
            double duration = Double.parseDouble(line[0]);
            G.addEdge(new DirectedEdge(i, i + N, duration));
            G.addEdge(new DirectedEdge(s, i, 0.0));
            G.addEdge(new DirectedEdge(i + N, t, 0.0));
            for (int j = 1; j < line.length; j++) {
                int successor = Integer.parseInt(line[j]);
                G.addEdge(new DirectedEdge(i + N, successor, 0.0));
            }
        }
        AcyclicLP lp = new AcyclicLP(G, s);
        System.out.println("Start times:");
        for (int i = 0; i < N; i++) { System.out.printf("%4d: %5.1f\n", i, lp.distTo(i)); }
        System.out.printf("Finish time: %5.1f\n", lp.distTo(t));
    }
}
```
可以得到如下结果:
```
Start times:
   0:   0.0
   1:  41.0
   2: 123.0
   3:  91.0
   4:  70.0
   5:   0.0
   6:  70.0
   7:  41.0
   8:  91.0
   9:  41.0
Finish time: 173.0
```
之所以最长路径路径能够解决并行任务调度问题在于s到t的最长路径是所有任务完成所需的最短时间.

### 一般加权有向图的最短路径算法 ###
考虑包含deadline限制的相对最后期限限制下的并行任务调度问题, 一般的deadline限制都是相对与第一个的任务的开始时间而言, 即在任务调度的问题中加入某个任
务必须在指定的时间点之前开始. 该问题等价于加权有向图中最短路径问题(可以有环和负权重).
构造一个加权有向图与关键路径类似, 但需要为每条deadline限制添加一条边: 如果任务v必须在任务w启动后的d个单位时间内开始, 则添加一条从v指向w且权重为负的
边. 将所有边的权重取反即可得到一个加权有向图的最短路径问题.

> 当且仅当加权有向图中至少存在一条从s到v的有向路径且其上的任意顶点都不存在与负权重环中时, 最短路径才存在.
> 对于任意一个负权重的环(权重之和为负数), 重复该环即可得到权重任意小的路径, 而这是无意义的.

因此该算法要求能够:
- 对于从起点不可达的顶点, 最短路径是$ +\infty $
- 对于从起点可达但存在与一个负权重环的顶点, 最短路径为$ -\infty $
- 对于其他顶点, 能够计算最短路径的权重

Bellman-Ford算法可以解决这样的问题, 基本思想是按节点依次构建最短路径, 于是该算法的时间与VE成正比, 空间和V成正比. 该方法非常通用, 因为其没有规定
边的放松顺序, 且总是放松VE条边. 我不会着力于这个版本, 而是介绍另外一个改进的版本.

__基于队列的Bellman-Ford算法__

我们很容易发现, 原始的版本在每一轮中有很多`relax`操作都不会成功, 只有在上一轮中起点到该点路径长度改变的点指出的边才能够改变起点到其他顶点的路径长.
为了记录这样的顶点, 可以使用一个FIFO的队列来记录这些点.
```java
public class BellmanFordSP {
    private double[] distTo;
    private DirectedEdge[] edgeTo;
    private boolean[] onQ;
    private Queue<Integer> queue;
    private int cost;  // relax调用的次数
    private Iterable<Integer> cycle;

    public BellmanFordSP(EdgeWeightedDigraph G, int s) {
        distTo = new double[G.V()];
        edgeTo = new DirectedEdge[G.V()];
        onQ = new boolean[G.V()];
        queue = new Queue<>();
        for (int v = 0; v < G.V(); v++) { distTo[v] = Double.POSITIVE_INFINITY; }
        distTo[s] = 0.0;
        queue.enqueue(s);
        onQ[s] = true;
        while (!queue.isEmpty() && !hasNegativeCycle()) {
            int v = queue.dequeue();
            onQ[v] = false;
            relax(G, v);
        }
    }

    private void relax(EdgeWeightedDigraph G, int v) {
        for (DirectedEdge e : G.adj(v)) {
            int w = e.to();
            if (distTo[w] > distTo[v] + e.weight()) {
                distTo[w] = distTo[v] + e.weight();
                edgeTo[w] = e;
                if (!onQ[w]) {
                    queue.enqueue(w);
                    onQ[w] = true;
                }
            }
            cost += 1;
            if (cost % G.V() == 0) { findNegativeCycle(); }
        }
    }

    private void findNegativeCycle() {
        int V = edgeTo.length;
        EdgeWeightedDigraph spt = new EdgeWeightedDigraph(V);
        for (int v = 0; v < V; v++) {
            if (edgeTo[v] != null) { spt.addEdge(edgeTo[v]); }
        }
        EdgeWeightedCycleFinder cf = new EdgeWeightedCycleFinder(spt);
        cycle = cf.cycle();
    }
    
    public boolean hasNegativeCycle() { return cycle != null; }

    public Iterable<Integer> negativeCycle() { return cycle; }

    public double distTO(int v) { return distTo[v]; }

    public boolean hasPathTo(int v) { return distTo[v] < Double.POSITIVE_INFINITY; }

    public Iterable<DirectedEdge> pathTo(int v) {
        Stack<DirectedEdge> path = new Stack<>();
        for (DirectedEdge e = edgeTo[v]; e != null; e = edgeTo[e.from()]) { path.push(e); }
        return path;
    }
}
```
为了避免负权重出现, 我们需要在每轮进行负权重环的检测.

### 加权有向图最短路径算法比较 ###

| 算法 | 限制 | 时间复杂度 | 空间复杂度 | 优势 |
| :-: | :-: | :------: | :------: | :--: |
| Dijkstra算法 | 权重需要为正 | ElgV | V | 最坏情况下仍有较好性能 |
| 利用拓扑排序的最短路径算法 | 只适用于无环加权有向图 | E + V | V | 是无环图中的最优算法 |
| Bellman-Ford算法 | 不能存在负权重环 | E + V(最坏EV) | V | 适用性高 |
