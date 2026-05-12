# 列表 List

这一页整理 Ch03 List。重点是 ADT 思想、List ADT、数组实现、链表实现、插入删除时指针顺序、带头结点、双向循环链表、多项式 ADT、多重链表，以及 cursor implementation。

## 1. ADT 是什么

ADT 是 Abstract Data Type，抽象数据类型。

课件里的定义可以拆成一句更好懂的话：

```text
ADT = 对象集合 + 操作集合
```

也就是：

- 这个数据类型里有什么对象。
- 对这些对象允许做什么操作。
- 不关心对象具体怎么存，也不关心操作具体怎么实现。

例如 `int` 可以理解成：

```text
Objects: ..., -2, -1, 0, 1, 2, ...
Operations: +, -, *, /, %, ...
```

ADT 最重要的思想是 **specification 和 implementation 分离**。

```text
Specification: 使用者需要知道的规则
Implementation: 程序内部怎么存、怎么做
```

比如栈的 ADT 说可以 `Push` 和 `Pop`，但没有规定必须用数组实现，还是链表实现。

## 2. List ADT

List 是一个有限有序序列：

```text
(item0, item1, ..., itemN-1)
```

这里的“有序”不是指大小有序，而是指每个元素有位置顺序。  
`item0` 在 `item1` 前面，`item1` 在 `item2` 前面。

常见操作：

| 操作 | 含义 |
| --- | --- |
| `Length(L)` | 返回表长 `N` |
| `PrintList(L)` | 打印所有元素 |
| `MakeEmpty(L)` | 清空列表 |
| `FindKth(k, L)` | 找第 `k` 个元素，通常 `0 <= k < N` |
| `Insert(x, k, L)` | 在第 `k` 个元素之后插入 `x` |
| `Delete(x, L)` | 删除元素 `x` |
| `Next(p, L)` | 找当前位置的后继 |
| `Previous(p, L)` | 找当前位置的前驱 |

学习 List 的关键不是背函数名，而是比较不同实现下这些操作的代价。

## 3. 数组实现 List

数组实现也叫 sequential mapping。  
如果 `array[i] = itemi`，那么第 `i` 个元素直接存在数组下标 `i` 的位置。

```text
index:   0    1    2    3
value:  a0   a1   a2   a3
```

### 优点

找第 `k` 个元素非常快：

```text
FindKth(k) = array[k]
```

复杂度：

```text
O(1)
```

原因是数组支持随机访问，地址可以直接算出来。

### 缺点

数组实现有两个主要问题。

第一，`MaxSize` 必须提前估计。

```text
开小了：不够用
开大了：浪费空间
```

第二，插入和删除通常要移动很多元素。

例如在中间插入 `x`：

```text
插入前:
index:  0   1   2   3   4
value:  A   B   C   D   E

在 B 后插入 X:
index:  0   1   2   3   4   5
value:  A   B   X   C   D   E
```

为了给 `X` 腾位置，`C, D, E` 都要向后移动。

复杂度：

```text
Insert: O(N)
Delete: O(N)
```

数组表适合：

- 经常按下标访问。
- 插入删除不频繁。
- 最大规模大致可估计。

## 4. 单链表 Linked List

链表不要求元素在内存中连续。  
每个结点保存两部分：

```text
data + next
```

结构大致是：

```c
typedef struct list_node *list_ptr;

struct list_node {
    ElementType data;
    list_ptr next;
};
```

一条链表可以画成：

```text
ptr -> A -> B -> C -> NULL
```

这里 `ptr` 是头指针，指向第一个结点。  
每个结点的 `next` 指向下一个结点。

链表的重点是：**逻辑上相邻，不代表物理地址相邻**。

## 5. 单链表插入

假设要把新结点 `temp` 插到 `node` 后面：

```text
插入前:
node -> ai+1 -> ...

目标:
node -> temp -> ai+1 -> ...
```

正确顺序是：

```c
temp->next = node->next;
node->next = temp;
```

为什么不能反过来？

如果先写：

```c
node->next = temp;
```

那么原来的 `node->next`，也就是 `ai+1` 的地址就丢了。  
后面的链表会断掉。

可视化：

```text
正确:
1. temp 先连到 node 原来的后继
2. node 再连到 temp

错误:
1. node 先连到 temp
2. 原来的后继找不到了
```

如果已经有 `node` 指针，插入本身是：

```text
O(1)
```

