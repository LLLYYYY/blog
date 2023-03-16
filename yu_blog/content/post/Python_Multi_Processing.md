---
title: Python Multi-Processing
date: 2019-08-27
categories:
- Programming_Language
tags:
- blogs
---
Code speaks:
``` python
from multiprocessing import Pool
cplexProcessPool = Pool(processes=program_process_number)
cplexPoolResult = []
for variable in variableList:
    cplexPoolResult.append(cplexProcessPool.apply_async(multi_processing_function, args= (arg1, arg2, arg3)))
cplexProcessPool.close()
cplexProcessPool.join()

results = []
for res in cplexPoolResult:
    results.append(res.get())
results = [x for x in results if x is not None]
print(str(results))
```



