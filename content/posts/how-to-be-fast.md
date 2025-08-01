---
title: "How to programming fast"
date: 2023-08-08T21:17:05+08:00
---

I have a good understanding of algorithms, data structures, logic, program verification, and Vim keybindings. Why am I not fast as I thought?

<!--more-->

This is what I thought when I was struggling with Leetcode years ago.

At that time, I could easily solve medium problems in half an hour, and most hard problems in one hour. I
considered myself an intermediate Vim user and can type everything very fast.

Sincerely, I'm not bad, but also not good enough.

How can I reach the top players' level where they can solve problems, write correct and beautiful solutions in twenty minutes, and even have no wrong submissions ever?

## Profiling

I watched some Youtube videos to see how they did it:

They just stared at the problem, scrolling the screen several times. After several minutes,
they started to write code and never made a mistake, just like there was a piece of
completed correct code in front of them. They are well-trained athletes and do well in
all the details.

I set up OBS Studio to "profile" what I did during contests. The result was surprising. I was not effectively solving the problem most of the time:

1. When I saw the problem, I read the problem and thought I understood it. But I didn't read and understand the example input/output carefully. Finally, I spent a lot of time on a solution that tried to solve another problem!
2. I didn't estimate the scale of the data. I don't even have a clue about it. Finally, I spent a lot of time on solutions that were not fast enough and always had higher time complexity.
3. When I started to write code, I don't know the whole image of the final code.
4. I wrote code from top to bottom. I didn't know the next exact expression; I only knew the rough steps.
5. I always wrote the definition of variables/functions/macros before using them. So I always created some useless variables and functions and finally removed them.
6. I sometimes interrupted my current flow and jumped to another random place to make some changes, then went back and didn't remember what to do next. It sounds stupid, but that's what I did.
7. When I found a non-obvious bug, I ran the program many times and always got the same results. If I couldn't immediately know where it came from, my mind would go blank and do nothing for several minutes. When I decided to reason and solve it, it usually took less than one minute to solve it.
8. My Vim operations are fast, but most operations were unnecessary. Some were useless; some can be replaced with one rare keybinding. That's because I wasn't sure what I exactly need.

It's crazy that I made all these mistakes. It could be a proof that I am stupider than an average person 😄.

## Approach

This is what I do now:

1. Resist the desire to code. For some reason, times would disappear very easily when writing code but move much slower when thinking.
2. understand the problem from various perspectives, give some examples, and estimate the size of the data. If the data size is unknown, it should be estimated again after completing the algorithm.
3. Try to find a simpler solution even if I have a valid but complex solution in mind. A simpler solution is always easier to write than a complex one.
4. Do a rough or full proof of the solution before coding.
5. Write code in reverse or prioritize main logic when I have rough steps in mind. For example, I write important code directly, using undefined variables/functions/macros, and determine their meaning when I use them. So the core logic can be very compact and simple. Then gradually complete the undefined variables/functions/macros in BFS (breadth first) manner. In this way, I rarely write useless code and every function/macro will be simple. The "reverse" means I sometimes write the last expression of the function, then the previous one...
6. don't be busy to touch the keyboard, think about the operation before touching it.
7. imagine the shape of the code

Point 5 is an important method, that greatly improved my speed and my code quality.

I may not be at the level of top players yet, but I have improved significantly from where I was before. It also applies to all aspects of programming.

## unfocused Parts

The following points are not the focus of this post, but there is a lot of room for them:

- problem-solving methodology. There are many common strategies for solving problems. The most common one is probably to identify the properties of known data and objects and the relationship between that data and the question. A different strategy sometimes leads to a simpler solution or simpler to implement.
- program proof. My brain runs a JIT (just in time) prover when writing code to reduce potential bugs. It's tiring but helps to write the correct code.
- debug. Effective debugging requires reasoning like a detective, careful investigation, imagination, and a bit of luck.
- abstraction. It is one of the biggest factors in programming, but a wrong choice can also add extra cost.
- programming language. A good programming language would also be helpful. That's why I like Lisp or Racket in particular.
