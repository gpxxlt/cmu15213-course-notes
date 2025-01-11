#malloc #Memory 
#### Overview
- Programmers use **dynamic memory allocators** (such as `malloc()`) to acquire **virtual memory** (VM) at runtime. Dynamic memory allocators manage an area of process VM known as the **heap**. Heap grows toward ***higher*** addresses (contrary to stack), and its growth or shrinkage is controlled by internal function `sbrk()`. 

- There are two major types of dynamic memory allocators. An **explicit allocator** requires application to allocate and free space, e.g., `malloc` and `free` in C. An **implicit allocator** only requires application to allocate, but does not free space, e.g., `new` and garbage collection in Java.

- The major ***reason*** of using dynamic memory allocation is that we often do not know the sizes of certain data structures until the program actually runs. 

#### Memory Allocation Performance
An allocator achieves desired performance if satisfy the following goals:
1. Maximizing **throughput**. By maximizing throughput, operations can be done efficiently. 
2. Minimizing **overhead**. This is equivalent to ***maximizing peak memory utilization.*** By minimizing overhead, we can ensure that the maximum proportion of allocated memory is used for ***actual*** program data. Also, virtual memory is a finite resource, so we should use it efficiently.
However, these 2 goals usually conflict; there's a tradeoff between them.

>[!note] Throughput, *CSAPP, p.845*
>Defined as number of completed requests per unit time. For example, if there are 5000 malloc calls and 5000 free calls in 10 seconds, **throughput** is $(5000+5000)/10 = 1000$ operations/second.

>[!note] Overhead, *CSAPP, p.845*
>**Overhead $O_k$​** is defined by the ***fraction*** of heap space ***not*** used for program data. $$O_k = \left(\frac{H_k}{\max_{i\leq k} P_i}\right) - 1.0$$
>Derived using following parameters:
>- **Payload**: `malloc(p)` results in a block with a payload of `p` bytes. This is ***different*** from the ***actual*** allocated amount of memory to account for header, footer, and alignment purposes.
>- **Aggregate Payload $P_k$:** The sum of currently allocated payloads.
>- **Peak Aggregate Payload** $\max_{i\leq k} P_i$: The maximum aggregate payload at any point in the sequence up to the request.
>- **Current Heap Size $H_k$​:** Assume the heap only ***grows*** when the allocator uses `sbrk()` and never shrinks.

Oftentimes, ***poor*** memory utilization is caused by **fragmentation**. There are two types of fragmentation.

1. **Internal Fragmentation.** Occurs when an allocated **block** is larger than the **payload**. Measured by sum of the differences between the sizes of the allocated blocks and their payloads.
2. **External Fragmentation.** Occurs when there is enough ***aggregate*** free memory, but no single block is large enough.
Now that we know everything about the motivation and metrics and dynamic memory allocation, we want to actually implement such an allocator. Specifically, there are four major concerns:
1. How do we keep track of free blocks? Choosing a **data structure** to manage free blocks. 
2. How to choose an appropriate block to place newly allocated block? Choosing a **placement policy.**
3. After we place a newly allocated block in some free block, what do we do with the remainder of the free block? Choosing a **splitting policy.**
4. What to do with the block that just get freed? Choosing a **coalescing policy.** 

#### Data Structure
##### 1. Implicit List
- Use a linked list to link ***all*** blocks.
- Need to record **block size**, that is, the amount of actual allocated memory. 
- Need to **tag** each block as allocated or free.
- Allocation time complexity is $O(n)$ where $n$ is the number of ***total*** blocks. In the worst case, we need to go through all blocks to find a free one.
##### 2. Explicit List
- Use a linked list to link ***all free*** blocks.
- Need to record block size and tag as implicit list.
- Need an **insertion policy** to determine where to put a ***newly freed*** block.
- Allocation time complexity is ***linear*** in number of ***all free*** blocks. Performance get extremely well when memory is near full.
##### 3. Segregated Free List
- Maintain multiple free lists, where each list holds blocks that are roughly the same size. 
- A popular approach is to choose sizes of powers of 2 for all lists. This is called a **buddy system**. The major advantage is its fast searching and coalescing. The major disadvantage is significant internal fragmentation.
#### Placement Policy
When an application requests a block of k bytes, the allocator searches the free list for a free block that is large enough to hold the requested block. The manner in which the allocator performs this search is determined by the **placement policy** (CSAPP, p.849).
##### 1. First Fit
- Searches the free list from the beginning and chooses the first free block that fits.
- Advantage: Tends to retain large free blocks at the end of the list.
- Disadvantage: Fragmentizes the front of the list.
##### 2. Next Fit. 
- Starts each search where the previous search left off. 
- Faster than **first fit** but even worse memory utilization.
##### 3. Best Fit
- Examines every free block and chooses the free block with the smallest size that fits.
- Good memory utilization, but too time consuming.

