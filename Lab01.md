<link rel="stylesheet" type="text/css" href="http://zlyd.iccnconn.com/markdowncss/stylelib/typora-forest-theme-0.1.6/forest.css">

<link rel="stylesheet" type="text/css" href="http://zlyd.iccnconn.com/markdowncss/stylelib/typora-forest-theme-0.1.6/forest.css">

# Lab01

> ### 题目描述
> 
> - 有 $n$ 个人排成一圈，按顺时针方向依次编号 $0,1,2,...,n-1$ 。从编号为 $0$ 的人开始顺时针“一二”报数，报到 $2$ 的人退出圈子。这样不断循环下去，圈子里的人将不断减少，最终一定会剩下一个人。编写一段程序，输入 $n$ ，输出最后剩下的人的编号。
> 
> - 例如，如果有 $5$ 个人，那么依次是编号为 $1,3,0,4$ 的退出，最后剩下编号为 $2 $ 的。

### 源代码

```c
#include <stdio.h>
#include <stdlib.h>

/**
 * @brief Calculate the solution of josephus problem by the flow chart mentioned in the lecture
 *
 * @param n : the number of people
 * @return int
 */

// 定义链表节点结构
typedef struct Node {
    int data;
    struct Node* next;
} Node;

// 创建新节点
Node* createNode(int value) {
    Node* newNode = (Node*)malloc(sizeof(Node));
    newNode->data = value;
    newNode->next = NULL;
    return newNode;
}

// 创建循环链表
Node* createCircularLinkedList(int n) {
    if (n <= 0) return NULL;

    Node* head = createNode(0); // 创建头节点，值为0
    Node* current = head;

    for (int i = 1; i <= n-1; i++) {
        current->next = createNode(i); // 创建新节点并连接到当前节点后面
        current = current->next; // 移动到新创建的节点
    }

    current->next = head; // 将最后一个节点的next指针指向头节点，形成循环

    return head;
}

// 第一个解决方案
int josephus_solution1(int n) {
    int count = 2, pre_winner = 0;

    while (count <= n){
        pre_winner = (pre_winner + 2) % count;
        count += 1;
    }

    return pre_winner;
}

// 第二个解决方案
int josephus_solution2(int n) {
    int i, j, count = n;
    int people[n];

    for (i = 0; i < n; i++) {
        people[i] = 1; // 初始化数组，表示所有人都参与游戏
    }

    for (i = 0; i < n; i++) {
        people[i] = 1; // 初始化数组，表示所有人都参与游戏
    }

    for (i = 0; count > 1; i = (i + 1) % n) {
        if (people[i]) {
            i = (i + 1) % n;
            while (!people[i]) {
                i = (i + 1) % n;
            }
            people[i] = 0; // 将被移除的人标记为0
            count--;
        }
    }

    for (i = 0; i < n; i++) {
        if (people[i]) {
            return i; // 返回位置
        }
    }
}

// 第三个解决方案
int josephus_solution3(Node* head) {
    Node* current = head;
    Node* previous = NULL;

    while (current->next != current) {
        for (int i = 0; i < 1; i++) {
            previous = current;
            current = current->next;
        }

        // 移除当前节点
        previous->next = current->next;

        // 释放当前节点的内存
        free(current);

        // 更新当前节点为下一个节点
        current = previous->next;
    }

    // 返回最后一个幸存者的值
    return current->data;
}

int main(int argc, char** argv) {
    int n;

    printf("The number of people is: ");
    scanf("%d", &n);

    printf("The last person's number is (Solution 1): %d\n", josephus_solution1(n));
    printf("The last person's number is (Solution 2): %d\n", josephus_solution2(n));

    Node* head = createCircularLinkedList(n);
    printf("The last person's number is (Solution 3): %d\n", josephus_solution3(head));

    return 0;
}
```

### 算法解释

#### 第一个解决方案

```c
int josephus_solution1(int n) {
    int count = 2, pre_winner = 0;

    while (count <= n){
        pre_winner = (pre_winner + 2) % count;
        count += 1;
    }

    return pre_winner;
}
```

        设 $f(n)$ 为有 $n$ 个人时最后剩下的人的编号，由算法可推导出 $f$ 的递推关系式：

