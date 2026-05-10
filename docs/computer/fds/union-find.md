# 并查集

这一页整理 Ch08 The Disjoint Set ADT。重点是等价关系、动态等价问题、Union/Find 的森林表示、按规模合并、按高度合并、路径压缩，以及最终的均摊复杂度。

## 1. 等价关系

一个关系 `R` 定义在集合 `S` 上，如果对任意 `a, b ∈ S`，命题 `a R b` 要么为真，要么为假。

如果 `a R b` 为真，就说：

```text
a is related to b
```

等价关系需要满足三条性质：

| 性质 | 含义 |
| --- | --- |
| reflexive 自反性 | `a ~ a` |
| symmetric 对称性 | 如果 `a ~ b`，则 `b ~ a` |
| transitive 传递性 | 如果 `a ~ b` 且 `b ~ c`，则 `a ~ c` |

如果 `x ~ y`，则 `x` 和 `y` 属于同一个等价类（equivalence class）。

## 2. 动态等价问题

动态等价问题是：

> 给定一个等价关系 `~`，不断读入关系 `a ~ b`，并能随时判断两个元素是否属于同一个等价类。

例子：

```text
S = {1, 2, 3, ..., 12}

关系:
12≡4, 3≡1, 6≡10, 8≡9, 7≡4,
6≡8, 3≡5, 2≡11, 11≡12
```

最后得到的等价类：

```text
{2, 4, 7, 11, 12}
{1, 3, 5}
{6, 8, 9, 10}
```

这个问题适合用并查集，因为关系是动态读入的，可以边读边合并。

## 3. Union / Find 思路

算法流程：

```text
Initialize N disjoint sets

while read a ~ b:
    if Find(a) != Find(b):
        Union(Find(a), Find(b))

while read query a, b:
    if Find(a) == Find(b):
        output true
    else:
        output false
```

两个核心操作：

- `Find(x)`：找到元素 `x` 所在集合的代表元。
- `Union(i, j)`：把两个集合合并成一个集合。

在实现中，代表元通常就是树的根。

## 4. 森林表示

并查集把每个集合表示成一棵树，多个集合组成森林。

指针方向是：

```text
child -> parent
```

根结点代表整个集合。

例如：

```text
S1 = {6, 7, 8, 10}
S2 = {1, 4, 9}
S3 = {2, 3, 5}
```

可以表示为：

```text
      10             4             2
    / | \          /   \         /   \
   6  7  8        1     9       3     5
```

## 5. 双亲数组

如果元素编号是 `1..N`，就可以用数组表示父指针。

一种基础约定：

```text
S[x] = x 的父结点
S[root] = 0
```

例如：

```text
      10             4             2
    / | \          /   \         /   \
   6  7  8        1     9       3     5
```

可以写成：

```text
下标:  1  2  3  4  5  6  7  8  9  10
S[i]:  4  0  2  0  2 10 10 10  4   0
```

其中：

- `S[10] = 0`，说明 `10` 是根。
- `S[6] = 10`，说明 `6` 的父结点是 `10`。

优化版常用另一种约定：

```text
S[root] = -size 或 -height
S[x] >= 0 表示父结点
```

这样根结点还能顺便保存集合规模或高度信息。

## 6. 基础 Find

`Find(x)` 沿着父指针一路向上，直到根。

```c
SetType Find(ElementType X, DisjSet S) {
    for (; S[X] > 0; X = S[X]) {
        ;
    }
    return X;
}
```

如果使用 `S[root] <= 0` 表示根，则可以写成：

```c
int Find(int S[], int x) {
    while (S[x] >= 0) {
        x = S[x];
    }
    return x;
}
```

复杂度取决于树高：

```text
O(depth of x)
```

## 7. 基础 Union

如果已经知道两个根 `Root1` 和 `Root2`，最简单的合并就是让其中一个根指向另一个根。

```c
void SetUnion(DisjSet S, SetType Root1, SetType Root2) {
    S[Root2] = Root1;
}
```

这一步本身是 `O(1)`。  
但如果总是随便挂，树可能变得很高。

最坏例子：

```text
union(2, 1), find(1)
union(3, 2), find(1)
union(4, 3), find(1)
...
union(N, N-1), find(1)
```

会形成一条长链：

```text
1 -> 2 -> 3 -> 4 -> ... -> N
```

反复 `find(1)` 的总代价可以达到：

```text
Θ(N^2)
```

所以并查集的关键不只是“能合并”，而是要避免树长成链。

## 8. Union-by-Size

Union-by-size 的策略：

```text
总是把小树挂到大树下面。
```

根结点保存负的集合大小：

```text
S[root] = -size
```

初始化：

```text
S[i] = -1
```

表示每个集合一开始只有一个元素。

代码：

```c
void UnionBySize(int S[], int Root1, int Root2) {
    if (Root1 == Root2) {
        return;
    }

    if (S[Root1] < S[Root2]) {
        S[Root1] += S[Root2];
        S[Root2] = Root1;
    } else {
        S[Root2] += S[Root1];
        S[Root1] = Root2;
    }
}
```

注意根保存的是负数：

```text
-5 < -3
```

所以 `S[Root1] < S[Root2]` 表示 `Root1` 的集合更大。

## 9. Union-by-Size 的高度界

课件给出结论：

```text
height(T) <= floor(log2 N) + 1
```

直觉证明：

