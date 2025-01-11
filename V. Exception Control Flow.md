#Exception #Process #Signal
#### Exceptions
- Simply put, an exception is the change in control flow in response to a system event.
- More formally, an **exception** is an abrupt change in the control flow in response to some change in the processor’s state (system event).
- There are four classes of exceptions:

| Class     | Cause                         | Async/Sync | Return Behavior                     |
| --------- | ----------------------------- | ---------- | ----------------------------------- |
| Interrupt | Signal from I/O device        | Async      | Always returns to next instruction  |
| Trap      | Intentional exception         | Sync       | Always returns to next instruction  |
| Fault     | Potentially recoverable error | Sync       | Might return to current instruction |
| Abort     | Nonrecoverable error          | Sync       | Never returns                       |
Exceptions are the basic building blocks that allow the operating system kernel to provide the notion of a process. This is achieved by, for example, context switching. 
#### Processes
>[!note] Definition of Process
>A **process** is an instance of a running program.

>[!quote] Process, Context, State, *CSAPP chapter 8 opening*
>Each program in the system runs in the **context** of some process. The context consists of the state that the program needs to run correctly. This state includes the program’s code and data stored in memory, its stack, the contents of its general-purpose registers, its program counter, environment variables, and the set of open file descriptors. 
>

Process provides each program with 2 key abstractions:

1. **Private address space.** Each program seems to have exclusive use of main memory, provided by kernel mechanism called virtual memory.
2. **Logical control flow.** Each program seems to have exclusive use of the CPU. Provided by kernel mechanism called **context switching.**

>[!quote] Context Switching, *CSAPP, p.736*
At certain points during the execution of a process, the **kernel** can decide to preempt the current process and restart a previously preempted process. This decision is known as **scheduling** and is handled by code in the kernel, called the **scheduler**. When the kernel selects a new process to run, we say that the kernel has scheduled that process. After the kernel has scheduled a new process to run, it preempts the current process and transfers control to the new process using a mechanism called a **context switch** that (1) saves the context of the current process, (2) restores the saved context of some previously preempted process, and (3) passes control to this newly restored process.

Simply put, the kernel can decide to temporarily stop a process, and returns to it when appropriate, for example, the kernel receives some signal.

In reality, **logical control flows** refers to a sequence of program counter values that correspond exclusively to instructions in executable object files. Some important concepts: ^de33ee
- **Concurrent flow** is a logical flow whose execution overlaps in time with another flow, and the two flows are said to run concurrently (even if they are running on the ***same*** processor). 
- **Concurrency** is the general phenomenon of multiple flows executing concurrently. 
- **Multitasking** is the notion of a process taking turns with other processes.
- If two flows are running ***concurrently*** on ***different*** processor cores or computers, then we say that they are **parallel flows**, that they are running in parallel.

Note that ***only one process an execute at a time on any single core.*** Two processes cannot execute at the same time on any single core (but can execute on different cores in parallel).
#### Process Control
At any time, each process is either in ***one*** of the following states:
1. Running. Process is either executing instructions, or it could be executing instructions if there were enough CPU cores.
2. Blocked / Sleeping. Process cannot execute any more instructions until some external event happens (usually I/O).
3. Stopped. Process has been prevented from executing by user action.
4. Terminated / **Zombie**. Process is finished. Parent process has not yet been notified.

##### 1. Spawning Processes
A **parent process** creates a new running **child process** by calling `fork()`. 
- `fork()` returns 0 to the child process, child’s PID to parent process.
- Child process gets a duplicated (but separated) copy of parent's virtual address space.
- Child process gets an identical copy of the parent’s open file descriptors.
Note that `fork()` is called once but returned twice as both child and parent need to return. Interestingly, we cannot predict execution order of parent and child; they are ***concurrently*** executed.

##### 2. Reaping Child Processes
When process terminates, it becomes a zombie that still consumes system resources. To synchronize with children (by reaping them), one of two systems calls `wait()` or `waitpit()` need to be invoked. Parent is given exit status information, and kernel then deletes zombie child process.

##### 3. Loading and Running Programs
```c
#include <unistd.h>
int execve(const char *filename, const char *argv[], const char *envp[]);
```
The `execve` function loads and runs the executable object file `filename` with the argument list `argv` and the environment variable list `envp`.
>[!note] Program vs. Process, *CSAPP, p.753*
>A **program** is a collection of code and data; programs can exist as object files on disk or as segments in an address space. A **process** is a ***specific instance*** of a program in execution; a program always runs in the context of some process. 

With this distinction, let us revisit the functions mentioned above:
- `fork()` runs the same program in a new child process that is a duplicate of the parent. 
- `execve()` loads and runs a new program in the context of the current process. While it overwrites the address space of the current process, it ***does not create a new process***. The new program still has the same PID, and it inherits all of the file descriptors that were open at the time of the call to `execve`.

#### Shell
[CS213 Shell Lab](https://github.com/cmu15213-s24/tshlab-s24-gpxxlt/blob/master/tsh.c)
>[!note] Definition of Shell
>A shell is an application that runs programs according to user input.

A shell needs **signals** to function property, unless it never creates child processes. A basic shell without signals can only reap foreground jobs, so children run in background will become zombies. 
#### Signals
>[!note] Definition of Signal
>A **signal** is a small message that notifies a process that an event of some type has occurred in the system. 

##### 1. Sending Signal
The kernel delivers a signal to a destination process (might be itself) by updating some state in the context of the destination process. The signal is delivered for one of two reasons,
1. The kernel has detected a system event such as a divide-by-zero error or the termination of a child process.
2. A process has invoked the `kill()` function to explicitly request the kernel to send a signal to the destination process.

##### 2. Receiving Signal
A destination process receives a signal when it is forced by the kernel to react in some way to the delivery of the signal. The process can either ***ignore*** the signal, ***terminate***, or ***catch*** the signal by executing a user-level function called a **signal handler.** 

##### 3. Signal Status: Pending
- A signal is **pending** if sent but not yet received.
- There can be ***at most one*** pending signal of each type.
- Signals are ***not queued.*** If a process has a pending signal of type k, then subsequent signals of type k that are sent to that process are discarded.
##### 4. Signal Status: Blocked
- A process can defer receiving some signals by temporarily **blocking** them, and unblock later. Blocked signals can still be sent, but will not be received until the signal is unblocked.
- Not all signals can be blocked, for example, `SIGKILL` and `SIGSTOP`.
##### 5. Default Action
Each signal type has a predefined **default action**, which is one of:
- The process terminates.  
- The process stops until restarted by a `SIGCONT` signal.
- The process ignores the signal.
This default action can be overwritten by `signal()` function by providing targeted signal number and user-defined **signal handler.**

##### 6. Writing Safe Signal Handler
See CSAPP p.766-p.776 and Lecture 17 Slides p.52 for details.
