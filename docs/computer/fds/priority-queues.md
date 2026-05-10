# 优先队列

这一页整理 Ch05 Priority Queues (Heaps)。重点是优先队列 ADT、二叉堆的结构性质和堆序性质、插入、DeleteMin、BuildHeap、kth largest，以及 d-heap。

## 1. 优先队列 ADT

优先队列（Priority Queue）不是按照进入时间决定谁先出，而是按照优先级决定。

常见模型：

```text
DeleteMin: 删除并返回最小优先级元素
DeleteMax: 删除并返回最大优先级元素
```

课件主要讨论 **min priority queue**，也就是每次删除最小元素。

ADT 操作：

| 操作 | 含义 |
| --- | --- |
| `Initialize(MaxElements)` | 初始化优先队列 |
| `Insert(X, H)` | 插入元素 `X` |
| `DeleteMin(H)` | 删除并返回最小元素 |
| `FindMin(H)` | 返回最小元素但不删除 |

## 2. 简单实现方式对比

优先队列可以用数组、链表、有序数组、有序链表实现，但各有代价。

| 实现 | Insert | DeleteMin | 说明 |
| --- | --- | --- | --- |
| 无序数组 | `Θ(1)` | `Θ(N)` | 插入快，删除要扫描最小值 |
| 无序链表 | `Θ(1)` | `Θ(N)` | 删除要扫描，移除结点本身快 |
| 有序数组 | `O(N)` | `Θ(1)` | 插入要找位置并移动元素 |
| 有序链表 | `O(N)` | `Θ(1)` | 插入要找位置，删除首/尾很快 |
| 平衡搜索树 | `O(log N)` | `O(log N)` | 可行但实现复杂，指针开销大 |

为什么不用 BST/AVL 直接做优先队列？

- 优先队列通常只关心最小值或最大值。
- AVL 可以支持很多操作，但维护平衡成本更高。
- 对这个特定任务，堆更简单、更紧凑。

## 3. 二叉堆 Binary Heap

二叉堆同时满足两个性质：

1. **结构性质**：是一棵完全二叉树。
2. **堆序性质**：对 min heap，每个结点都不大于它的孩子。

这两个性质缺一不可：

- 只有完全二叉树，没有大小关系，不能快速找最小值。
- 只有大小关系，但形状混乱，不能用数组高效存储。

## 4. 结构性质：完全二叉树

完全二叉树的结点从上到下、从左到右连续填充。

```text
          1
       /     \
      2       3
    /  \     / \
   4    5   6   7
  / \
 8   9
```

它非常适合用数组表示。  
通常下标从 `1` 开始，`H->Elements[0]` 不放堆元素，可以当 sentinel。

数组表示：

```text
index:      1   2   3   4   5   6   7   8   9
element:    A   B   C   D   E   F   G   H   I
```

## 5. 堆的下标关系

对下标为 `i` 的结点：

```text
parent(i) = floor(i / 2)      i != 1
left(i)   = 2i
right(i)  = 2i + 1
```

前提是对应下标没有超过当前 `Size`。

| 位置 | 下标 |
| --- | --- |
| parent | `i / 2` |
| left child | `2 * i` |
| right child | `2 * i + 1` |

这就是二叉堆常用数组而不是链式结构的原因：不用真的存指针，父子关系可以直接算出来。

## 6. 堆序性质

min heap 的堆序性质：

```text
每个结点的 key <= 它的孩子的 key
```

因此最小值一定在根结点：

```text
H->Elements[1]
```

例子：

```text
          10
       /      \
      20      83
     /
    50
```

这是一个 min heap。

max heap 类似，只是方向反过来：

```text
每个结点的 key >= 它的孩子的 key
```

## 7. 堆的初始化

堆结构通常类似：

```c
struct HeapStruct {
    int Capacity;
    int Size;
    ElementType *Elements;
};
```

初始化时：

```c
PriorityQueue Initialize(int MaxElements) {
    PriorityQueue H;

    if (MaxElements < MinPQSize) {
        Error("Priority queue size is too small");
    }

    H = malloc(sizeof(struct HeapStruct));
    if (H == NULL) {
        FatalError("Out of space!!!");
    }

    H->Elements = malloc((MaxElements + 1) * sizeof(ElementType));
    if (H->Elements == NULL) {
        FatalError("Out of space!!!");
    }

    H->Capacity = MaxElements;
    H->Size = 0;
    H->Elements[0] = MinData;

    return H;
}
```

