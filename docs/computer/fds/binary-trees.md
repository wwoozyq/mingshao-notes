# 二叉树

这一页整理 Ch04 Binary Trees。重点是树的基本术语、树的实现、二叉树、表达式树、树遍历、目录递归，以及线索二叉树。

## 1. 树 Tree

树是一组结点的集合。这个集合可以为空；如果非空，则由下面两部分组成：

1. 一个特殊结点 `r`，称为根结点（root）。
2. 零个或多个非空子树 `T1, T2, ..., Tk`，每棵子树的根都由一条从 `r` 出发的边连接。

树的递归定义很重要：一棵树由根和若干棵子树组成，而每棵子树本身又是一棵树。

注意：

- 子树之间不能互相连接。
- 一棵有 `N` 个结点的树有 `N - 1` 条边。
- 通常画树时，根在最上面。

## 2. 树的基本术语

假设有一棵树：

```text
              A
        /     |     \
       B      C      D
     /  \     |    / | \
    E    F    G   H  I  J
   / \             |
  K   L            M
```

### 结点的度

一个结点的 **degree** 是它的子树个数，也就是孩子个数。

例如：

```text
degree(A) = 3
degree(F) = 0
```

### 树的度

树的 degree 是所有结点 degree 的最大值。

上面这棵树中，`A` 和 `D` 都有 `3` 个孩子，所以：

```text
degree(tree) = 3
```

### 父亲、孩子、兄弟

- parent：有子树的结点。
- children：某个 parent 的子树根结点。
- siblings：同一个 parent 的孩子。
- leaf / terminal node：degree 为 `0` 的结点，也就是叶子。

例如：

- `A` 是 `B, C, D` 的 parent。
- `B, C, D` 是 siblings。
- `F, G, I, J, K, L, M` 都是 leaves。

### 度数和叶子数公式

对任意非空树，边数有两种数法。

第一种：有 `n` 个结点的树有：

```text
n - 1
```

条边。

第二种：把所有结点的孩子数加起来，也就是把所有结点的 degree 加起来。  
如果 `ni` 表示 degree 为 `i` 的结点个数，那么：

```text
边数 = 1*n1 + 2*n2 + 3*n3 + ... + k*nk
```

总节点数是：

```text
n = n0 + n1 + n2 + ... + nk
```

两式相减，可以得到普通树常用公式：

```text
n0 = 1 + n2 + 2n3 + 3n4 + ... + (k - 1)nk
```

也就是：

```text
叶子数 = 1 + 所有非叶结点的“多出来的孩子数”之和
```

!!! homework "例题：树的 degree 和叶子数"

    Given a tree of degree 3. Suppose that there are 3 nodes of degree 2 and 2 nodes of degree 3. Then the number of leaf nodes must be ___.

    A. `5`  
    B. `6`  
    C. `7`  
    D. `8`

点拨：普通树用公式 `n0 = 1 + n2 + 2n3`。这里 `n2 = 3`，`n3 = 2`。

答案：

```text
n0 = 1 + 3 + 2*2 = 8
```

## 3. 路径、深度和高度

### 路径 Path

从 `n1` 到 `nk` 的路径是一串结点：

```text
n1, n2, ..., nk
```

其中每个 `ni` 都是 `n(i+1)` 的 parent。  
树中从根到任意结点的路径是唯一的。

路径长度是路径上的边数。

### 深度 Depth

结点 `x` 的 depth 是从根到 `x` 的路径长度。

```text
Depth(root) = 0
```

例如上图：

```text
Depth(A) = 0
Depth(B) = 1
Depth(E) = 2
Depth(K) = 3
```

### 高度 Height

结点 `x` 的 height 是从 `x` 到叶子的最长路径长度。

```text
Height(leaf) = 0
```

例如：

```text
Height(D) = 2       因为 D -> H -> M
```

树的高度就是根结点的高度，也等于最深叶子的深度。

### Ancestors 和 Descendants

- ancestors：从当前结点往上到根路径上的所有结点。
- descendants：当前结点所有子树中的结点。

## 4. 树的普通表示为什么麻烦

如果每个结点直接保存所有孩子指针，那么不同结点的孩子数不同，结点结构就很难统一。

例如：

```text
A 有 3 个孩子
B 有 2 个孩子
F 有 0 个孩子
```

