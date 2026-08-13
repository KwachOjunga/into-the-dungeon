**Inside LLVM: Engineering a Target-Independent Compiler**  
**Part 3 — Reusing Optimizations Across Architectures**  
**The Middle-End Contract**

### 1. The Question

Parts 1 and 2 established two foundational abstractions. LLVM is organized as a collection of
reusable libraries rather than a monolithic compiler, and LLVM IR serves as the stable contract 
that lets many languages feed a single optimizer.

A new question now arises.

If the same IR is produced by frontends as different as Clang, Rustc, and the Swift compiler, why
should the *same* sequence of optimizations improve all of them? More precisely:

**Why can the same optimization improve programs written in completely different languages and still produce good code for completely different processors?**

The answer lies in what the middle-end(the contract - LLVMIR) deliberately refuses to know.

### 2. The Limitation

Once a program has been lowered to LLVM IR, a large collection of analyses and transformations 
become possible: 
dead-code elimination, common-subexpression elimination, sparse conditional constant propagation,
loop-invariant code motion, inlining, vectorization, and many others.

If each of these transformations had to be written with intimate knowledge of a particular 
language or a particular micro-architecture, the reuse promised by Parts 1 and 2 would collapse. 
An inliner that understood Rust’s ownership semantics would be useless for C. A loop optimizer that 
hard-coded the characteristics of an Intel Golden Cove branch predictor would generate poor code 
for an ARM Neoverse core or a simple in-order RISC-V microcontroller.

The middle-end therefore faces a new engineering pressure:

It must improve code *without* knowing the source language that produced the IR and *without* 
knowing most of the micro-architectural details of the processor that will eventually execute it.

Traditional optimizers often failed this test. Many were deeply entangled with the frontend that fed them 
or with the backend they fed. The result was duplicated effort and optimizations that could not travel.

### 3. The New Abstraction

LLVM’s response is the **middle-end contract**: a suite of analyses and transformations that operate almost 
exclusively on the properties of LLVM IR itself, together with a narrow, carefully controlled set of target 
queries.

The optimizer is allowed to rely on:

- the SSA property,
- the explicit control-flow graph,
- the type system,
- the use-def chains,
- and a small number of target descriptors (DataLayout, TargetLibraryInfo, TargetTransformInfo).

It is *not* allowed to assume:

- a particular finite register file,
- a particular pipeline depth or issue width,
- the presence or accuracy of a branch predictor,
- a particular memory-reordering model,
- or the existence of speculative execution with specific side effects.

Assumptions that are implementation specific.

By refusing to know these facts, the same pass can run on IR that originated in C, Rust,
or Swift and later be lowered to x86-64, AArch64, or RISC-V.

I think it is quite accurate to say that the IR explicitly refuses to rely on 
specific information unless it is absolutely neccessary.

### 4. How It Works

The middle-end is organized as a pipeline of **passes**. Each pass is either an analysis 
(computing information such as dominator trees, loop nests, or alias sets) or a transformation 
(rewriting the IR while preserving semantics).

Modern LLVM uses the New Pass Manager. Analyses are requested on demand and cached; transformations 
declare which analyses they preserve or invalidate. This machinery lets the compiler avoid recomputing
expensive results and keeps the pipeline composable.

Two further design decisions keep the middle-end largely target-independent.

**Canonicalization**  
Many passes exist solely to drive the IR toward a smaller set of preferred forms. InstCombine, SimplifyCFG, 
and related passes rewrite instruction sequences into canonical patterns so that later passes see less 
surface variation. Canonical forms are chosen because they are easier to reason about, not because they 
match any particular machine.

**Controlled target queries**  
When a pass truly needs hardware knowledge—most often for cost modeling—it consults TargetTransformInfo (TTI) or
TargetLibraryInfo (TLI). Vectorizers ask TTI about legal vector widths and the relative cost of shuffles. The inliner 
may ask about call overhead. These queries are deliberate, narrow leaks; the great majority of transformations never 
make them.

#### What the IR Abstracts Away

LLVM IR presents a simplified model of execution that deliberately erases many hardware realities:

- **Infinite virtual registers.** Real processors have small, architecturally visible register files and must spill. 
  The middle-end largely ignores this constraint; register allocation occurs later.
- **Simple control flow.** IR has basic blocks and terminators. It does not expose branch-predictor tables, 
  return-address stacks, or the speculative execution machinery that modern CPUs use to keep pipelines full.
- **Sequential consistency within a thread (with explicit atomics).** Real machines reorder loads and stores. 
  x86 provides a relatively strong model; ARM and RISC-V provide weaker models that require more barriers. 
  The middle-end works with the IR’s memory model and leaves the insertion of the precise fences to later stages
  or to target-specific lowering.
- **Uniform instruction costs.** An `add` in IR is just an `add`. On a real machine its latency, throughput, 
  and ability to execute on multiple ports vary dramatically. Only cost-model-driven passes consult TTI 
  for this information.

#### Concrete Architectural Differences the Middle-End Mostly Ignores

Consider three capabilities that hardware exposes to varying degrees and that aggressive programmers
sometimes exploit directly.

