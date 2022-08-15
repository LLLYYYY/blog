---
title: The road to LLVM pass (Part III)
date: 2020-12-29
categories:
- LLVM
tags:
- blogs
---

LLVM pass framework is one of the most important and fundamental elements in the LLVM system. While a program is compiling, passes will be applied to perform any code transformation, code optimization and code analysis works. However, the official LLVM document does not mention a lot of important details of writing a working pass for the LLVM system. Therefore, this blog will work as a reference for my own experience of how to write a fully functional LLVM pass.

The blog will be divided into three parts, Part I of the blog will introduce a way to implement the most easy dynamic loaded pass. Beginning from the second blog, we will introduce the way to statically link the pass **into** the LLVM library, to be able to run your LLVM pass as a default optimization step in the LLVM pipeline. Furthermore, you can even implement your LLVM pass to be able to run at **Linking time**. These kinds of passes that run at link time are also being referred as LTO(Link Time Optimization) pass. And LTO-passes currently can only be implemented by statically linked into the LLVM library(currently latest LLVM 12.0.0), knowing how to statically link your pass into the LLVM library can be very important. At last, since the new PassManager Passes API has been officially exposed and used in the LLVM system, we will introduce both the Legacy PassManager interface (Part II of the blog) and the new PassManager Pass interface (Part III of the blog). Let us begin the journey!



-----------------------------------------------------------------------------------------

## Writing a statically linked pass. (New Pass Manager)

I assume we have LLVM-9.0.0 source code downloaded and successfully compiled. 

I will only introduce how to create a LLVM pass that is not Target Machine sensitive. For example, if you are creating a pass that is targeting only X86 platform, the creating process would be different. The pass we create in this blog should be applied to all different target architectures. 

There are two different kinds of passes: 
    - Analysis Pass: That they gather information that can be used by other passes. These passes should be placed in `llvm/lib/Analysis` folder.
    - Transforms Pass: Apply transformations to the IR. These passes should be placed in `llvm/lib/Transforms` folder. 

Let's assume we are building a Transformation pass. Similar to the legacy pass version, we create a subfolder `MyPass` in `llvm/lib/Transforms` and `llvm/include/llvm/Transforms`. Then we create three files in `llvm/lib/Transforms/MyPass`: which are `MyCustomPass.cpp`, `CMakeLists.txt` and `LLVMBuild.txt`.

In file `MyCustomPass.cpp`:

```C++
#include "llvm/ADT/Statistic.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/LegacyPassManager.h"
#include "llvm/IR/Module.h"
#include "llvm/Support/Debug.h"
#include "llvm/Transforms/IPO/PassManagerBuilder.h"
#include "llvm/IR/LLVMContext.h"
#include "llvm/Transforms/Utils/BasicBlockUtils.h"
#include "llvm/Support/FileSystem.h"
#include "llvm/Support/Path.h"
#include "llvm/Support/raw_ostream.h"

using namespace llvm;

PreservedAnalyses MyCustomPass::run(Module &M, ModuleAnalysisManager &MAM) {
    return PreservedAnalyses::all();
}
```

In file `MyCustomPass.cpp`:

```C++
#ifndef LLVM_TRANSFORMS_MYPASS_MYCUSTOMPASS_H
#define LLVM_TRANSFORMS_MYPASS_MYCUSTOMPASS_H

#include "llvm/IR/PassManager.h"

class MyCustomPass : public PassInfoMixin<MyCustomPass> {
    public:
      PreservedAnalyses run(Module &M, ModuleAnalysisManager &MPM);
};

#endif // LLVM_TRANSFORMS_MYPASS_MYCUSTOMPASS_H
```

Well, you can notice that the code is much more cleaner with the new pass manager method. We do not need to call initialize pass or create or register pass function. Just declare and implement the pass would be fine. Additionally, we gain more control on the return value of the `run(Module&, ModuleAnalysisManager&)` function, the return value could be PresererdAnalysis::all(), PresererdAnalysis::none(), PresererdAnalysis::allInSet

But how does the LLVM system acknowledge our new declared pass? Well. **We also need to declare our pass in the `llvm/lib/Passes/PassRegistry.def` file.** For example, now we are creating a new ModulePass, then the pass registry line should be:

```C++
#ifndef MODULE_PASS
#define MODULE_PASS(NAME, CREATE_PASS)
#endif
...
MODULE_PASS("mycustompass", MyCustomPass())
...
#undef MODULE_PASS

```

Don't forget to add the required linker dependencies in the `CMakeLists.txt` and `LLVMBuild.txt`. Especially in the `llvm/lib/Passes` folder, where the passes will be registered and called. 

### Run the pass at Link time. (LTO)

Once the new pass method has been implemented and compiled into the LLVM system, we can call this pass in the LTO. We choose to use the LLVM linker **lld** in this command. (You have to choose to compile the LLVM project with lld enabled. Then these passes including our custom created pass would be statically linked into the lld application) **Please notice that on LLVM-9.0.0, only the new pass (and not the legacy one) can be called by this method**.

Note that `-Wl,--XXX` represents that passing the following parameters into the linker. No matter it is gold, lld or gcc linker. 

```sh
clang-9 -flto -fuse-ld=lld -Wl,--lto-newpm-passes=afl ./MyRandomCFile.c -o MyRandomFile.o
```

Check!