如果给每个结点都开很多指针，会浪费空间。  
如果每个结点结构大小不一样，实现又会很麻烦。

这就引出一种很重要的统一表示法：**FirstChild-NextSibling**。

## 5. FirstChild-NextSibling 表示法

每个结点只保留两个指针：

```c
struct TreeNode {
    ElementType Element;
    PtrToNode FirstChild;
    PtrToNode NextSibling;
};
```

含义：

- `FirstChild`：指向第一个孩子。
- `NextSibling`：指向右边下一个兄弟。

例如：

```text
              A
        /     |     \
       B      C      D
     /  \     |
    E    F    G
```

可以理解成：

```text
A.FirstChild = B
B.NextSibling = C
C.NextSibling = D

B.FirstChild = E
E.NextSibling = F

C.FirstChild = G
```

这个表示法的好处是：无论树的度是多少，每个结点都只需要两个指针。

注意：普通树中孩子之间没有固定顺序，所以 FirstChild-NextSibling 表示不唯一；孩子顺序不同，表示也不同。

## 6. 二叉树 Binary Tree

二叉树是每个结点最多有两个孩子的树。

在二叉树中，这两个孩子有明确含义：

- left child
- right child

常见结点结构：

```c
struct TreeNode {
    ElementType Element;
    PtrToNode Left;
    PtrToNode Right;
};
```

二叉树和普通树的区别不只是“最多两个孩子”。  
二叉树的左右孩子有顺序，左和右不能随便交换。

## 7. 普通树转二叉树

FirstChild-NextSibling 表示法天然可以看成一棵二叉树：

- `FirstChild` 变成 `Left`
- `NextSibling` 变成 `Right`

也就是：

```text
普通树的第一个孩子  -> 二叉树左孩子
普通树的下一个兄弟  -> 二叉树右孩子
```

可视化：

```text
普通树:
        A
      / | \
     B  C  D
    / \
   E   F

FirstChild-NextSibling / 二叉树视角:
        A
       /
      B
     / \
    E   C
     \   \
      F   D
```

这也是课件里说“把 FirstChild-NextSibling 树顺时针旋转 45°”的直觉来源。

### 普通树和二叉树遍历对应关系

普通树 `T` 转成 FirstChild-NextSibling 二叉树 `BT` 后，有两个常用对应关系：

```text
T 的 preorder  = BT 的 preorder
T 的 postorder = BT 的 inorder
```

为什么 `T` 的 postorder 对应 `BT` 的 inorder？

在普通树中，postorder 是：

```text
先访问所有孩子子树，最后访问根
```

转成 `BT` 后：

- 左孩子表示 first child。
- 右孩子表示 next sibling。

所以对 `BT` 做 inorder：

```text
先访问左子树 -> 访问根 -> 访问右兄弟
```

正好对应普通树里“先处理当前结点的孩子们，再处理当前结点，然后处理兄弟”。

!!! homework "例题：普通树转二叉树后的遍历"

    If a general tree `T` is converted into a binary tree `BT`, then which of the following `BT` traversals gives the same sequence as that of the post-order traversal of `T`?

    A. Pre-order traversal  
    B. In-order traversal  
    C. Post-order traversal  
    D. Level-order traversal

点拨：普通树转 FirstChild-NextSibling 二叉树后，普通树的 postorder 对应二叉树的 inorder。

答案：

```text
B. In-order traversal
```

## 8. 表达式树 Expression Tree

表达式树用树表示表达式：

- 叶子结点通常是操作数。
- 内部结点通常是运算符。

例如中缀表达式：

```text
A + B * C / D
```

对应表达式树可以看成：

```text
        +
      /   \
     A     /
          / \
         *   D
        / \
       B   C
```

这棵树的含义是：

```text
A + ((B * C) / D)
```

表达式树和遍历关系非常密切：

| 遍历 | 得到的表达式 |
| --- | --- |
| inorder | 中缀表达式 |
| preorder | 前缀表达式 |
| postorder | 后缀表达式 |

对上面这棵树：

```text
inorder:    A + B * C / D
preorder:   + A / * B C D
postorder:  A B C * D / +
```

## 9. 用后缀表达式构造表达式树

给定后缀表达式：

```text
a b + c d e + * *
```

