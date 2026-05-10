# 栈和队列

这一页整理 Ch03 Stack and Queue。重点是 ADT 定义、数组/链表实现、括号匹配、后缀表达式、infix 转 postfix、系统栈，以及循环队列。

## 1. 栈 Stack

栈是一种 **Last-In-First-Out** 的线性表，简称 **LIFO**。

也就是说：

```text
最后放进去的元素，最先被取出来。
```

可以把栈想成一摞盘子：

```text
top ->  6   最后放入，最先拿走
        5
        4
        3
        2
        1   最早放入，最后拿走
```

栈只允许在一端操作，这一端叫 **top**。

## 2. 栈 ADT

栈的对象是一个有限有序表，可以有零个或多个元素。

常见操作：

| 操作 | 含义 |
| --- | --- |
| `IsEmpty(S)` | 判断栈是否为空 |
| `CreateStack()` | 创建一个栈 |
| `DisposeStack(S)` | 销毁栈 |
| `MakeEmpty(S)` | 清空栈 |
| `Push(X, S)` | 把 `X` 压入栈顶 |
| `Top(S)` | 返回栈顶元素，但不删除 |
| `Pop(S)` | 删除栈顶元素 |

注意：

- 对空栈执行 `Pop` 或 `Top` 是 **ADT 错误**。
- 对满栈执行 `Push` 是 **实现层面的错误**，因为“满”取决于具体实现。

这句话很容易考概念：栈这个 ADT 本身没有规定容量上限；容量上限通常来自数组实现。

## 3. 栈的链表实现

栈可以用链表实现，常见写法是带一个 header node。  
栈顶就是 header 后面的第一个结点。

```text
S(header) -> first -> next -> next -> NULL
              top
```

### Push

把新结点插到 header 后面：

```text
TmpCell->Next = S->Next
S->Next = TmpCell
```

可视化：

```text
插入前:
S -> A -> B -> NULL

插入 X:
S -> X -> A -> B -> NULL
     top
```

### Pop

删除 header 后面的第一个结点：

```text
FirstCell = S->Next
S->Next = S->Next->Next
free(FirstCell)
```

链表栈的优点是不需要预先固定容量。  
缺点是每次 `Push`/`Pop` 可能涉及 `malloc` 和 `free`，开销比数组大。

课件里提到一个优化思路：可以维护一个“回收栈”保存释放掉的结点，减少频繁申请和释放内存。

## 4. 栈的数组实现

数组实现通常维护：

```c
struct StackRecord {
    int Capacity;
    int TopOfStack;
    ElementType *Array;
};
```

其中：

- `Capacity`：栈容量。
- `TopOfStack`：栈顶下标。
- `Array`：真正存元素的数组。

常见约定：

```text
TopOfStack = -1      表示空栈
Push 时 TopOfStack++
Pop 时 TopOfStack--
```

可视化：

```text
Array index:   0   1   2   3   4
Array value:   A   B   C
TopOfStack:            ^
                       2
```

### 数组栈代码模板

```c
int IsEmpty(Stack S) {
    return S->TopOfStack == -1;
}

int IsFull(Stack S) {
    return S->TopOfStack == S->Capacity - 1;
}

void Push(ElementType X, Stack S) {
    if (IsFull(S)) {
        Error("Full stack");
    } else {
        S->Array[++S->TopOfStack] = X;
    }
}

ElementType Top(Stack S) {
    if (IsEmpty(S)) {
        Error("Empty stack");
    }
    return S->Array[S->TopOfStack];
}

void Pop(Stack S) {
    if (IsEmpty(S)) {
        Error("Empty stack");
    } else {
        S->TopOfStack--;
    }
}
```

数组栈实现时要注意封装：除了栈操作函数，其他代码不应该直接访问 `Array` 或 `TopOfStack`。

## 5. 栈应用：括号匹配

目标：检查表达式中的 `()`, `[]`, `{}` 是否匹配。

核心思想：

- 遇到左括号，入栈。
- 遇到右括号，检查栈顶是不是对应的左括号。
- 如果不匹配，出错。
- 扫描结束后，如果栈不空，也出错。

算法：

```text
MakeEmpty(S)
while read character c:
    if c is opening symbol:
        Push(c, S)
    else if c is closing symbol:
        if S is empty:
            ERROR
        else if Top(S) does not match c:
            ERROR
        else:
            Pop(S)

if S is not empty:
    ERROR
```

例子：

```text
表达式: { [ ( ) ] }

读到 { 入栈: {
读到 [ 入栈: { [
读到 ( 入栈: { [ (
读到 ) 匹配 (，弹出
读到 ] 匹配 [，弹出
读到 } 匹配 {，弹出
最后栈空，合法
```

复杂度：

```text
O(N)
```

其中 `N` 是表达式长度。  
这是一个 on-line algorithm，因为它可以边读边判断，不需要先读完整个输入。

## 6. 栈应用：后缀表达式求值

中缀表达式：

```text
a + b * c - d / e
```

后缀表达式：

```text
a b c * + d e / -
```