**Branch prediction and speculative execution**  
High-performance x86 and ARM cores invest heavily in predictors and execute far beyond unresolved branches. 
A misprediction is expensive, so the shape of control flow matters. Some code bases are hand-tuned to make 
branches more predictable or to use conditional moves to avoid them. LLVM IR has no first-class notion of
“this branch is highly predictable” or “this value was produced by speculation.” The middle-end may convert 
branches into selects (or vice versa) based on simple heuristics and TTI cost models, but it does not try to
model predictor state. The final decision about branch versus conditional move is left to the backend, where 
target-specific knowledge is available.

**Instruction reordering and out-of-order execution**  
Out-of-order cores (most modern x86 and high-end ARM designs) dynamically reorder independent instructions to 
hide latency. In-order cores (many embedded RISC-V implementations, older ARM cores) do not. 
The middle-end performs *static* reordering only when it is profitable under a simplified cost model; 
it does not attempt to schedule for a particular out-of-order window or reservation-station configuration. 
Those decisions belong to the machine scheduler, which runs after instruction selection and has access to 
detailed pipeline models.

**Explicit software pipelining or prefetching**  
Some architectures and some performance-sensitive code sequences rely on software pipelining or on explicit prefetch instructions. LLVM IR has no direct encoding of “this load should be issued two iterations early.” Loop transformations 
may create the conditions under which a later backend can software-pipeline, but the middle-end itself does not emit
architecture-specific prefetch intrinsics unless a pass has been told (via TTI or target hooks) that they are profitable.

In each case the IR and the middle-end provide a portable substrate. The features that differ most across micro-architectures are consulted late and through narrow interfaces, or are left entirely to the backend.

### 5. Design Trade-offs

The middle-end’s refusal to know hardware details buys massive reuse: the same LoopVectorize pass, the same GVN, 
the same inliner, improve C, Rust, and Swift alike, and the resulting IR can be handed to any LLVM backend.

The cost is that some optimization opportunities are left on the table until later stages, and a few must be handled 
by target-specific passes. A purely target-agnostic middle-end cannot, for example, know that a particular sequence 
will saturate a specific execution port on a particular CPU, nor can it know the exact misprediction penalty of a given
branch.

LLVM accepts this trade-off because the alternative—writing and maintaining separate optimization pipelines for every
language and every major micro-architecture—reintroduces the N × M problem that Part 1 set out to solve.

### 6. The Abstraction Leaks

The middle-end is only *mostly* target-independent. Three important leaks exist:

- **DataLayout** — already present in the IR (Part 2). Alignment and pointer-size decisions affect many optimizations.
- **TargetLibraryInfo** — tells the optimizer which library calls exist and what they mean (e.g., whether `memcpy` may be
  assumed, or whether a math function is available).
- **TargetTransformInfo** — the primary channel for cost modeling. Vectorization, inlining heuristics, and some
  simplification decisions consult it.

In addition, a few passes are allowed to behave differently when they know they are compiling for a particular target, 
and some language frontends inject target-aware attributes early. These leaks are intentional. They let the optimizer 
remain useful on real hardware without forcing every transformation to become target-specific.

### 7. Source Tour

The middle-end lives primarily in three places:

- `llvm/lib/Transforms/` — the transformations themselves (Scalar, Vectorize, IPO, Utils, \ldots)
- `llvm/lib/Analysis/` — analyses that transformations query (LoopInfo, AliasAnalysis, ScalarEvolution, \ldots)
- `llvm/lib/Passes/` — the New Pass Manager infrastructure and the pipelines that assemble analyses and 
  transformations into `-O2`, `-O3`, etc.

Target queries surface through headers in `llvm/include/llvm/Analysis/` (especially TargetTransformInfo) and are 
implemented by each backend.

### 8. Looking Ahead

The middle-end can now improve IR without knowing the original language and without knowing most 
micro-architectural details. That IR, however, still describes an ideal machine: infinite registers, 
uniform instructions, and simple control flow.


Real processors have finite registers, illegal types and operations, complex calling conventions, 
and idiosyncratic instruction encodings. The next engineering problem is therefore:

**How does LLVM turn ideal IR instructions into legal machine instructions for many different ISAs without forcing the middle-end to know about those ISAs?**

That problem forces the introduction of instruction selection and Machine IR — the subject of Part 4.

### Design Principle #3 — Optimize Against a Simplified Model

When many different hardware implementations must be supported, perform the bulk of optimization against a simplified, mostly hardware-agnostic model, and consult detailed target information only through narrow, explicit interfaces.

LLVM IR erases finite registers, pipeline structure, branch-predictor behavior, and most memory-reordering details so that the same passes can serve many languages and many architectures. The few facts that must be known are obtained through DataLayout, TTI, and TLI. Everything else is deferred until the backend, where target-specific knowledge is unavoidable.

### Series Thread

Every abstraction in LLVM exists because the previous one could not adequately contain a particular form of complexity.

- The monolithic compiler could not solve the N × M problem → libraries (Part 1).  
- ASTs and assembly could not serve as a shared medium → LLVM IR (Part 2).  
- Language-specific or architecture-specific optimizers could not be reused → a middle-end that operates 
  on a simplified, mostly target-independent model of the IR (Part 3).

The next abstraction will confront the fact that this simplified model must eventually be reconciled with the 
messy reality of actual instruction sets.