`Elements[0]` 是 sentinel，值应当不大于堆中任何可能元素。  
这样插入时向上比较可以自然停在下标 `0` 前。

## 8. Insert：上滤 Percolate Up

插入必须先保持结构性质。  
完全二叉树中新结点只能放在数组最后：

```text
i = ++H->Size
```

然后检查堆序性质。  
如果新元素比父结点小，就让父结点下移，新元素继续向上找位置。

这叫 **percolate up**。

代码：

```c
void Insert(ElementType X, PriorityQueue H) {
    int i;

    if (IsFull(H)) {
        Error("Priority queue is full");
        return;
    }

    for (i = ++H->Size; H->Elements[i / 2] > X; i /= 2) {
        H->Elements[i] = H->Elements[i / 2];
    }

    H->Elements[i] = X;
}
```

这里不是每次都 swap，而是一路把父结点往下挪，最后把 `X` 放进正确位置。  
这样比反复交换更快。

复杂度：

```text
O(log N)
```

因为堆高是 `O(log N)`。

## 9. Insert 例子

原堆：

```text
          10
       /      \
      12      20
     /  \    /
    15  18  21
```

插入 `9`：

1. 先放到最后一个位置，也就是 `21` 的右边。
2. `9 < 20`，`20` 下移。
3. `9 < 10`，`10` 下移。
4. `9` 成为根。

结果：

```text
           9
       /      \
      12      10
     /  \    /  \
    15  18  21  20
```

## 10. DeleteMin：下滤 Percolate Down

DeleteMin 要删除根结点，因为 min heap 的最小值在根。

但删除根会破坏结构性质。  
做法：

1. 保存根结点作为 `MinElement`。
2. 拿最后一个元素 `LastElement` 填补根的位置。
3. `Size--`，删除最后位置。
4. 让 `LastElement` 向下找合适位置。

向下过程中，每次都和较小的孩子比较。  
如果 `LastElement` 比较小孩子还大，就让较小孩子上移。

这叫 **percolate down**。

## 11. DeleteMin 代码

```c
ElementType DeleteMin(PriorityQueue H) {
    int i, Child;
    ElementType MinElement, LastElement;

    if (IsEmpty(H)) {
        Error("Priority queue is empty");
        return H->Elements[0];
    }

    MinElement = H->Elements[1];
    LastElement = H->Elements[H->Size--];

    for (i = 1; i * 2 <= H->Size; i = Child) {
        Child = i * 2;

        if (Child != H->Size &&
            H->Elements[Child + 1] < H->Elements[Child]) {
            Child++;
        }

        if (LastElement > H->Elements[Child]) {
            H->Elements[i] = H->Elements[Child];
        } else {
            break;
        }
    }

    H->Elements[i] = LastElement;
    return MinElement;
}
```

关键判断：

```c
Child != H->Size
```

表示当前结点有右孩子，才能比较 `Child + 1`。  
如果省略这个条件，可能访问不存在的右孩子。

复杂度：

```text
O(log N)
```

## 12. DecreaseKey 和 IncreaseKey

### DecreaseKey

`DecreaseKey(P, Δ, H)` 表示把位置 `P` 的 key 减小 `Δ`。

对 min heap 来说，key 变小后可能比父结点还小，所以需要：

```text
percolate up
```

### IncreaseKey

`IncreaseKey(P, Δ, H)` 表示把位置 `P` 的 key 增大 `Δ`。

对 min heap 来说，key 变大后可能比孩子还大，所以需要：

```text
percolate down
```

## 13. Delete 任意位置

删除位置 `P` 的结点可以用：

```text
DecreaseKey(P, infinity, H)
DeleteMin(H)
```

对 min heap 来说，把它降到足够小，它就会上滤到根，然后执行 `DeleteMin` 删除。

复杂度：

```text
O(log N)
```

## 14. BuildHeap

BuildHeap 的任务是：把 `N` 个输入 key 一次性建成一个堆。

如果一个一个 Insert：

```text
N 次 Insert，每次 O(log N)
总复杂度 O(N log N)
```

