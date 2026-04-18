# 编程错题本



## C 语言并查集易错点

### 1. 语法错

#### `printf` 要用双引号

- 错：`printf('yes');`
- 对：`printf("yes\n");`
- 记法：单引号是字符，双引号才是字符串

#### `union` 不能当函数名

- 错：`void union(int S[], int a, int b)`
- 对：`void UnionSet(int S[], int a, int b)`
- 记法：`union` 是 C 语言关键字

#### 数组传参不要乱写 `[]`

- 错：
  - `void Initial(S[], int n)`
  - `Initial(S[], n)`
- 对：
  - `void Initial(int S[], int n)`
  - `Initial(S, n)`
- 记法：定义时要写类型，调用时只写数组名

### 2. 输入输出错

#### `scanf("%c")` 容易读到回车

- 错：`scanf("%c", &op)`
- 对：`scanf(" %c", &op)`
- 记法：`%c` 前面加空格，先吃掉空白字符

#### `scanf` 的返回值不是读到的内容

- 错：`while (scanf(" %c", &op) != 'S')`
- 对：`while (scanf(" %c", &op) != EOF && op != 'S')`
- 记法：
  - `scanf(...)` 比较的是有没有读成功
  - `op` 比较的才是读到了什么

### 3. 并查集逻辑错

#### `Find` 判断条件别写错

- 错：`for (root = x; root >= 0; root = S[root])`
- 也不稳：`S[root] > 0`
- 对：`for (root = x; S[root] >= 0; root = S[root]);`
- 记法：
  - `S[i] < 0`：`i` 是根
  - `S[i] >= 0`：`i` 还有父结点
  - 如果下标从 `0` 开始，写 `> 0` 会漏掉父结点刚好是 `0` 的情况

#### 合并时比的是集合大小，不是根编号

- 错：`if (root1 > root2) ...`
- 对：`if (S[root1] > S[root2]) ...`
- 记法：
  - 比较的是 `S[root]`
  - 根里存的是负数
  - 数值越大，集合越小

#### 合并前一定先找根

- 错：`UnionSet(S, S[b], S[c]);`
- 对：`UnionSet(S, Find(S, b), Find(S, c));`
- 记法：并查集的 `Union` 永远合并两个根

