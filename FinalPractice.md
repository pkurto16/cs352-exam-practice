CS352 Practice Questions
Week 7 - Optimizations
(Based on week7-1 (1).pdf)

Question 11: Constant Folding and Propagation
Prompt: Given the following CPS/MiniScala code snippet, apply constant folding and constant propagation optimizations. Show the optimized code.

scala
Copy
Edit
val_l n1 = 5;
val_l n2 = 10;
val_p n3 = n1 + n2;
val_l n4 = 2;
val_p n5 = n3 * n4;
val_p n6 = n1 + 7; // Assume n1 is not used after this
c_ret(n5)
<details><summary>Answer</summary>
scala
Copy
Edit
val_l n1 = 5
val_l n2 = 10
val_l n3 = 15        // folded 5 + 10
val_l n4 = 2
val_l n5 = 30        // folded 15 * 2
c_ret(30)
</details>
Question 12: Dead Code Elimination
Prompt: Identify and eliminate any dead code (unused variable/function definitions) in the following CPS/MiniScala code. Explain why the code you removed is considered dead.

scala
Copy
Edit
def_f funcA(c_ret, x) = {
  val_l temp1 = 5; // temp1 is never used
  val_p result = x + 10;
  c_ret(result)
};
def_f funcB(c_ret, y) = { // funcB is never called
  val_p result = y * 2;
  c_ret(result)
};
def_f funcC(c_ret, z) = {
  val_l unused_val = 100;
  val_p temp_z = z - 1;
  funcA(c_ret, temp_z)
};

val_l start_val = 20;
def_c final_cont(res) = {
  print(res)
};
funcC(final_cont, start_val)
<details><summary>Answer</summary>
Removed:

val_l temp1 = 5 in funcA (never used).

Entire funcB definition (never called).

val_l unused_val = 100 in funcC (dead).

Optimized code:

scala
Copy
Edit
def_f funcA(c_ret, x) = {
  val_p result = x + 10
  c_ret(result)
};
def_f funcC(c_ret, z) = {
  val_p temp_z = z - 1
  funcA(c_ret, temp_z)
};

val_l start_val = 20;
def_c final_cont(res) = print(res);
funcC(final_cont, start_val)
</details>
Question 13: Common Subexpression Elimination (CSE)
Prompt: Apply Common Subexpression Elimination (CSE) to the following code snippet. Identify the common subexpression and show the optimized code.

scala
Copy
Edit
val_p a = x + y;
val_p b = a * 2;
// ... other code that does not modify x or y ...
val_p c = x + y; // Common subexpression
val_p d = c * 3;
c_ret(b + d)
<details><summary>Answer</summary>
The common subexpression is x + y. We reuse a instead of recomputing:

scala
Copy
Edit
val_p a = x + y
val_p b = a * 2
// ... other code ...
val_p d = a * 3    // use a directly
c_ret(b + d)
</details>
Question 14: Inlining
Prompt: Consider the following functions. Explain the potential problems with naively inlining addOne into applyFunc directly on an AST if argExpr has side effects (e.g., print("Eval"); 5). How does using an intermediate representation (like CPS/MiniScala where arguments must be atoms) or binding arguments to temporary variables mitigate this?

scala
Copy
Edit
def addOne(n: Int): Int = n + 1;
def applyFunc(argExpr: Int): Int = addOne(argExpr);
<details><summary>Answer</summary>
Naive inlining:

scala
Copy
Edit
def applyFunc(argExpr: Int): Int = argExpr + 1
causes argExpr to be evaluated twice if it has side effects (once for inlining, once for the addition), e.g.:

scala
Copy
Edit
applyFunc({ print("Eval"); 5 })
// would print "Eval" twice
By using CPS/MiniScala (where arguments must be atoms) or by doing:

scala
Copy
Edit
val tmp = argExpr
tmp + 1
we ensure argExpr is evaluated exactly once, preserving semantics.