但更好的方法是：先把所有元素按顺序放进数组，然后从最后一个非叶结点开始向前做 `PercolateDown`。

```text
for (i = N / 2; i > 0; i--) {
    PercolateDown(i);
}
```

为什么从 `N/2` 开始？  
因为下标大于 `N/2` 的结点都是叶子，不需要下滤。

## 15. BuildHeap 为什么是 O(N)

BuildHeap 看起来像 `N` 次下滤，容易误以为是 `O(N log N)`。  
但实际上是：

```text
O(N)
```

原因是：越靠近底层的结点越多，但它们高度很小，下滤不了几层；越靠近根的结点高度大，但数量很少。

对满二叉树，可以把总工作量看成“所有结点高度之和”：

```text
sum of heights = O(N)
```

课件给出的结论是：对高度为 `h` 的 perfect binary tree，结点高度和为：

```text
2^(h+1) - 1 - (h + 1)
```

所以 BuildHeap：

```text
T(N) = O(N)
```

## 16. kth largest 问题

题目：给定 `N` 个元素和整数 `k`，找第 `k` 大元素。

常见方法：

| 方法 | 思路 | 复杂度 |
| --- | --- | --- |
| 排序 | 全部排序后取第 `k` 大 | `O(N log N)` |
| max heap | 建 max heap，执行 `k` 次 DeleteMax | `O(N + k log N)` |
| min heap of size k | 维护大小为 `k` 的 min heap | `O(N log k)` |

最常用的是大小为 `k` 的 min heap：

1. 先把前 `k` 个元素建成 min heap。
2. 对剩下每个元素 `x`：
   - 如果 `x <= heap min`，说明它进不了前 `k` 大，忽略。
   - 如果 `x > heap min`，删除堆顶，再插入 `x`。
3. 最后堆顶就是第 `k` 大。

适合 `k` 远小于 `N` 的情况。

## 17. d-Heaps

d-heap 是每个结点最多有 `d` 个孩子的堆。

二叉堆是 `d = 2` 的特殊情况。

3-heap 示意：

```text
             1
       /     |     \
      2      3      5
    / | \  / | \  / | \
   4  7 10 13 15 6 8 17 9
```

d-heap 的特点：

- 高度变成 `O(log_d N)`，比二叉堆更矮。
- 但 DeleteMin 时要在 `d` 个孩子中找最小，需要 `d - 1` 次比较。

DeleteMin 复杂度大约是：

```text
O(d log_d N)
```

所以不是 `d` 越大越好。

课件还提醒：二叉堆中的 `*2`、`/2` 可以用位移优化，但 `*d`、`/d` 通常不能。  
当优先队列太大，无法完全放入内存时，d-heap 才会变得更有吸引力。

## 18. 考点速记

| 考点 | 必会内容 |
| --- | --- |
| Priority Queue | 按优先级删除，不按进入顺序 |
| Min Heap | 根是最小元素 |
| 结构性质 | 完全二叉树 |
| 堆序性质 | 父结点 `<=` 孩子 |
| 数组下标 | parent `i/2`，left `2i`，right `2i+1` |
| Insert | 放末尾，上滤 |
| DeleteMin | 删根，用最后元素补根，下滤 |
| Insert / DeleteMin | `O(log N)` |
| FindMin | `O(1)` |
| 查找任意 key | 需要线性扫描 `O(N)` |
| DecreaseKey | key 变小，上滤 |
| IncreaseKey | key 变大，下滤 |
| BuildHeap | 从 `N/2` 到 `1` 下滤，`O(N)` |
| kth largest | 大小为 `k` 的 min heap 可做 `O(N log k)` |
| d-heap | DeleteMin 为 `O(d log_d N)` |

## 19. 易错点

- 堆不是 BST，左右子树之间没有整体大小关系。
- min heap 只能保证根最小，不能保证数组整体有序。
- 找任意元素不能二分，通常要 `O(N)` 扫描。
- Insert 必须先放到完全二叉树的最后位置，再上滤。
- DeleteMin 必须用最后一个元素补根，再下滤。
- 下滤时要选择较小的孩子。
- `Child != H->Size` 是为了确认右孩子存在。
- BuildHeap 是 `O(N)`，不是 `O(N log N)`。
