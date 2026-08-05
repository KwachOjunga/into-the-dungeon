I now find myself seeking justification for 

pulling a massive dependency into the kisumu lang project.

This one is particularly intriguing since part of it actually 
makes sense and the case for its requirement may be valid.

Normally, the structure of a compiler's pipeline is in the following stages:
  - Parse source input program, generate a form that represents the program's structure
  - Transform the program's structure into an intermediate representation (IR)
  - Optimize the IR, perform code generation, and emit the  output.

These three steps recur across the multiple compiler phases.
Rust for instance uses at least two IRs before finally emitting the llvm IR.

The first IR is the HIR (High-Level Intermediate Representation), which is basically source code
but with items such as iteration constructs desugared.

The following is an instance that illustrates this.

```rust
#![feature(ascii_char)]

fn main() {
    let a = "a".as_bytes()[0];
    let z = "z".as_bytes()[0];
    let x : Vec<_> = (a..z).map(|x| x as u8).collect();
    for i in &x {
        print!("{:#?} :", i.as_ascii().unwrap());
        println!("{i:#?}");
    }
}
```

This snippet works, perhaps with the additional need to use the nightly
feature `ascii_char`.

But as can be seen we have in the snippet some of rust's advanced features;
the use of proc_macros for one.
There is also the use of range expressions in defining var x.
In the for loop statement we see declarative macros.

```rust
// equivalent rust hir code

#![attr = Feature([ascii_char#0])]
extern crate std;
#[attr = PreludeImport]
use ::std::prelude::rust_2015::*;


fn main() {
    let a = "a".as_bytes()[0];
    let z = "z".as_bytes()[0];
    let x: Vec<_> = Range { start: a, end: z }.map(|x| x as u8).collect();
    {
        let _t =
            match into_iter(&x) {
                mut iter =>
                    loop {
                        match next(&mut iter) {
                            None {} => break,
                            Some {  0: i } => {
                                {
                                    ::std::io::_print({
                                            super let args = (&i.as_ascii().unwrap(),);
                                            super let args = [format_argument::new_debug(args.0)];
                                            unsafe {
                                                format_arguments::new(b"\xc1 \x00\x80`\x02 :\x00", &args)
                                            }
                                        });
                                };
                                {
                                    ::std::io::_print({
                                            super let args = (&i,);
                                            super let args = [format_argument::new_debug(args.0)];
                                            unsafe {
                                                format_arguments::new(b"\xc1 \x00\x80`\x01\n\x00", &args)
                                            }
                                        });
                                };
                            }
                        }
                    },
            };
        _t
    }
