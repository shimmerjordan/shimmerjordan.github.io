---
title: Five shortest-circuit algorithms & two ways to store maps
date: 2021-08-02 21:15:22
tags: 
  - graph
  - shortest path
  - SPFA
  - Dijkstra
  - Floyd
  - Bellman-Ford
id: LC743-shortestCircuit_mapsStorage
---

This article is a record of the answer to Leetcode 743, learning about the five shortest path solutions as follows: Floyd, Plain Dijkstra, Heap Optimization Dijkstra, Bellman Ford, SPFA. and three ways of storing graphs, including adjacency tables, adjacency matrices, and class & adjacency tables.

<!--more-->

# Original question

> You are given a network of `n` nodes, labeled from `1` to `n`. You are also given times, a list of travel times as directed edges `times[i] = (ui, vi, wi)`, where `ui` is the source node, `vi` is the target node, and `wi` is the time it takes for a signal to travel from source to target.
>
> We will send a signal from a given node k. Return the time it takes for all the n nodes to receive the signal. If it is impossible for all the n nodes to receive the signal, return `-1`.
>
> <center>
> <img style="border-radius: 0.3125em;
> box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
> src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed/blog/LC743-shortestCircuit_mapsStorage/pic1.png" width='25%'>
> <br>
> <div style="color:orange; border-bottom: 1px solid #d9d9d9;
> display: inline-block;
> color: #999;
> padding: 2px;">Example 1</div>
> </center>
>
> **Example 1:**
>
> > **Input:** times = [[2,1,1],[2,3,1],[3,4,1]], n = 4, k = 2
> > **Output:** 2
>
> **Example 2:**
>
> >**Input:** times = [[1,2,1]], n = 2, k = 1
> **Output:** 1
>
> **Example 3:**
>
> > **Input:** times = [[1,2,1]], n = 2, k = 2
> > **Output:** -1
>
> **Constraints:**
>
> - `1 <= k <= n <= 100`
> - `1 <= times.length <= 6000`
> - `times[i].length == 3`
> - `1 <= ui, vi <= n`
> - `ui != vi`
> - `0 <= wi <= 100`
> - All the pairs `(ui, vi)` are **unique**. (i.e., no multiple edges.)

# Basic analysis
For convenience, we have agreed that `n` is the number of points and `m` is the number of edges.

According to the question, the data range of `n` is `100` and the data range of `m` is `6000`, so the graph can be stored using either the 'adjacency table' or the 'adjacency matrix'.

Also, the problem is **'the shortest time for all points to be visited from point `k`'**, which translates to **'the shortest distance from point `k` to other points `x`'**.

# Floyd (Adjacency matrix)

Based on the **Basic Analysis**, we can use the Floyd algorithm, a multi-source sink shortest-circuit algorithm with complexity $O(n^3)$, to solve the problem, and the adjacency matrix to store the graph.

The computational effort at this point is about $10^6$ which can be passed.

Running through Floyd, we can obtain the 'shortest distance from any starting point to any starting point'. Then take `max` from all `w[k][x]` which is 'the maximum of the shortest distance from point `k` to other points `x`'.

## code

```java
//java
class Solution {
    int N = 110, M = 6010;
    int[][] w = new int[N][N];
    int INF = 0x3f3f3f3f;
    int n, k;
    public int networkDelayTime(int[][] ts, int _n, int _k) {
        n = _n; k = _k;
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= n; j++) {
                w[i][j] = w[j][i] = i == j ? 0 : INF;
            }
        }
        for (int[] t : ts) {
            int u = t[0], v = t[1], c = t[2];
            w[u][v] = c;
        }
        floyd();
        int ans = 0;
        for (int i = 1; i <= n; i++) {
            ans = Math.max(ans, w[k][i]);
        }
        return ans > INF / 2 ? -1 : ans;
    }
    void floyd() {
        for (int p = 1; p <= n; p++) {
            for (int i = 1; i <= n; i++) {
                for (int j = 1; j <= n; j++) {
                    w[i][j] = Math.min(w[i][j], w[i][p] + w[p][j]);
                }
            }
        }
    }
}
```

- Time complexity: stored graph plus shortest-circuit algorithm with complexity $O(m+n^3)$
- Space complexity: $O(n^2)$

# Plain Dijkstra (Adjacency matrix)

Let us now review the algorithm, the main idea of which is greed.

All nodes are divided into two categories: those for which the shortest path from the starting point to the current point has been determined, and those for which the shortest path from the starting point to the current point has not been determined (hereinafter referred to as 'undetermined nodes' and 'determined nodes').

Each time a point with the shortest distance from the starting point is taken from the Undetermined Node, it is classified as a Determined Node and used to 'update' the distance from the starting point to all other Undetermined Nodes. This is done until all points are classified as 'identified nodes'.

