# Kisumu Lang Roadmap

Kisumu Lang's prmary objective is a scholarly one. 
It offers a place one can get to observe the internals of compiler design
but it also bridges a gap of what is capable in an academic setting with
the current standard industrial processes.


There are several domains that open up to exploration with such a task.
- Of primary interest is type theory. 
  - The introduction of formal type theory in computing has enabled definition of 
    methods that offer a degree of determinism when it comes to unravelling 
    abstractions and employing them as tools to wield in describing various other 
    domains.
- Cybersecurity. Much of the weaknesses that hackers exploit tend to be those 
  exposed by various programming languages. In a similar vein much of the work that 
  addresses exploits also involves certain loop holes being sealed at the programming 
  language interace.


# What to do.

With the primary objective of retaining a long term view of the project.
What ought to happen is that this project allows individuals involved in 
it the capacity to gainfully participate in virtually all the branches
of CS that rely on this information.

To achieve this i am hard pressed to get a currently more important task than
updating the llir/llvm codebase to the current LLVM version 22.

This gives the apprentice exposure to a few things.
- Fundamentally Compiler Theory
  - From the onset, a beginner sees the stressing of the importance of concepts 
    such as static single assignment and its utility as propagated by LLVM.
  - How a program can be structured for datalow analysis to be performed
    effectively

  This will be the primary learning ground since it offers much of the context 
  neccessary to learn more advanced concepts in designing compiler backends and 
  know of considerations to be made when designing custom backends.

This is a lot of grunt work. And it also means  playing catch up. So why would it 
even be a remotely good idea to do this.

Updating the infrastructure to what is latest in the industry also has certain 
advantages;
  the LLVM framework has expanded its coverage to include a dynamic composable and 
  extensible IR, MLIR. Updating the go package offers an avenue to build an 
  ecosystem in Go that can leverage all that is made available by this extension.

  NOTE: The opportunity i am basically hinting at touches upon the capacity to 
  develop tools across the entire semi-conductor tool plane.
    - From building entire languages, DSLs
    - Hardware Level Synthesis Tools
    - Hardware DEscription Languages - this by extension means designing custom 
      targets for AI workloads
  
# Relevant Things to Consider
