---
title: 图论知识梳理+模板
date: 2024-11-26 21:12:05
tags:
    - 图论
    - 模板
category: 知识梳理
mathjax: true
---

### 拓扑排序

操作步骤：先得到所有零度点，再依次让零度点出度为1的点入度减1，直到所有入度为0的点，这样就能得到拓扑排序序列。

拓扑排序还可以用来判环，同时若改为优先队列可以取字典序最小等。

```c++
bool toposort() {
    int cnt = 0;
    queue<int> q;
    for (int i = 1; i <= n; i++)
        if (!indeg[i]) q.push(i);
    while (!q.empty()) {
        int u = q.front(); q.pop();
        cnt++;
        for (auto v : g[u])
            if (--indeg[v] == 0) q.push(v);
    }
    return cnt == n;
}
```

### 最短路

#### Dijkstra算法

```c++
typedef pii = pair<int, int>;
void dijkstra (int s) {
    fill(d+1, d+n+1, inf);
    fill(vst+1, vst+n+1, false);
    d[s] = 0;
    priority_queue<pii, vector<pii>, greater<pii>> q;
    q.push({0, s});
    while (!q.empty()) {
        auto [dis, u] = q.top(); q.pop();
        if (vst[u]) continue;
        vst[u] = true;
        for (auto [v, w] : g[u])
            if (d[u] + w < d[v]) {
                d[v] = d[u] + w;
                q.push({d[v], v});
            }
    }
}
```

#### Floyd-Warshall算法

```c++
void floyd_warshall() {
    for (int k = 1; k <= n; k++)
        for (int i = 1; i <= n; i++)
            for (int j = 1; j <= n; j++)
                d[i][j] = min(d[i][j], d[i][k] + d[k][j]);
}
```

#### SPFA算法

SPFA可用于判断负环。

```c++
bool spfa(int s) {
    fill(d+1, d+n+1, inf);
    fill(vst+1, vst+n+1, false);
    fill(cnt+1, cnt+n+1, 0);
    d[s] = 0;
    queue<int> q;
    q.push(s);
    while (!q.empty()) {
        int u = q.front(); q.pop();
        vst[u] = false;
        for (auto [v, w] : g[u])
            if (d[v] > d[u] + w) {
                d[v] = d[u] + w;
                if (!vst[v]) {
                    vst[v] = true;
                    if (++cnt[v] > n) return false;
                    q.push(v);
                }
            }
    }
    return true;
}
```

#### 应用：差分约束系统

例如我们有 $x_a - x_b \geq c$，或 $x_a - x_b \leq c$，或 $x_a = x_b$ 这几种形式，则分别可以转化为 $x_b - x_a \leq -c$，$x_a - x_b \leq c$，及 $x_b - x_a \leq 0$ 且 $x_a - x_b \leq 0$。分别加边 $(a, b, -c)$，$(b, a, c)$，$(a, b, 0)$ 和 $(b, a, 0)$，可以用上文的 SPFA 算法解决，若无负环存在，则有解。

### 最小生成树

#### Kruskal算法

```c++
void kruskal() {
    sort(e+1, e+m+1, [](const edge &a, const edge &b) { return a.w < b.w; });
    for (int i = 1; i <= n; i++) fa[i] = i;
    for (int i = 1; i <= m; i++) {
        int u = e[i].u, v = e[i].v;
        int fu = find(u), fv = find(v);
        if (fu != fv) {
            ans += e[i].w;
            fa[fu] = fv;
        }
    }
}
```

#### Prim算法

每次要选择距离最小的一个结点，以及用新的边更新其他结点的距离，与Dijkstra算法相似。注意只会获得最小生成树权值和，并无法输出方案，需要记录边，此时Kruskal可能更好些。

```c++
void prim (int s) {
    fill(d+1, d+n+1, inf);
    fill(vst+1, vst+n+1, false);
    d[s] = 0;
    priority_queue<pii, vector<pii>, greater<pii>> q;
    q.emplace(0, s);
    int res = 0, cnt = 0;
    while (!q.empty()) {
        if (cnt >= n) break;
        auto [dis, u] = q.top(); q.pop();
        if (vst[u]) continue;
        vst[u] = true;
        res += dis;
        cnt++;
        for (auto [v, w] : g[u])
            if (!vst[v] && d[v] > w)
                d[v] = w, q.emplace(d[v], v);
    }
}
```

### 树相关问题

#### 树的直径

树上任意两节点之间最长的简单路径称为树的直径。

##### 两次DFS

注意有负权变时，无法使用两次DFS求解树的直径。

```c++
int n, c, d[N];
vector<int> g[N];
void dfs(int u, int fa) {
    for (auto v: g[u]) {
        if (v == fa) continue;
        d[v] = d[u] + 1;
        if (d[v] > d[c]) c = v;
        dfs(v, u);
    }
}
int main() {
    // ...
    dfs(1, 0);
    d[c] = 0; dfs(c, 0);
    printf("%d\n", d[c]);
    // ...
}
```

##### 树形DP

令 $dp_u$ 为 从 $u$ 开始的最长路径长度，那么 $dp_u = \max \{dp_u, \max_{v \in g_u} \{dp_v + w(u, v)\}\}$。则可以使用树形DP，时间复杂度为 $O(n)$。树的直径可以顺便枚举两最大值和得出。

```c++
int n, d, dp[N];
vector<int> g[N];
void dfs(int u, int fa) {
    for (auto v: g[u]) {
        if (v == fa) continue;
        dfs(v, u);
        d = max(d, dp[u]+dp[v]+1);
        dp[u] = max(dp[u], dp[v]+1);
    }
}
int main() {
    // ...
    dfs(1, 0);
    printf("%d\n", d);
    // ...
}
```