To 'update' node `B` with node `A` means to compare the length of the shortest path from the starting point to node `A` with the length of the edge from node `A` to node `B`, and if the former is less than the latter, update the latter with the former. This operation is also called **'slackening'**.

The implicit message here is that each time an 'undetermined node' is selected, the length of the shortest path from the starting point to it can be determined.

This can be understood as we have already updated the current node with every 'determined node' and do not need to update it again (as a point cannot be reached more than once). The current node is already the shortest distance to the starting point of all the 'undetermined nodes' and cannot be updated by any other 'undetermined node'. Therefore the current node can be classified as a 'determined node'.

## code

According to the question, the time it takes for the signal from node `k` to reach node `x` is the length of the shortest path from node `k` to node `x`. Therefore we need to find the shortest path from node `k` to all other points, the maximum of which is the answer. If there is a point that cannot be reached from `k`, then `-1` is returned.

The following code decreases the node number by `1` so that the node number lies in the range `[0,n-1]`

```python
# python
class Solution:
    def networkDelayTime(self, times: List[List[int]], n: int, k: int) -> int:
        g = [[float('inf')] * n for _ in range(n)]
        for x, y, time in times:
            g[x - 1][y - 1] = time

        dist = [float('inf')] * n
        dist[k - 1] = 0
        used = [False] * n
        for _ in range(n):
            x = -1
            for y, u in enumerate(used):
                if not u and (x == -1 or dist[y] < dist[x]):
                    x = y
            used[x] = True
            for y, time in enumerate(g[x]):
                dist[y] = min(dist[y], dist[x] + time)

        ans = max(dist)
        return ans if ans < float('inf') else -1
```

- Time complexity: stored graph plus shortest-circuit algorithm with complexity $O(m+n^2)$
- Space complexity: $O(n^2)$

# Heap-optimised Dijkstra (Adjacency tables)
Since the range of edge data is not too large, we can also use the **heap-optimised** Dijkstra algorithm with a complexity of $O(m\log{n})$.

Both heap-optimised Dijkstra and plain Dijkstra are 'single-source shortest-circuit' algorithms.

After running through the heap-optimised Dijkstra algorithm to find the shortest circuit, max is taken from all the shortest circuits to be the **'maximum of the shortest distance from point k to other points x'**.

At this point, the complexity of the algorithm is $O(m\log{n})$, which can be passed.

## code

```java
//java
class Solution {
    int N = 110, M = 6010;
    int[] he = new int[N], e = new int[M], ne = new int[M], w = new int[M];
    int[] dist = new int[N];
    boolean[] vis = new boolean[N];
    int n, k, idx;
    int INF = 0x3f3f3f3f;
    void add(int a, int b, int c) {
        e[idx] = b;
        ne[idx] = he[a];
        he[a] = idx;
        w[idx] = c;
        idx++;
    }
    public int networkDelayTime(int[][] ts, int _n, int _k) {
        n = _n; k = _k;
        Arrays.fill(he, -1);
        for (int[] t : ts) {
            int u = t[0], v = t[1], c = t[2];
            add(u, v, c);
        }
        dijkstra();
        int ans = 0;
        for (int i = 1; i <= n; i++) {
            ans = Math.max(ans, dist[i]);
        }
        return ans > INF / 2 ? -1 : ans;
    }
    void dijkstra() {
        Arrays.fill(vis, false);
        Arrays.fill(dist, INF);
        dist[k] = 0;
        PriorityQueue<int[]> q = new PriorityQueue<>((a,b)->a[1]-b[1]);
        q.add(new int[]{k, 0});
        while (!q.isEmpty()) {
            int[] poll = q.poll();
            int id = poll[0], step = poll[1];
            if (vis[id]) continue;
            vis[id] = true;
            for (int i = he[id]; i != -1; i = ne[i]) {
                int j = e[i];
                if (dist[j] > dist[id] + w[i]) {
                    dist[j] = dist[id] + w[i];
                    q.add(new int[]{j, dist[j]});
                }
            }
        }
    }
}
```

- Time complexity: $O(m\log{n}+n)$
- Space complexity: $O(m)$

# Bellman Ford (Class & Adjacency table)
Although the title specifies that there are no 'negatively weighted edges', we can still use Bellman Ford, which can be solved by 'shortest-circuiting in a negatively weighted graph', which is also a **'single-source shortest-circuiting'** algorithm with $O(n \cdot m)$ complexity.
Usually, to ensure $O(n \cdot m)$, a separate class can be constructed to represent the edges, and all edges can be stored in the set, and the set of edges can be traversed directly in the `n` times relaxation operations (see P1 for code).
Since the order of magnitude of edges in this problem is greater than the order of magnitude of points, it is also possible to continue to use the 'adjacency table' approach to edge traversal, with a lower bound of $O(n)$ on the complexity of traversing all edges and an upper bound that ensures that it does not exceed $O(m)$ (see P2 for code).