用栈构造表达式树：

1. 遇到操作数：创建单结点树，入栈。
2. 遇到运算符：弹出两棵树，作为左右子树，创建新树再入栈。

注意弹出顺序：

- 先弹出的是右子树。
- 后弹出的是左子树。

例如遇到 `+`，如果先弹出 `b`，再弹出 `a`，则新树是：

```text
    +
   / \
  a   b
```

最后栈中唯一的树就是表达式树。

## 10. 树遍历

遍历就是访问树中每个结点一次，且只访问一次。

常见遍历：

- preorder：先序遍历。
- inorder：中序遍历，主要用于二叉树。
- postorder：后序遍历。
- levelorder：层序遍历。

## 11. 先序和后序遍历

### Preorder

顺序：

```text
先访问根，再访问每棵子树
```

伪代码：

```c
void preorder(tree_ptr tree) {
    if (tree) {
        visit(tree);
        for (each child C of tree) {
            preorder(C);
        }
    }
}
```

### Postorder

顺序：

```text
先访问每棵子树，最后访问根
```

伪代码：

```c
void postorder(tree_ptr tree) {
    if (tree) {
        for (each child C of tree) {
            postorder(C);
        }
        visit(tree);
    }
}
```

## 12. 二叉树中序遍历

中序遍历只对二叉树有自然定义：

```text
左子树 -> 根 -> 右子树
```

递归代码：

```c
void inorder(tree_ptr tree) {
    if (tree) {
        inorder(tree->Left);
        visit(tree->Element);
        inorder(tree->Right);
    }
}
```

对表达式树来说，中序遍历通常对应中缀表达式。  
但严格输出时，有时需要补括号来避免歧义。

## 13. 中序遍历的非递归写法

递归本质上使用系统栈。  
所以中序遍历也可以手动用栈写出来。

核心思想：

1. 一路向左走，把沿途结点压栈。
2. 左边走到底后，弹出栈顶并访问。
3. 转向它的右子树。
4. 重复以上过程。

代码：

```c
void iter_inorder(tree_ptr tree) {
    Stack S = CreateStack(MAX_SIZE);

    for (;;) {
        for (; tree; tree = tree->Left) {
            Push(tree, S);
        }

        if (IsEmpty(S)) {
            break;
        }

        tree = Top(S);
        Pop(S);
        visit(tree->Element);

        tree = tree->Right;
    }
}
```

课件中的伪代码在 `Top(S)` 前需要保证栈非空；否则空栈取顶会出错。

## 14. 层序遍历 Levelorder

层序遍历按照从上到下、从左到右访问结点。

它需要队列：

```c
void levelorder(tree_ptr tree) {
    enqueue(tree);

    while (queue is not empty) {
        T = dequeue();
        visit(T);

        for (each child C of T) {
            enqueue(C);
        }
    }
}
```

可视化：

```text
        1
      /   \
     2     3
    / \   / \
   4   5 6   7

levelorder: 1 2 3 4 5 6 7
```

队列保证：先看到的结点先被访问，所以可以一层一层推进。

## 15. 目录树例子

文件系统可以看成树：

- 目录是内部结点。
- 文件通常是叶子。
- 子目录和文件是某个目录的孩子。

### 列出目录

目录列表可以用先序遍历：先打印当前目录/文件，再递归打印孩子。

```c
static void ListDir(DirOrFile D, int Depth) {
    if (D is a legitimate entry) {
        PrintName(D, Depth);

        if (D is a directory) {
            for (each child C of D) {
                ListDir(C, Depth + 1);
            }
        }
    }
}
```

`Depth` 用来控制缩进：

```text
depth = 0    /usr
depth = 1        mark
depth = 2            book
```

注意：`Depth` 是内部变量，不应该暴露给用户。  
可以再包一层接口：

```c
void ListDirectory(DirOrFile D) {
    ListDir(D, 0);
}
```

复杂度：

```text
O(N)
```

因为每个目录或文件只访问一次。

### 计算目录大小

目录大小可以用后序遍历：先算完所有孩子，再汇总当前目录。

```c
static int SizeDir(DirOrFile D) {
    int TotalSize = 0;

    if (D is a legitimate entry) {
        TotalSize = FileSize(D);

        if (D is a directory) {
            for (each child C of D) {
                TotalSize += SizeDir(C);
            }
        }
    }

    return TotalSize;
}
```

