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

> These failure modes had names:  
> A **dangling pointer**: you free memory but keep a reference to it, and the next allocation reuses that address for something completely unrelated; you think you are writing to your data structure; you are overwriting someone else's.
> 
>   
> A ***memory leak***: you allocate memory and simply forget to free it. The program runs perfectly for hours, days, even, and then dies when the heap silently fills. Both failures shared the same pathology cause and symptoms separated in time and space by millions of instructions, with nothing in between to tell you where the wound was.

![](https://cdn.hashnode.com/uploads/covers/69a498aaa7428b958decac2d/20538b7d-1201-4014-a999-fa26f8da2890.png align="center")

**The Erasure Problem**

John McCarthy arrived at the Dartmouth Summer Research Project on Artificial Intelligence (YES AI) in 1956 with a conviction that would seem audacious to anyone familiar with computers at the time: machines could be made to *reason*. Not to calculate but to **reason**.

To do that, he needed a language that thought in lists. Nested, recursive, symbolic lists. The kind of structures that logic itself uses: a sentence is a list of words, a proof is a list of steps, a derivative is a transformation of an expression. Lists, as McCarthy would discover, introduced a problem that took two years to identify.

In the summer of 1958, McCarthy was at IBM's Information Research Department. He chose a sample problem: write a program that symbolically differentiates algebraic expressions. The derivative of **x²** is **2x**. The derivative of **sin(x)** is **cos(x)**. LISP could represent these expressions as list structures, nested pairs of symbols, and apply the rules recursively.

He wrote the definition. It was beautiful. Clean. Recursive. Mathematically elegant.

Yet, he noticed that each recursion created new list structures for intermediate expressions, partial derivatives, and sub-expressions. These structures emerged and became irrelevant as recursion unwound and the final answer assembled. There was no mechanism to free them, leading to their silent accumulation throughout the program's duration.

During that period, his colleagues were using FLPL (Fortran List Processing Language) for list processing, where programmers explicitly managed the lifecycle of their list calls. However, McCarthy realized this method clashed with his objectives. In his paper, he articulates:

> *"The recursive definition of differentiation made no provision for erasure of abandoned list structure. No solution was apparent at the time, but the idea of complicating the elegant definition of differentiation with explicit erasure was unattractive."*

The term "unattractive" carries significant philosophical weight. McCarthy wasn't suggesting that explicit erasure was technically impossible; he meant it contradicted the language's fundamental purpose.  
  
He was creating a tool for artificial intelligence to enable machines to think more like humans. Humans don't monitor the ownership of every intermediate thought. When solving a math problem mentally, you don't remember to "free" the partial sums; they simply become irrelevant and disappear.  
  
McCarthy wanted LISP to function similarly. The programmer should focus on what to compute, while the machine manages where to store it.

This was, in 1958, a radical idea.  
  
**ACT TWO: THE BIRTH**

**The Two Alternatives**

By the fall of 1958, McCarthy had become an Assistant Professor at MIT. Together with Marvin Minsky, he initiated the MIT Artificial Intelligence Project. With an IBM 704, a small team, and a significant challenge to address before LISP could advance, the team identified precisely two potential solutions.

**Option One: Reference Counting.**

In 1960, George E. Collins published "A method for overlapping and erasure of lists" in Communications of the ACM, volume 3. This coincided with the release of McCarthy's LISP paper, with both addressing the erasure problem independently in the same year.

Here's how it works:

Each memory block has a counter. When something references it, the counter increases. When the reference is removed, it decreases. Once the count reaches zero, the memory is freed instantly and automatically, without any pauses or scanning. If successful, every object would clean itself up as soon as it became unreachable. LISP's unused intermediate expressions would vanish upon becoming irrelevant.

However, reference counting has a critical flaw: cycles. Reference cycles render reference counting ineffective as a garbage collector. Data structures with back-pointers, such as doubly-linked lists, trees with parent pointers, and object graphs, can create cycles that lead to perpetual memory leaks.

This is not a minor edge case; it is a fundamental characteristic of any language where object graphs can form cycles, which applies to nearly every non-trivial program.

Collins implemented his approach on a 48-bit CDC computer, which, unlike the IBM 704, had sufficient bits in a word to store a reference count.

So,

**Option Two: The Mark & Sweep**

In McCarthy’s words:

> *"storage is abandoned until the free storage list is exhausted, the storage accessible from program variables and the stack is marked, and the unmarked storage is made into a new free storage list."*

Let's break that down slowly, as this single sentence forms the foundation for everything that follows.

*Storage is abandoned* — do not track it. Do not count it. Just allocate freely and let the dead accumulate.

*Until the free storage list is exhausted,* run until you run out of memory. Only then act.

*The storage accessible from program variables and the stack is marked*, starting from everything the program can currently reach, following every pointer, recursively, and marking every reachable object as alive.

*The unmarked storage is made into a new free storage list*, everything not marked is garbage. Thread it back into the pool.

The insight completely reverses the logic of reference counting. While reference counting directly tracks an object's death when its count reaches zero, mark-and-sweep disregards death and infers it from the absence of life. Instead of detecting when objects become garbage, it identifies what remains alive and declares everything else as garbage by exclusion.

**The Algorithm (Pseudo code):**

The heap in LISP 1.5 consisted of a pool of cons-cells, which are pairs of pointers known as **car** and **cdr** (Contents of Address Register and Contents of Decrement Register, named after IBM 704 hardware fields). Each cons cell was a fixed-size word, and the free storage list was a linked list of unallocated cells.

**DATA STRUCTURES**

```c
Every cons cell in the heap has: 
struct ConsCell: 
    car: pointer or atom    // first element 
    cdr: pointer or atom    // rest of list 
    mark: bit               // 0 = unmarked, 1 = marked 
// (mark bit was stored in the tag field of the IBM 704 word)

// "Roots" = all locations in the running program 
// can currently reach: 
// - all variables on the stack 
// - all registers holding pointers 
// - all global variables
```

**PHASE 1: MARK**

```c
function mark_all_roots(): 
    for each pointer P in roots: 
        mark(P)

function mark(P): 
    if P is null: 
        return 
    if P is an atom (not a cons cell): 
        return 
    if P.mark == 1:    // already visited 
        return         // CRITICAL: prevents infinite loop on cycles

    P.mark = 1         // mark as alive

    mark(P.car)        // recurse into left child
    mark(P.cdr)        // recurse into right child
```

**PHASE 2: SWEEP**

```c
function sweep(): 
    free_list = null         // rebuild the free list from scratch

for each ConsCell C in heap (linearly, address by address):
    if C.mark == 1:
        C.mark = 0           // reset for NEXT collection cycle
    else:
        // C is garbage: no living pointer reaches it
        C.cdr = free_list   // link it into free list
        free_list = C
```

**ALLOCATION**

```c
function cons(a, d) -> ConsCell: 
    if free_list == null: 
        collect() // trigger GC 
    if free_list == null: 
        ERROR "out of memory" // THIS is what happened at the demo
    cell = free_list
    free_list = free_list.cdr
    cell.car = a
    cell.cdr = d
    cell.mark = 0
    return cell
```

**FULL COLLECTION**

```c
function collect(): 
    mark_all_roots() 
    sweep() 
    // print verbose statistics ← THIS printed on the Flexowriter
    print_gc_message() 
```

That's the entire process: two phases, one mark bit per object, and a linear scan. McCarthy's differentiation function, which he insisted on keeping elegant without explicit erasure, could now operate smoothly. It could create and discard list structures without manual tracking, as the collector would identify the unused ones when the heap was full.

However, McCarthy didn't emphasize this aspect, noting that:

> *"Its actual implementation could be postponed, as only toy examples were being used."*

He considered it primarily theoretical.