但如果题目只给你“第 k 个位置”，你还要先从头走到第 `k` 个结点，这一步是 `O(N)`。

## 6. 单链表删除

删除单链表中的某个结点时，通常需要知道它的前驱 `pre`。

假设：

```text
pre -> node -> next
```

删除 `node`：

```c
pre->next = node->next;
free(node);
```

顺序也很重要。  
必须先让 `pre` 跳过 `node`，再释放 `node`。

```text
删除前:
pre -> node -> next

删除后:
pre --------> next
```

如果已经有 `pre` 指针，删除本身是：

```text
O(1)
```

但如果要先查找待删结点或它的前驱，查找过程通常是 `O(N)`。

## 7. 为什么要带头结点

课件问了两个很典型的问题：

- 怎样插入新的第一个元素？
- 怎样删除第一个结点？

如果没有头结点，第一个结点要特殊处理，因为头指针本身会改变。

例如删除第一个结点：

```text
ptr -> A -> B -> C -> NULL
```

删除 `A` 后，`ptr` 必须改成指向 `B`。

为了减少这种特殊情况，可以加入 dummy head node，也叫 header node。

```text
H -> A -> B -> C -> NULL
```

此时真正的数据从 `H->next` 开始。  
插入或删除第一个数据结点，也变成“在 `H` 后面操作”。

```text
删除 A:
H -> A -> B -> C

pre = H
pre->next = A->next

结果:
H -> B -> C
```

带头结点的好处：

- 空表和非空表处理更统一。
- 首结点插入删除不需要单独写一套逻辑。
- 代码更不容易出错。

## 8. 双向循环链表

单链表找后继方便，但找前驱不方便。  
如果只有某个结点 `ptr`，要找它前面的结点，通常要从头走一遍。

双向链表在每个结点里同时存：

```text
llink + item + rlink
```

```c
typedef struct node *node_ptr;

struct node {
    node_ptr llink;
    ElementType item;
    node_ptr rlink;
};
```

其中：

- `llink` 指向前驱。
- `rlink` 指向后继。

循环链表把尾结点再连回头结点。  
如果带 head node，大致是：

```text
        +-----------------------------+
        |                             v
H <-> item1 <-> item2 <-> item3 <----+
^                                    |
|____________________________________|
```

空的双向循环链表也很统一：

```text
H->llink = H
H->rlink = H
```

双向循环链表适合：

- 需要频繁找前驱。
- 需要在已知结点附近插入删除。
- 希望头尾操作统一。

缺点是每个结点多存一个指针，指针维护也更复杂。

## 9. 多项式 ADT

多项式可以看成若干项组成的有序集合：

```text
P(x) = a1*x^e1 + a2*x^e2 + ... + an*x^en
```

每一项可以表示成：

```text
<exponent, coefficient>
```

其中：

- `coefficient` 是系数。
- `exponent` 是指数。

常见操作：

| 操作 | 含义 |
| --- | --- |
| `FindDegree(P)` | 找最高次数 |
| `Add(P1, P2)` | 多项式加法 |
| `Sub(P1, P2)` | 多项式减法 |
| `Mul(P1, P2)` | 多项式乘法 |
| `Differentiate(P)` | 求导 |

## 10. 多项式的数组表示

一种直接表示法是用数组下标表示指数：

```c
typedef struct {
    int CoeffArray[MaxDegree + 1];
    int HighPower;
} *Polynomial;
```

例如：

```text
P(x) = 10x^1000 + 5x^14 + 1
```

可以存成：

```text
CoeffArray[1000] = 10
CoeffArray[14]   = 5
CoeffArray[0]    = 1
```

优点：

- 按指数访问很直接。
- 加法、减法实现简单。

问题：

- 如果最高指数很大，但非零项很少，会浪费大量空间。
- 多项式乘法可能要遍历很多零系数位置。

所以数组表示适合“指数范围不大、比较稠密”的多项式。

## 11. 多项式的链表表示

链表表示只存非零项。  
每个结点保存：

```text
coefficient + exponent + next
```

代码结构：

```c
typedef struct poly_node *poly_ptr;

struct poly_node {
    int Coefficient;
    int Exponent;
    poly_ptr Next;
};
```

通常按指数从大到小排序：

```text
A(x) = a_m x^e_m + ... + a_0 x^e_0

链表:
[a_m, e_m] -> ... -> [a_0, e_0] -> NULL
```

优点：

- 只存非零项，适合 sparse polynomial。
- 加法可以像归并一样比较指数。