## code

### P1

```java
//java
class Solution {
    class Edge {
        int a, b, c;
        Edge(int _a, int _b, int _c) {
            a = _a; b = _b; c = _c;
        }
    }
    int N = 110, M = 6010;
    int[] dist = new int[N];
    boolean[] vis = new boolean[N];
    int INF = 0x3f3f3f3f;
    int n, m, k;
    List<Edge> es = new ArrayList<>();
    public int networkDelayTime(int[][] ts, int _n, int _k) {
        n = _n; k = _k;
        m = ts.length;
        for (int[] t : ts) {
            int u = t[0], v = t[1], c = t[2];
            es.add(new Edge(u, v, c));
        }
        bf();
        int ans = 0;
        for (int i = 1; i <= n; i++) {
            ans = Math.max(ans, dist[i]);
        }
        return ans > INF / 2 ? -1 : ans;
    }
    void bf() {
        Arrays.fill(dist, INF);
        dist[k] = 0;
        for (int p = 1; p <= n; p++) {
            int[] prev = dist.clone();
            for (Edge e : es) {
                int a = e.a, b = e.b, c = e.c;
                dist[b] = Math.min(dist[b], prev[a] + c);
            }
        }
    }
}
```

### P2

```java
//java
class Solution {
    int N = 110, M = 6010;
    int[] he = new int[N], e = new int[M], ne = new int[M], w = new int[M];
    int[] dist = new int[N];
    boolean[] vis = new boolean[N];
    int INF = 0x3f3f3f3f;
    int n, m, k, idx;
    void add(int a, int b, int c) {
        e[idx] = b;
        ne[idx] = he[a];
        he[a] = idx;
        w[idx] = c;
        idx++;
    }
    public int networkDelayTime(int[][] ts, int _n, int _k) {
        n = _n; k = _k;
        m = ts.length;
        Arrays.fill(he, -1);
        for (int[] t : ts) {
            int u = t[0], v = t[1], c = t[2];
            add(u, v, c);
        }
        bf();
        int ans = 0;
        for (int i = 1; i <= n; i++) {
            ans = Math.max(ans, dist[i]);
        }
        return ans > INF / 2 ? -1 : ans;
    }
    void bf() {
        Arrays.fill(dist, INF);
        dist[k] = 0;
        for (int p = 1; p <= n; p++) {
            int[] prev = dist.clone();
            for (int a = 1; a <= n; a++) {
                for (int i = he[a]; i != -1; i = ne[i]) {
                    int b = e[i];
                    dist[b] = Math.min(dist[b], prev[a] + w[i]);
                }
            }
        }
    }
}
```

- Time complexity: $O(m \cdot n)$
- Space complexity: $O(m)$

# SPFA (Adjacent Table)
SPFA is an optimised implementation of Bellman Ford, **either using a queue for optimisation or a stack for optimisation.**
The complexity is usually $O(k \cdot m)$, with `k` typically being `4` to `5`, and still $O(n \cdot m)$ in the worst case, degrading from $O(k \cdot m)$ to $O(n \cdot m)$ when the data is a grid graph.

## code

```java
//java
class Solution {
    int N = 110, M = 6010;
    int[] he = new int[N], e = new int[M], ne = new int[M], w = new int[M];
    int[] dist = new int[N];
    boolean[] vis = new boolean[N];
    int INF = 0x3f3f3f3f;
    int n, k, idx;
    void add(int a, int b, int c) {
        e[idx] = b;
        ne[idx] = he[a];
        he[a] = idx;
        w[idx] = c;
        idx++;
    }
    public int networkDelayTime(int[][] ts, int _n, int _k) {
        n = _n; k = _k;
        Arrays.fill(he, -1);
        for (int[] t : ts) {
            int u = t[0], v = t[1], c = t[2];
            add(u, v, c);
        }
        spfa();
        int ans = 0;
        for (int i = 1; i <= n; i++) {
            ans = Math.max(ans, dist[i]);
        }
        return ans > INF / 2 ? -1 : ans;
    }
    void spfa() {
        Arrays.fill(vis, false);
        Arrays.fill(dist, INF);
        dist[k] = 0;
        Deque<Integer> d = new ArrayDeque<>();
        d.addLast(k);
        vis[k] = true;
        while (!d.isEmpty()) {
            int poll = d.pollFirst();
            vis[poll] = false;
            for (int i = he[poll]; i != -1; i = ne[i]) {
                int j = e[i];
                if (dist[j] > dist[poll] + w[i]) {
                    dist[j] = dist[poll] + w[i];
                    if (vis[j]) continue;
                    d.addLast(j);
                    vis[j] = true;
                }
            }
        }
    }
}
```

- Time complexity: $O(m \cdot n)$
- Space complexity: $O(m)$