</details>
Question 15: Contification
Prompt: Explain the concept of contification. Under what conditions can a CPS/MiniScala function be contified? Why is this transformation beneficial? Give a small example.

<details><summary>Answer</summary>
Contification is transforming a function into a continuation when it is always called in tail position and never used as a first-class value.
Conditions:

The function is only ever invoked in tail-call position.

There are no non-tail calls or references to its closure.

Benefits:

Eliminates allocation of continuation closures.

Enables further inlining and simplification.

Example:

scala
Copy
Edit
// Before contification
def loop_c(c_ret, n) =
  if (n <= 0) c_ret(0)
  else loop_c(c_ret, n-1)

// After contification (direct tail recursion elimination)
def loop(n) =
  if (n <= 0) 0
  else loop(n-1)
</details>
Week 8 - Dataflow Analysis
(Based on week8-1.pdf)

Question 16: Available Expressions
Prompt: Consider the following CFG. Calculate the set of available expressions at the entry and exit of each node using dataflow analysis. Assume the set of all non-trivial expressions is {a+b, c-d, a*b}. Show the initial state and the state after each iteration until a fixed point is reached.

yaml
Copy
Edit
  Node 1: t1 = a + b
      |
      V
  Node 2: t2 = c - d
      |
      V
  Node 3: if (...)
     /   \
    V     V
Node 4: a = 5   Node 5: t3 = a * b
     \   /
      V
  Node 6: t4 = a + b
<details><summary>Answer</summary>
Initial (IN = ∅ for all, OUT = all expressions):

Node	IN	OUT
1	∅	{a+b}
2	{a+b}	{a+b, c-d}
3	{a+b, c-d}	{a+b, c-d}
4	{a+b, c-d}	{c-d}
5	{a+b, c-d}	{a+b, c-d, a*b}
6	{c-d, a*b}	{a+b, c-d, a*b}

Fixed point reached after 2 iterations.

</details>
Question 17: Live Variables
Prompt: For the CFG in Question 16, perform a live variable analysis. Calculate the set of live variables at the entry and exit of each node. Assume the variables involved are a, b, c, d, t1, t2, t3, t4. Show the initial state and the state after each iteration until a fixed point is reached. What is one potential optimization enabled by this analysis?

<details><summary>Answer</summary>
Fixed point:

Node	IN	OUT
1	{a, b, c, d}	{a, b, c, d}
2	{a, b, c, d}	{a, b, c, d}
3	{a, b, c, d}	{a, b, c, d}
4	{c, d}	{c, d}
5	{a, b, c, d}	{a, b, c, d}
6	{a, b, c, d}	∅

Optimization: Dead store elimination of assignments to variables that are not live afterwards.

</details>
Question 18: Reaching Definitions
Prompt: Perform a reaching definitions analysis on the following CFG. Assign a unique label to each definition (e.g., d1: x=1, d2: y=2, etc.). Calculate the set of definitions reaching the entry and exit of each node. How can the results be used for constant propagation?

yaml
Copy
Edit
    d1: x = 1
       |
       V
    d2: y = 2
       |
       V
Loop Header:
       |
       V
    Node A: if (x < 10)
      /    \
     V      V
d3: y = x + y   Node B (Exit)
     |
     V
d4: x = x + 1
     |
     V
    Goto Loop Header
<details><summary>Answer</summary>
Fixed point:

Node	IN	OUT
d1	∅	{d1}
d2	{d1}	{d1,d2}
LoopHdr	{d1,d2,d3,d4}	{d1,d2,d3,d4}
Node A	{d1,d2,d3,d4}	{d1,d2,d3,d4}
d3	{d1,d2,d3,d4}	{d1,d2,d3,d4}
d4	{d1,d2,d3,d4}	{d1,d2,d3,d4}
Exit B	{d1,d2,d3,d4}	{d1,d2,d3,d4}

Use for constant propagation: If at a use site only one reaching definition assigns a constant, that constant can replace the variable.

