Actual file

 - add.mlir

```mlir

module {
        func.func @add(%a : i32, %b : i32) -> i32 {
                %sum = arith.addi %a, %b : i32
                %c0 = arith.constant 0 :index
                %c10 = arith.constant 10: index
                %step = arith.constant 1:index
                scf.for %i = %c0 to %c10 step %step {
                        scf.yield
                }
                return %sum : i32
        }
}
```


Transformation 1 

>> mlir-opt --convert-to-llvm  add.mlir| xdsl-opt -p convert-scf-to-x86-scf

```mlir
builtin.module {
  llvm.func @add(%arg0: i32, %arg1: i32) -> i32 {
    %0 = llvm.add %arg0, %arg1 : i32
    llvm.return %0 : i32
  }
}
```


- Equivalent code converted via:

>> mlir-opt --convert-to-llvm  add.mlir| xdsl-opt -p convert-scf-to-x86-scf|mlir-translate --mlir-to-llvmir

```llvm

; ModuleID = 'LLVMDialectModule'
source_filename = "LLVMDialectModule"

define i32 @add(i32 %0, i32 %1) {
  %3 = add i32 %0, %1
  ret i32 %3
}

!llvm.module.flags = !{!0}

!0 = !{i32 2, !"Debug Info Version", i32 3}

```


From this it is obvious that the loop is optimized away leaving neccessary the actual sum.
 

I tried a different optimization sequence just for the sake of it.

>> mlir-opt --convert-scf-to-cf --convert-to-llvm add.mlir

As can be seen from the instance, the objective is to first convert portions 
of the code containing scf dialect to cf dialects.

```mlir
module {
  llvm.func @add(%arg0: i32, %arg1: i32) -> i32 {
    %0 = llvm.add %arg0, %arg1 : i32
    %1 = llvm.mlir.constant(0 : index) : i64
    %2 = llvm.mlir.constant(10 : index) : i64
    %3 = llvm.mlir.constant(1 : index) : i64
    llvm.br ^bb1(%1 : i64)
  ^bb1(%4: i64):  // 2 preds: ^bb0, ^bb2
    %5 = llvm.icmp "slt" %4, %2 : i64
    llvm.cond_br %5, ^bb2, ^bb3
  ^bb2:  // pred: ^bb1
    %6 = llvm.add %4, %3 : i64
    llvm.br ^bb1(%6 : i64)
  ^bb3:  // pred: ^bb1
    llvm.return %0 : i32
  }
}
```

The emitted llvm-ir;

>>  mlir-opt --convert-scf-to-cf --convert-to-llvm add.mlir|mlir-translate --mlir-to-llvmir

```llvm

; ModuleID = 'LLVMDialectModule'
source_filename = "LLVMDialectModule"

define i32 @add(i32 %0, i32 %1) {
  %3 = add i32 %0, %1
  br label %4

4:                                                ; preds = %7, %2
  %5 = phi i64 [ %8, %7 ], [ 0, %2 ]
  %6 = icmp slt i64 %5, 10
  br i1 %6, label %7, label %9

7:                                                ; preds = %4
  %8 = add i64 %5, 1
  br label %4

9:                                                ; preds = %4
  ret i32 %3
}

!llvm.module.flags = !{!0}

!0 = !{i32 2, !"Debug Info Version", i32 3}

```

My previous assumption that llvm ir by definition neccessarily had an entry block has been shattered.
In both instances there are defined unctions in the llvm output with none having an entry basic block.


Secondly why exactly does the scf to cf conversion introduce uneccessary branching.
Can the for loop construct be implemented using branching constructs. 
And what exactly is happening in the second code snippet.