```

In this snippet, one can see how the compiler reconstructs the original source code to the HIR.
So, much of what has been made possible at the initial high level of expression is largely
an outcome of this IR's capacity to capture what the user at a high level leaves unsaid.

As a final illustration, its equivalent MIR(Mid-Level Intermediate Representation) is shown at the 
bottom. It's structure is fairly similar to the HIR, but with more fine-grained details. It's also 
closely related to the LLVM IR. - It captures the essence of lowering perfectly.


Rust uses its HIR format for [type inference, trait solving and type checking](https://rustc-dev-guide.rust-lang.org/overview.html#ast-lowering).

Type inference is largely a feature to improve upon user experience.
Trait solving serves mostly as bookkeeping over which traits are implemented for which types
and this helps in determining valid methods for a given type.
This alone offers support for the features of method chaining, operator overloading
not forgetting the highly coveted need for memory safety.

My working assumption which by default will drive the presumptions of Kisumu Lang's 
architecture is that lowering the language's AST to an IR prior to
emitting the LLVM IR will be the most efficient way to support a class of language 
constructs without being bogged down by the needed research in implementation
processes since the AST we will have by default will be custom to the language's grammar.

If we lower it to a known IR format we can have reference points with which to compare
against existing LLVM IR generators.

But more specifically, i think lowering it to an IR like Pliron does a bunch for the entire ecosystem.
1. It neccessitates the growth of PlironIR to support more of MLIR's known IR dialects.
    - It currently supports the the llvm, tensor, index and the cf dialects.

2. Directly leveraging pliron implicitly also means leveraging mlir infrastructure. - There is always much one can and needs to learn with such a mature ecosystem.

3. The developer gets to see how certain abstractions made in the programming language design space
   are useful in other domains ie. machine learning systems and hardware design through the same
   code base and set of tools.

```rust
fn main() -> () {
    let mut _0: ();
    let _1: u8;
    let mut _2: &[u8];
    let mut _3: &str;
    let _4: usize;
    let mut _5: usize;
    let mut _6: bool;
    let mut _8: &[u8];
    let mut _9: &str;
    let _10: usize;
    let mut _11: usize;
    let mut _12: bool;
    let mut _14: std::iter::Map<std::ops::Range<u8>, {closure@char.rs:7:33: 7:36}>;
    let mut _15: std::ops::Range<u8>;
    let mut _16: std::slice::Iter<'_, u8>;
    let mut _17: &std::vec::Vec<u8>;
    let mut _19: std::option::Option<&u8>;
    let mut _20: &mut std::slice::Iter<'_, u8>;
    let mut _21: isize;
    let _23: ();
    let mut _24: std::fmt::Arguments<'_>;
    let mut _26: &std::ascii::Char;
    let _27: std::ascii::Char;
    let mut _28: std::option::Option<std::ascii::Char>;
    let mut _30: core::fmt::rt::Argument<'_>;
    let mut _31: &[u8; 9];
    let _32: &[core::fmt::rt::Argument<'_>; 1];
    let _33: ();
    let mut _34: std::fmt::Arguments<'_>;
    let mut _36: &&u8;
    let mut _38: core::fmt::rt::Argument<'_>;
    let mut _39: &[u8; 8];
    let _40: &[core::fmt::rt::Argument<'_>; 1];
    let mut _41: &std::ascii::Char;
    let mut _42: &&u8;
    scope 1 {
        debug a => _1;
        let _7: u8;
        scope 2 {
            debug z => _7;
            let _13: std::vec::Vec<u8>;
            scope 3 {
                debug x => _13;
                let mut _18: std::slice::Iter<'_, u8>;
                scope 4 {
                    debug iter => _18;
                    let _22: &u8;
                    scope 5 {
                        debug i => _22;
                        let _25: (&std::ascii::Char,);
                        let _35: (&&u8,);
                        scope 6 {
                            debug args => _25;
                            let _29: [core::fmt::rt::Argument<'_>; 1];
                            scope 7 {
                                debug args => _29;
                            }
                        }
                        scope 8 {
                            debug args => _35;
                            let _37: [core::fmt::rt::Argument<'_>; 1];
                            scope 9 {
                                debug args => _37;
                            }
                        }
                    }
                }
            }
        }
    }

    bb0: {
        _3 = const "a";
        _2 = core::str::<impl str>::as_bytes(move _3) -> [return: bb1, unwind continue];
    }

    bb1: {
        _4 = const 0_usize;
        _5 = PtrMetadata(copy _2);
        _6 = Lt(copy _4, copy _5);
        assert(move _6, "index out of bounds: the length is {} but the index is {}", move _5, copy _4) -> [success: bb2, unwind continue];
    }

    bb2: {
        _1 = copy (*_2)[_4];
        _9 = const "z";
        _8 = core::str::<impl str>::as_bytes(move _9) -> [return: bb3, unwind continue];
    }

    bb3: {
        _10 = const 0_usize;
        _11 = PtrMetadata(copy _8);
        _12 = Lt(copy _10, copy _11);
        assert(move _12, "index out of bounds: the length is {} but the index is {}", move _11, copy _10) -> [success: bb4, unwind continue];
    }

    bb4: {
        _7 = copy (*_8)[_10];
        _15 = std::ops::Range::<u8> { start: copy _1, end: copy _7 };
        _14 = <std::ops::Range<u8> as Iterator>::map::<u8, {closure@char.rs:7:33: 7:36}>(move _15, const ZeroSized: {closure@char.rs:7:33: 7:36}) -> [return: bb5, unwind continue];
    }

    bb5: {
        _13 = <Map<std::ops::Range<u8>, {closure@char.rs:7:33: 7:36}> as Iterator>::collect::<Vec<u8>>(move _14) -> [return: bb6, unwind continue];
    }

    bb6: {
        _17 = &_13;
        _16 = <&Vec<u8> as IntoIterator>::into_iter(move _17) -> [return: bb7, unwind: bb21];
    }

    bb7: {
        _18 = move _16;
        goto -> bb8;
    }

    bb8: {
        _20 = &mut _18;
        _19 = <std::slice::Iter<'_, u8> as Iterator>::next(copy _20) -> [return: bb9, unwind: bb21];
    }

    bb9: {
        _21 = discriminant(_19);
        switchInt(move _21) -> [0: bb12, 1: bb11, otherwise: bb10];
    }

    bb10: {
        unreachable;
    }

    bb11: {
        _22 = copy ((_19 as Some).0: &u8);
        _28 = core::num::<impl u8>::as_ascii(copy _22) -> [return: bb13, unwind: bb21];
    }

    bb12: {
        drop(_13) -> [return: bb20, unwind continue];
    }

    bb13: {
        _27 = Option::<std::ascii::Char>::unwrap(move _28) -> [return: bb14, unwind: bb21];
    }

    bb14: {
        _26 = &_27;
        _25 = (move _26,);
        _41 = no_retag copy (_25.0: &std::ascii::Char);
        _30 = core::fmt::rt::Argument::<'_>::new_debug::<std::ascii::Char>(copy _41) -> [return: bb15, unwind: bb21];
    }

    bb15: {
        _29 = [move _30];
        _31 = const b"\xc1 \x00\x80`\x02 :\x00";
        _32 = &_29;
        _24 = Arguments::<'_>::new::<9, 1>(move _31, copy _32) -> [return: bb16, unwind: bb21];
    }

    bb16: {
        _23 = std::io::_print(move _24) -> [return: bb17, unwind: bb21];
    }

    bb17: {
        _36 = &_22;
        _35 = (move _36,);
        _42 = no_retag copy (_35.0: &&u8);
        _38 = core::fmt::rt::Argument::<'_>::new_debug::<&u8>(copy _42) -> [return: bb18, unwind: bb21];
    }

    bb18: {
        _37 = [move _38];
        _39 = const b"\xc1 \x00\x80`\x01\n\x00";
        _40 = &_37;
        _34 = Arguments::<'_>::new::<8, 1>(move _39, copy _40) -> [return: bb19, unwind: bb21];
    }

    bb19: {
        _33 = std::io::_print(move _34) -> [return: bb23, unwind: bb21];
    }

    bb20: {
        return;
    }

    bb21 (cleanup): {
        drop(_13) -> [return: bb22, unwind terminate(cleanup)];
    }

    bb22 (cleanup): {
        resume;
    }

    bb23: {
        goto -> bb8;
    }
}

alloc4 (size: 8, align: 1) {
    c1 20 00 80 60 01 0a 00                         │ . ..`...
}

alloc3 (size: 9, align: 1) {
    c1 20 00 80 60 02 20 3a 00                      │ . ..`. :.
}

alloc2 (size: 1, align: 1) {
    7a                                              │ z
}

alloc1 (size: 1, align: 1) {
    61                                              │ a
}

fn main::{closure#0}(_1: &mut {closure@char.rs:7:33: 7:36}, _2: u8) -> u8 {
    debug x => _2;
    let mut _0: u8;

    bb0: {
        _0 = copy _2;
        return;
    }
}
```