</details>
Question 19: Very Busy Expressions
Prompt: Explain the concept of "very busy expressions". What optimization does this analysis enable? For the following CFG, determine the set of very busy expressions at the entry and exit of each node. Assume the expressions are {a+b, x*y}.

yaml
Copy
Edit
  Node 1: Entry
      |
      V
  Node 2: if (...)
     /   \
    V     V
Node 3: z = a + b  Node 4: w = a + b
     \   /
      V
  Node 5: q = x * y
      |
      V
  Node 6: Exit
<details><summary>Answer</summary>
Very busy expressions are those that are guaranteed to be evaluated along every path to a use, enabling early code motion or hoisting.

Fixed point:

Node	IN	OUT
1	{a+b, x*y}	{a+b, x*y}
2	{a+b, x*y}	{a+b, x*y}
3	{a+b, x*y}	{x*y}
4	{a+b, x*y}	{x*y}
5	{x*y}	∅
6	∅	∅

</details>
Question 20: Speeding Up Dataflow Analysis
Prompt: Briefly explain two techniques used to speed up dataflow analysis computations. For instance, how does using basic blocks as CFG nodes help? How can bit vectors be used to represent sets efficiently for these analyses?

<details><summary>Answer</summary>
Basic‐block granularity: Grouping statements into basic blocks reduces the number of graph nodes, decreasing iterations.

Bit vectors: Represent sets (e.g., gen/kill) as bit vectors so that union/intersection/kill operations become fast bitwise operations.

</details>
Week 10 - Register Allocation
(Based on week10-2.pdf)

Question 21: Interference Graph
Prompt: Given the following simple program snippet (assuming variables a, b, c, d, t are virtual registers) and the liveness information provided, construct the interference graph. Nodes should include the virtual registers.

Instruction	Live Out
t = a + b	{a, b, t}
c = t * 2	{a, b, c}
d = a - b	{c, d}
print c	{d}
print d	{}

<details><summary>Answer</summary>
Interference edges:

Between t and a,b

Between c and a,b at second instruction

etc.

Graph (adjacency):

makefile
Copy
Edit
a: b, t, c, d
b: a, t, c, d
t: a, b
c: a, b, d
d: a, b, c
</details>
Question 22: Graph Coloring by Simplification
Prompt: Using the interference graph from Question 21, attempt to color it using 
𝐾
=
2
K=2 colors (registers R1, R2) via the simplification heuristic. Show the stack of removed nodes and the final coloring if successful. If simplification gets stuck (no node has fewer than K neighbors), explain what the next step would be.

<details><summary>Answer</summary>
Removal order (stack):

Remove t (degree 2)

Remove c (degree 2)

Remove d (degree 2)

Remove a (degree 2)

Remove b (degree 2)

No node found with degree < 2 ⇒ simplification stuck ⇒ spill a node (e.g., pick the one with lowest spill cost) and restart coloring.

</details>
Question 23: Spilling
Prompt: Suppose during graph coloring with 
𝐾
K registers, the simplification phase gets stuck, and all remaining nodes have a degree of 
𝐾
K or more. Explain the concept of "spilling" a node. How is a node typically chosen for spilling (mention spill cost calculation)? What needs to happen to the code after a variable is spilled?

<details><summary>Answer</summary>
Spilling: Select a variable to store in memory instead of in a register.

Spill cost: frequency of use × estimated load/store cost.

After spill: Insert load before each use and store after each definition, then rebuild interference graph.

</details>
Question 24: Coalescing
Prompt: What is the purpose of coalescing in register allocation? Consider the instruction v1 = v2. Under what condition can v1 and v2 be coalesced? Explain one conservative coalescing heuristic (e.g., Briggs or George) and why it is considered "safe" but potentially "conservative".

<details><summary>Answer</summary>
Coalescing eliminates move instructions by assigning the same register to both source and destination.

