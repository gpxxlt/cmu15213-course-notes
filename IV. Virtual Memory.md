#Cache #DRAM

>[!quote] Virtual Memory Brief, *CSAPP chapter 9 opening*
>In order to manage memory more efficiently and with fewer errors, modern systems provide an ***abstraction*** of **main memory** known as **virtual memory** (VM)... Virtual memory gives applications powerful capabilities to create and destroy chunks of memory, map chunks of memory to portions of disk files, and share memory with other processes.

#### Physical and Virtual Addresses
The **main memory** of a computer system is organized as an array of $M$ contiguous byte-size cells. Each byte has a unique **physical address** (PA). Modern computer system adds virtual addressing on top of that. 

With virtual addressing, the CPU accesses main memory by generating a virtual address (VA), which is translated to the appropriate physical address before being sent to main memory. The translation is done by **MMU**, memory management unit.

#### Virtual Pages
**Virtual pages** are blocks of **virtual memory.** They are stored on **disk**, and ***cached*** in **DRAM**. At any point in time, the set of virtual pages is partitioned into three disjoint subsets:
- ***Unallocated.*** Pages that have not yet been allocated (or created) by the VM system. Unallocated blocks do not have any data associated with them, and thus do not occupy any space on disk.
- ***Cached.*** Allocated pages that are currently cached in **physical memory** (DRAM).
- ***Uncached***. Allocated pages that are not cached in physical memory (DRAM).
#### DRAM Cache Organization
Notably, disk is about ***100,000 times slower*** than a DRAM. Therefore, miss penalties are very costly, and this is the major concern when designing DRAM caches:
1. Virtual pages tend to be ***large***. They are typically at least 4KB.
2. DRAM caches are [[II. Memory Hierarchy and Cache#Cache Types|fully associative]] (one set rules all). Any virtual page (in disk) can be placed in any physical page (in DRAM cache).
3. More advanced replacement policy with sophisticated algorithms.
4. Always use [[II. Memory Hierarchy and Cache#Write Hit|write-back]] instead of write-through.

Like all caches, there are hits and misses. 
- A DRAM cache hit is called **page hit**: reference to VM word that is in physical memory. 
- A DRAM cache miss is called **page fault**: reference to VM word that is not in physical memory.
#### Page Tables
Page table is a data structure for the VM system to know caching details of a virtual page. Specifically,
1. Whether a **virtual page** is cached somewhere in DRAM.
2. If it is cached (page hit), which **physical page** it is cached in.
3. If it is not (page fault), where it is stored on disk (virtual address).

Essentially, a page table is an array of page table entries (**PTEs**). ***Each page*** in the **virtual address space** has a PTE at a fixed offset in the page table. For our purposes, we will assume that each PTE consists of a **valid bit** and an **n-bit address field.**
- If the valid bit is set, the address field indicates the start of the corresponding **physical page** in DRAM where the virtual page is cached.
- Valid bit not set and address field is null. Virtual page has not yet been allocated.
- Valid bit not set and address field is not null. The address points to the start of the **virtual page** on disk.
#### Address Translation

>[!note] Translation from Virtual Address to Physical Address by MMU
>1. CPU sends a virtual address (**VA**) to MMU.
>2. MMU split **VA** into virtual page number (**VPN**) and virtual page number (**VPO**), and indexing into the **page table** using VPN.
>3. MMU gets **PPN** from the page table.
>4. MMU constructs a physical address (**PA**) by combining physical page number **PPN** and VPO (equals to physical page offest **PPO**).
>5. PA is then used to fetch actual data from cache or main memory.


#### VM as a Tool for Memory Management
1. Simplifying memory allocation. Each virtual page can be mapped to any physical page, and a virtual page can be stored in different physical pages at different times.
2. Sharing code and data among processes. Each **process** has its own virtual address space that gets mapped to a shared physical space.

#### Summary
- Virtual memory uses DRAM as a cache for parts of a virtual address space. 
- Virtual memory allows each process to have its own private linear address space, so cannot be corrupted by other processes.