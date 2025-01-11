#Memory #Cache #Locality

### 1. Memory Hierarchy
#### Definition and Motivation
Storage on modern computer system utilize the design of **memory hierarchy**, which is an optimal and natural choice given by some fundamental and enduring properties of hardware and software:
1. Fast storage technologies ***cost*** more per byte, have less capacity, and require more power.
2. The gap between CPU and main memory ***speed*** is widening (accessing speed, or latency).
3. Well-written programs tend to exhibit good ***locality***.

>[!quote] Use of Memory Hierarchy, *CSAPP*
>In practice, a ***memory system*** is a ***hierarchy of storage devices*** with different capacities, costs, and access times. CPU registers hold the most frequently used data. Small, fast cache memories nearby the CPU act as staging areas for a subset of the data and instructions stored in the relatively slow main memory. The main memory stages data stored on large, slow disks, which in turn often serve as staging areas for data stored on the disks or tapes of other machines connected by networks.

#### Locality
Such a hierarchical design allows all level of modern computer system, i.e., hardware, OS, web browser, to exploit **locality.** The **principle of locality** states that many programs tend to use data and instructions with addresses near or equal to those they have used recently. These data and instructions are typically stored in the staging area named **cache**.

>[!note] 2 Types of Locality
>1. **Temporal Locality.** ***Recently*** referenced items are likely to be referenced again in the near future. For example, updating the ***same*** variable each iteration.
>2. **Spatial Locality.** Items with ***nearby addresses*** tend to be referenced close together in time. For example, stride one reference pattern in loops. We move to the ***next*** (nearby) address every iteration.


---

### 2. Cache
#### Cache and Memory
More formally, a **cache** is a smaller, faster storage device that acts as a staging area for a subset of the data in a larger, slower device. It serves as a fundamental idea of a memory hierarchy:
> [!note] Fundamental Idea of Memory Hierarchy
> For each level $k$ in memory hierarchy, the faster, smaller device at level $k$ serves as a **cache** for the larger, slower device at level $k+1$.

As we mentioned earlier, a memory hierarchy (hardware) allows us to exploit locality. But to think another way, hardware design actually dictates how we should write our code (software) in order to achieve better performance. They are, thus, two inseparable parts of modern pc's storage system.

|              |      Cache       |    Memory     |
| :----------: | :--------------: | :-----------: |
|  Managed by  |     Hardware     |   Software    |
| Optimization | Without software | With software |
#### Cache Reads: Misses and Policies
Two operations happened in cache are `read` and `write`. When a program request data (in a block $b$) in cache (which is a `read` operation), two things could happen: **hit** (data in cache) and **miss** (data not in cache). 
> [!note] 3 Types of Cache Miss, *CSAPP,  p.612-613*
> - **Cold (compulsory) miss.** Occurs because the cache starts empty (called a **cold cache**) and this is the ***first*** reference to the block.
> - **Capacity miss.** ﻿﻿Occurs when the set of active cache blocks (working set) is larger than the cache. For example, when we access an array repeated inside a loop, but the cache is too small to hold the entire array.
> - **Conflict miss.** Occurs when the level k cache is ***large enough***, but multiple data objects all map to the same level k block. In other words, misses ***due to actual placement policy.***

Whenever there is a miss, the cache at level $k$ must implement some **placement policy** that determines where to place the block $b$ it has retrieved from level $k + 1$. 

>[!note] 2 Types of Placement Policy, *CSAPP,  p.613*
>- **Random placement.** Allows any block from level $k + 1$ to he stored in any block at level $k$. That is, place the block to ***wherever available***. This policy is usually implemented for caches at lower level in the hierarchy due to its high cost.
>- **Restrictive placement.** Use a ***deterministic*** rule to place blocks. For example, a block $i$ at level $k + 1$ must be placed in block ($i$ mod 4) at level $k$. This may leads to **conflict misses**.

If the cache is full, we also need to replace, or evict a block, determined by **replacement policy**.
>[!note] 2 Types of Replacement Policy, *CSAPP,  p.612*
>- A cache with a **random replacement policy**  would choose a random victim block. 
>- A cache with a **least recently used (LRU) replacement policy** would choose the block that was last accessed the furthest in the past.

#### Cache Terms 
There are different designs of cache, though their functionalities and properties remain the same as stated above. There are several parameters and terms to talk about.
>[!note] Cache Terms
>- A **block** is a fixed-size packet of information that moves back and forth between a cache and main memory (or a lower-level cache). 
>- A **line** is a container in a cache that stores a **block**, as well as other information such as the **valid bit** and the **tag bits**. 
>- A **set** is a collection of one or more **lines**. Sets in **direct-mapped caches** consist of a single line. Sets in **set associative** and **fully associative caches** consist of multiple lines. 

>[!note] Cache Parameters and Size, CSAPP, p.617
>- Number of **bytes per block** denoted as $B$.
>- Number of **lines per set** (equivalent to **block per set**) denoted as $E$. 
>- Number of **sets per cache** denoted as $S$.
>- The size of a cache is given by $B \times E \times S$ data bytes.

#### Cache Types
There are 3 types of cache.
1. **Direct Map Cache.** Each set contains only 1 line (block), we have $E=1$.
2. **E-way Set Cache.** Each set contains $E$ lines (blocks), we have $E=E$.
3. **Fully Associative Cache.** The cache only contains 1 set, we have $S = 1$.

These parameters are closely related to **physical addresses** that are used to locate data in cache. A physical address consists of 3 parts: tag, set index, and block offset. 
1. **Tag** determines if there is a cache **hit**. It is given the first $t$ bits in address.
2. **Set index** indicates which set the targeted block should be, given by the middle $s = \log_2 S$ bits.
3. **Block offset** tells us which byte in the block should we start. Think of a block as an array of bytes, and the byte offset as an index into that array. It is given by the last $b = \log_2 B$ bits.
4. Actual layout: `[tag][set index][blockoffest]`
#### Cache Writes
So far we have limited our discussion to `read` operation. There are also notions of **hit** and **miss** in terms of writing data. 
##### Write Hit
1. **Write-through.** Write immediately to memory.
2. **Write-back.** Defer write to memory until replacement of line from cache. If adopting this policy, ﻿each cache line needs a ***dirty bit*** (set if data has been written to cache to distinguish from `read`).
##### Write Miss
1. **No-write-allocate.** Writes straight to memory, does not load into cache.
2. **Write-allocate.** Load into cache, update line in cache.
