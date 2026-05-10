# 搜索树

这一页整理 Ch04 Search Tree。重点是二叉树性质、Binary Search Tree 的定义、查找、最小/最大值、插入、删除、lazy deletion，以及树高对复杂度的影响。

## 1. 二叉树补充性质

在普通树里，孩子之间的顺序通常不重要。  
但在二叉树里，左孩子和右孩子是不同的。

例如：

```text
    A            A
   /              \
  B                B
```

这两棵是不同的二叉树。

## 2. 斜树和完全二叉树

### 斜树

如果每个结点都只有左孩子，或者都只有右孩子，就会形成斜树。

```text
左斜树:
    A
   /
  B
 /
C

右斜树:
A
 \
  B
   \
    C
```

斜树虽然是二叉树，但形状接近链表。  
很多二叉树操作的复杂度会退化到 `O(N)`。

### 完全二叉树

完全二叉树（complete binary tree）可以这样定义：

> 一棵有 `n` 个结点、高度为 `h` 的二叉树是完全二叉树，当且仅当它的结点正好对应高度为 `h` 的满二叉树中按层序编号 `1` 到 `n` 的那些结点。

更直观地说：

- 除最后一层外，前面的层都是满的。
- 最后一层可以不满，但必须从左到右连续填充，不能中间空着。

```text
        A
      /   \
     B     C
    / \   / \
   D   E F   G
  / \
 H   I
```

完全二叉树比较“矮”，高度大约是 `O(log N)`。

## 3. 二叉树结点数量性质

### 每层最多结点数

如果根在第 `1` 层，那么第 `i` 层最多有：

$$
2^{i-1}
$$

个结点。

### 深度为 k 的二叉树最多结点数

深度为 `k` 的二叉树最多有：

$$
2^k - 1
$$

个结点。

这里课件采用的是“第 1 层为根，深度 k 表示有 k 层”的写法。  
如果你用“root depth = 0”的定义，公式会相应写成高度为 `h` 最多有：

$$
2^{h+1} - 1
$$

个结点。

## 4. 叶子数和二度结点数

对任意非空二叉树：

```text
n0 = n2 + 1
```

其中：

- `n0`：叶子结点数，即 degree 为 `0` 的结点数。
- `n1`：degree 为 `1` 的结点数。
- `n2`：degree 为 `2` 的结点数。

证明：

总节点数：

```text
n = n0 + n1 + n2
```

树有 `n - 1` 条边。  
另一方面，所有边都从父结点连向孩子：

```text
边数 = n1 + 2n2
```

所以：

```text
n - 1 = n1 + 2n2
```

代入 `n = n0 + n1 + n2`：

```text
n0 + n1 + n2 - 1 = n1 + 2n2
n0 - 1 = n2
n0 = n2 + 1
```

这个公式很常考，记住“叶子数 = 二度结点数 + 1”。

## 5. 二叉搜索树 BST

Binary Search Tree，简称 BST。  
BST 是一种特殊的二叉树，可以为空；如果非空，则满足：

1. 每个结点有一个 key，课件默认 key 是互不相同的整数。
2. 左子树中所有 key 都小于根结点 key。
3. 右子树中所有 key 都大于根结点 key。
4. 左右子树也都是 BST。

可视化：

```text
          30
        /    \
       5      40
      / \    /  \
     2  25  35  80
```

对任意结点都满足：

```text
左子树 < 当前结点 < 右子树
```

## 6. BST ADT

常见操作：

| 操作 | 含义 |
| --- | --- |
| `MakeEmpty(T)` | 清空搜索树 |
| `Find(X, T)` | 查找 key 为 `X` 的结点 |
| `FindMin(T)` | 找最小 key |
| `FindMax(T)` | 找最大 key |
| `Insert(X, T)` | 插入 `X` |
| `Delete(X, T)` | 删除 `X` |
| `Retrieve(P)` | 取出位置 `P` 对应的元素 |

这些操作的复杂度通常和树高有关。  
如果树高为 `h`，查找、插入、删除都是：

```text
O(h)
```

## 7. Find 查找

查找利用 BST 的大小关系：

- 如果 `X` 小于当前结点，去左子树。
- 如果 `X` 大于当前结点，去右子树。
- 如果相等，找到。
- 如果走到 `NULL`，说明不存在。

