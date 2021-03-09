---
title: The Welfare Crook
date: 2021-01-21 20:52:12
updated: 2021-01-21
katex: true
tags:
  - algorithm
  - program verification
---

题目 The Welfare Crook, 出自 "Science of Programming":

> Suppose we have three long magnetic tapes, each containing a list of names in alphabetical order. The first list contains the names of people woring at IBM Yorktown, the second the names of students at Columbia University and the third the names of people on welfare in New York City. Practically speaking, all three lists are endless, so no upper bounds are given. It is known that at least one person is on all three lists. Write a program to locate the first such person (the one with the alphabetically smallest person).

这个题目看起来不太容易, 而且可能需要几十行代码, 如果不使用形式化方法的话.

<!-- more -->

假设这三个列表分别表示为 f, g, h. 最后找到的位置是 iv, jv, kv. 这样, 我们有了后置条件:

$$
R: 0\le \text{iv}\land 0\le \text{jv}\land 0\le \text{kv}\land f[\text{iv}] = g[\text{jv}] = h[\text{kv}]
$$

为了编写循环, 使用 i, j, k 替换 iv, jv, kv, 并弱化谓词, 得到不变式:

$$
P: 0\le i\le \text{iv}\land 0\le j\le \text{jv}\land 0\le k\le \text{kv}
$$

显然, 我们需要迭代三个变量 i, j, k, 在满足某些情况时, 递增某个变量. 最简单的递增方法就是递增 1:

```python
def the_welfare_crook(f, g, h) -> str:
    i, j, k = 0, 0, 0
    while f[i] != g[j] or f[i] != h[k]:
        if IF1:
            i = i + 1
        if IF2:
            j = j + 1
        if IF3:
            k = k + 1
    return f[i]
```

接下来需要确定 IF1, IF2, IF3. 首先看看 IF1: (这里使用了 predicate transformer 来推导 IF1 的最弱形式)

$$
\begin{aligned}
\text{IF1} & = \text{wp}("i := i + 1", P)\cr
& = 0\le i+1\le \text{iv}\land 0\le j\le \text{jv}\land 0\le k\le \text{kv}\cr
& = 0\le i + 1\le \text{iv}\cr
& = i < \text{iv}
\end{aligned}
$$

所以只要满足 i < iv 即可递增 i. 问题在于 iv 就是我们想要找到的结果, 现在是未知的. 我们可以换个思路, 现在已知的有 i, j, k, f[i], g[j], h[k], 只要 f[i] < g[j] $\lor$ f[i] < h[k] 就一定意味着 i < iv, 反之则不一定. 因此

$$
\text{IF1} = f[i] < g[j]\land f[i] < h[k]
$$

同理, 推导出 IF2 和 IF3:

```python
def the_welfare_crook(f, g, h) -> str:
    i, j, k = 0, 0, 0
    while f[i] != g[j] or f[i] != h[k]:
        if f[i] < g[j] or f[i] < h[k]:
            i = i + 1
        if g[j] < f[i] or g[j] < h[k]:
            j = j + 1
        if h[k] < f[i] or h[k] < g[j]:
            k = k + 1
    return f[i]
```

这样不变式得到了维护. 下面需要证明循环会终止. 令整数函数 t = iv - i + jv - j + kv - k, 显然 t 每次循环都会减小, 且 t 有下界 0. 因此循环一定会终止.

下面证明如果循环终止, R 成立. 令循环条件为 BB, 首先需要证明 BB 是循环中各种条件的析取 (因为 IF1, IF2, IF3 需要覆盖 BB 成立的条件下的所有可能).

$$
\begin{aligned}
\text{IF1}\lor \text{IF2}\lor \text{IF3} & = f[i] < g[j] \lor f[i] < h[k]\cr
& \lor g[j] < f[i] \lor g[j] < h[k]\cr
& \lor h[k] < f[i] \lor h[k] < g[j]\cr
& = f[i] \not= g[j]\lor f[i] \not= h[k]\cr
& = \text{BB}
\end{aligned}
$$

另外, 需要证明 P $\land \neg $ BB $\Rightarrow$ R:

$$
\begin{aligned}
& P\land \neg \text{BB}\cr
& = P\land \neg (f[i]\not= g[j]\lor f[i]\not= h[k])\cr
& = P\land f[i]=g[j]=h[k]\cr
& = R
\end{aligned}
$$

这样, 我们证明了程序是正确的. 不过现在的条件判断太多了, 我们可以用更少的条件来证明 BB 是三个条件的析取. 经过仔细研究发现可以去掉一半:

```python
if f[i] < g[j]:
    i = i + 1
if g[j] < h[k]:
    j = j + 1
if h[k] < f[i]:
    k = k + 1
```

因为三个条件互斥, 最后得到:

```python
def the_welfare_crook(f, g, h) -> str:
    i, j, k = 0, 0, 0
    while True:
        if f[i] < g[j]:
            i = i + 1
        elif g[j] < h[k]:
            j = j + 1
        elif h[k] < f[i]:
            k = k + 1
        else:
            break
    return f[i]
```

最后的代码十分简单, 但是如果不使用形式化的方法, 几乎很难得到这样的代码, 甚至给出现成的答案也很难理解.

## References

- Science of Programming
