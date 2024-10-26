---
title: AFL_Note
date: 2020-11-30
categories:
- Fuzzer
tags:
- random_notes
---

* [x] Pulling JPEGs out of thin air
* [x] LLVM IR.
* [x] afl_clang_fast.c and afl_llvm_pass.so.cc
* [x] AFL_min.c
* [x] How to trim the input file?
* [x] How do the program manage its queue?
* [x] http://lcamtuf.blogspot.com/2014/08/binary-fuzzing-strategies-what-works.html (Fuzzing strategies details)
* [x] http://lcamtuf.blogspot.com/2015/01/afl-fuzz-making-up-grammar-with.html (Dictionaries)
* [x] http://lcamtuf.blogspot.com/2014/11/afl-fuzz-crash-exploration-mode.html. (Investigating crashes.)
* [x] https://lcamtuf.blogspot.com/2016/02/say-hello-to-afl-analyze.html (Automatically inferring file syntax with afl-analyze)
* [ ] Difference between afl-tmin.c and afl-cmin.c?
* [ ] QEMU
* [x] AFL-analyze. 
* [ ] ClusterFuzz (Google Chrome) 
* [ ] OSSFuzz.
* [ ] Project Springfield
* [ ] Fork is not scalable.
* [ ] fork() needs to update the reverse mapping of a physical page for swapping under memory contention, which is a well-known scalability bottleneck in the current Linux kernel [8, 9, 11]. (In paper Designing New Operating Primitives to Improve Fuzzing Performance)
* [ ] combine fuzzing with symbolic execution [13, 27, 36]. (In paper Designing New Operating Primitives to Improve Fuzzing Performance)
* [ ] Evolving the input queue. (In technical_details.txt) 3 links.
* [x] Perf_tips.txt
* [x] env_variables.txt
* [ ] Forward Symbolic analysis. (Not finished)


- C++ **__thread** keyword. Global variables but no need for across threads global protection's values. If the variables are shared between threads, then each thread will maintain its own variable version. Can only works with Plain Old Data, cannot work with class, struct, union etc. (***C language model***)

-  AFL can generate larger size mutations than the original inputs, however, it depends on whether the program has character over character level identifications.

- How the AFL trimming process works? (Not afl-min.c)
    - The inputs are split into several blocks, the original block size is around 1/16 of the total length of the inputs, and increase the size x0.5 every time. The trimming process iterate through the trimming block, removing them and see if the checksum of the track_bits changed or not. If changed, remove the trimming block permanently, otherwise, keep iterating. Stop if the remaining input length <= 4, or trimming block being too small (1/1024 of total length). 

- How the AFL manage its queue list structure?
    - add_to_queue is the main function to edit new queue entry.
    - queue_top means the last elements in the queue, which will be pop() at last. 
    - queue_cur is the one that is iterating, not the queue variable itself. Every time when queue_cur finish its cycle, then queue_cur = queue to reset.

- How to auto generate new dictionary? 1. If flip the bytes always generate a new path, that is distinct from the neighboring regions, then we can add these bytes as a new syntax to the dictionary. 2. Because of the nature of the fuzzer (coverage), we can easily distinguish between nonsensical and the one that actually follow the rules of the grammar. Keep the already constructed syntax and keep refining the others, we could progressively construct more complex and meaningful syntax.

- AFL parallelization: Each process runs independently normally. After a fix times of run cycles, each process will look into other processes' output directory, scan their output interesting queues, and execute them in their own process area, if new trace_bits regions turn up, then import the newly discovered queue to its local queue list. It seems that each process will mark the latest scanned queue position from other queues (seems to be global, weird). 
    - It is recommended to run the first process as -M (Master process), which will forced deterministic mutations, and run the other processes as -S (Slave process), which will indicate random mutations. Running multiple -M process is feasible, but low efficient. (Because duplicate deterministic mutations)


