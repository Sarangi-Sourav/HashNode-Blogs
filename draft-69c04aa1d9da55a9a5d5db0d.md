---
title: "1st draft of GC"

---

**Ten Characters Per Second**

**Cambridge, Massachusetts. 1960.**

Picture the room. Fourth floor of MIT, a crowd of industry executives, the kind of people who fund research, who decide which ideas live and which ones die. They are watching a closed-circuit television screen. Downstairs, in the computer room, John McCarthy is running a demonstration of something nobody in this room has ever seen before.  
A *language* that could reason about *<mark class="bg-yellow-200 dark:bg-yellow-500/30">symbolic mathematics</mark>* the way a human mind does.  
  
The problem chosen for the demo was elegant: determine whether a first-order differential equation of the form **M dx + N dy** was exact by testing whether **∂M/∂y = ∂N/∂x** symbolic algebra. Live. On a computer. In 1960.  
  
Everything is going well, and then the Flexowriter begins to type:  
THE GARBAGE COLLECTOR HAS BEEN CALLED.

SOME INTERESTING STATISTICS ARE AS FOLLOWS:

And then it keeps going *on and on and on.* A full page of statistics: how many words were marked, how many were collected, and the total size of list space. All of it, at ten characters per second, slowly consuming every remaining second of the allocated demonstration time.

McCarthy later wrote: *"Nothing had ever been said about a garbage collector, and I could only imagine the reaction of the audience... both the lecturer and the audience were incapacitated by laughter. I think some of them thought we were victims of a practical joke."*

But here is the thing nobody immediately noticed then: it’s the GC, **which worked perfectly.**  
  
The real story isn't about the embarrassing demo. The real story is about why a garbage collector needed to exist in the first place and why solving that problem required a kind of thinking so radical that it changed every program you have ever run, on every device you have ever touched, for the rest of your life.

> *<mark class="bg-yellow-200 dark:bg-yellow-500/30">What is Symbolic Math?</mark>*
> 
> *Think of the difference between* ***doing math*** *and* ***learning the rules of math****:*
> 
> *   ***Arithmetic (Regular Math):*** *If I ask you, "What is 2 + 3 = ?" you say* ***5****. You are working with numbers to get a final result.*
>     
> *   ***Symbolic Math:*** *If I ask you, "What is the rule for adding any two things?" you say* ***x + y****. You aren't looking for a number; you are looking for the pattern or the rule itself.*
>     
> 
> *John McCarthy wanted to teach computers to move "symbols" (like x, y, +, or \\) around. He wanted the computer to follow a "rule" to change one math sentence into another, for example, automatically turning x^2 into 2x without even knowing what x stands for.*  
>   
> ⚠️ **The Catch:** Moving these symbols around required the computer to constantly create "scratchpad" notes to keep track of the patterns. This is where McCarthy noticed a massive problem...

**ACT ONE: THE WORLD BEFORE**

**The Machine Owns Everything**

In the 1950s, if you wrote a program, memory was *your* problem. You decided which memory addresses your variables lived in. You requested memory when you needed it. And when you were done with it, you freed it. Manually. Explicitly. Line by line.

This sounds manageable. It wasn't.

The problem isn't that programmers were careless. The problem is that **memory ownership is a property of time**, and time in a program is non-linear. A function allocates a block of memory. It passes a pointer to that block to another function. That function passes it somewhere else. Three call frames deep, a fourth function decides it's done with it and frees it. But the first function still has a pointer. And now that pointer points to memory that has been reallocated for something completely different.

You write through your old pointer, thinking you are updating your data. You are silently corrupting something else. The crash, when it comes, is nowhere near the cause.

The failure modes had names. A **dangling pointer**: you free memory but keep a reference to it, and the next allocation reuses that address for something completely unrelated; you think you are writing to your data structure; you are overwriting someone else's. A **memory leak**: you allocate memory and simply forget to free it. The program runs perfectly for hours, days, even, and then dies when the heap silently fills. Both failures shared the same pathology cause and symptoms separated in time and space by millions of instructions, with nothing in between to tell you where the wound was.

![](https://cdn.hashnode.com/uploads/covers/69a498aaa7428b958decac2d/20538b7d-1201-4014-a999-fa26f8da2890.png align="center")

**The Erasure Problem**

John McCarthy arrived at the Dartmouth Summer Research Project on Artificial Intelligence (YES AI) in 1956 with a conviction that would seem audacious to anyone who knew how computers worked: machines could be made to *reason*. Not to calculate but to **reason**.

To do that, he needed a language that thought in lists. Nested, recursive, symbolic lists. The kind of structures that logic itself uses: a sentence is a list of words, a proof is a list of steps, a derivative is a transformation of an expression. And lists, as McCarthy would discover, created a problem that would take two years to name.

In the summer of 1958, McCarthy was at IBM's Information Research Department. He chose a sample problem: write a program that symbolically differentiates algebraic expressions. The derivative of **x²** is **2x**. The derivative of **sin(x)** is **cos(x)**. LISP could represent these expressions as list structures, nested pairs of symbols, and apply the rules recursively.

He wrote the definition. It was beautiful. Clean. Recursive. Mathematically elegant.

Yet, he noticed that each recursion created new list structures for intermediate expressions, partial derivatives, and sub-expressions. These structures emerged and became irrelevant as recursion unwound and the final answer assembled. There was no mechanism to free them, leading to their silent accumulation throughout the program's duration.