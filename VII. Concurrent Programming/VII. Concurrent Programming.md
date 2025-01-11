
### Threads
As we have discussed [[V. Exception Control Flow#Processes|earlier]] we can view process as **process context** plus program code, data, stack, etc. Alternatively, we can treat process as multiple **threads** that have (relatively) ***separated*** stack space but ***shared*** code and data (there is also kernel context, but for simplicity it is not mentioned). 

##### Thread Memory Model
There are multiple threads running in the same process with *some* sort of independency, but at the same time sharing some other properties.
- Each thread has its *own* **thread id (TID)**
- Each thread has its *own* [[V. Exception Control Flow#^de33ee|logical control flow]]  
- Each thread has its *own* stack for local variables but ***not protected*** from other threads
- Each thread *shares* the same code, data, and kernel context

| Separated - Thread Context                                       | Shared - Process Context                                                                                              |
| ---------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| TID, stack, stack pointer, PC, condition codes, and GP registers | code, data, heap, and shared library segments of the process virtual address space, open files and installed handlers |

##### Mapping Variable Instances to Memory
Note that even each thread has its own separate thread context, ***any thread can read and write the stack of any other thread.*** In multi-threaded C programs, there is not, unfortunately, a golden rule for determining whether a variable is shared or not. It highly depends on the execution reality. 

A less strict rule is that, a variable `x` is shared if and only if multiple threads reference ***at least one instance*** of `x`. To reason about variable sharing among threads, we could take advantage variable **scope**. 

- Global variables
	- ﻿﻿Variable declared outside of a function.
	- ﻿﻿Virtual memory contains ***exactly one*** instance of any global variable. That is, writes to this variable from any thread can modify it.
- ﻿﻿Local automatic variables
	- ﻿﻿Variable declared inside function ***without*** `static` attribute
	- ﻿﻿Each thread stack contains one instance of each local variable. That is, modification on ***non-static*** variables is updated in every thread.
- ﻿﻿Local static variables
	- ﻿﻿Variable declared inside function ***with*** `static` attribute
	- Virtual memory contains exactly one instance of any local static variable.
- ﻿﻿`errno`
	- ﻿﻿Declared outside a function, but ***each*** thread stack contains one instance.


---

### Synchronization

While shared variables are handy, we use them under the risk of **data races** and ***synchronization errors.*** Shared variables need to be modified according to our design. However, due to concurrent execution of threads it is highly possible that the instructions are not executed in the order we want.

##### Process Graphs
A **progress graph** depicts the discrete **execution state space** of concurrent threads. Each point on the graph corresponds to a possible **execution state.** State changes each time an instruction is executed. In other words, one state change correspond to one instruction in **assembly code.** 

Other important terms (included in slides) include **trajectory, critical section, and unsafe region.**

![[Process Graph.pdf]]

##### Fixing Data Races 
###### 1. Mutex (MUTual EXclusion)
Our goal is to avoid data races by avoiding unsafe trajectories. To achieve this, we need to guarantee mutually exclusive access to each **critical section.** Mutex achieves this by surrounding critical session with `lock` and `unlock` operations:
- `m` is a mutex, an opaque object which is either locked or unlocked
	- Starts out unlocked
	- `static pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;`
- `lock(m)`
	- If the mutex is currently not locked, lock it and return 
	- Otherwise, wait until it becomes unlocked, then retry
- `unlock(m)`
	- Can ***only be called*** when mutex is locked, and if so, change mutex to unlocked

###### 2. Semaphores
Semaphore is a generalization version of mutex. While mutex uses a `bool` to control lock and unlock, semaphores uses an `unsigned int` to adapt for more complex environment.
- `s` is a semaphore, created with some non-negative value
	- `static sem_t s;`
- `P(s)` 
	- If `s` is zero, wait for a `V(s)` operation to happen, then subtract 1 from `s` and return
- `V(s)`
	- Simply add 1 to `s`
	- If there are any threads waiting inside a `P(s)` operation, resume ***one*** of them
- Unlike **mutex**, no requirement to call `P(s)` before calling `V(s)`

###### 3. Atomic Memory Operations
These are special hardware instructions that prevent data races. Atomic memory operations are actually used to implement mutexes, semaphores, etc. They are fast but very hard to use correctly. 

#### Synchronization Dilemma

1. **Deadlock.** None of the parties make progress because they don't want to give out the resources they already have. 
	- Example. Suppose we have two locks and two threads (each thread calls lock mutex1 and mutex2, so need both locks to progress), if one thread grabs both locks then no deadlock will occur. However if each of the threads grabs one lock then a deadlock occurs.
2. **Livelock**. None of the parties make progress (but they tried) because they consistently get ***denied*** from their request of some resources.
3. **Starvation**. One thread cannot get access to shared resources for an unacceptably long time. 
	- Typically because others are given higher ***priority***. This priority could be implicit, i.e., others do things much faster than you so you don't even get a chance to perform any tasks.
	- Algorithms that guarantee no starvation are **fair**.

#### Synchronization Patterns

##### 1. Producer-Consumer Model
The producer-consumer model is a common synchronization pattern. Some real-world examples include multimedia processing and event-driven graphical user interfaces. It mainly involves three components:
- **Buffer.** A shared queue-like structure where producers add items and consumers remove them.
- **Producer.** A thread(s) that generates data and places it into the buffer.
- **Consumer.** A thread(s) that retrieves data from the buffer and processes it.

The goal is to ensure proper synchronization so that:
1. The producer does not add data when the buffer is ***full***.
2. The consumer does not retrieve data when the buffer is ***empty***.
3. Access to the buffer is **thread-safe** to avoid race conditions.

Assuming buffer is an n-element queue, the simplest way to ensure proper synchronization is using 2 semaphores and 1 mutex. 2 semaphores are used to control counts of empty and occupied entry to satisfy point 1 and 2. Accessing buffer is protected by 1 mutex, achieving point 3. Specifically,
- `pthread_mutex_t mutex` enforces mutually exclusive access to the queue’s innards 
- `sem_t slots` counts the available slots in the queue  
- `sem_t items` counts the available items in the queue

##### 2. Readers-Writers Problem
Another synchronization pattern is the reader-writer problem. Real world examples include online airline reservation system and multithreaded caching web proxy. Problem statement:  
- Reader threads only read the object
	- ***Unlimited*** number of readers can access the object
- Writer threads modify the object (read/write access)
	- Writers must have ***exclusive*** access to the object  

Read and write lock operations for reference:
- Acquire read lock: `pthread_rwlock_rdlock(pthread_rwlock_t *rwlock)`
- Acquire write lock: `pthread_rwlock_wrlock(pthread_rwlock_t *rwlock)`
- Release (either) lock: `pthread_rwlock_unlock(pthread_rwlock_t *rwlock)`

Improper design of read and write may lead to starvation. For example, if 2 readers interleave with each other, they produce an endless stream of reading. This means acquiring write lock takes endless time.