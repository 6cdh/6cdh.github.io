---
title: 二叉树的常数空间遍历
date: 2020-11-26 16:46:42
updated: 2020-11-27
tags:
  - tree
---

如果你不了解 Morris 遍历, 不使用栈(或者说额外空间)遍历二叉树, 你可能认为是不可能的. 为什么这么说? 因为在遍历时我们需要空间保存父结点或相关的信息, 如果不保存, 遍历时, 就只能下降而不能上升:

```plaintext
             +---+
       +-----| 0 |-----+
       |     +---+     |
       |               |
     +---+           +---+
  +--| 1 |--+     +--| 2 |--+
  |  +---+  |     |  +---+  |
  |         |     |         |
+---+     +---+ +---+     +---+
| 3 |     | 4 | | 5 |     | 6 |
+---+     +---+ +---+     +---+
```

如图, 假设我们已经访问了结点 0 -> 1 -> 3, 因为没有保存父节点的信息, 没有办法返回到结点 1 然后继续访问结点 4, 更别说回到结点 0 并访问结点 2, 5, 6 了. 理论上, 我们需要 O(h) 空间来保存我们需要的信息, 这里 h 是树的高度. 当然, 如果每个结点内还保存了一个指向其父结点的指针或者这个树是一个单链表, 我们也可以不使用额外的空间, 但是这里谈论的并不是特例, 而是适用于所有二叉树的方法.

Morris 可以不使用额外的空间而遍历二叉树. 它是如何做到的呢? 显然, 它也一定需要空间来保存信息, 这些空间不来自于堆或者栈, 而是来自二叉树本身 (当然, 二叉树本身或许也是在堆或栈上的).

<!--more-->

二叉树中有哪些空间被浪费了呢? 显然是空节点或空 (null) 字段. 这些字段在二叉树中大量重复, 可以说是一种浪费. 对于 n 个结点的二叉树, 空的字段一共有 n+1 个 (n 个结点共有 2n 个字段, 其中 n-1 个字段指向正常结点, 剩下 n+1 个字段表示空), 这足以满足 O(h) 的需求了. 使用这些空的字段, 便是 Morris 遍历的基本思想. 此外, 还有一些细节问题, 例如, 我们使用了这些字段, 破坏了树的结构, 还能有容易的方法来遍历它吗? 遍历后又怎么恢复空字段呢? 这些细节在之后描述.

Morris 使用了现成的利用空字段的方法: 线索二叉树 _(threaded binary tree)_. 如果你不了解线索二叉树, 那么它的定义如下:

> A binary tree is threaded by making all right child pointers that would normally be null point to the in-order successor of the node (if it exists), and all left child pointers that would normally be null point to the in-order predecessor of the node.

即所有没有右子结点的结点中保存右子节点的字段 (即 null) 都被利用, 保存该节点的后继. 当访问到这个结点时, 可以借助这个 null 字段来访问该结点的后继. 所谓的后继即中序遍历的后继. 如果我们用于前序/后序遍历, 那么它是前序/后序遍历的后继了.

多说无益, 看图:

```plaintext
             +---+
       +-----| 0 |-----+
       |     +---+     |
       |               |
     +---+           +---+
  +--| 1 |--+     +--| 2 |--+
  |  +---+  |     |  +---+  |
  |         |     |         |
+---+     +---+ +---+     +---+
| 3 |     | 4 | | 5 |     | 6 |
+---+     +---+ +---+     +---+
```

这个树的中序遍历结果如下:

```plaintext
3 1 4 0 5 2 6
```

对于某个结点 x, 如果 x 的左节点非空, 那么中序遍历应该先访问 x 的左子树, 然后是 x, 然后是 x 的右子树. 为了在访问完它的左子树后还能回到 x, 在遍历树的指针到达 x 并且还没有访问它的左子树时, 将 x 挂到 x 的左结点的右结点的右结点的右结点...(即 x 的左子树的所有叶子的最右边那个叶子) 下面, 这样访问完左子树以后, 就可以回到 x 了, 再次回到 x 后, 将前一个叶子(即 x 的左子树的最右边的叶子) 重新置回 null.

## Morris 中序遍历的实现

我们有 Morris 中序遍历的 C++ 代码 (假设结点的元素类型是 int):

```c++
vector<int> inorderTraversal(TreeNode *root) {
    vector<int> inorder_list;

    auto curr = root;
    while (curr != nullptr) {
        if (curr->left == nullptr) {
             // visit curr
            inorder_list.push_back(curr->val);
            curr = curr->right;
        } else {
            auto prev = curr->left;
            while (true) {
                if (prev->right == nullptr) {
                    prev->right = curr;
                    curr = curr->left;
                    break;
                }
                if (prev->right == curr) {
                    prev->right = nullptr;
                    // visit curr
                    inorder_list.push_back(curr->val);
                    curr = curr->right;
                    break;
                }
                prev = prev->right;
            }
        }
    }
    return inorder_list;
}
```

## 栈的视角

我们在二叉树中寻找 null 结点并保存某个结点的信息, 访问完后再将信息恢复原状. 这满足先进先出的规则, 也可以称为栈. 只不过这个栈没有实体, 是抽象的, 只由操作定义. 栈的 push 是访问 x 的左子树时, 将 x 的信息保存到 x 的左子树的最右边的叶子. 栈的 pop 是访问完左子树后, 将原来保存 x 的那个叶子恢复.

为了使这个想法更明显, 代码重写如下:

```c++
// Is stack empty?
bool empty(TreeNode *curr) {
    auto prev = curr->left;
    while (prev->right != nullptr && prev->right != curr) {
        prev = prev->right;
    }
    return prev->right == nullptr;
}

// push curr into stack
void push(TreeNode *curr) {
    auto prev = curr->left;
    while (prev->right != nullptr) {
        prev = prev->right;
    }
    prev->right = curr;
}

// pop curr
void pop(TreeNode *curr) {
    auto prev = curr->left;
    while (prev->right != curr) {
        prev = prev->right;
    }
    prev->right = nullptr;
}

vector<int> inorderTraversal(TreeNode *root) {
    vector<int> inorder_list;

    auto curr = root;
    while (curr != nullptr) {
        if (curr->left == nullptr) {
            inorder_list.push_back(curr->val);
            curr = curr->right;
        } else if (empty(curr)) {
            push(curr);
            curr = curr->left;
        } else {
            inorder_list.push_back(curr->val);
            pop(curr);
            curr = curr->right;
        }
    }
    return inorder_list;
}
```

本文只描述了中序遍历, 但前序和后序的思路类似.

## References

- [Tree traversal - Wikipedia](https://en.wikipedia.org/wiki/Tree_traversal)
- [Threaded binary tree - Wikipedia](https://en.wikipedia.org/wiki/Threaded_binary_tree)
