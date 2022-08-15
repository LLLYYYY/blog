---
title: The road to LLVM pass (Part II)
date: 2020-12-29
categories:
- LLVM
tags:
- blogs
---

LLVM pass framework is one of the most important and fundamental elements in the LLVM system. While a program is compiling, passes will be applied to perform any code transformation, code optimization and code analysis works. However, the official LLVM document does not mention a lot of important details of writing a working pass for the LLVM system. Therefore, this blog will work as a reference for my own experience of how to write a fully functional LLVM pass.

The blog will be divided into three parts, Part I of the blog will introduce a way to implement the most easy dynamic loaded pass. Beginning from the second blog, we will introduce the way to statically link the pass **into** the LLVM library, to be able to run your LLVM pass as a default optimization step in the LLVM pipeline. Furthermore, you can even implement your LLVM pass to be able to run at **Linking time**. These kinds of passes that run at link time are also being referred as LTO(Link Time Optimization) pass. And LTO-passes currently can only be implemented by statically linked into the LLVM library(currently latest LLVM 12.0.0), knowing how to statically link your pass into the LLVM library can be very important. At last, since the new PassManager Passes API has been officially exposed and used in the LLVM system, we will introduce both the Legacy PassManager interface (Part II of the blog) and the new PassManager Pass interface (Part III of the blog). Let us begin the journey!



-----------------------------------------------------------------------------------------

## Writing a statically linked pass. (Legacy PassManager)

I assume we have LLVM-9.0.0 source code downloaded and successfully compiled. 

I will only introduce how to create a LLVM pass that is not Target Machine sensitive. For example, if you are creating a pass that is targeting only X86 platform, the creating process would be different. The pass we create in this blog should be applied to all different target architectures. 

There are two different kinds of passes: 
    - Analysis Pass: That they gather information that can be used by other passes. These passes should be placed in `llvm/lib/Analysis` folder.
    - Transforms Pass: Apply transformations to the IR. These passes should be placed in `llvm/lib/Transforms` folder. 

Let us assume we are creating a Transform pass. In the `llvm/lib/Transforms` folder, create a sub folder, named `My_Pass` (Any names you want actually). Then we create four files: `CMakeLists.txt`, `LLVMBuild.txt`, `MyCustomPass.cpp` and `MyPass.cpp`. And in the `llvm/include/llvm/Transforms` folder, create a sub folder too, named `My_Pass` (Any names you want actually), create header files `MyCustomPass.h` in `llvm/include/llvm/Transforms/My_Pass` and `MyPass.h` in `llvm/include/llvm/Transforms`.

Let's borrow the code from Part I, and copy and paste it into the `llvm/lib/Transforms/MyCustomPass.cc`. **Please notice that the registration function is different in this case!!!**

In file `CMakeLists.txt`

```
add_llvm_library(LLVMMyPassOpts
  MyPass.cpp
  MyCustomPass.cpp

  ADDITIONAL_HEADER_DIRS
  ${LLVM_MAIN_INCLUDE_DIR}/llvm/Transforms
  ${LLVM_MAIN_INCLUDE_DIR}/llvm/Transforms/MyPass

  DEPENDS
  intrinsics_gen
  )

```

In file `LLVMBuild.txt`

```
[component_0]
type = Library
name = MyPass
parent = Transforms
library_name = MyPassOpts
required_libraries = AggressiveInstCombine Analysis Core InstCombine Support
```

In file `MyCustomPass.cpp`

