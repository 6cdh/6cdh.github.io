---
title: "On Read/Write Code"
date: 2024-06-27T15:19:28+08:00
---

Recently I've been thinking about how to improve reading and writing code. Here are some ideas for discussion.

<!--more-->

My intuition is that reading code is similar to reading books or stories, which we do since a very young age and are familiar with. This analogy might provide some insights into how to read and write code better and how programming tools should help.

## Goals

There are a lot of books in the world. But we only read a small part of them. For many books we have read, we don't even remember the fact that we have read them. For some books, we may only have vague impressions and know what they are about, but can't remember the details. For the rest of the books, we have deep impressions, understand the theme, and can even remember some paragraphs and images clearly. Therefore we need to determine if a book is interesting to read.

This highlights our **first goal**:

> Determine if the given book/software source code is what we are interested in.

If we want to read a book, we need to make sure we can understand it. For example, if we can't understand the language the book is written in, or we have trouble understanding words or grammar, then it will be difficult to make progress. It would be great if it's written in our native language. But if it's not, and we have only a basic understanding of the language, then we can only read slowly and hope the book uses as many as simple words as possible. Personally, I'm not a native English speaker, and I feel at ease with simple English texts, but work more slowly and less aggressively in complex English contexts.

So we get the **second goal**:

> Understand the grammar. And only write text/code in simple grammar as if the reader is a beginner language learner.

After that, we can go through the book. The book may be long or short, the chapter may be interesting or boring. We usually want to identify the key points quickly, try our best to understand them, and skim through the normal texts. A good understanding of keypoints includes not only what the concept is, but also why it exists and how the author uses it throughout the whole book. This is our **third goal**:

> Recognize and understand the key parts.

Now we almost completely understand the book. What's next? We enjoyed the book and interaction with the author. But that's not enough. We might wonder how others think the book, whether the author's opinion is common in the field, and what else the book doesn't mention.

So this is the final and long-term **goal 4**:

> Read other books/software which has a similar subject to this book.

Let's discuss each goal briefly.

## Goal 1

When we get a book, we usually read the table of contents first, turn pages quickly, stop randomly or stop at interesting chapters, look at some tables and pictures, read a few highlighted terms, paragraphs and pages, etc.

Similarly, we should assume that readers always **read code randomly** first. There should be

* A clear and unified **structure** so that the readers can easily browse and read any part randomly.
* **Documentation** about the structure that covers from the highest (project) level to the lowest (file or class) level, is accessible to people who know nothing about the project.
* **Other materials** which help readers understand the code. It corresponds to tables, images, and references in the book.
* **Less connection** between different components. If we assume readers read randomly, then a module, a class, or a function should have a single responsibility and work independently, or at least clearly describe which other components it serves, depends on, and belongs to.

These requirements are difficult to maintain between iterations. At this point, some tools can help our lives. For example, a language server should provide a function list at the file level, jump to symbols, hover documentations, references, goto definitions, and even more to help readers easily browse code randomly. The language should also be designed to write well-organized code and integrate these tools more easily.

## Goal 2

We assume that people will read randomly and quickly before reading carefully. So the readers should know the language beforehand. If they don't, then they expect the language to be simple, intuitive, or share many features and syntax with other languages they are familiar with.

This puts demands on authors and language designers. They should avoid making/using unnecessary complex features and syntax, and prefer simpler ones.

For readers, they should have a basic grasp before or during reading, and guess as little as possible about the meaning of expressions or statements.

However, we know, that some wide used languages are not easy to read for beginners, such as Lisp, or have quite complex syntax and features, such as C++ or Rust, or have many quirks, such as Javascript. In my imagination, there will be some tools that render plain code text into another form that mixes graphics, visualizations, and even animations to simplify the recognition of syntax and understanding of the meaning of the code, just like how we write HTML/Latex and render them for reading. There are some efforts to build structured code editors but it yet not benefit general programming languages.

## Goal 3

After finishing the first two goals, we located and decided to read the important parts. We usually have two reasons to read the important code:

1. Know what it does. Most of the time, we only want to know the logic of that part, and what it does in some specific situation. We are not interested in the code style, or why the author wrote it in that way.
2. Understand it. When we do this, we consider ourselves learners and try to understand how the author thinking. We also expect to learn wonderful ideas and technologies from the code.

No matter what reasons, by [Rice's theorem](https://en.wikipedia.org/wiki/Rice%27s_theorem), reading code is not enough to determine the behavior of the program. You always need to execute the program, modify it, and try different inputs to fully understand what it does. **Never think you fully understand a program before running it.** This is another reason why the author should write independent and isolated code. It simplifies the readers' attempt to run the subroutines. This is also a use of debuggers.

Understanding the behavior of the program is not enough. We also want to know the idea of the author. A program is similar to a proof. It might cover many pages but only one or several core ideas. Sometimes we just wonder about the ideas instead of reading all the details of the program. Therefore the authors should add comments about the key ideas in the code. If the code is related to some complex theories, they should indicates these theories, and how they connects to their code.

## Goal 4

We now fully understand the code. We can stop and enjoy our life. Or we still have questions about the subject and want to read more similar code. This is a lifelong stage where we read, understand, and compare ideas and details of different implementations, never ending.

Here a isolated software component is also useful. We can split different implementations from the whole complex project, compare them, or even embed them in another software and switch them as needed.

## Conclusion

We briefly discussed the similarities between reading books and reading code. We expanded to some advice on how write and design code. Some of the advice are difficult to follow, but I hope it increases the understanding of reading and writing code. 

Many ideas here come from [How to Read a book by Mortimer J. Adler and Charles Van Doren](https://www.amazon.com/How-Read-Book-Classic-Intelligent/dp/0671212095).

