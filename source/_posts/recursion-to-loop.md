---
title: 递归转换为迭代的通用技术
date: 2020-10-07 11:54:55
mathjax: true
tags:
- C++
- Recursion
- Iteration
---

递归 _(Recursion)_ 和迭代 _(Iteration)_ 是等价的, 任何的递归算法都可以转换成循环, 任何迭代都可以转换成递归. 如果说到将递归转换成迭代的技术, 两种常见的方法是尾递归和模拟运行时栈.

<!-- more -->

## 尾递归 _(Tail recursive)_

一个函数最后执行的操作是递归调用自身, 这种调用称为尾递归调用. 尾递归消除技术可以对尾递归的代码优化将其转换成生成迭代.

一个阶乘 _(factorial)_ 的例子:
$$
n! = \begin{cases}
1 & n = 0\\
n\cdot (n-1)! & n > 0
\end{cases}
$$
最直观的 C++ 代码如下, 为了方便, 称为版本 1 (v1).

```c++
template <typename T>
T factorial_v1(T n) {
    if (n == 0) {
        return 1;
    }
    return n * factorial_v1(n - 1);
}
```

注意 `factorial_v1` 虽然末尾进行了递归, 但不是尾递归, 因为它最后执行的操作是乘法而不是递归调用.

对于 `factorial_v1(6)` 的递归过程如下:

```scheme
factorial_v1(6)
(6 * factorial_v1(5))
(6 * (5 * factorial_v1(4)))
(6 * (5 * (4 * factorial_v1(3))))
(6 * (5 * (4 * (3 * factorial_v1(2)))))
(6 * (5 * (4 * (3 * (2 * factorial_v1(1))))))
(6 * (5 * (4 * (3 * (2 * (1 * factorial_v1(0)))))))
(6 * (5 * (4 * (3 * (2 * (1 * 1))))))
(6 * (5 * (4 * (3 * (2 * 1)))))
(6 * (5 * (4 * (3 * 2))))
(6 * (5 * (4 * 6)))
(6 * (5 * 24))
(6 * 120)
720
```

这个递归过程先扩张后收缩. 扩张发生在构建延迟求值操作链的过程中, 随着延迟求值操作链越来越长, 运行时栈保存的状态也越来越多. 当延迟求值操作链达到最长时, 扩张结束. 然后发生收缩, 收缩时, 实际求值延迟求值的操作链, 这时用到了运行时栈保存的状态, 随着不断的求值, 运行时栈需要保存的状态减少, 求值操作链也越来越短, 最后得到结果. 延迟操作链随着 $n$ 的增长而线性增长, 这称为线性递归.

所以 `factorial_v1` 所需时间是 $O(n)$, 递归保存状态所需空间是 $O(n)$. 但是理想的迭代应该使用 $O(1)$ 的空间.

为了获得尾递归的版本, 代码修改如下:

```c++
template <typename T>
T factorial_impl(T n, T product) {
    if (n == 0) {
        return product;
    }
    return factorial_impl(n - 1, n * product);
}

template <typename T>
T factorial(T n) {
    return factorial_impl(n, static_cast<T>(1));
}
```

现在的 `factorial_impl` 接收两个参数, 将最后本该需要的乘法转换到了参数上, 且是尾递归. 这很容易被编译器优化成等价的尾部 goto:

```c++
template <typename T>
T factorial_impl_optimized(T n, T product) {
start:
    if (n == 0) {
        return product;
    }
    product = n * product;
    n = n - 1;
    goto start;
}
```

显然这就变成了迭代版本. 因此虽然表面上 `factorial_impl` 的代码使用了递归, 但是它实际上会被优化成迭代. 如果你对循环有特殊偏好, 也可以很容易的手动把 `factorial_imple_optimized` 转换成一个循环结构.

```c++
template <typename T>
T factorial_loop(T n) {
    T product = 1;
    while (n > 0) {
        product = n * product;
        --n;
    }
    return product;
}
```

总之, 尾递归的代码满足如下的模式 (伪代码)

```c++
T func(A arg1, B arg2, ...) {
    if (condition) {
        return x;
    }
    // code block 1
    return func(transform(arg1), transform(arg2), ...);
}
```

它可以被转换成以下循环结构:

```c++
T func(A arg1, B arg2, ...) {
    while (not condition) {
        // code block 1
        arg1 = transform(arg1);
        arg2 = transform(arg2);
    }
    return x;
}
```

尾递归的思想很简单直观. 不过模拟运行时栈要更难一些.

## 模拟运行时栈

普通的递归实际上利用了系统的运行时栈. 函数 fa 调用 fb 是将 fa 函数的帧 _(frame)_ 压入栈中并运行 fb, 函数 fb 返回到 fa 是将栈中的 fa 的栈帧推出. 因此, 一个直观的想法是使用一个自定义的栈模拟这个运行时栈, 这样就可以直接运行递归过程.

新的自定义的栈中的元素应该是函数栈帧, 但是为了方便和效率, 并不会真正完整的模拟运行时栈的行为, 即每次保存整个函数的栈帧, 只要对每个函数栈帧保存一些必要的信息即可.

一个完整的函数栈帧包括:

- 参数. 这个函数从其调用者那里接收到的函数参数.
- 返回地址. 这个函数调用的函数返回时, 用于指示返回地址.
- 局部变量. 这个函数调用其他函数时, 已存在的局部变量.

其中, 返回地址 (或类似的概念) 是必须的, 否则无法返回到其调用者. 但是在我们的例子中, 栈中的元素是栈帧, 栈以栈帧为单位进行操作, 并不担心无法返回到调用者的问题. 但是还是需要类似的概念, 例如对于下面的代码:

```c++
T func() {
    // code block 1
    func();
    // code block 2
    func();
    // code block 3
    func();
    // code block 4
}
```

它一共递归调用了三次, 如果没有返回地址, 返回后不知道应该执行代码块 2 还是 3 还是 4. 显然, 可以使用数字编号表示状态, 这样就代替了返回地址的功能.

参数和局部变量可以视情况判断是否保存.

## 迭代版本的二叉树遍历

总之, 已经得到了大致的思路, 下面对二叉树的前序遍历算法应用这个技术. 树的定义如下:

```c++
class Tree {
    struct TreeNode {
        int val = 0;
        TreeNode *left = nullptr;
        TreeNode *right = nullptr;
    };

    TreeNode *m_root;
};
```

为了方便, 结点内的元素设定为 `int`.

递归的版本:

```c++
void preorder_recur(TreeNode *root, vector<int> &vi) {
    // code block 0
    if (root != nullptr) {
        vi.push_back(root->val);
        // code block 0
        preorder_recur(root->left, vi);
        // code block 1
        preorder_recur(root->right, vi);
        // code block 2
    }
}

vector<int> preorder_traversal(TreeNode *root) {
    vector<int> ret;
    preorder_recur(root, ret);
    return ret;
}
```

`preorder_recur` 中每个栈帧需要保存的状态是 root 和代码块编号. 因此栈帧声明如下:

```c++
struct Frame {
    TreeNode *node;
    int codeblock;
};
```

完整的代码如下:

```c++
vector<int> preorder_iter(TreeNode *root) {
    struct Frame {
        TreeNode *node = nullptr;
        int codeblock = 0;
    };

    stack<Frame> runtime_stack;
    runtime_stack.push({root});

    vector<int> ret;
    while (!runtime_stack.empty()) {
        Frame &now_frame = runtime_stack.top();
        switch (now_frame.codeblock) {
            case 0:
                if (now_frame.node == nullptr) {
                    runtime_stack.pop();
                } else {
                    ret.push_back(now_frame.node->val);
                    runtime_stack.push({now_frame.node->left});
                    ++now_frame.codeblock;
                }
                break;
            case 1:
                runtime_stack.push({now_frame.node->right});
                ++now_frame.codeblock;
                break;
            case 2:
                runtime_stack.pop();
                break;
        }
    }
    return ret;
}
```

经过测试, 现在它可以正常工作了! 不过这么多的 `switch` 和 `case` 还是臃肿了点. 为了优化代码, 使用一个优化准则: 两个或多个连续的递归调用可以以相反的顺序一次性入栈. 这样就得到了第一个优化版本的代码:

```c++
vector<int> preorder_iter_v1(TreeNode *root) {
    struct Frame {
        TreeNode *node = nullptr;
        int codeblock = 0;
    };

    stack<Frame> runtime_stack;
    runtime_stack.push({root});

    vector<int> ret;
    while (!runtime_stack.empty()) {
        Frame &now_frame = runtime_stack.top();
        switch (now_frame.codeblock) {
            case 0:
                if (now_frame.node == nullptr) {
                    runtime_stack.pop();
                } else {
                    ret.push_back(now_frame.node->val);
                    runtime_stack.push({now_frame.node->right});
                    runtime_stack.push({now_frame.node->left});
                    ++now_frame.codeblock;
                }
                break;
            case 1:
                runtime_stack.pop();
                break;
        }
    }
    return ret;
}
```

注意到只要 `now_frame.node != nullptr` 且 `now_frame.codeblock == 0`, 就会 `push` 两个子节点, 否则从栈中取出 `now_frame.node`. 代码优化如下:

```c++
vector<int> preorder_iter_v2(TreeNode *root) {
    struct Frame {
        TreeNode *node = nullptr;
        int codeblock = 0;
    };

    stack<Frame> runtime_stack;
    runtime_stack.push({root});

    vector<int> ret;
    while (!runtime_stack.empty()) {
        Frame &now_frame = runtime_stack.top();
        if (now_frame.node != nullptr && now_frame.codeblock == 0) {
            ret.push_back(now_frame.node->val);
            runtime_stack.push({now_frame.node->right});
            runtime_stack.push({now_frame.node->left});
            ++now_frame.codeblock;
        } else {
            runtime_stack.pop();
        }
    }
    return ret;
}
```

这样每个帧就剩下了两个状态, 一个状态是初始状态, 初始状态进入 `if` 分支并执行一些操作. 另一个状态是终止状态, 终止状态在遍历完该结点的所有子节点后, 此时将该结点也从栈中弹出. 如下:

```c++
void preorder_recur(TreeNode *root, vector<int> &vi) {
    // Initial state
    if (root != nullptr) {
        vi.push_back(root->val);
        preorder_recur(root->left, vi);
        preorder_recur(root->right, vi);
    }
    // Pop root
}
```

但是其实弹出 `root` 的过程不一定得在遍历完它的所有子节点之后, 可以将它放在开头:

```c++
void preorder_recur(TreeNode *root, vector<int> &vi) {
    // Initial state
    now_frame.node = root;
    // Pop root
    if (now_frame.node != nullptr) {
        vi.push_back(now_frame.node->val);
        preorder_recur(now_frame.node->left, vi);
        preorder_recur(now_frame.node->right, vi);
    }
}
```

这样终止状态也不是必须的了. 因为只有一个状态, 表示状态的 `Frame.codeblock` 也不是必须的了. 优化过的代码:

```c++
vector<int> preorder_loop(TreeNode *root) {
    stack<TreeNode *> runtime_stack;
    runtime_stack.push(root);

    vector<int> ret;
    while (!runtime_stack.empty()) {
        TreeNode *now_frame = runtime_stack.top();
        runtime_stack.pop();
        if (now_frame != nullptr) {
            ret.push_back(now_frame->val);
            runtime_stack.push({now_frame->right});
            runtime_stack.push({now_frame->left});
        }
    }
    return ret;
}
```

前序遍历的循环版本就这样完成了.

类似的, 有中序遍历:

```c++
vector<int> inorder_iter_v1(TreeNode *root) {
    struct Frame {
        TreeNode *node = nullptr;
        bool left_has_traveled = false;
    };

    stack<Frame> runtime_stack;
    runtime_stack.push({root});

    vector<int> ret;
    while (!runtime_stack.empty()) {
        Frame &now_frame = runtime_stack.top();
        if (!now_frame.left_has_traveled) {
            if (now_frame.node == nullptr) {
                runtime_stack.pop();
            } else {
                runtime_stack.push({now_frame.node->left});
                now_frame.left_has_traveled = true;
            }
        } else {
            TreeNode *tmp = now_frame.node;
            runtime_stack.pop();
            ret.push_back(tmp->val);
            runtime_stack.push({tmp->right});
        }
    }
    return ret;
}
```

似乎很难再优化了.