后缀表达式也叫 Reverse Polish notation。  
它的好处是：不需要括号，也不需要运算符优先级规则。

### 求值规则

从左到右扫描 token：

- 如果是操作数，入栈。
- 如果是运算符，弹出两个操作数，计算后把结果压回栈。

注意二元运算的顺序。  
如果先弹出的是 `right`，后弹出的是 `left`，则计算：

```text
left op right
```

例子：

```text
6 2 / 3 - 4 2 * +
```

步骤：

| token | 动作 | 栈 |
| --- | --- | --- |
| `6` | 入栈 | `6` |
| `2` | 入栈 | `6, 2` |
| `/` | `6 / 2 = 3` | `3` |
| `3` | 入栈 | `3, 3` |
| `-` | `3 - 3 = 0` | `0` |
| `4` | 入栈 | `0, 4` |
| `2` | 入栈 | `0, 4, 2` |
| `*` | `4 * 2 = 8` | `0, 8` |
| `+` | `0 + 8 = 8` | `8` |

答案：

```text
8
```

复杂度：

```text
O(N)
```

## 7. 中缀转后缀

目标：把中缀表达式转成后缀表达式。

例子：

```text
a + b * c - d
```

转换结果：

```text
a b c * + d -
```

两个观察：

- 操作数在中缀和后缀中的相对顺序不变。
- 运算符要按照优先级和结合性决定什么时候输出。

## 8. 中缀转后缀算法

用一个栈保存暂时还不能输出的运算符。

从左到右扫描 token：

1. 如果是操作数，直接输出。
2. 如果是左括号 `(`，入栈。
3. 如果是右括号 `)`，不断弹出并输出，直到遇到左括号；左括号只弹出，不输出。
4. 如果是普通运算符：
   - 当栈顶运算符优先级高于或等于当前运算符时，弹出栈顶并输出。
   - 然后把当前运算符入栈。
5. 扫描结束后，把栈中剩余运算符全部弹出并输出。

### 例子一

```text
infix:   a + b * c - d
postfix: a b c * + d -
```

直觉是：

- `*` 优先级比 `+` 高，所以 `b c *` 先输出。
- 遇到 `-` 时，前面的 `+` 必须先输出。

### 例子二

```text
infix:   a * ( b + c ) / d
postfix: a b c + * d /
```

括号的作用是强制 `b + c` 先算。  
处理右括号时，要一直弹出到左括号为止。

## 9. 括号和结合性细节

课件里强调两个细节。

### 左括号的特殊处理

左括号 `(` 在栈外和栈内的优先级不同：

- 作为 incoming symbol 时，优先级很高，因为它必须入栈。
- 在栈中时，优先级很低，不能因为外面的运算符而被弹出。

所以：

```text
Never pop '(' except when processing ')'.
```

### 右结合运算符

普通的 `+`, `-`, `*`, `/` 通常是左结合。

例如：

```text
a - b - c
```

应当理解为：

```text
(a - b) - c
```

转换为：

```text
a b - c -
```

但指数运算 `^` 通常是右结合：

```text
2 ^ 2 ^ 3
```

应当理解为：

```text
2 ^ (2 ^ 3)
```

所以后缀表达式是：

```text
2 2 3 ^ ^
```

而不是：

```text
2 2 ^ 3 ^
```

## 10. 系统栈与函数调用

函数调用会使用系统栈。每次调用函数，系统会创建一个 stack frame，里面通常包含：

- 返回地址。
- 旧的 frame pointer。
- 局部变量。
- 参数和临时信息。

递归函数会不断压入新的 stack frame，所以递归太深可能导致栈溢出。

课件里的例子：

```c
void PrintList(List L) {
    if (L != NULL) {
        PrintElement(L->Element);
        PrintList(L->Next);
    }
}
```

如果链表有一百万个元素，这种递归写法可能导致系统栈不够。

可以改成迭代：

```c
void PrintList(List L) {
    while (L != NULL) {
        PrintElement(L->Element);
        L = L->Next;
    }
}
```

尾递归理论上可以被编译器优化成循环，但不要在 C 里默认依赖编译器一定帮你优化。

## 11. 队列 Queue

队列是一种 **First-In-First-Out** 的线性表，简称 **FIFO**。

也就是说：

```text
最先进入队列的元素，最先离开队列。
```

可以想成排队：

```text
Front -> A -> B -> C -> Rear
先出                    后进
```

队列在一端插入，在另一端删除：

- `Enqueue`：从 rear 入队。
- `Dequeue`：从 front 出队。

## 12. 队列 ADT

常见操作：

| 操作 | 含义 |
| --- | --- |
| `IsEmpty(Q)` | 判断队列是否为空 |
| `CreateQueue()` | 创建队列 |
| `DisposeQueue(Q)` | 销毁队列 |
| `MakeEmpty(Q)` | 清空队列 |
| `Enqueue(X, Q)` | 入队 |
| `Front(Q)` | 返回队首元素，但不删除 |
| `Dequeue(Q)` | 出队 |