Condition: Interfering neighbors of v1 ∪ neighbors of v2 < 
𝐾
K.

Briggs heuristic: Only coalesce if the combined interference degree is < 
𝐾
K. Safe (no extra spill risk) but conservative (may miss some safe coalescings).

</details>
Question 25: Linear Scan Register Allocation
Prompt: Briefly describe the Linear Scan register allocation algorithm. What are the main steps involved? How does it handle the situation when no registers are free for a new live range? What is a key advantage and a potential disadvantage compared to graph coloring?

<details><summary>Answer</summary>
Sort live ranges by start point.

Maintain an active set of ranges currently in registers.

When a new range begins, expire old ones, then if free register exists assign it; else spill the range with latest end.
Advantage: 
𝑂
(
𝑛
)
O(n) time, simple.
Disadvantage: May spill more than optimal graph coloring.

</details>
Week 11 - Instruction Scheduling
(Based on week11-2.pdf)

Question 26: Instruction Dependencies
Prompt: Identify the types of dependencies (True/RAW, Anti/WAR, Output/WAW) between the following pairs of instructions. Can renaming registers remove any of these dependencies? If so, which ones and why?

a)

ini
Copy
Edit
R1 = R2 + R3
R4 = R1 * R5
b)

ini
Copy
Edit
R1 = R2 / R3
R2 = R4 + R5
c)

ini
Copy
Edit
R1 = R2 * R3
R1 = R4 - R5
<details><summary>Answer</summary>
a) RAW on R1 ⇒ cannot remove.
b) WAR (first writes R1 from R2, second reads R2) ⇒ can remove by renaming the second R2.
c) WAW on R1 ⇒ can remove by renaming the second R1.

</details>
Question 27: Dependence Graph
Prompt: Construct the dependence graph for the following sequence of instructions. Label the edges with the type of dependence (True, Anti, Output). Assume a, b, c, d are registers.

makefile
Copy
Edit
1: a = Mem[addr1]
2: b = a * 2
3: c = Mem[addr2]
4: a = b + c  // Reuses register 'a'
5: d = a / 3
6: Mem[addr3] = d
<details><summary>Answer</summary>
1→2: RAW on a

1→4: WAR on a

2→4: RAW on b

3→4: RAW on c

4→5: RAW on a

5→6: RAW on d

1→4: WAW on a (also)

</details>
Question 28: List Scheduling
Prompt: Consider the following instruction sequence and latencies:

Load (Ld): 3 cycles

Add (Add): 1 cycle

Multiply (Mul): 2 cycles

makefile
Copy
Edit
1: R1 = Ld Mem[A]
2: R2 = Ld Mem[B]
3: R3 = Add R1, R2
4: R4 = Ld Mem[C]
5: R5 = Mul R3, R4
6: R6 = Ld Mem[D]
7: R7 = Mul R3, R6
Perform list scheduling for this basic block. Assume the priority of an instruction is the length of the longest latency-weighted path from it to a root node. Show the state of the ready and active lists at each cycle, and the final schedule. What is the total number of cycles required?

<details><summary>Answer</summary>
Cycle-by-cycle ready/active lists omitted for brevity.
Final schedule:

sql
Copy
Edit
Cycle 0–2:   1:Ld A  
Cycle 1–3:   2:Ld B  
Cycle 3–3:   3:Add  
Cycle 3–5:   4:Ld C  
Cycle 5–6:   5:Mul R3,R4  
Cycle 4–6:   6:Ld D  
Cycle 6–8:   7:Mul R3,R6  
Total cycles: 9

</details>
Question 29: Instruction Scheduling vs. Register Allocation
Prompt: Explain the conflict between instruction scheduling and register allocation. Why does performing register allocation first potentially hinder scheduling? Why does performing scheduling first potentially get undone by register allocation? What is a common approach to handle this interaction?

<details><summary>Answer</summary>
Scheduling first may increase live ranges ⇒ hurts allocation.