注意到当栈收缩时, 栈中所有元素的 `left_has_traveled` 都变成 `true`. 即当栈收缩时, 将结点 `pop` 的同时 `push` 右边结点. 可以进一步优化如下.

```c++
vector<int> inorder_iter_v2(TreeNode *root) {
    stack<TreeNode *> runtime_stack;
    runtime_stack.push(root);

    vector<int> ret;
    while (!runtime_stack.empty()) {
        TreeNode *now_frame = runtime_stack.top();
        if (now_frame != nullptr) {
            runtime_stack.push(now_frame->left);
        } else {
            runtime_stack.pop();
            if (!runtime_stack.empty()) {
                TreeNode *tmp = runtime_stack.top();
                ret.push_back(tmp->val);
                runtime_stack.pop();
                runtime_stack.push(tmp->right);
            }
        }
    }
    return ret;
}
```

由于个人能力有限, 无法进一步优化了. 理想的中序遍历的循环版本代码应该如下:

```c++
vector<int> inorder_loop(TreeNode *root) {
    stack<TreeNode *> runtime_stack;

    TreeNode *probe = root;
    vector<int> ret;
    while (true) {
        if (probe != nullptr) {
            runtime_stack.push(probe);
            probe = probe->left;
        } else if (!runtime_stack.empty()) {
            probe = runtime_stack.top();
            runtime_stack.pop();
            ret.push_back(probe->val);
            probe = probe->right;
        } else {
            break;
        }
    }
    return ret;
}
```

后序遍历:

```c++
vector<int> postorder_iter(TreeNode *root) {
    struct Frame {
        TreeNode *node = nullptr;
        bool codeblock = false;
    };

    stack<Frame> runtime_stack;
    runtime_stack.push({root});

    vector<int> ret;
    while (!runtime_stack.empty()) {
        Frame &now_frame = runtime_stack.top();
        if (!now_frame.codeblock) {
            if (now_frame.node == nullptr) {
                runtime_stack.pop();
            } else {
                runtime_stack.push({now_frame.node->right});
                runtime_stack.push({now_frame.node->left});
                now_frame.codeblock = true;
            }
        } else {
            ret.push_back(now_frame.node->val);
            runtime_stack.pop();
        }
    }
    return ret;
}
```

因为个人能力有限, 不会进一步优化了. 一个理想的简洁的后序遍历的循环版本如下:

```c++
vector<int> postorder_iter(TreeNode *root) {
    stack<TreeNode *> runtime_stack;
    runtime_stack.push(root);

    stack<int> ret_stack;
    while (!runtime_stack.empty()) {
        TreeNode *now_frame = runtime_stack.top();
        runtime_stack.pop();
        if (now_frame != nullptr) {
            ret_stack.push(now_frame->val);
            runtime_stack.push({now_frame->left});
            runtime_stack.push({now_frame->right});
        }
    }
    vector<int> ret;
    while (!ret_stack.empty()) {
        ret.push_back(ret_stack.top());
        ret_stack.pop();
    }
    return ret;
}
```

总之, 模式为

```c++
T func(args) {
    // code block 1
    func(transform1(args));
    // code block 2
    func(transform2(args));
    // code block 3
    func(transform3(args));
    // code block 4
    return x;
}
```

的递归可以优化为

```c++
struct Frame {
    // ...
    int status;
};

T func_loop() {
    stack<Frame> runtime_stack;
    runtime_stack.push(Frame(args));
    while (!stack.empty()) {
        auto now_Frame = stack.top();
        switch () {
            case 0:
                // code block 1
                runtime_stack.push(Frame(transform1(args)));
                ++status;
                break;
            case 1:
                // code block 2
                runtime_stack.push(Frame(transform2(args)));
                ++status;
                break;
            // ...
            default:
                throw runtime_error();
        }
    }
    return x;
}
```

这样, 就再也不担心将递归转化成循环的问题了. 当然, 这只是将原有的递归代码以循环的方式运行而已, 如果追求更高的优化, 需要仔细的深入研究, 调整代码. 例如二叉树的中序和后序遍历都是比较难优化的.

## 迭代版本的快速排序

快速排序是一个常见算法. 下面是一个递归版本.

