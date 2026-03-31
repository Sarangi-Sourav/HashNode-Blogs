---
title: "The 1960 Algorithm Your Phone Runs Millions of Times a Day"
seoTitle: "The 1960 Algorithm Your Phone Runs Millions of Times a Day"
seoDescription: "Discover the history of Garbage Collection. From John McCarthy’s LISP to modern ZGC, learn how a 1960s breakthrough still manages your phone's memory."
datePublished: 2026-03-31T13:00:00.000Z
cuid: cmnemlk25000121dyaurp7vvz
slug: the-1960-algorithm-your-phone-runs-millions-of-times-a-day
cover: https://cdn.hashnode.com/uploads/covers/69a498aaa7428b958decac2d/277d6da7-a31d-439f-99af-e59598451e59.jpg
ogImage: https://cdn.hashnode.com/uploads/og-images/69a498aaa7428b958decac2d/d62b9b72-f28c-4023-819f-8cad36754349.jpg
tags: software-development, programming-blogs, computer-science, lisp, garbagecollection, mark-and-sweep, programminghistory

---

**Ten Characters Per Second**

**Cambridge, Massachusetts. 1962.**

Picture the room. On the fourth floor of MIT, a crowd of industry executives, the kind of people who fund research, who decide which ideas live and which ones die, are watching a closed-circuit television screen. Downstairs, in the computer room, **John McCarthy** is running a demonstration of something nobody in this room has ever seen before.  
A *language* that could reason about *symbolic mathematics* the way a human mind does.

The problem chosen for the demo was elegant: determine whether a first-order differential equation of the form **M dx + N dy** was exact by testing whether **∂M/∂y = ∂N/∂x**, symbolic algebra live on a computer in 1962.

Everything is going well, and then the Flexowriter begins to type:  
THE GARBAGE COLLECTOR HAS BEEN CALLED.

SOME INTERESTING STATISTICS ARE AS FOLLOWS:

And then it keeps going *on and on and on.* A full page of statistics: how many words were marked, how many were collected, and the total size of list space. All of it, at a speed of ten characters per second, and the printer was really determined and persistent in typing out these long, technical messages, slowly consuming every remaining second of the allocated demonstration time.

McCarthy later wrote: *"Nothing had ever been said about a garbage collector, and I could only imagine the reaction of the audience... both the lecturer and the audience were incapacitated by laughter. I think some of them thought we were victims of a practical joke."*

But here is the thing nobody immediately noticed then: it’s the GC, **which worked perfectly.**

The real story isn't about the embarrassing demo. The real story is about why a garbage collector needed to exist in the first place and why solving that problem required a kind of thinking so radical that it changed every program you have ever run, on every device you have ever touched, for the rest of your life.

> *What is Symbolic Math?*
> 
> *Think of the difference between* ***doing math*** *and* ***learning the rules of math***:
> 
> *   ***Arithmetic (Regular Math):*** *If I ask you, "What is 2 + 3 = ?" you say* ***5***. You are working with numbers to get a final result.
>     
> *   ***Symbolic Math:*** *If I ask you, "What is the rule for adding any two things?" you say* ***x + y***. You aren't looking for a number; you are looking for the pattern or the rule itself.
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
> A ***memory leak***: you allocate memory and simply forget to free it. The program runs perfectly for hours, days, even, and then dies when the heap silently fills. Both failures shared the same pathology cause and symptoms separated in time and space by millions of instructions, with nothing in between to tell you where the wound was.