Allocation first may insert spills ⇒ hurts scheduling.
Common approach: Iteratively co‐optimize (e.g., integrated register scheduling) or use heuristics to balance both.

</details>
Question 30: Pipeline Stalls
Prompt: Why do modern pipelined processors benefit from instruction scheduling? Give two reasons why pipeline stalls might occur, referencing instruction latencies and data dependencies.

<details><summary>Answer</summary>
Benefits: Reduces bubbles by reordering independent instructions.
Stall reasons:

Data hazards (RAW/WAW/WAR) ⇒ waiting for operand availability.

Structural hazards ⇒ resource conflicts (e.g., multiple loads).

</details>
Week 12 - Tail Calls & Interpreters/VMs
(Based on week12-1.pdf & week12-2.pdf)

Question 31: Identifying Tail Calls
Prompt: In the following Scala-like code, identify which of the function calls (g, h, k, sum, f) are tail calls. Explain your reasoning for each.

scala
Copy
Edit
def f(x: Int, y: Int): Int = {
  if (x > y) {
    g(x-1) + 1 // Call 1 to g
  } else if (x < y) {
    val temp = h(y) // Call 2 to h
    k(temp)       // Call 3 to k
  } else {
    sum(x, y)     // Call 4 to sum
  }
}

def g(a: Int): Int = f(a, a*2) // Call 5 to f
<details><summary>Answer</summary>
Call 1 (g(x-1)): Not tail (result +1 afterward).

Call 2 (h(y)): Not tail (binding to temp).

Call 3 (k(temp)): Tail (returned directly).

Call 4 (sum(x,y)): Tail.

Call 5 (f(a,a*2)): Tail (direct return).

</details>
Question 32: Tail Call Elimination (TCE)
Prompt: Explain the core problem that Tail Call Elimination (TCE) solves, especially in functional languages that use recursion for looping. How does TCE work conceptually? Why can the activation frame of the caller be deallocated before a tail call?

<details><summary>Answer</summary>
TCE prevents unbounded stack growth by reusing the caller’s frame for a tail call. Since nothing in the caller’s context is needed after the call, its frame can be popped before jumping to the callee.

</details>
Question 33: TCE Implementation Techniques
Prompt: Briefly describe two different techniques for implementing TCE when the target environment (like C without guaranteed TCE, or the JVM) doesn't directly support it. Choose from: Single Function Approach, Trampolines, or Baker's Technique (CPS). Explain the basic idea of each chosen technique.

<details><summary>Answer</summary>
Trampolines: Functions return a “thunk” describing the next call; a loop repeatedly invokes thunks without growing the native stack.

Baker’s Technique (CPS): Compile to CPS so all calls are tail calls; the language runtime uses direct jumps for CPS calls.

</details>
Question 34: Interpreters vs. Virtual Machines
Prompt: What is the fundamental difference between a tree-based interpreter and a bytecode virtual machine (VM) in terms of how they execute a program? What is one advantage of using a VM over directly compiling to native code?

<details><summary>Answer</summary>
Tree-based interpreter: Walks the AST at runtime.

Bytecode VM: Executes a linear sequence of compact instructions.
Advantage: Portability—bytecode runs on any platform with the VM.