- The theory behind afl-analysis.c
    - Perform walking bytes, first bit flipping, sub_10 and add_10 operations across the input sequence, run with target binary, and observe the trace_bits checksums.
        - If checksums is not changing at all, then the current byte should be **no-op (raw) data**.
        - If ANY bit flip causes the same change, then it could be **checksums, magic values and maybe atomically compared tokens**.
        - Long sections with ANY bit flip causes the same change, it could be **encrypted data, checksums**. 
        - **"Pure" data sections**, where analyzer-injected changes consistently elicit differing changes to control flow. (Not inferring in the code?)
    - Paper "ProFuzzer: On-the-ﬂy Input Type Probing for Better Zero-day Vulnerability Discovery"
        - Not using random mutations from the AFL, but use the heuristic from the fuzzing input analysis (input semantic) to put more focus on more targeted mutations. For example, not mutating **raw data** input sections etc. 
        - Two step process:
            1. Iterating bytes with all possible values (0x00 to 0xff). Gain the rough idea of input data field semantics. 
            2. Combine related fields together, and mutates them together. For example, the consecutive bytes whose specific content leads to the same exception path under mutation can be grouped into a ﬁeld.
        - 6 input field types:
            - assertion: only a single valid value that allows the program to execute correctly.
            - raw data: Nothing happens here.
            - enumeration: Only a small set of valid values, with other values causing the program to erroneously terminate.
            - loop count: Might impact loop counts.
            - size: An input ﬁeld of the size type determines the amount of data the program should read from the input file for further processing. Similar to the offset type, if a size value causes any out-of-range data access, the program terminates with errors; within the valid range, different sizes may result in different execution paths, since different data are processed. The difference between offset and size lies in that if a size value is set to zero, errors must occur, while setting an offset to 0 likely does not cause similar problems.
            - offset: The location from which the program can access the subsequent data in the input file. 
        - The algorithm has a reprobing function, where a new input comes in, the algorithm will firstly verified the similar input field semantics with the newly inputs, if successful, directly use the same input fields information from the old file.
        
Original code for afl-analysis:

```C
case RESP_NONE:     SAYF("no-op block\n"); break;
case RESP_MINOR:    SAYF("superficial content\n"); break;
case RESP_VARIABLE: SAYF("critical stream\n"); break;
case RESP_FIXED:    SAYF("\"magic value\" section\n"); break;
case RESP_LEN:      SAYF("suspected length field\n"); break;
case RESP_CKSUM:    SAYF("suspected cksum or magic int\n"); break;
case RESP_SUSPECT:  SAYF("suspected checksummed block\n"); break;
```

- AFL persistant mode:
    - Use a long-live process to test multiple inputs. 
    - The program and the inputs are required to be stateless, which means we must guarantee that in different runs the memory structure of the process is unchanged (does not corrupted by previous runs).

- AFL LLVM mode:
    - Afl_clang_fast use compiler level instrumentation, instead of the more crude assembly-level rewriting. Could be much faster than traditional methods, not feasible with GCC. I still do not understand the how that works.
    - Use the LLVM Intermedia Language to inject the instruments, compile handles optimization. Might because of the reduction of function jumps (trampoline). 
    - As high as 2x faster runtime. 
    - In file afl-clang-fast.c and afl-llvm-pass.so.cc. 
    - **__afl_prev_loc** and **__afl_area_ptr** being declared in afl_llvm_rt.o.c, and being referenced as (GlobalValue::ExternalLinkage, GlobalVariable::GeneralDynamicTLSModel) in the afl_llcm_pass.so.cc.

- Paper "Designing New Operating Primitives to Improve Fuzzing Performance"
    - A performance optimization focused approach to improve AFL.
    - Github link: https://github.com/tarafans/perf-fuzz.git
    - Three contributions:
        1. Use system call snapshot() rather than fork().
            - fork() is not scalable with multiple cores. 
                * [ ] ***This includes a lot of unnecessary features such as creating the reverse mapping of the child’s physical pages for swapping, which is a well-known performance bottleneck in Linux [8, 9, 11].***
            - snapshot() is a custom made system calls. 
        2. Create a duel file system to cache seed writing to the disk. Cache the file on memory.
        3. Shared In_memory Test_case log. For AFL syncing. Shared across different processes of AFL. Because it is in memory, it could be very fast, and avoid file system read/write violations and files iterations (latency). 