复杂度同样是：

```text
O(N)
```

因为每个结点只处理一次。

## 16. 二叉树中的空指针数量

线索二叉树前有一个经典问题：一棵有 `n` 个结点的二叉树中，有多少个空指针？

每个结点有两个指针域：

```text
Left 和 Right
```

所以总指针域数量是：

```text
2n
```

一棵有 `n` 个结点的树有 `n - 1` 条边。每条边会占用一个非空指针。  
所以非空指针有：

```text
n - 1
```

空指针数量为：

```text
2n - (n - 1) = n + 1
```

这说明普通二叉树里有很多空指针域。  
线索二叉树就是想利用这些空指针。

### 二叉树结点数例题

!!! homework "例题：二叉树是否存在"

    There exists a binary tree with `2016` nodes in total, and with `16` nodes having only one child.

    判断这句话是否正确。

点拨：二叉树满足 `n0 = n2 + 1`，且 `n = n0 + n1 + n2`。已知 `n = 2016`，`n1 = 16`。

答案：

```text
n0 = n2 + 1
2016 = n0 + 16 + n2
2016 = (n2 + 1) + 16 + n2
1999 = 2n2
```

`n2 = 999.5` 不是整数，所以这样的二叉树不存在。原命题是 false。

## 17. 线索二叉树 Threaded Binary Tree

线索二叉树把部分空指针替换成遍历意义上的前驱/后继指针。

以中序线索二叉树为例：

- 如果 `Tree->Left` 为空，就让它指向中序前驱。
- 如果 `Tree->Right` 为空，就让它指向中序后继。

规则：

```text
Rule 1: Left 为空时，替换为 inorder predecessor
Rule 2: Right 为空时，替换为 inorder successor
Rule 3: 不能有 loose threads，所以通常需要 head node
```

## 18. 线索二叉树结点结构

因为 `Left` 和 `Right` 有时是真孩子，有时是线索，所以需要标记位：

```c
typedef struct ThreadedTreeNode *ThreadedTree;

struct ThreadedTreeNode {
    int LeftThread;
    ThreadedTree Left;
    ElementType Element;
    int RightThread;
    ThreadedTree Right;
};
```

含义：

- `LeftThread == TRUE`：`Left` 是线索，指向中序前驱。
- `LeftThread == FALSE`：`Left` 是真正的左孩子。
- `RightThread == TRUE`：`Right` 是线索，指向中序后继。
- `RightThread == FALSE`：`Right` 是真正的右孩子。

线索二叉树的目的：让中序遍历更方便，减少对递归栈或显式栈的依赖。

## 19. 考点速记

| 考点 | 必会内容 |
| --- | --- |
| 树定义 | 根 + 若干互不相交子树 |
| 边数 | `N` 个结点的树有 `N - 1` 条边 |
| degree(node) | 结点孩子数 |
| degree(tree) | 最大结点度 |
| depth | 根到该结点的路径长度 |
| height | 该结点到叶子的最长路径长度 |
| FirstChild-NextSibling | 普通树统一成两个指针 |
| 二叉树 | 每个结点最多两个孩子，左右有区别 |
| 表达式树 | 叶子是操作数，内部结点是运算符 |
| preorder | 根 -> 子树 |
| inorder | 左 -> 根 -> 右 |
| postorder | 子树 -> 根 |
| levelorder | 用队列逐层访问 |
| 目录列表 | 先序思想 |
| 目录大小 | 后序思想 |
| 空指针数量 | `n + 1` |
| 线索二叉树 | 空指针改成前驱/后继线索 |

## 20. 易错点

- 树可以为空，但非空树必须有 root。
- depth 从根往下数，height 从结点往叶子方向数。
- 叶子的 height 是 `0`，不是 `1`。
- 普通树孩子顺序不唯一，FirstChild-NextSibling 表示也不唯一。
- 二叉树左右孩子不能随便交换。
- 后缀表达式构造表达式树时，先弹出的是右子树。
- 层序遍历用队列，不是栈。
- 中序遍历主要是二叉树概念，普通多叉树没有唯一自然中序。
- 线索指针不是孩子指针，必须用 `LeftThread/RightThread` 区分。
