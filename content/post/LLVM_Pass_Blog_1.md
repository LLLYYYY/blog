---
title: The road to LLVM pass (Part I)
date: 2020-12-29
categories:
- LLVM
tags:
- blogs
---

LLVM pass framework is one of the most important and fundamental elements in the LLVM system. While a program is compiling, passes will be applied to perform any code transformation, code optimization and code analysis works. However, the official LLVM document does not mention a lot of important details of writing a working pass for the LLVM system. Therefore, this blog will work as a reference for my own experience of how to write a fully functional LLVM pass.

The blog will be divided into three parts, Part I of the blog will introduce a way to implement the most easy dynamic loaded pass. Beginning from the second blog, we will introduce the way to statically link the pass **into** the LLVM library, to be able to run your LLVM pass as a default optimization step in the LLVM pipeline. Furthermore, you can even implement your LLVM pass to be able to run at **Linking time**. These kinds of passes that run at link time are also being referred as LTO(Link Time Optimization) pass. And LTO-passes currently can only be implemented by statically linked into the LLVM library(currently latest LLVM 12.0.0), knowing how to statically link your pass into the LLVM library can be very important. At last, since the new PassManager Passes API has been officially exposed and used in the LLVM system, we will introduce both the Legacy PassManager interface (Part II of the blog) and the new PassManager Pass interface (Part III of the blog). Let us begin the journey!



-----------------------------------------------------------------------------------------

## Writing a dynamic loaded pass. (Legacy PassManager)

### Compile LLVM

First of all, download and install the LLVM system. In this blog, we are using version LLVM-9.0.0. We suggest you to download the LLVM system from the official llvm.org, or from the [Github repo](https://github.com/llvm/llvm-project). Remember to check the correct releases/commits when you are using the git version of the source code. 

Assuming we are working on the Linux machine (or possibly MacOS), to compile the LLVM system, go to the /llvm sub folder, and create a new build folder. Use the **cmake** command to generate the make file. And then directly call **make** to compile.

```sh
cd ./llvm
mkdir -p build
cmake -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_PROJECTS="clang;clang-tools-extra;lld" -G "Unix Makefiles" ../
make
```

Note that you should manually enables the LLVM projects that you want to compile, in this case, we are compiling clang, clang-tools-extra, and the LLVM linker lld. The complete list of these projects are: **clang;clang-tools-extra;compiler-rt;debuginfo-tests;libc;libclc;libcxx;libcxxabi;libunwind;lld;lldb;openmp;parallel-libs;polly;pstl**. Compile just what you need, and the compilation time will be pretty long...


### Dynamic loaded pass. (Legacy PassManager)

Writing a dynamic loaded pass that can be directly called by **clang** is straightfoward and easy. I will recommend reading the official LLVM document first. [Official Document Link.](https://llvm.org/docs/WritingAnLLVMPass.html#introduction-what-is-a-pass) Note that the official document is implementing the pass using cmake, and they place the pass file into the LLVM system. However, this is not mandatory. As an easy to implement pass, we place the pass source file anywhere we want in the file system, we can call clang to load it anytime as we required.

Next, let us create and write at the pass source file:

```c++
// File MyPass.cc
#include "llvm/Pass.h"
#include "llvm/IR/Module.h"

#include "llvm/IR/LegacyPassManager.h"
#include "llvm/Transforms/IPO/PassManagerBuilder.h"

using namespace llvm;

namespace {
    struct MyPass : public ModulePass {
      static char ID;
      MyPass() : ModulePass(ID) {}
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

char MyPass::ID = 0;
static RegisterPass<MyPass> X("hello", "My Pass",
                             false /* Only looks at CFG */,
                             false /* Analysis Pass */);

static RegisterStandardPasses Y(
    PassManagerBuilder::EP_EarlyAsPossible,
    [](const PassManagerBuilder &Builder,
       legacy::PassManagerBase &PM) { PM.add(new MyPass()); });
```

To implement your own pass, we have to derive our pass struct from one of the following struct: ModulePass, FunctionPass, LoopPass and RegionPass. The most commonly used passes are ModulePass and FunctionPass, but you can choose based on what you need. The ModulePass and also extract and use Function level passes. Next, we override the **runOn<Module, Function, etc>** function. This is the main function that we used to implement your logic for your pass. The return value for this function is a bool value, return `true` if you have modified the IR, otherwise `false`. At last, we have to register the pass. Use `RegisterPass<Hello> X` to register the pass, or we can use `RegisterStandardPasses` to register our pass to the existing pipeline. All the possible Extension points are: `EP_EarlyAsPossible, EP_ModuleOptimizerEarly, EP_LoopOptimizerEnd, EP_ScalarOptimizerLate, EP_OptimizerLast, EP_VectorizerStart, EP_EnabledOnOptLevel0, EP_Peephole, EP_LateLoopOptimizations, EP_CGSCCOptimizerLate, EP_FullLinkTimeOptimizationEarly, EP_FullLinkTimeOptimizationLast` However, `EP_FullLinkTimeOptimizationEarly, EP_FullLinkTimeOptimizationLast` doesn't seems to work correctly (LLVM-12.0.0), since I failed to call the pass in LTO using this register pipeline. Maybe they are values for future features. 

After finishing the source file, we can use clang to compile it as a shared library. The command is:

```sh
clang -O3 -funroll-loops -Wall -D_FORTIFY_SOURCE=2 -g -Wno-pointer-sign -fPIC -c MyPass.cc -o MyPass.o
```

Then we can load the pass by using clang!

```sh
clang -Xclang -load -Xclang ./MyPass.o ./MyRandomCFile -o MyRandomCFile.o
```

Check!





