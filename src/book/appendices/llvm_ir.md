An overview of some of llvm's internals illustrating their usages and syntax.
 
 Categories
 - Attributes
 - linkages
 - Terminator instructions
 - Metadata
 - Debug Information
 
 For a complete reference of the IR, lookup [LLVM_IR](https://llvm.org/docs/LangRef.html)
 
 ## Examples
 
### SSA features 
 
 This instance show cases the constrains laid out by SSA 
  ```rust
  fn greater_than (a :u32, b: u32) bool {
      if  a > b {
          true
      }else{
          false
      }
  }
  
  
  fn main() {
      let mut a = 6;
      let mut b = 8;
      if greater_than(a, b) {
          a += 1
      }else{
          b += 1
      }
  }
 ```
 
 ```llvm-ir
    define i1 @greater_than(i32 %a, i32 %b) {
      start:
        %cmp = icmp ugt i32 %a, i32 %b
        br i1 %cmp, label %if_true, l abel %if_false
      
      if_true:
        ret i1 true
        
      if_false:
        ret i1 false      
    }
    
    define void @main() {
      start:
        %0 = alloca i32
        %1 = alloca i32
        store i32 6, ptr %0
        store i32 8, ptr %1
        %cmp = call i1 @greater_than(i32 %0, i32 %1)
        br i1 %cmp, label %if_greater, label %if_less
        
      if_greater:
        %_0 = load i32, ptr %0
        %_01 = add i32 %0, 1
        ret void
        
      if_less:
        %_1 = load i32, ptr %1
        %_11 = add i32 &_1, 1
        ret void        
    }
   ```
  Actual rustc output with the added difference of printing the output to stdout
     
   ```llvm-ir
      ; great::main
      ; Function Attrs: nonlazybind uwtable
      define hidden void @_RNvCs3TUjdV8qTsL_5great4main() unnamed_addr #1 {
      start:
        %_13 = alloca [16 x i8], align 8
        %args = alloca [16 x i8], align 8
        %b = alloca [4 x i8], align 4
        %a = alloca [4 x i8], align 4
        store i32 6, ptr %a, align 4
        store i32 8, ptr %b, align 4
        %_4 = load i32, ptr %a, align 4
        %_5 = load i32, ptr %b, align 4
      ; call great::greater_than
        %_3 = call zeroext i1 @_RNvCs3TUjdV8qTsL_5great12greater_than(i32 %_4, i32 %_5)
        br i1 %_3, label %bb2, label %bb4
      
      bb4:                                              ; preds = %start
        %0 = load i32, ptr %b, align 4
        %_7.0 = add i32 %0, 1
        %_7.1 = icmp ult i32 %_7.0, %0
        br i1 %_7.1, label %panic, label %bb5
      
      bb2:                                              ; preds = %start
        %1 = load i32, ptr %a, align 4
        %_6.0 = add i32 %1, 1
        %_6.1 = icmp ult i32 %_6.0, %1
        br i1 %_6.1, label %panic1, label %bb3
      
      bb5:                                              ; preds = %bb4
        store i32 %_7.0, ptr %b, align 4
        br label %bb6
      
      panic:                                            ; preds = %bb4
      ; call core::panicking::panic_const::panic_const_add_overflow
        call void @_RNvNtNtCsgc7BJoiPOQP_4core9panicking11panic_const24panic_const_add_overflow(ptr align 8 @alloc_9d4dbda1dd74df7697d1e3a0acc956d8) #9
        unreachable
      
      bb6:                                              ; preds = %bb3, %bb5
      ; call <core::fmt::rt::Argument>::new_display::<u32>
        call void @_RINvMNtNtCsgc7BJoiPOQP_4core3fmt2rtNtB3_8Argument11new_displaymECs3TUjdV8qTsL_5great(ptr sret([16 x i8]) align 8 %_13, ptr align 4 %b) #7
        %2 = getelementptr inbounds nuw %"core::fmt::rt::Argument<'_>", ptr %args, i64 0
        call void @llvm.memcpy.p0.p0.i64(ptr align 8 %2, ptr align 8 %_13, i64 16, i1 false)
      ; call <core::fmt::Arguments>::new::<4, 1>
        %3 = call { ptr, ptr } @_RINvMs2_NtCsgc7BJoiPOQP_4core3fmtNtB6_9Arguments3newKj4_Kj1_ECs3TUjdV8qTsL_5great(ptr align 1 @alloc_61247b90e1706a3f65e71312b599d3d1, ptr align 8 %args) #7
        %_9.0 = extractvalue { ptr, ptr } %3, 0
        %_9.1 = extractvalue { ptr, ptr } %3, 1
      ; call std::io::stdio::_print
        call void @_RNvNtNtCskKV3BO88lSU_3std2io5stdio6__print(ptr %_9.0, ptr %_9.1)
        ret void
      
      bb3:                                              ; preds = %bb2
        store i32 %_6.0, ptr %a, align 4
        br label %bb6
      
      panic1:                                           ; preds = %bb2
      ; call core::panicking::panic_const::panic_const_add_overflow
        call void @_RNvNtNtCsgc7BJoiPOQP_4core9panicking11panic_const24panic_const_add_overflow(ptr align 8 @alloc_7b5440927130137bf397d791bde43b7e) #9
        unreachable
      }
      
   ```
   If you are keen you'll notice no defined variable is re-assigned twice.
   In SSA there is no concept of shadowing. Each defined variable has to be the only instance of its initialization; 
   reusing it requires the instantiation of a new variable.
 
 ## Terminator instructions
 
 The terminator instructions are: ‘ret’, ‘br’, ‘switch’, ‘indirectbr’, ‘invoke’, ‘callbr’ ‘resume’, ‘catchswitch’, ‘catchret’, ‘cleanupret’, and ‘unreachable’.
 
  ### `ret`
    return a value from a function.
    
    _syntax
    ```text
      ret <type> <value>
    ```
  
    _use-cases
    ```llvm-ir
      ret void
      ret i64 %_0
      ret i32 0
      ret { i32, i8 } { i32 4, i8 2 } ; a struct with fields of i32 and i8
    ```
    The return type must be of a first class type so basically it must be of the primitive types `i8`,`i32`,`i32`,`float` etc. If it is an aggregate type like a struct of some sort it must be delineated to illustrate the individual types within the aggregate type.
  
  ### `br`
  
  branch conditionally or unconditionally from a block.
  
   _syntax
   ```text
      ;conditional branch
      br i1 <cond>, label <iftrue>, label <iffalse>
      
      ; unconditional branch
      br label %block
  ```
  
   _use-cases
   ```llvm-ir
      br label %bb14
      br i1 %_2, label %bb2, label %bb3
   ```
  
  ### `switch`
  
  _syntax
   ```text
      switch <intty> <value>, label <defaultdest> [ <intty> <val>, label <dest> ... ]
   ```
  
   _use-cases
   ```llvm-ir
      switch i64 %_2, label %bb1 [
          i64 0, label %bb5
          i64 1, label %bb4
          i64 2, label %bb3
          i64 3, label %bb2
      ]
   ```
  From the langref :- "Depending on properties of the target machine and the particular switch instruction, this instruction may be code generated in different ways. For example, it could be generated as a series of chained conditional branches or with a lookup table."
  
  ## Linkages 
  
  These appear to be information being passed to the linker.