如果一个结点的深度增加，说明它所在的树被挂到了另一棵至少同样大的树下面。  
因此，每当深度增加 `1`，它所在集合的大小至少翻倍。

所以一个结点深度增加 `k` 次后，集合大小至少是：

```text
2^k
```

而总元素数最多是 `N`：

```text
2^k <= N
k <= log2 N
```

所以树高是 `O(log N)`。

如果有 `N` 次 Union 和 `M` 次 Find，使用 union-by-size 后：

```text
O(N + M log N)
```

## 10. Union-by-Height

Union-by-height 的策略：

```text
总是把矮树挂到高树下面。
```

如果两棵树高度相同，随便选一个作为根，并让新根高度加 `1`。

它和 union-by-size 很像，目的都是控制树高。  
实际实现中常称为 union-by-rank，因为后面路径压缩会改变真实高度，rank 只是一个估计值。

## 11. 路径压缩

路径压缩在 `Find` 时顺手优化树结构。

普通 Find 只是找到根。  
路径压缩会把查找路径上的所有结点直接挂到根下面。

递归写法：

```c
SetType Find(ElementType X, DisjSet S) {
    if (S[X] <= 0) {
        return X;
    } else {
        return S[X] = Find(S[X], S);
    }
}
```

迭代写法：

```c
SetType Find(ElementType X, DisjSet S) {
    ElementType root, trail, lead;

    for (root = X; S[root] > 0; root = S[root]) {
        ;
    }

    for (trail = X; trail != root; trail = lead) {
        lead = S[trail];
        S[trail] = root;
    }

    return root;
}
```

可视化：

```text
压缩前:
1 -> 2 -> 3 -> 4 -> 5

Find(1) 后:
1 -> 5
2 -> 5
3 -> 5
4 -> 5
```

单次 Find 可能比普通 Find 多做一点修改，但对一连串操作会更快。

## 12. 路径压缩和按高度合并

课件提醒：

```text
路径压缩不完全兼容真实 height
```

原因是路径压缩会改变树的实际高度。  
如果根里保存的是“真实高度”，压缩后高度信息可能不准。

解决方法：

```text
把 height 当作 rank，也就是估计高度。
```

这就是常说的 union-by-rank + path compression。

## 13. 复杂度：反阿克曼函数

如果同时使用 union-by-rank 和 path compression，Tarjan 结论是：

```text
T(M, N) = Θ(M α(M, N))
```

其中：

- `N` 是元素个数。
- `M` 是操作次数，通常 `M >= N`。
- `α(M, N)` 是反阿克曼函数。

反阿克曼函数增长极慢。  
实际规模下几乎可以看成常数，课件中写到：

```text
α(M, N) <= O(log* N) <= 4
```

所以竞赛或工程里常说：

```text
并查集单次操作均摊近似 O(1)
```

更严谨地说：

```text
amortized O(α(N))
```

## 14. 常见使用模板

```c
void Init(int S[], int n) {
    for (int i = 1; i <= n; i++) {
        S[i] = -1;
    }
}

int Find(int S[], int x) {
    if (S[x] < 0) {
        return x;
    }
    return S[x] = Find(S, S[x]);
}

void Union(int S[], int a, int b) {
    int ra = Find(S, a);
    int rb = Find(S, b);

    if (ra == rb) {
        return;
    }

    if (S[ra] < S[rb]) {
        S[ra] += S[rb];
        S[rb] = ra;
    } else {
        S[rb] += S[ra];
        S[ra] = rb;
    }
}

int IsSameSet(int S[], int a, int b) {
    return Find(S, a) == Find(S, b);
}
```

注意上面 `Find` 的调用写法要统一。  
如果函数定义是：

```c
int Find(int S[], int x)
```

递归时应写：

```c
return S[x] = Find(S, S[x]);
```

## 15. 适用场景

并查集适合处理“合并集合 + 查询是否同集合”的问题。

常见应用：

- 无向图连通块统计。
- 判断两个点是否连通。
- 岛屿数量。
- 冗余连接。
- Kruskal 最小生成树。
- 社交圈、朋友圈、账户合并。

## 16. 考点速记

| 考点 | 必会内容 |
| --- | --- |
| 等价关系 | 自反、对称、传递 |
| 等价类 | 互相等价的元素组成一类 |
| 动态等价问题 | 边读关系边合并，在线查询 |
| Find | 找根，即集合代表元 |
| Union | 合并两个根代表的集合 |
| 森林表示 | child 指向 parent |
| 根结点 | 代表整个集合 |
| 双亲数组 | 非根存父结点，根存负规模或负高度 |
| 朴素 Union | 可能形成链，最坏很慢 |
| Union-by-size | 小树挂大树 |
| Union-by-height/rank | 矮树挂高树 |
| 路径压缩 | Find 后把路径结点直接挂到根 |
| rank | 路径压缩后高度只作估计 |
| 最优复杂度 | 均摊 `O(α(N))`，近似常数 |

## 17. 易错点

- `Union` 通常应该传入两个根，而不是随便两个元素。
- 如果传入普通元素，要先 `Find` 再合并。
- 根结点保存负数时，绝对值越大集合越大。
- `S[root] < 0` 和 `S[x] >= 0` 的约定要始终一致。
- 路径压缩会改变树形，不要再把 height 当成真实高度。
- 并查集只能高效回答“是否同集合”，不能直接列出集合全部元素，除非额外维护。