$$
f(n)=\left\{
\begin{matrix}
0 \;,\;if\;n=1\\
[f(n-1) + 2]\;mod\;n\;,\;if\;n>1
\end{matrix}
\right.
$$

        我们使用数学归纳法证明此递归关系式：

- 当 $n=1$ 时，递推显然成立。

- 假设当 $n=k$ 时，递推成立。
  
  - 先考虑有 $k$ 个人的情况下，所有人的编号依次为 $0,1,2,3,...,f(k),...,k-1$ ，最后剩下的人的编号为 $f(k)$ 。
  
  - $[f(k)+2]$ 的含义是将编号为 $f(k)$ 向右移动两个位置，此人获得新的编号 $f'(k) = f(k) + 2$ 。此时，$k$ 个人都相应被赋予了新的编号，如 $2,3,4,5,...,f'(k),...,k+1$ ，但是其他人的编号不重要，只需要关注那个最后剩下来的人的新的编号 $f'(k)$ 便可。
  
  - 假设 $f'(k)$ 是一个小于等于 $k$ 的数，那么不会出现问题。在整个队列前面加上两个人，分别标号 $0,1$ 。做这个操作的原因是，当 $n = k+1$ 时，第一个退出的一定是编号为 $1$ 的人。这时便可以转换为 $n=k$ 的情况，编号为 $2$ 的人开始报数。
  
  - 假设 $f'(k) = k+1$ ，那么这个人的编号超过了 $k+1$ 个人的情况下的编号范围。超出范围的处理方式便是循环到第一个编号 $0$ ， $mod\;(k+1)$ 可以实现这个操作。

- 有此可得，递推关系式成立。

#### 第二个解决方案

```c
for (i = 0; count > 1; i = (i + 1) % n) {
        if (people[i]) {
            i = (i + 1) % n;
            while (!people[i]) {
                i = (i + 1) % n;
            }
            people[i] = 0; // 将被移除的人标记为0
            count--;
        }
    }
```

        这段代码是第二个解决方案里的核心代码。它含义是：

- 当场上人数大于 $1$ 个时，执行 `for` 循环。每次循环结束， $i$ 的值会增加 $1$ 。

- `if` 条件判断，如果编号为 $i$ 的人还在场上，那么使 $i$ 的值增加 $1$ 。

- `while` 循环作用为，如果这个位置上没人，那么继续往后遍历直到有人的位置。

- 使得那个位置的人下场，得到 `people[i] = 0` 。

- 场上的人数减少 $1$ 。

#### 第三个解决方案

```c
typedef struct Node {
    int data;
    struct Node* next;
} Node;
```

        创建一个名为 `Node` 的数据类型，该类型包含两个成员变量：

- `int data`: 这是一个整数类型的变量，用于存储节点的值。

- `struct Node* next`: 这是一个指向结构体 `Node` 的指针，用于指向下一个节点。

- 最后一行 `Node;` 将 `Node` 作为该结构体类型的别名，以便在代码中更方便地使用它。

```c
Node* createNode(int value) {
    Node* newNode = (Node*)malloc(sizeof(Node));
    newNode->data = value;
    newNode->next = NULL;
    return newNode;
```

        这段代码定义了一个名为 `createNode` 的函数，其作用是创建一个新的节点并返回指向该节点的指针。

- `Node* createNode(int value) {`：这是函数的定义，它接受一个整数参数 `value` 作为节点的值，并返回一个指向 `Node` 类型的指针。

- `Node* newNode = (Node*)malloc(sizeof(Node));`：这一行使用 `malloc` 函数动态分配了足够的内存空间来存储一个 `Node` 结构体的大小。`malloc` 返回的是 `void*` 类型的指针，所以需要将其强制类型转换为 `Node*` 类型。

- `newNode->data = value;`：`->` 是一个C/C++中用于访问结构体或类的成员的运算符。它用于指向一个结构体或类的指针，并访问该结构体或类的成员。这一行将传入的参数 `value` 赋值给新节点的 `data` 成员，表示新节点的数据为传入的值。

- `newNode->next = NULL;`：`NULL` 是一个在C和C++中经常使用的特殊值，它表示一个空指针或者空地址，这里表示空地址。通常用来指示一个指针没有指向任何有效的内存地址。这一行将新节点的 `next` 指针初始化为 `NULL`，表示新节点目前没有下一个节点，这在创建节点时是一个常见的初始状态。

- `return newNode;`：最后，函数返回指向新节点的指针，以便在调用时可以将新节点连接到链表中。
  
  ```c
  Node* createCircularLinkedList(int n) {
      if (n <= 0) return NULL;
  
      Node* head = createNode(0); // 创建头节点，值为0
      Node* current = head;
  
      for (int i = 1; i <= n-1; i++) {
          current->next = createNode(i); // 创建新节点并连接到当前节点后面
          current = current->next; // 移动到新创建的节点
      }
  
      current->next = head; // 将最后一个节点的next指针指向头节点，形成循环
  
      return head;
  }
  ```

         这段代码定义了一个名为 `createCircularLinkedList` 的函数，它接受一个整数参数 `n` 作为节点的个数，其作用是创建一个新的节点并返回指向该节点的指针。

```c
int josephus_solution3(Node* head) {
    Node* current = head;
    Node* previous = NULL;

    while (current->next != current) {
        for (int i = 0; i < 1; i++) {
            previous = current;
            current = current->next;
        }

        // 移除当前节点
        previous->next = current->next;

        // 释放当前节点的内存
        free(current);

        // 更新当前节点为下一个节点
        current = previous->next;
    }

    // 返回最后一个幸存者的值
    return current->data;
}
```

- `while` 循环的含义是，只要链表中还有超过一个节点，就会继续执行。循环的条件是当前节点的下一个节点不等于当前节点，表示链表中还有多于一个节点。
  
  - 在每次循环中，通过一个 `for` 循环将 `previous` 设置为当前节点，将 `current` 移动到下一个节点。
  
  - `previous->next = current->next;` 将前一个节点的 `next` 指针设置为当前节点的下一个节点，从而绕过了当前节点。
  
  - `free(current);` 释放了当前节点的内存，将其从链表中移除。
  
  - `current = previous->next;` 更新了当前节点为下一个节点，以继续游戏。

- 最后，返回最后一个幸存者节点的数据值。

### 结果统计

| $n$  | Result | Screenshot                                                       |
|:----:|:------:|:----------------------------------------------------------------:|
| $10$ | $4$    | ![](/Users/agathe_qian_file/Desktop/截屏2023-09-20%2020.08.58.png) |
| $11$ | $6$    | ![](/Users/agathe_qian_file/Desktop/截屏2023-09-20%2020.09.19.png) |
| $12$ | $8$    | ![](/Users/agathe_qian_file/Desktop/截屏2023-09-20%2020.09.33.png) |
| $13$ | $10$   | ![](/Users/agathe_qian_file/Desktop/截屏2023-09-20%2020.09.47.png) |
| $14$ | $12$   | ![](/Users/agathe_qian_file/Desktop/截屏2023-09-20%2020.10.02.png) |
| $15$ | $14$   | ![](/Users/agathe_qian_file/Desktop/截屏2023-09-20%2020.10.14.png) |
| $16$ | $0$    | ![](/Users/agathe_qian_file/Desktop/截屏2023-09-20%2020.10.27.png) |
| $17$ | $2$    | ![](/Users/agathe_qian_file/Desktop/截屏2023-09-20%2020.10.40.png) |
| $18$ | $4$    | ![](/Users/agathe_qian_file/Desktop/截屏2023-09-20%2020.10.51.png) |
| $19$ | $6$    | ![](/Users/agathe_qian_file/Desktop/截屏2023-09-20%2020.11.03.png) |
| $20$ | $8$    | ![](/Users/agathe_qian_file/Desktop/截屏2023-09-20%2020.11.15.png) |
