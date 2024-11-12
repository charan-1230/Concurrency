## Copy-On-Write (COW) Fork Performance Analysis

### Page Fault Frequency:

- In the kernel/trap.c, at the start of code,a global variable `performance_cnt` was initialized to 0.
- Whenever there is a pagefault,the execution goes through usertrap() function.So,this variable is incremented and printed in usertrap() function (of same file) at the part where pagefault is handled.
- By running lazytest.c file, the following results were obtained
    -   simpletest():   1 pagefault
    -   threetest():    In the range of 18810 to 18835 pagefaults
    -   filetest(): 4 pagefaults
- When tested with readonly process, 3 pagefaults were observed.
- When tested with process that only modifies memory, 4 pagefaults were observed.

### Brief Analysis:

#### Benifits of COW fork:

-  COW delays actual copying of pages until a write operation is needed.So, it greatly reduces the memory usage in cases where the child process mainly reads from the parent’s memory space. If a child process simply needs to read data from the parent or exits shortly after forking, the entire memory duplication is avoided.So it helps in `Memory conservation`.
-  In normal fork operations, all pages of the parent process are duplicated, which may use lot of time especially in case of large processes. Whereas in COW, the pages get duplicted only on modification, which minimizes the time required to fork and allows the child to start executing almost immediately.Hence COW improves `Efficiency`.
- When multiple processes or threads are performing similar tasks,they  share the read-only data.This avoids duplication. So, COW is useful in the cases where high concurrency is involved.

#### Further Optimization Areas for COW:

- Some adaptive policies can be used to predict whether a process is likely to modify a page. So, the system can decide more intelligently when and where to apply COW.This minimizes unnecessary copying in situations where changes to pages are unlikely. This approach reduces overhead,resource usage and increases performance.
- We need to try to find a better implementation ways to decrease the overhead of counting reference for pages,allocation/dealloaction etc.

