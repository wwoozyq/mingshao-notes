# 线段树

线段树（Segment Tree）是一种维护**区间信息**的数据结构，适合解决这类问题：

- 查询一个区间的和、最值、最大公约数等
- 修改某一个位置的值
- 修改一整个区间的值

它的核心优势是：在数组会动态变化时，仍然能高效地完成区间查询与更新。

## 1. 为什么需要线段树

假设有一个数组：

```text
[1, 3, 5, 7, 9, 11]
```

如果我们只做一次区间求和，比如求 `[2, 5]` 的和，直接遍历就行。  
但如果有很多次操作：

- 查询区间和
- 修改某个位置
- 再查询另一个区间

这时每次都重新遍历，复杂度会很高。

### 前缀和为什么不够

前缀和适合处理**静态数组的区间查询**：

- 查询快，`O(1)`
- 但一旦数组中某个元素被修改，后面的前缀和都要重新更新

所以：

- 静态区间和：前缀和很合适
- 动态区间查询：线段树更合适

## 2. 基本思想

线段树本质上是一棵二叉树。  
它把一个区间不断二分，每个结点维护一个子区间的信息。

例如数组下标范围是 `[1, 8]`，可以这样划分：

```text
                  [1,8]
                /       \
             [1,4]     [5,8]
            /   \      /   \
         [1,2] [3,4] [5,6] [7,8]
         / \    / \    / \    / \
       [1,1][2,2]...[7,7][8,8]
```

叶子结点对应单个元素，非叶子结点对应一个区间。  
如果我们维护的是区间和，那么父结点的值就是左右子结点之和。

## 3. 存储方式

线段树通常不用链式存储，而是直接用数组模拟：

```cpp
int tree[4 * MAXN];
```

常见约定：

- 根结点下标为 `1`
- 左孩子下标为 `p * 2`
- 右孩子下标为 `p * 2 + 1`

之所以常开 `4 * n`，是为了保证数组空间足够。

如果维护区间和，`tree[p]` 表示结点 `p` 对应区间的和。

## 4. 建树

设原数组为 `a[1..n]`，建树过程如下：

```cpp
void build(int p, int l, int r) {
    if (l == r) {
        tree[p] = a[l];
        return;
    }

    int mid = (l + r) >> 1;
    build(p * 2, l, mid);
    build(p * 2 + 1, mid + 1, r);
    tree[p] = tree[p * 2] + tree[p * 2 + 1];
}
```

这里：

- `p` 表示当前结点编号
- `[l, r]` 表示当前结点维护的区间

建树完成后，根结点 `tree[1]` 存的就是整个区间 `[1, n]` 的信息。

## 5. 区间查询

比如查询区间 `[ql, qr]` 的和：

```cpp
int query(int p, int l, int r, int ql, int qr) {
    if (ql <= l && r <= qr) {
        return tree[p];
    }

    int mid = (l + r) >> 1;
    int sum = 0;

    if (ql <= mid) {
        sum += query(p * 2, l, mid, ql, qr);
    }
    if (qr > mid) {
        sum += query(p * 2 + 1, mid + 1, r, ql, qr);
    }

    return sum;
}
```

查询时有三种典型情况：

- 当前区间被完整包含，直接返回
- 查询区间完全落在左半边，递归左子树
- 查询区间完全落在右半边，递归右子树
- 查询区间横跨中点，左右都查

## 6. 单点更新

如果把某个位置 `x` 修改成 `val`：

```cpp
void update(int p, int l, int r, int x, int val) {
    if (l == r) {
        tree[p] = val;
        return;
    }

    int mid = (l + r) >> 1;
    if (x <= mid) {
        update(p * 2, l, mid, x, val);
    } else {
        update(p * 2 + 1, mid + 1, r, x, val);
    }

    tree[p] = tree[p * 2] + tree[p * 2 + 1];
}
```

思路是：

1. 从根结点一路找到叶子结点
2. 修改这个叶子结点
3. 回溯时更新沿途所有祖先结点

## 7. 懒标记

如果题目要求对整个区间同时加上一个值，例如：

- 把 `[2, 6]` 中每个数都加 `3`