递归版：

```c
Position Find(ElementType X, SearchTree T) {
    if (T == NULL) {
        return NULL;
    }

    if (X < T->Element) {
        return Find(X, T->Left);
    } else if (X > T->Element) {
        return Find(X, T->Right);
    } else {
        return T;
    }
}
```

这是尾递归，可以很自然地改成迭代。

迭代版：

```c
Position Iter_Find(ElementType X, SearchTree T) {
    while (T) {
        if (X == T->Element) {
            return T;
        } else if (X < T->Element) {
            T = T->Left;
        } else {
            T = T->Right;
        }
    }

    return NULL;
}
```

复杂度：

```text
O(d)
```

其中 `d` 是查找路径深度。最坏情况是 `O(h)`。

## 8. FindMin 和 FindMax

BST 的最小值一定在最左边。  
BST 的最大值一定在最右边。

### FindMin

```c
Position FindMin(SearchTree T) {
    if (T == NULL) {
        return NULL;
    } else if (T->Left == NULL) {
        return T;
    } else {
        return FindMin(T->Left);
    }
}
```

### FindMax

```c
Position FindMax(SearchTree T) {
    if (T != NULL) {
        while (T->Right != NULL) {
            T = T->Right;
        }
    }

    return T;
}
```

复杂度都是：

```text
O(h)
```

## 9. Insert 插入

插入时先按照查找逻辑往下走。  
如果发现 key 已经存在，课件默认什么都不做。  
如果走到空位置，就在那里创建新结点。

例子：插入 `35`

```text
          30
        /    \
       5      40
      / \    /
     2  25  35
```

路径：

```text
35 > 30，向右
35 < 40，向左
左边为空，插入
```

代码：

```c
SearchTree Insert(ElementType X, SearchTree T) {
    if (T == NULL) {
        T = malloc(sizeof(struct TreeNode));
        if (T == NULL) {
            FatalError("Out of space!!!");
        } else {
            T->Element = X;
            T->Left = T->Right = NULL;
        }
    } else if (X < T->Element) {
        T->Left = Insert(X, T->Left);
    } else if (X > T->Element) {
        T->Right = Insert(X, T->Right);
    }

    return T;
}
```

最后的：

```c
return T;
```

不能忘。因为递归插入后，父结点需要接回更新后的子树根。

复杂度：

```text
O(h)
```

## 10. Delete 删除

删除是 BST 中最容易错的操作。  
分三种情况讨论。

### 情况一：删除叶子结点

叶子没有孩子，直接删掉，并把父结点对应指针设为 `NULL`。

```text
删除 25:

    30              30
   /  \     ->     /  \
  5   40          5   40
   \
   25
```

### 情况二：删除 degree 为 1 的结点

该结点只有一个孩子，直接用它的孩子顶替它。

```text
删除 5:

    30              30
   /  \     ->     /  \
  5   40          2   40
 /
2
```

### 情况三：删除 degree 为 2 的结点

如果结点有两个孩子，不能直接拿某个孩子顶替，否则 BST 顺序可能被破坏。

常用做法：

1. 找右子树中的最小结点，也就是 inorder successor。
2. 用它的 key 替换当前结点。
3. 再从右子树中删除那个 successor。

也可以找左子树中的最大结点，即 inorder predecessor。

例子：

```text
删除 60:

          60
        /    \
       40     70
      / \    /  \
     20 55  65  80

右子树最小值是 65，用 65 替换 60，再删除原来的 65。
```

删除后：

```text
          65
        /    \
       40     70
      / \      \
     20 55      80
```

## 11. Delete 代码

课件中的代码整理如下：

```c
SearchTree Delete(ElementType X, SearchTree T) {
    Position TmpCell;

    if (T == NULL) {
        Error("Element not found");
    } else if (X < T->Element) {
        T->Left = Delete(X, T->Left);
    } else if (X > T->Element) {
        T->Right = Delete(X, T->Right);
    } else if (T->Left && T->Right) {
        TmpCell = FindMin(T->Right);
        T->Element = TmpCell->Element;
        T->Right = Delete(T->Element, T->Right);
    } else {
        TmpCell = T;
        if (T->Left == NULL) {
            T = T->Right;
        } else if (T->Right == NULL) {
            T = T->Left;
        }
        free(TmpCell);
    }

    return T;
}
```

