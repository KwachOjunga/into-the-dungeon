Describing type theory from a beginner's perspective.
We'll delve into as much of the math as is humanly possible and undesirable.

Specifically capture the applications of types in compiler theory and how it influences 
the programming language design.

Also delve into the collection of algorithms used by compilers 
to do things like type inference and type checking.

Big words those ones.

---
It is obvious that the types used in strongly typed languages influence how one can use a prticular type.
It prevents variable reuse.
What one can store in them and 
The sort of methods one can apply to said types.

You'll be seeing alot of rust here, get comfortable.

```rust
    let mut message = String::new();
```

The defined message is incapable of holoding a type other than a string type.


We'll explore the repurcussions of this.