如果逐个修改，复杂度就会退化。  
这时需要用到**懒标记**（Lazy Propagation）。

懒标记的想法是：

- 当前结点代表一个大区间
- 如果整个区间都要加同一个值，就先记录下来
- 等以后真的访问到子区间时，再把这个修改下传

通常会再开一个数组：

```cpp
int lazy[4 * MAXN];
```

其中 `lazy[p]` 表示结点 `p` 还有一个尚未下传的修改。

### 下传标记

```cpp
void pushDown(int p, int l, int r) {
    if (lazy[p] == 0) {
        return;
    }

    int mid = (l + r) >> 1;
    int left = p * 2;
    int right = p * 2 + 1;

    tree[left] += lazy[p] * (mid - l + 1);
    tree[right] += lazy[p] * (r - mid);

    lazy[left] += lazy[p];
    lazy[right] += lazy[p];
    lazy[p] = 0;
}
```

### 区间修改

下面的模板表示“把 `[ql, qr]` 每个元素都加上 `val`”：

```cpp
void updateRange(int p, int l, int r, int ql, int qr, int val) {
    if (ql <= l && r <= qr) {
        tree[p] += val * (r - l + 1);
        lazy[p] += val;
        return;
    }

    pushDown(p, l, r);
    int mid = (l + r) >> 1;

    if (ql <= mid) {
        updateRange(p * 2, l, mid, ql, qr, val);
    }
    if (qr > mid) {
        updateRange(p * 2 + 1, mid + 1, r, ql, qr, val);
    }

    tree[p] = tree[p * 2] + tree[p * 2 + 1];
}
```

### 带懒标记的区间查询

```cpp
int queryRange(int p, int l, int r, int ql, int qr) {
    if (ql <= l && r <= qr) {
        return tree[p];
    }

    pushDown(p, l, r);
    int mid = (l + r) >> 1;
    int sum = 0;

    if (ql <= mid) {
        sum += queryRange(p * 2, l, mid, ql, qr);
    }
    if (qr > mid) {
        sum += queryRange(p * 2 + 1, mid + 1, r, ql, qr);
    }

    return sum;
}
```

## 8. 时间复杂度

以区间和线段树为例：

- 建树：`O(n)`
- 单次区间查询：`O(log n)`
- 单点修改：`O(log n)`
- 区间修改（带懒标记）：`O(log n)`

空间复杂度一般记为 `O(n)`，实现时通常开 `4n` 大小的数组。

## 9. 线段树适合哪些题

线段树尤其适合下面几类题目：

- 多次区间求和、区间最值
- 单点修改 + 区间查询
- 区间修改 + 区间查询
- 动态维护区间信息

常见关键词：

- 区间和
- 区间最大值 / 最小值
- 区间加法
- 区间赋值
- 动态查询

## 10. 和树状数组的区别

线段树和树状数组都能处理区间问题，但侧重点不完全一样。

### 树状数组

- 实现更短
- 常用于前缀和、单点修改
- 更适合模板简单的题

### 线段树

- 功能更强
- 更容易扩展到区间最值、区间赋值、懒标记
- 写起来更长，但适用面更广

## 11. 适合记住的要点

- 线段树维护的是“区间信息”。
- 每个结点对应一个区间。
- 父结点的信息由左右子结点合并得到。
- 查询和更新的时间复杂度通常是 `O(log n)`。
- 遇到“区间修改 + 区间查询”，通常要考虑懒标记。

## 12. 常见模板骨架

```cpp
const int MAXN = 100000 + 5;
int a[MAXN];
int tree[MAXN * 4];
int lazy[MAXN * 4];

void pushUp(int p) {
    tree[p] = tree[p * 2] + tree[p * 2 + 1];
}

void build(int p, int l, int r) {
    if (l == r) {
        tree[p] = a[l];
        return;
    }
    int mid = (l + r) >> 1;
    build(p * 2, l, mid);
    build(p * 2 + 1, mid + 1, r);
    pushUp(p);
}
```

如果以后你要维护的不是区间和，而是区间最值，只需要把“合并规则”从求和改成 `max` 或 `min` 即可。
