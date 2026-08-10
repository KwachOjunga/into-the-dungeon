
```mermaid
%% ============================================================
%% COMPILER PIPELINE
%% Detailed view of the major compiler stages and their
%% intermediate representations.
%% ============================================================

flowchart TD

    %% ========================================================
    %% SOURCE
    %% ========================================================

    SRC["Source Program<br/><b>foo.c</b>"]

    PP["Preprocessing<br/><br/>
    #include expansion<br/>
    #define expansion<br/>
    Conditional compilation"]

    SRC --> PP


    %% ========================================================
    %% FRONT END
    %% ========================================================

    subgraph FE["FRONT-END — LANGUAGE DEPENDENT"]
        direction TB

        %% ----------------------------------------------------
        %% Lexical Analysis
        %% ----------------------------------------------------

        subgraph LEXICAL["1. Lexical Analysis"]
            direction TB

            CHAR["Source Characters"]

            SCAN["Scanner / Lexer<br/><br/>
            Recognize lexemes<br/>
            Keywords<br/>
            Identifiers<br/>
            Literals<br/>
            Operators<br/>
            Punctuation"]

            TOK["Token Stream<br/><br/>
            IDENTIFIER<br/>
            INTEGER_LITERAL<br/>
            PLUS<br/>
            LPAREN<br/>
            ..."]

            CHAR --> SCAN --> TOK
        end

        %% ----------------------------------------------------
        %% Syntax Analysis
        %% ----------------------------------------------------

        subgraph SYNTAX["2. Syntax Analysis"]
            direction TB

            PARSER["Parser<br/><br/>
            Grammar matching<br/>
            Recursive descent / LR / PEG<br/>
            Error recovery"]

            CST["Concrete Syntax Tree<br/><br/>
            Grammar structure"]

            AST["Abstract Syntax Tree<br/><br/>
            Expressions<br/>
            Statements<br/>
            Declarations<br/>
            Functions"]

            TOK --> PARSER
            PARSER --> CST
            CST --> AST
        end

        %% ----------------------------------------------------
        %% Semantic Analysis
        %% ----------------------------------------------------

        subgraph SEMANTIC["3. Semantic Analysis"]
            direction TB

            NAME["Name Resolution<br/><br/>
            Identifier lookup<br/>
            Scope resolution<br/>
            Declaration binding"]

            TYPES["Type Checking<br/><br/>
            Type inference<br/>
            Implicit conversions<br/>
            Operator compatibility<br/>
            Function signatures"]

            SYMBOLS["Symbol Table<br/><br/>
            Variables<br/>
            Functions<br/>
            Types<br/>
            Storage information"]

            CONST["Constant / Compile-Time Analysis<br/><br/>
            Constant expressions<br/>
            Enum values<br/>
            Compile-time constraints"]

            AST --> NAME
            AST --> TYPES
            AST --> SYMBOLS
            AST --> CONST
        end

        %% ----------------------------------------------------
        %% IR Generation
        %% ----------------------------------------------------

        subgraph IRGEN["4. Intermediate Representation Generation"]
            direction TB

            LOWER["AST Lowering<br/><br/>
            High-level constructs → simpler operations"]

            CFG0["Initial Control-Flow Graph<br/><br/>
            Basic blocks<br/>
            Branches<br/>
            Returns"]

            IR0["Initial IR<br/><br/>
            Three-address operations<br/>
            Loads / Stores<br/>
            Arithmetic<br/>
            Calls / Branches"]

            SSA["SSA Construction<br/><br/>
            Single assignment<br/>
            Φ nodes<br/>
            Explicit data-flow"]

            LOWER --> CFG0
            CFG0 --> IR0
            IR0 --> SSA
        end

        %% Front-end internal connections
        SCAN --> PARSER
        AST --> LOWER
        NAME --> LOWER
        TYPES --> LOWER
        SYMBOLS --> LOWER
        CONST --> LOWER
    end

    PP --> CHAR

%% ----------------------------------------------------
%%  partition from here
%% ----------------------------------------------------

    %% ========================================================
    %% MIDDLE END
    %% ========================================================

    subgraph ME["MIDDLE-END — TARGET INDEPENDENT"]
        direction TB

        %% ----------------------------------------------------
        %% IR representation
        %% ----------------------------------------------------

        subgraph IR["INTERMEDIATE REPRESENTATION"]
            direction TB

            IRMODULE["Module<br/><br/>
            Functions<br/>
            Global variables<br/>
            Types<br/>
            Metadata"]

            IRFUNC["Function IR<br/><br/>
            Basic Blocks<br/>
            Instructions<br/>
            SSA Values"]

            IRMODULE --> IRFUNC
        end

        %% ----------------------------------------------------
        %% Analysis
        %% ----------------------------------------------------

        subgraph ANALYSIS["5. Program Analysis"]
            direction TB

            CFG["Control-Flow Analysis<br/><br/>
            Basic blocks<br/>
            Predecessors / successors<br/>
            Dominators<br/>
            Loop structure"]

            DATAFLOW["Data-Flow Analysis<br/><br/>
            Def-use chains<br/>
            Liveness<br/>
            Reaching definitions"]

            ALIAS["Alias Analysis<br/><br/>
            Memory dependencies<br/>
            Pointer relationships"]

            CALLGRAPH["Call-Graph Analysis<br/><br/>
            Caller / callee relationships<br/>
            Recursion<br/>
            Inlining candidates"]

            LOOP["Loop Analysis<br/><br/>
            Loop detection<br/>
            Induction variables<br/>
            Loop nesting"]

            IRFUNC --> CFG
            IRFUNC --> DATAFLOW
            IRFUNC --> ALIAS
            IRFUNC --> CALLGRAPH
            CFG --> LOOP
        end

        %% ----------------------------------------------------
        %% Canonicalization
        %% ----------------------------------------------------

        subgraph CANON["6. IR Canonicalization"]
            direction TB

            SIMPLIFY["CFG Simplification<br/><br/>
            Remove unreachable blocks<br/>
            Merge blocks<br/>
            Simplify branches"]

            INSTSIMPLIFY["Instruction Simplification<br/><br/>
            Normalize operations<br/>
            Simplify expressions<br/>
            Fold identities"]

            SSAOPT["SSA Cleanup<br/><br/>
            Simplify Φ nodes<br/>
            Remove redundant values<br/>
            Repair SSA form"]

            CFG --> SIMPLIFY
            DATAFLOW --> INSTSIMPLIFY
            SSA --> SSAOPT
        end

        %% ----------------------------------------------------
        %% Scalar Optimizations
        %% ----------------------------------------------------

        subgraph SCALAR["7. Scalar Optimizations"]
            direction TB

            CONSTFOLD["Constant Folding<br/><br/>
            Compile known expressions"]

            CONSTPROP["Constant Propagation<br/><br/>
            Propagate known values"]

            CSE["Common Subexpression Elimination<br/><br/>
            Reuse equivalent computations"]

            DCE["Dead Code Elimination<br/><br/>
            Remove unused computations"]

            COPY["Copy Propagation<br/><br/>
            Eliminate unnecessary copies"]

            GVN["Global Value Numbering<br/><br/>
            Detect equivalent values"]

            INSTCOMB["Instruction Combining<br/><br/>
            Replace operation sequences<br/>
            with simpler operations"]

            CONSTFOLD --> CONSTPROP
            CONSTPROP --> CSE
            CSE --> DCE
            COPY --> GVN
            GVN --> INSTCOMB
        end

        %% ----------------------------------------------------
        %% Memory Optimizations
        %% ----------------------------------------------------

        subgraph MEMORY["8. Memory Optimizations"]
            direction TB

            MEM2REG["Memory-to-Register Promotion<br/><br/>
            Promote stack variables<br/>
            into SSA values"]

            SROA["Scalar Replacement of Aggregates<br/><br/>
            Break structures into scalars"]

            LOADSTORE["Load / Store Optimization<br/><br/>
            Eliminate redundant memory operations"]

            DSE["Dead Store Elimination<br/><br/>
            Remove stores that cannot be observed"]

            LICM["Loop-Invariant Code Motion<br/><br/>
            Move invariant computations<br/>
            outside loops"]

            ALIAS --> LOADSTORE
            MEM2REG --> SROA
            LOADSTORE --> DSE
            LOOP --> LICM
        end

        %% ----------------------------------------------------
        %% Loop Optimizations
        %% ----------------------------------------------------

        subgraph LOOPS["9. Loop Optimizations"]
            direction TB

            LOOPUNROLL["Loop Unrolling<br/><br/>
            Replicate loop body"]

            LOOPROTATE["Loop Rotation<br/><br/>
            Restructure loop control"]

            INDUCTION["Induction Variable Optimization<br/><br/>
            Simplify loop counters"]

            VECTORIZ["Loop Vectorization<br/><br/>
            Scalar operations → vector operations"]

            LOOPUNROLL --> INDUCTION
            INDUCTION --> VECTORIZ
            LOOPROTATE --> LOOPUNROLL
        end

        %% ----------------------------------------------------
        %% Interprocedural Optimizations
        %% ----------------------------------------------------

        subgraph IPO["10. Interprocedural Optimization"]
            direction TB

            INLINE["Function Inlining<br/><br/>
            Replace calls with function body"]

            DEvirt["Devirtualization<br/><br/>
            Resolve indirect calls"]

            ARGPROP["Argument / Return Propagation<br/><br/>
            Propagate known values across calls"]

            GLOBALOPT["Global Optimization<br/><br/>
            Analyze module-wide state"]

            CALLGRAPH --> INLINE
            INLINE --> DEvirt
            DEvirt --> ARGPROP
            ARGPROP --> GLOBALOPT
        end

        %% ----------------------------------------------------
        %% Target-independent transformations
        %% ----------------------------------------------------

        subgraph TRANSFORM["11. IR Transformation Pipeline"]
            direction TB

            PIPE["Pass Pipeline<br/><br/>
            Analysis → Transform → Analysis<br/>
            Repeated until useful canonical form"]

            OPTIR["Optimized IR<br/><br/>
            Target-independent<br/>
            optimized program"]

            PIPE --> OPTIR
        end

        %% Middle-end broad connections
        IRFUNC --> PIPE

        SIMPLIFY --> PIPE
        INSTSIMPLIFY --> PIPE
        SSAOPT --> PIPE

        CONSTFOLD --> PIPE
        CONSTPROP --> PIPE
        CSE --> PIPE
        DCE --> PIPE
        COPY --> PIPE
        GVN --> PIPE
        INSTCOMB --> PIPE

        MEM2REG --> PIPE
        SROA --> PIPE
        LOADSTORE --> PIPE
        DSE --> PIPE
        LICM --> PIPE

        LOOPUNROLL --> PIPE
        LOOPROTATE --> PIPE
        INDUCTION --> PIPE
        VECTORIZ --> PIPE

        INLINE --> PIPE
        DEvirt --> PIPE
        ARGPROP --> PIPE
        GLOBALOPT --> PIPE
    end

    SSA --> IRMODULE


    %% ========================================================
    %% BACK END
    %% ========================================================

    subgraph BE["BACK-END — TARGET DEPENDENT"]
        direction TB

        %% ----------------------------------------------------
        %% Target information
        %% ----------------------------------------------------

        subgraph TARGETINFO["12. Target Description"]
            direction TB

            ISA["Target ISA<br/><br/>
            x86-64 / AArch64 / RISC-V / ..."]

            REGS["Register File<br/><br/>
            General-purpose registers<br/>
            Vector registers<br/>
            Special registers"]

            COST["Target Costs<br/><br/>
            Instruction latency<br/>
            Throughput<br/>
            Register pressure"]

            ISA --> REGS
            ISA --> COST
        end

        %% ----------------------------------------------------
        %% Target-independent lowering
        %% ----------------------------------------------------

        subgraph LOWERING["13. Target Lowering"]
            direction TB

            LEGALIZE["Legalization<br/><br/>
            Convert unsupported IR operations<br/>
            into target-supported operations"]

            TYPELEGAL["Type Legalization<br/><br/>
            Adjust unsupported data types"]

            CUSTOM["Target-Specific Lowering<br/><br/>
            Addressing modes<br/>
            Calling conventions<br/>
            Special operations"]

            LEGALIZE --> TYPELEGAL
            TYPELEGAL --> CUSTOM
        end

        %% ----------------------------------------------------
        %% Instruction selection
        %% ----------------------------------------------------

        subgraph ISEL["14. Instruction Selection"]
            direction TB

            PATTERN["Pattern Matching<br/><br/>
            Match IR operations<br/>
            against target instruction patterns"]

            DAG["Selection DAG / GlobalISel<br/><br/>
            Represent instruction dependencies"]

            SELECT["Select Target Instructions<br/><br/>
            IR operations → target operations"]

            PSEUDO["Pseudo Instructions<br/><br/>
            Temporary target instructions"]

            DAG --> PATTERN
            PATTERN --> SELECT
            SELECT --> PSEUDO
        end

        %% ----------------------------------------------------
        %% Machine IR
        %% ----------------------------------------------------

        subgraph MIR["15. Machine Representation"]
            direction TB

            MI["Machine Instructions<br/><br/>
            Target-specific operations"]

            MBB["Machine Basic Blocks<br/><br/>
            Target-specific CFG"]

            MF["Machine Function<br/><br/>
            Complete target-level function"]

            MI --> MBB
            MBB --> MF
        end

        %% ----------------------------------------------------
        %% Machine optimization
        %% ----------------------------------------------------

        subgraph MOPT["16. Machine-Dependent Optimization"]
            direction TB

            PEEPHOLE["Peephole Optimization<br/><br/>
            Replace inefficient instruction sequences"]

            SCHED["Instruction Scheduling<br/><br/>
            Reorder instructions<br/>
            Respect dependencies<br/>
            Exploit pipeline parallelism"]

            BLOCKLAYOUT["Basic Block Layout<br/><br/>
            Improve fall-throughs<br/>
            Branch behavior"]

            TAIL["Tail Duplication / Tail Merging<br/><br/>
            Optimize control-flow layout"]

            PEEPHOLE --> SCHED
            SCHED --> BLOCKLAYOUT
            BLOCKLAYOUT --> TAIL
        end

        %% ----------------------------------------------------
        %% Register allocation
        %% ----------------------------------------------------

        subgraph REGALLOC["17. Register Allocation"]
            direction TB

            LIVE["Live-Range Analysis<br/><br/>
            Determine where values are live"]

            COLOR["Register Assignment<br/><br/>
            Map virtual registers<br/>
            to physical registers"]

            SPILL["Spill / Reload<br/><br/>
            Move values between<br/>
            registers and memory"]

            COALESCE["Register Coalescing<br/><br/>
            Eliminate unnecessary moves"]

            LIVE --> COLOR
            COLOR --> SPILL
            COALESCE --> COLOR
        end

        %% ----------------------------------------------------
        %% Prologue / epilogue
        %% ----------------------------------------------------

        subgraph FRAME["18. Machine Frame / ABI"]
            direction TB

            FRAMELOWER["Frame Lowering<br/><br/>
            Stack frame layout"]

            PROLOGUE["Prologue / Epilogue<br/><br/>
            Save registers<br/>
            Allocate stack frame<br/>
            Restore registers"]

            CALLLOWER["Calling Convention Lowering<br/><br/>
            Arguments<br/>
            Return values<br/>
            Register / stack passing"]

            FRAMELOWER --> PROLOGUE
            CALLLOWER --> PROLOGUE
        end

        %% ----------------------------------------------------
        %% Assembly emission
        %% ----------------------------------------------------

        subgraph EMIT["19. Code Emission"]
            direction TB

            MC["Machine Code Representation<br/><br/>
            Encoded instructions<br/>
            Relocations<br/>
            Symbols"]

            ASM["Assembly Emission<br/><br/>
            Target assembly"]

            OBJ["Object File<br/><br/>
            .o / .obj"]

            MC --> ASM
            MC --> OBJ
        end

        %% Back-end connections
        OPTIR --> LEGALIZE

        ISA --> LEGALIZE
        COST --> SCHED
        REGS --> COLOR

        CUSTOM --> DAG
        PSEUDO --> MI

        MF --> PEEPHOLE
        TAIL --> LIVE

        COLOR --> FRAMELOWER
        SPILL --> FRAMELOWER
        FRAMELOWER --> CALLLOWER

        PROLOGUE --> MC
        CALLLOWER --> MC
    end


    %% ========================================================
    %% OUTPUT
    %% ========================================================

    ASM --> ASMOUT["Target Assembly<br/><b>foo.s</b>"]
    OBJ --> OBJOUT["Object Code<br/><b>foo.o</b>"]

    ASMOUT --> LINK["Linker"]
    OBJOUT --> LINK

    LINK --> EXE["Executable<br/><b>foo</b>"]


    %% ========================================================
    %% STYLING
    %% ========================================================

    classDef source fill:#fff4cc,stroke:#c99400,stroke-width:2px,color:#222;
    classDef frontend fill:#e8f2fc,stroke:#3978b9,stroke-width:1.5px,color:#222;
    classDef middle fill:#f0e8fa,stroke:#8148b5,stroke-width:1.5px,color:#222;
    classDef backend fill:#e9f5e2,stroke:#58963c,stroke-width:1.5px,color:#222;
    classDef artifact fill:#ffffff,stroke:#777,stroke-width:1px,stroke-dasharray:5 4,color:#222;
    classDef output fill:#fff4cc,stroke:#c99400,stroke-width:2px,color:#222;

    class SRC,PP source;

    class CHAR,SCAN,TOK,PARSER,CST,AST frontend;
    class NAME,TYPES,SYMBOLS,CONST frontend;
    class LOWER,CFG0,IR0,SSA frontend;

    class IRMODULE,IRFUNC,CFG,DATAFLOW,ALIAS,CALLGRAPH,LOOP middle;
    class SIMPLIFY,INSTSIMPLIFY,SSAOPT middle;
    class CONSTFOLD,CONSTPROP,CSE,DCE,COPY,GVN,INSTCOMB middle;
    class MEM2REG,SROA,LOADSTORE,DSE,LICM middle;
    class LOOPUNROLL,LOOPROTATE,INDUCTION,VECTORIZ middle;
    class INLINE,DEvirt,ARGPROP,GLOBALOPT middle;
    class PIPE,OPTIR middle;

    class ISA,REGS,COST,LEGALIZE,TYPELEGAL,CUSTOM backend;
    class PATTERN,DAG,SELECT,PSEUDO backend;
    class MI,MBB,MF backend;
    class PEEPHOLE,SCHED,BLOCKLAYOUT,TAIL backend;
    class LIVE,COLOR,SPILL,COALESCE backend;
    class FRAMELOWER,PROLOGUE,CALLLOWER backend;
    class MC,ASM,OBJ backend;

    class ASMOUT,OBJOUT,EXE output;

    %% ========================================================
    %% SUBGRAPH STYLING
    %% ========================================================

    style FE fill:#f7fbff,stroke:#3978b9,stroke-width:2px,stroke-dasharray:6 4
    style ME fill:#fbf8ff,stroke:#8148b5,stroke-width:2px,stroke-dasharray:6 4
    style BE fill:#f8fcf5,stroke:#58963c,stroke-width:2px,stroke-dasharray:6 4

    style LEXICAL fill:#ffffff,stroke:#75a9d6,stroke-width:1px
    style SYNTAX fill:#ffffff,stroke:#75a9d6,stroke-width:1px
    style SEMANTIC fill:#ffffff,stroke:#75a9d6,stroke-width:1px
    style IRGEN fill:#ffffff,stroke:#75a9d6,stroke-width:1px

    style IR fill:#ffffff,stroke:#a779c9,stroke-width:1px
    style ANALYSIS fill:#ffffff,stroke:#a779c9,stroke-width:1px
    style CANON fill:#ffffff,stroke:#a779c9,stroke-width:1px
    style SCALAR fill:#ffffff,stroke:#a779c9,stroke-width:1px
    style MEMORY fill:#ffffff,stroke:#a779c9,stroke-width:1px
    style LOOPS fill:#ffffff,stroke:#a779c9,stroke-width:1px
    style IPO fill:#ffffff,stroke:#a779c9,stroke-width:1px
    style TRANSFORM fill:#ffffff,stroke:#a779c9,stroke-width:1px

    style TARGETINFO fill:#ffffff,stroke:#75ad5c,stroke-width:1px
    style LOWERING fill:#ffffff,stroke:#75ad5c,stroke-width:1px
    style ISEL fill:#ffffff,stroke:#75ad5c,stroke-width:1px
    style MIR fill:#ffffff,stroke:#75ad5c,stroke-width:1px
    style MOPT fill:#ffffff,stroke:#75ad5c,stroke-width:1px
    style REGALLOC fill:#ffffff,stroke:#75ad5c,stroke-width:1px
    style FRAME fill:#ffffff,stroke:#75ad5c,stroke-width:1px
    style EMIT fill:#ffffff,stroke:#75ad5c,stroke-width:1px
```