```c++
// File MyCustomPass.cpp
#include "llvm/Pass.h"
#include "llvm/IR/Module.h"

#include "llvm/Transforms/MyPass.h"
#include "llvm/Transforms/MyPass/MyCustomPass.h"

#include "llvm/IR/LegacyPassManager.h"
#include "llvm/Transforms/IPO/PassManagerBuilder.h"

using namespace llvm;

namespace {
    struct MyCustomLegacyPass : public ModulePass {
      static char ID;
      MyCustomLegacyPass() : ModulePass(ID) {}
      bool runOnModule(Module &M) override {
            // Do whatever you want for the pass in this section.
            /* // Example BasicBlock IRBuilder syntax.
            for (auto &F : M){  // Extract Function from Module
                for (auto &BB : F) {  //Extract BasicBlock from Function
                    BasicBlock::iterator IP = BB.getFirstInsertionPt();
                    IRBuilder<> IRB(&(*IP));
                    // IRB.CreateLoad();
                    // IRB.CreateStone();
                    // ............
                }
            } */
        }
    }
            return false; // Not performing any transformation for the IR.
            }
    }; // end of struct Hello
}  // end of anonymous namespace

char MyCustomLegacyPass::ID = 0;

/* The registration code is different from the dynamic loaded passes. */

INITIALIZE_PASS(MyCustomLegacyPass, 
                "mycustompass", /* arg of your pass. opt will use this arg to identify and call your pass.*/
                "A short introduction of your pass.", 
                false, /* Only looks at CFG */
                false /* Analysis Pass */
                )

Pass *llvm::createMyCustomPass() { return new MyCustomLegacyPass(); }
```

In `MyCustomPass.h`: Potentially included some not used headers.

```c++
// File MyCustomPass.h
#ifndef LLVM_TRANSFORMS_MYPASS_MYCUSTOMPASS_H
#define LLVM_TRANSFORMS_MYPASS_MYCUSTOMPASS_H

#include "llvm/IR/PassManager.h"
#include "llvm/IR/LegacyPassManager.h"
#include "llvm/Transforms/My_Pass.h"

namespace llvm {
  Pass *createMyCustomPass(); //Legacy version
}

#endif // LLVM_TRANSFORMS_MYPASS_MYCUSTOMPASS_H
```

In `MyPass.cpp`. Potentially included some not used headers.

```C++
// File MyPass.cpp
#include "llvm/Transforms/MyPass.h"
#include "llvm/Transforms/AFL/MyCustomPass.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/IR/LegacyPassManager.h"
#include "llvm/IR/Verifier.h"
#include "llvm/InitializePasses.h"
#include "llvm/Transforms/Utils/UnifyFunctionExitNodes.h"

using namespace llvm;
void LLVMAddMyCustomPass(LLVMPassManagerRef PM) {
  unwrap(PM)->add(createMyCustomPass());
}
```

In `MyPass.h`.

```C++
#ifndef LLVM_TRANSFORMS_MYPASS_H
#define LLVM_TRANSFORMS_MYPASS_H

#include "llvm-c/Types.h"

#ifdef __cplusplus
extern "C" {
#endif

void LLVMAddMyCustomPass(LLVMPassManagerRef PM);

#ifdef __cplusplus
}
#endif /* defined(__cplusplus) */

#endif
```

Do you think adding the files into the library, then you are done? Nope.... There are a lot of other works you should do. Notice that in the `MyCustomPass.cpp` file, the new `INITIALIZE_PASS` registration function is actually a C++ macro that expand to a initilizeXXXpasspass function. Then we should place the pass initialization function to the correct calling locations:

- Steps:
    - Place `initializeXXXPassPass` function into `llvm/include/llvm/InitializePasses.h`
    - Place `createMyCustomPass` function into `llvm/include/llvm/LinkAllPasses.h`
    - Very importantly, no matter where you inject your new pass code, remember, in these CMakeLists.txt, and LLVMBuild.txt, you have to manually write your build dependencies for your new pass code. The easiest way for me to do it, is to check another built-in pass libraries, such as Scaler. Anywhere in the CMakeLists.txt, LLVMBuild.txt when you see `Scalar` or `ScalarOpts`, also write in your own pass signature `MyPass` and `MyPassOpts`. 

At last, if you want your pass to be called by the compile and link pipeline, the normal way to do so is to call a populateXXXPassManager function. For example, you can place you custom pass manager's create function in `void PassManagerBuilder::populateModulePassManager` in `llvm/lib/Transform/IPO/PassManagerBuilder.cpp`. 

Check!