```c++
template <typename Container>
size_t partition(Container &arr, size_t low, size_t high) {
    size_t pivot = low;
    size_t end = high - 1;
    for (size_t j = low; j < end; ++j) {
        if (arr[j] < arr[end]) {
            swap(arr[pivot], arr[j]);
            ++pivot;
        }
    }
    swap(arr[pivot], arr[end]);
    return pivot;
}

template <typename Container>
void quick_sort(Container &arr, size_t low, size_t high) {
    if (low < high) {
        size_t pivot = partition(arr, low, high);
        quick_sort(arr, low, pivot);
        quick_sort(arr, pivot + 1, high);
    }
}

template <typename Container>
void quick_sort(Container &arr) {
    return quick_sort(arr, 0, arr.size());
}
```

观察到其模式类似于二叉树的前序遍历, 直接把二叉树的前序遍历模板套上即可, 或者可以自己再从初始的 `switch` 和 `case` 版本开始优化.

```c++
template <typename Container>
size_t partition(Container &arr, size_t low, size_t high) {
    size_t pivot = low;
    size_t end = high - 1;
    for (size_t j = low; j < end; ++j) {
        if (arr[j] < arr[end]) {
            swap(arr[pivot], arr[j]);
            ++pivot;
        }
    }
    swap(arr[pivot], arr[end]);
    return pivot;
}

template <typename Container>
void quick_sort(Container &arr) {
    using Frame = pair<size_t, size_t>;
    stack<Frame> runtime_stack;
    runtime_stack.push({0, arr.size()});
    while (!runtime_stack.empty()) {
        auto now_frame = runtime_stack.top();
        runtime_stack.pop();
        auto low = now_frame.first;
        auto high = now_frame.second;
        if (low < high) {
            auto pivot = partition(arr, low, high);
            runtime_stack.push({pivot + 1, high});
            runtime_stack.push({low, pivot});
        }
    }
}
```

## DFS 的迭代版本

不如试试迭代版本的 DFS.

```c++
void dfs(const vector<vector<size_t>> &graph, deque<bool> &visited, size_t node,
         vector<size_t> &ret) {
    const auto &vt = graph[node];
    visited[node] = true;
    ret.push_back(node);
    for (size_t t : vt) {
        if (!visited[t]) {
            dfs(graph, visited, t, ret);
        }
    }
}

vector<size_t> dfs_entry_point(const vector<vector<size_t>> &graph,
                               size_t start_node) {
    deque<bool> visited(graph.size(), false);
    vector<size_t> ret;
    dfs(graph, visited, start_node, ret);
    return ret;
}
```

迭代版本:

```c++
vector<size_t> dfs_entry_point_iter(vector<vector<size_t>> &graph,
                                    size_t start_node) {
    struct Frame {
        size_t neighbor = 0;
        size_t end_neighbor = 0;
        size_t node;
    };

    vector<size_t> ret;
    ret.push_back(start_node);
    deque<bool> visited(graph.size(), false);
    stack<Frame> runtime_stack;
    runtime_stack.push({0, graph[start_node].size(), start_node});
    while (!runtime_stack.empty()) {
        Frame &now_frame = runtime_stack.top();
        if (now_frame.neighbor == now_frame.end_neighbor) {
            runtime_stack.pop();
        } else {
            size_t t = graph[now_frame.node][now_frame.neighbor];
            if (!visited[t]) {
                visited[t] = true;
                ret.push_back(t);
                runtime_stack.push({0, graph[t].size(), t});
            }
            ++now_frame.neighbor;
        }
    }
    return ret;
}
```

## 总结

有的算法用递归表达很自然和优雅, 有的算法适合用循环写. 转换为另一种形式会破坏这种和谐, 使代码变得更复杂. 另外, 迭代通常相对于递归有着性能优势, 但并不总是如此.

## References

- [Converting Recursion to Iteration](https://www.cs.odu.edu/~zeil/cs361/latest/Public/recursionConversion/index.html#conversion-example-quicksort)
- [Way to go from recursion to iteration - Stack Overflow](https://stackoverflow.com/questions/159590/way-to-go-from-recursion-to-iteration/8512072#8512072)
- [Postorder Tree Traversal | Iterative & Recursive - Techie Delight](https://www.techiedelight.com/postorder-tree-traversal-iterative-recursive/)