![](https://cdn.hashnode.com/uploads/covers/69a498aaa7428b958decac2d/2dadc173-5367-45cd-a7e0-78adf5515379.png align="center")

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

However, all these were just theories that McCarthy proposed in his 1960's paper "[Recursive Functions of Symbolic Expressions and Their Computation by Machine](https://www-formal.stanford.edu/jmc/recursive.pdf)", in his words:

> *"Its actual implementation could be postponed, as only toy examples were being used."*

He considered it primarily theoretical.

**Steve Russell, Dan Edwards, and the Implementation**

In the fall of 1958, a twenty-two-year-old from rural Washington state, with a keen interest in electronics, arrived at MIT after completing his undergraduate studies at Dartmouth, where McCarthy had recruited him.

It was **Steve Russell**. Russell initially worked on manually translating Lisp functions like car and cdr into machine code to assist in building a compiler. While reading McCarthy's paper, he noticed the *eval* function and believed he could implement it to create a Lisp interpreter. When he proposed this idea, McCarthy famously told him:

> "Ho , ho, you are confusing theory with practice."

Despite McCarthy's skepticism, Russell proceeded to hand-code eval, creating the first functional Lisp system. While Russell's efforts made Lisp operational, it was **Daniel Edwards**, a fellow MIT graduate student, who primarily developed the first practical garbage collection algorithm to address the "erasure problem" of nested list structures.

> In early Lisp, eval(e, a) was the core mechanism that empowered the language. It took an expression '**e'** (written as an S-expression) and an environment '**a'** (an association list mapping variables to values) and evaluated the expression recursively based on its structure. If 'e' was a constant, it returned it directly; if it was a variable, it looked it up in 'a'; if it was a special form like QUOTE or COND, it handled it with specific rules; and if it was a function call, it evaluated all arguments and then applied the function.
> 
> ```c
> function eval(e, a):
>     if is_constant(e):
>         return e
> 
>     else if is_variable(e):
>         return lookup(e, a)
> 
>     else if is_special_form(e):
>         if operator(e) == 'QUOTE':
>             return argument(e)
> 
>         if operator(e) == 'COND':
>             for each (predicate, expr) in clauses(e):
>                 if eval(predicate, a) == true:
>                     return eval(expr, a)
> 
>     else:  // function call
>         func = eval(operator(e), a)
>         args = eval_list(arguments(e), a)
>         return apply(func, args)
> ```
> 
> Before Russell implemented eval, Lisp was just a collection of subroutines that had to be manually translated into assembly language to run. By implementing the eval function itself in machine code, Russell created a system that could read Lisp code and execute it directly. This established the "Read-Eval-Print Loop" (REPL) that remains the heart of Lisp development today.

The implementation demanded a meticulously crafted table identifying every location in the running program where a list pointer might reside, every stack slot, register, and global variable, so the mark phase could locate all the roots. This table was compiled by manually examining the assembly code line by line and recording each pertinent address.

But it was incomplete.

> *"For at least a year, and I think longer than that, we would periodically get bugs where somebody had done some perfectly innocent function and had gotten the free storage list tacked in the middle of its list structure, because something that shouldn't have been garbage collected was."*

The collector was silently reclaiming memory still in use by programs, corrupting live data structures. These bugs were intermittent, surfacing only when the collector was activated, which happened when memory was nearly exhausted. This meant the cause and symptom were separated by vast amounts of computation, with no clear link between them.

Observe the parallel. The dangling pointer from Act One, the crash occurring far from its cause, had resurfaced within the tool designed to prevent it. The first garbage collector had its own bug: it discarded memory it should have retained.

Additionally, there was the glaring issue of the *Stop-The-World* problem associated with those garbage collectors.

> **Stop the world** is a phase in garbage collection (GC) where all application threads are halted so that the collector can perform its work safely. This is done to ensure the garbage collector has exclusive access to the memory and that the application does not interfere with the process, such as by allocating new objects or modifying references during the collection.

**ACT THREE: THE WORLD AFTER**

**The Idea That Wouldn't Stay Contained**

LISP survived the demo. The concept that a program shouldn't need to manage the lifetimes of its data structures proved too influential to be confined to just one language.

However, it also proved costly, sparking a more profound debate.

In systems programming, there was a deeply ingrained belief that programmers should have a complete understanding and control over the machine. Every allocation, every deallocation, every byte had to be accounted for. This wasn't mere obstinacy; it was the engineering principle of those creating real-time systems, where a pause for garbage collection could result in a missed deadline, a lost network packet, or a control loop that failed to execute.

In 2008, Steve Russell, who had witnessed the trade-offs of garbage collection for over fifty years, expressed the underlying tension:

> *"There's a basic problem with garbage collection. If you do it simply, it causes everything to stop when it happens. And if you do something to avoid that, the overhead on the garbage collection becomes a lot more... Free storage is not infinite. And reviewing code to establish or prove that it uses a bounded amount of storage, is really, really hard."*

For the next six decades, researchers worked to balance the cost, with each new algorithm striving to fulfill McCarthy's promise while minimizing Russell's concerns.

**Copying collectors**, introduced by Fenichel & Yochelson in 1969 and Cheney in 1970, addressed fragmentation by moving all live objects to a new memory region and completely discarding the old one. This made allocation as quick as a pointer bump, efficient, cache-friendly, and without the need to traverse a free list. The trade-off was that half of the heap had to be kept empty as a destination for evacuation.

**Concurrent collectors**, developed by Dijkstra and others in 1978, tackled the stop-the-world issue by introducing the tri-color abstraction. This method classified objects as white (not yet reached), grey (reached but not fully scanned), or black (fully processed). They formally demonstrated that running the collector alongside the program was safe, as long as write barriers intercepted every pointer write to maintain the invariant. The world didn't have to stop; it just needed to be monitored.

**Generational collection** (Lieberman & Hewitt, 1983; Ungar, 1984) shifted the focus entirely. Rather than figuring out how to collect the entire heap more efficiently, it questioned whether collecting the entire heap was even necessary.

The answer came from data. Researchers studying running programs in LISP, Smalltalk, and later Java found the same pattern everywhere: the vast majority of allocated objects are short-lived, often disappearing almost immediately, sometimes within microseconds, before the next similar-sized allocation occurs. A small fraction survives for a long time, with almost nothing in between. David Ungar called this the generational hypothesis, and his 1984 "Generation Scavenging" algorithm for Smalltalk-80 was built entirely around exploiting it.

The concept involves splitting the heap into a small nursery for new objects and a larger old generation for those that survive. The nursery is collected frequently. Since it's small enough to fit entirely in the CPU cache, a nursery collection takes only milliseconds. Most objects die before their first collection, allowing a nursery sweep to reclaim the majority of garbage. In Ungar's experiments, about 98% of objects perished in the nursery without being promoted to the old generation, which rarely needed collection due to its cost.

This was not a minor improvement. Generational collection turned garbage collection from a periodic disaster into a process that a program could maintain over hours of operation.

**G1 / G1GC** (Detlefs, Flood, Heller, Printezis - Sun Microsystems, 2004; default JVM GC since Java 9) divided the heap into equal-sized regions assigned dynamically, prioritizing the collection of regions with the most garbage. This approach allowed engineers to predict and limit pause times, rather than enduring them unpredictably.

**ZGC** (Liden, Karlsson - Oracle, 2018) utilized unused bits in 64-bit pointers to store GC metadata, intercepting each pointer read with a load barrier to dynamically correct stale references. This enabled sub-millisecond pauses, regardless of heap size, even on terabyte-scale heaps.

Java arrived in 1995, bringing automatic memory management into the mainstream. Suddenly, millions of programmers no longer needed to call free(). Languages like Python, Ruby, JavaScript, and Go embraced this approach. Yet, C and C++ remained prevalent.

Rust introduced a third option: automatic memory management through static analysis at compile time, enforcing ownership rules so rigorously that memory safety is ensured before execution, no collector, no pauses, no runtime overhead. The cost is a cognitive one, not in CPU cycles.

The argument that started in a basement at MIT in 1958 is still going.

### The Problem Nobody Has Fully Solved

After sixty-eight years, every garbage collection algorithm balances three key properties:

```plaintext
                                                            
   THROUGHPUT:    How much CPU goes to actual work           
                  vs. GC overhead?                           
                                                             
   LATENCY:       How long does the worst-case pause last?   
                                                             
   FOOTPRINT:     How much extra memory does GC need?        
                  (remembered sets, semi-spaces, metadata)   
                                                             
   You can optimize any two.                                 
   You cannot simultaneously optimize all three.                                                                 
```

This is not a temporary limitation awaiting a smarter algorithm. It is an inherent characteristic of the problem space more akin to the CAP theorem in distributed systems than to an engineering shortcoming.

**Sources:** `John McCarthy, "History of LISP," History of Programming Languages conference proceedings, 1981. Steve Russell, Oral History, Computer History Museum, August 9, 2008 (CHM Reference X4970.2009). McCarthy 1960 (CACM 3:4). Collins 1960 (CACM 3:655). Cheney 1970 (CACM 13:11). Dijkstra et al. 1978 (CACM 21:11). Lieberman & Hewitt 1983 (CACM 26:6). Ungar 1984 (SIGPLAN Notices 19:5). Detlefs et al. 2004 (ISMM '04).` [Detailed Sources](https://github.com/Sarangi-Sourav/HashNode-Blogs/blob/main/sources)