---
title: The Stack Sorting
date: 2021-03-10T12:32:42+08:00
tags: [data structure]
katex: true
---

Recently, I got stuck in the stack data structure. Everyone knows that the stack is a LIFO _(Last In First Out)_ data structure. This is easy to understand, but not easy to use sometimes.

<!--more-->

Consider the stack sorting problem which is introduced by Knuth in the 1960s:

> Given the n-sized permutation $p = a_1 a_2\cdots a_{n-1} a_n$. This permutation is known
> as the 'input'. The only tool we have for sorting is a stack.

How to do it? In the first step, we place $a_1$ into the stack. For the second step, we now compare it with the element $a_2$. If $a_1 > a_2$ then we place $a_2$ on the stack above of $a_1$, otherwise, we place $a_1$ into the output and place $a_2$ on the top of the stack.

For each step, we compare the leftmost element in the input with the element on the
top of the stack. The process ends when all the elements have been placed into the output
stack.

The Python code as follows:

```python
def stack_sorting(a: List) -> List:
    a.append(math.inf)
    stack = []
    output = []
    for v in a:
        while stack and stack[-1] <= v:
            output.append(stack.pop())
        stack.append(v)
    return output
```

This algorithm will cost O(n) and is a comparative sorting. So it should have somewhat limitations. In fact, the algorithm can't sort a permutation that contains a 231-pattern:

```python
>>> stack_sorting([2, 3, 1])
[2, 1, 3]
```

To sort this type of permutation, we can take the output of the one stack sorting and sort it again with the stack. If the permutation is ordered after k sorts, we say the initial permutation is k stack sortable.

Then Let's go back to the one stack sorting to study its properties.

> Consider the permutation $p = a_1 a_2 \cdots a_{n-1} a_n$. Let $x = \max (a_1, a_2, \cdots, a_n)$. Let $p_L$ and $p_R$ be the terms such that $p = p_L x p_R$. Let $s(p)$ is the permutation after one stack sorting. Then
>
> $$
> s(p) = s(p_L)s(p_R)x
> $$

The proof is trivial. Every element before $x$ will enter and leave the stack before $x$ enters since it is larger. After $x$ enters the stack, every element after $x$ will enter and leave the stack before $x$ leaves the stack.

This property implies:

- The predecessor of $x$ in the output is the maximum of the elements after $x$.
- The element on the top of the output is the maximum of the elements before $x$ when $x$
  just enters the stack.

Another property is the invariant of the one stack sorting algorithm:

> The top of the stack is the last greater element before $a_i$ when $a_i$ is ready to
> push to the stack.

The invariant is easy to prove. When $a_1$ is ready to push, the stack is empty and the
invariant holds. At the beginning of the ith cycle, the invariant holds, and the top of
the stack is $a_{i-1}$. If $a_{i-1} > a_i$, the invariant holds. If $a_{i-1} \le a_i$,
$a_{i-1}$ will be poped to the output. If the last greater element of $a_{i-1}$ less than
$a_i$, then we push $a_i$ to the stack. Otherwise, continue to pop it to the output. At
the end of the cycle, the invariant also holds.

Leetcode 503 [Next Greater Element II](https://leetcode.com/problems/next-greater-element-ii/) is an example:

> Given a circular array (the next element of the last element is the first element of the array), print the Next Greater Number for every element. The Next Greater Number of a number x is the first greater number to its traversing-order next in the array, which means you could search circularly to find its next greater number. If it doesn't exist, output -1 for this number.

We can reverse execution the stack sorting:

```python
def nextGreaterElements(nums: List[int]) -> List[int]:
    n = len(nums)
    stk = []
    ret = [-1] * n
    for i in reversed(range(2 * n)):
        while stk and stk[-1] <= nums[i % n]:
            stk.pop()
        if stk:
            ret[i % n] = stk[-1]
        stk.append(nums[i % n])
    return ret
```

Let $a$ be the input, $r_i = a_j$ be the next greater number of $a_i$ in $a$. That means

$$
i < j\land ( \forall k : i < k < j : a[i] \ge a[k] ) \land a[i] < a[j]
$$

So we have

```python
def nextGreaterElements(nums: List[int]) -> List[int]:
    n = len(nums)
    stk = []
    ret = [-1] * n
    for i in range(2 * n):
        while stk and nums[stk[-1]] < nums[i % n]:
            ret[stk.pop()] = nums[i % n]
        stk.append(i % n)
    return ret
```

## References

- [Brief Introduction on Stack Sorting - LIM, CONG HAN](https://math.uchicago.edu/~may/VIGRE/VIGRE2007/REUPapers/FINALAPP/Lim.pdf)
- [Stack-sortable permutation - Wikipedia](https://en.wikipedia.org/wiki/Stack-sortable_permutation)