</details>
Question 35: VM Optimization: Threaded Code
Prompt: Explain the difference between a standard switch-based VM dispatch loop and direct threaded code (using GCC's labels-as-values extension). Why is threaded code generally faster? Draw a simple diagram illustrating the control flow for a sequence of 3 instructions (e.g., ADD, LOAD, JUMP) in both approaches.

<details><summary>Answer</summary>
Switch VM: uses a switch on opcode → indirect jump each instruction.
Threaded code: embeds direct labels and uses computed goto → one indirect jump.
Threaded code reduces dispatch overhead by half.

</details>
Question 36: MiniScalaVM Function Calls
Prompt: Describe how function arguments are passed and how the caller's context (base registers, return address) is saved during a non-tail CALL instruction in MiniScalaVM. How does this differ for a tail call (TCALL)?

<details><summary>Answer</summary>
CALL: Push return address and old frame pointer, set up new frame.

TCALL: Reuses caller’s frame—no push of return address/frame pointer.

</details>
Week 13 - Memory Management
(Based on week13-1 (1).pdf)

Question 37: Explicit vs. Implicit Deallocation
Prompt: What are the main problems associated with explicit memory deallocation (e.g., free in C)? What is the fundamental assumption underlying implicit deallocation (garbage collection)? Does implicit deallocation completely prevent memory leaks? Explain.

<details><summary>Answer</summary>
Problems with explicit: Dangling pointers, double frees, manual burden.

GC assumption: Unreachable objects can never be used again.

Leaks still possible: If reachable objects reference unused data (“logical leaks”).

</details>
Question 38: Reference Counting
Prompt: Explain the reference counting garbage collection technique. What happens when an object's reference count drops to zero? What is the major limitation of simple reference counting? Illustrate this limitation with a diagram involving cyclic data structures.

<details><summary>Answer</summary>
When count → 0: Object is reclaimed immediately.

Limitation: Cannot collect cycles (two objects refer to each other).

less
Copy
Edit
A ↔ B   // both have count=1 → never dropped to 0
</details>
Question 39: Mark & Sweep GC
Prompt: Describe the two main phases of Mark & Sweep garbage collection. How are reachable objects typically identified and marked during the first phase? What happens during the second phase? Does this technique handle cyclic structures correctly?

<details><summary>Answer</summary>
Mark: Trace from roots, mark reachable objects.

Sweep: Scan heap, reclaim unmarked objects.
Cycles: Correctly handled since any cycle unreachable from roots is not marked.

</details>
Question 40: Copying GC (Cheney's Algorithm)
Prompt: Explain the basic principle of a copying garbage collector, including the concepts of from-space and to-space. Briefly describe how Cheney's algorithm performs the collection using a breadth-first traversal and avoids deep recursion. What is a "forwarding pointer" used for in this context?

<details><summary>Answer</summary>
From/To spaces: Two halves of heap; live objects are copied from from-space to to-space.

Cheney: Uses two pointers in to-space (scan & free) to traverse level-by-level.

Forwarding pointer: Replaces original object header in from-space to point to its copy.

</details>
Question 41: Generational GC
Prompt: What is the "generational hypothesis" that motivates generational garbage collection? Describe the basic structure of a two-generation (young/old) collector, including the concepts of minor and major collections, and object promotion. What is the role of the "remembered set" or "card marking" in generational GC?

<details><summary>Answer</summary>
Hypothesis: Most objects die young.

Two-gen: Young gen (frequent minor collections), Old gen (infrequent major).

Promotion: Surviving objects moved from young → old.

Remembered set: Tracks pointers from old → young to avoid scanning entire old space.

</details>
Question 42: Fragmentation
Prompt: Define external fragmentation and internal fragmentation in the context of heap memory management. Which type of fragmentation is typically eliminated by copying garbage collectors? Why?

<details><summary>Answer</summary>
External: Free memory is split into small non-contiguous holes.

Internal: Allocated blocks are larger than requested.
Copying GC eliminates external fragmentation by compacting live objects.

</details>
Week 14 - Object-Oriented Languages
(Based on week14-1.pdf)

Question 43: Object Layout (Single vs. Multiple Inheritance)
Prompt: Explain the simple object layout strategy commonly used in single-inheritance languages like Java. Why does this strategy ensure that accessing a field x defined in class A works correctly even when operating on an object of a subclass B extends A? Why does this simple strategy become problematic with multiple inheritance? Briefly mention one approach used to handle layout in multiple inheritance scenarios (e.g., unidirectional layout with waste, bidirectional layout, accessor methods).

<details><summary>Answer</summary>
Single inheritance layout: Subclass memory = superclass’s fields first, then subclass’s. Field offsets remain fixed.

Multiple inheritance issue: Two parents may define field x at different offsets.

Solution example: Use accessor methods or bidirectional layout with padding to resolve offset conflicts.

</details>
Question 44: Method Dispatch (VMTs)
Prompt: Describe how Virtual Method Tables (VMTs) are used for dynamic method dispatch in single-inheritance languages. Draw a simple diagram showing the VMTs and object layouts for the following classes, similar to slide 23:

java
Copy
Edit
class Point { int x; void move(int dx) { /*...*/ } }
class ColorPoint extends Point { int color; @Override void move(int dx) { /*...*/ } void changeColor() { /*...*/ } }
Show how a call p.move(5) would be dispatched if p refers to a ColorPoint object.

<details><summary>Answer</summary>
pgsql
Copy
Edit
[Point VMT]      [ColorPoint VMT]
-------------    ------------------
| move → Point.move | move → CP.move |
                    | changeColor → CP.cc |

Object layout (ColorPoint):
| VMT ptr → ColorPoint VMT |
| x                         |
| color                     |

Dispatch p.move(5):
1. Load VMT from p
2. Fetch vmt[0] (move slot)
3. Jump to ColorPoint.move
</details>
Question 45: Method Dispatch (Multiple Subtyping)
Prompt: Why can't the simple VMT approach be directly used for dispatching calls through Java interfaces or in languages with multiple inheritance? Briefly explain the concept of using a global dispatching matrix. What are the major drawbacks of a naive dispatch matrix, and what properties of the matrix are exploited by compression techniques like column displacement or compact dispatch tables?

<details><summary>Answer</summary>
Interface dispatch: Receiver may implement many interfaces ⇒ no single VMT per interface method.

Dispatch matrix: Rows = receiver type, cols = method signature.

Drawbacks: Very sparse, large memory.

Compression: Use column displacement to pack sparse rows/cols, exploit locality and few methods per class.

</details>
Question 46: Inline Caching
Prompt: What is the motivation behind inline caching? Explain how basic inline caching works, including the initial state of a call site, what happens on the first call, and what happens on subsequent calls (both cache hits and misses). How does Polymorphic Inline Caching (PIC) improve upon basic inline caching for call sites that frequently switch between a small number of receiver types?

<details><summary>Answer</summary>
Motivation: Speed up dynamic dispatch by remembering last lookup.

Basic IC:

Initially, no target cached.

On first call, do full method lookup and store in cache.

Subsequent calls: if receiver type matches cache, direct jump; else fallback to lookup and update cache.

PIC: Stores a small fixed-size list of (type→target) pairs to handle multiple receiver types without fallback to slow path each time.

</details>
Question 47: Membership Testing
Prompt: What is the "membership test" problem in OO languages? Explain one technique used for efficient membership testing in single subtyping contexts (e.g., Relative Numbering or Cohen's Encoding). Illustrate with a small class hierarchy.

<details><summary>Answer</summary>
Problem: Check at runtime if object’s class is a subtype of a given interface/parent.

Relative Numbering: Assign intervals to classes so that subtype’s interval ⊆ supertype’s.

css
Copy
Edit
A:[1..10], B:[2..5], C:[6..9]
B ∈ A if 2 ≥ 1 && 5 ≤ 10.

</details>
Question 48: Membership Testing (Multiple Subtyping)
Prompt: Why do techniques like simple Relative Numbering or Cohen's Encoding fail in the presence of multiple subtyping (e.g., with interfaces or multiple inheritance)? Name one technique designed to handle membership testing efficiently in multiple subtyping scenarios (e.g., Range Compression, Packed Encoding, PQ Encoding). You don't need to explain it in detail, just identify a valid technique.

<details><summary>Answer</summary>
Simple numbering fails because a class may appear in multiple disjoint intervals.
Valid technique: Packed Encoding (combines multiple intervals in a compressed form).

</details>