和栈类似：

- 对空队列执行 `Dequeue` 或 `Front` 是 ADT 错误。
- 对满队列执行 `Enqueue` 是实现层面的错误。

## 13. 队列的数组实现

数组队列通常维护：

```c
struct QueueRecord {
    int Capacity;
    int Front;
    int Rear;
    int Size;
    ElementType *Array;
};
```

其中：

- `Front` 指向队首。
- `Rear` 指向队尾。
- `Size` 记录当前元素个数，可选但很有用。

如果直接让 `Rear` 一直向右移动，会很快走到数组末尾，即使前面已经有空位。  
因此数组队列通常做成 **循环队列**。

## 14. 循环队列

循环队列用取模让下标绕回数组开头：

```text
rear = (rear + 1) % Capacity
front = (front + 1) % Capacity
```

可视化：

```text
index:  0     1     2     3     4     5
       [ ]   [A]   [B]   [C]   [ ]   [ ]
             F           R

Enqueue D:
index:  0     1     2     3     4     5
       [ ]   [A]   [B]   [C]   [D]   [ ]
             F                 R

Dequeue A:
index:  0     1     2     3     4     5
       [ ]   [ ]   [B]   [C]   [D]   [ ]
                   F           R
```

当 `Rear` 到末尾后，再入队可以回到 `0`：

```text
index:  0     1     2     3     4     5
       [E]   [ ]   [B]   [C]   [D]   [F]
        R          F
```

## 15. 空队列和满队列的区分

循环队列最容易错的地方是：只看 `Front` 和 `Rear`，空和满可能长得一样。

常见解决方法有两种。

### 方法一：保留一个空位

规定队列最多只放 `Capacity - 1` 个元素。  
当：

```text
(Rear + 1) % Capacity == Front
```

就认为队列满。

优点是不用额外字段。  
缺点是浪费一个数组位置。

### 方法二：增加 Size 字段

维护当前元素个数 `Size`：

```text
Size == 0          空队列
Size == Capacity   满队列
```

课件里说：加一个 `Size` 字段可以避免浪费一个空位。

## 16. 循环队列代码模板

下面是带 `Size` 字段的写法。

```c
int IsEmpty(Queue Q) {
    return Q->Size == 0;
}

int IsFull(Queue Q) {
    return Q->Size == Q->Capacity;
}

void Enqueue(ElementType X, Queue Q) {
    if (IsFull(Q)) {
        Error("Full queue");
    } else {
        Q->Rear = (Q->Rear + 1) % Q->Capacity;
        Q->Array[Q->Rear] = X;
        Q->Size++;
    }
}

ElementType Front(Queue Q) {
    if (IsEmpty(Q)) {
        Error("Empty queue");
    }
    return Q->Array[Q->Front];
}

void Dequeue(Queue Q) {
    if (IsEmpty(Q)) {
        Error("Empty queue");
    } else {
        Q->Front = (Q->Front + 1) % Q->Capacity;
        Q->Size--;
    }
}
```

不同教材对 `Front` 和 `Rear` 的初始含义可能不同。  
写题时最重要的是：保证自己的定义前后一致。

## 17. 栈和队列对比

| 结构 | 规则 | 插入 | 删除 | 典型应用 |
| --- | --- | --- | --- | --- |
| 栈 | LIFO | 栈顶 | 栈顶 | 括号匹配、表达式求值、函数调用 |
| 队列 | FIFO | 队尾 | 队首 | BFS、任务调度、缓冲区 |

## 18. 考点速记

| 考点 | 必会内容 |
| --- | --- |
| 栈 | LIFO，只在 top 操作 |
| 空栈错误 | `Top` / `Pop` on empty stack 是 ADT 错误 |
| 数组栈 | `TopOfStack = -1` 表示空 |
| 链表栈 | header 后第一个结点是栈顶 |
| 括号匹配 | 左括号入栈，右括号匹配栈顶 |
| 后缀表达式 | 操作数入栈，运算符弹两个数计算 |
| infix 转 postfix | 操作数直接输出，运算符借助栈 |
| 左括号 | 只有处理右括号时才弹出 |
| `^` | 通常右结合，`2^2^3` 转 `2 2 3 ^ ^` |
| 系统栈 | 递归太深可能栈溢出 |
| 队列 | FIFO，rear 入队，front 出队 |
| 循环队列 | 用 `% Capacity` 让下标回绕 |
| 判满 | 可浪费一个空位，或增加 `Size` 字段 |

## 19. 易错点

- 后缀表达式遇到运算符时，先弹出的是右操作数。
- 中缀转后缀时，操作数顺序不变，变的是运算符位置。
- 左括号在栈内不能随便被弹出。
- `a - b - c` 是左结合，应转成 `a b - c -`。
- `2 ^ 2 ^ 3` 是右结合，应转成 `2 2 3 ^ ^`。
- 循环队列中 `Front == Rear` 不一定能区分空和满，要看具体约定。
- 带 `Size` 字段的循环队列更直观，不必浪费一个位置。