多项式加法的思路：

```text
while P1 和 P2 都没结束:
    如果 P1 指数 > P2 指数，复制 P1 当前项
    如果 P1 指数 < P2 指数，复制 P2 当前项
    如果指数相等，系数相加
        如果结果不为 0，加入结果表
```

这个思路和“合并两个有序链表”很像。

## 12. Multilists

课件里的例子：

```text
40000 个学生
2500 门课程
需要：
1. 对每门课打印选课学生名单
2. 对每个学生打印他选的课
```

如果用二维数组：

```c
int Array[40000][2500];
```

含义是：

```text
Array[i][j] = 1  表示学生 i 选了课程 j
Array[i][j] = 0  表示没有选
```

问题是空间巨大：

```text
40000 * 2500 = 100000000
```

也就是一亿个位置。  
但真实选课关系通常很稀疏，大部分位置都是 `0`。

Multilist 的想法是：只为真实存在的关系建结点。  
一个选课关系结点可以同时挂在：

- 某个学生的课程链表里。
- 某门课的学生链表里。

这样既能从学生查课程，也能从课程查学生。

这个思想和稀疏矩阵表示很接近：不要存所有格子，只存非零项。

## 13. Cursor Implementation

Cursor implementation 是“没有指针的链表实现”。  
它用数组模拟链表。

普通链表需要两个能力：

1. 结点里有数据和指向下一个结点的指针。
2. 可以通过 `malloc` 申请结点，通过 `free` 释放结点。

Cursor implementation 用数组下标代替指针：

```c
struct Node {
    ElementType Element;
    int Next;
};

struct Node CursorSpace[SpaceSize];
```

这里 `Next` 不是真正的地址，而是数组下标。

```text
index:    0    1    2    3   ...
Element:       A    B    C
Next:     1    2    3    0
```

如果 `Next = 0`，通常表示链表结束，类似 `NULL`。

## 14. Cursor 中的 malloc 和 free

Cursor implementation 会维护一个 free list。  
课件中 `CursorSpace[0].Next` 指向空闲链表的第一个位置。

### malloc

申请一个空闲结点：

```c
p = CursorSpace[0].Next;
CursorSpace[0].Next = CursorSpace[p].Next;
```

意思是：

```text
从 free list 头部拿走一个结点 p
```

### free

释放一个结点 `p`：

```c
CursorSpace[p].Next = CursorSpace[0].Next;
CursorSpace[0].Next = p;
```

意思是：

```text
把 p 插回 free list 的头部
```

Cursor implementation 的接口可以和指针链表相同。  
也就是说，外部使用者还是调用 `Insert`、`Delete` 等操作，不需要知道内部到底是真指针还是数组下标。

课件提到它有时会更快，因为少了系统层面的 memory management routine。  
但理解时重点抓住一句：

```text
用数组下标模拟指针，用 free list 模拟 malloc/free。
```

## 15. 考点速记

| 考点 | 必会内容 |
| --- | --- |
| ADT | 对象集合 + 操作集合，规格和实现分离 |
| List | 有限有序序列，不一定按大小排序 |
| 数组表 FindKth | `O(1)` |
| 数组表插入删除 | 通常 `O(N)`，因为要移动元素 |
| 链表物理位置 | 逻辑相邻，不要求内存相邻 |
| 单链表插入 | 先 `temp->next = node->next`，再 `node->next = temp` |
| 单链表删除 | 先改前驱指针，再 `free` |
| 头结点 | 统一首结点和普通结点操作 |
| 双向链表 | 能直接找前驱，但多维护一个指针 |
| 多项式数组表示 | 适合指数范围小、比较稠密 |
| 多项式链表表示 | 只存非零项，适合稀疏多项式 |
| Multilist | 一个关系结点同时属于多条链 |
| Cursor implementation | 用数组下标模拟指针 |

## 16. 易错点

- List 的“有序”是位置有序，不是数值大小有序。
- 数组 `FindKth` 是 `O(1)`，但插入删除通常是 `O(N)`。
- 单链表插入时指针顺序不能反，反了会丢失后半段链表。
- 删除结点前要先保存或利用它的后继，不能先 `free` 再访问。
- 有头结点时，`H` 本身不存有效数据，真正第一个元素是 `H->next`。
- 链表已知位置插入删除是 `O(1)`，但“找到这个位置”可能要 `O(N)`。
- Cursor implementation 的 `Next` 是下标，不是真实地址。