复杂度：

```text
O(h)
```

其中 `h` 是树高。

## 12. Lazy Deletion

如果删除操作不多，或者频繁删除后又重新插入，可以使用 lazy deletion。

做法：给每个结点增加一个标记字段。

```text
active = true      正常结点
active = false     逻辑上已删除
```

删除时不真正释放结点，只把它标记为 deleted。  
如果之后重新插入同一个 key，可以直接把标记改回 active，避免再次 `malloc`。

优点：

- 删除实现简单。
- 可能减少内存申请释放开销。

缺点：

- 树中会保留无效结点。
- 如果 deleted 结点太多，查找效率会变差。
- 需要定期重建或清理。

课件提出的问题是：如果 deleted 结点和 active 结点数量差不多，会不会严重影响效率？  
直觉上会，因为树的实际规模变大了，很多查找路径会经过已经删除的结点。

## 13. 树高与平均情况

BST 操作复杂度是 `O(h)`，所以树高非常关键。

同样插入 `1,2,3,4,5,6,7`，不同插入顺序会得到完全不同的树。

### 较平衡的插入顺序

```text
插入顺序: 4, 2, 1, 3, 6, 5, 7

        4
      /   \
     2     6
    / \   / \
   1   3 5   7

h = 2
```

### 糟糕的插入顺序

```text
插入顺序: 1, 2, 3, 4, 5, 6, 7

1
 \
  2
   \
    3
     \
      4
       \
        5
         \
          6
           \
            7

h = 6
```

这棵树退化成链表，查找复杂度变成：

```text
O(N)
```

如果树保持平衡，高度大约是：

```text
O(log N)
```

操作也就可以做到：

```text
O(log N)
```

这就是后面 AVL 树、红黑树等平衡搜索树存在的原因。

## 14. BST 和中序遍历

BST 有一个很重要的性质：

```text
中序遍历 BST，会得到递增序列。
```

原因是中序遍历顺序是：

```text
左子树 -> 根 -> 右子树
```

而 BST 满足：

```text
左子树所有 key < 根 < 右子树所有 key
```

所以输出天然有序。

例子：

```text
        4
      /   \
     2     6
    / \   / \
   1   3 5   7

inorder: 1 2 3 4 5 6 7
```

这个性质经常用于判断一棵树是不是 BST，或把 BST 转成有序序列。

## 15. 考点速记

| 考点 | 必会内容 |
| --- | --- |
| 二叉树左右 | 左孩子和右孩子不同 |
| 每层最多结点 | 第 `i` 层最多 $2^{i-1}$ 个 |
| 深度为 k 最多结点 | $2^k - 1$ |
| 叶子公式 | `n0 = n2 + 1` |
| BST 定义 | 左子树 < 根 < 右子树 |
| Find | 小往左，大往右 |
| FindMin | 一直向左 |
| FindMax | 一直向右 |
| Insert | 找到空位置插入，重复 key 默认不处理 |
| Delete 叶子 | 直接删除 |
| Delete 一个孩子 | 用唯一孩子顶替 |
| Delete 两个孩子 | 用右子树最小值或左子树最大值替换 |
| Lazy deletion | 标记删除，不立即释放 |
| 复杂度 | `O(h)`，取决于树高 |
| 糟糕插入顺序 | 可能退化成链，`O(N)` |
| BST 中序遍历 | 得到递增序列 |

## 16. 易错点

- BST 的左右子树也必须分别是 BST。
- 只检查“左孩子 < 根 < 右孩子”不够，要检查整棵左子树和右子树范围。
- `FindMin(NULL)` 应返回 `NULL`，不要访问空指针。
- 插入递归后要 `return T`，并接回 `T->Left` 或 `T->Right`。
- 删除两个孩子的结点时，替换值之后还要删除原 successor/predecessor。
- Lazy deletion 会保留 deleted 结点，不能在查找时当作正常 active 结点。
- BST 的复杂度不是天然 `O(log N)`，只有树比较平衡时才接近 `O(log N)`。
