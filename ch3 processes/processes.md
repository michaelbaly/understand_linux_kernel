+ Processes are often called tasks or threads in the Linux source code
+
##3.1. Processes, Lightweight Processes, and Threads
+ beginning at the next instruction following the process creation system call
+ parent and child share the same *text*, but they have seperate copies of data, change to memory of either are invisible to another.
+ Linux uses lightweight processes(**LWP**) to offer better support for multithreaded applications

##3.2. Process Descriptor
+ refer to data structure of *struct task_struct*
+ This chapter **focuses on two types of fields** that refer to the **process state** and to **process parent/child relationships**.
###3.2.1. Process State
###3.2.2. Identifying a Process
+ The strict one-to-one correspondence between the process and process descriptor makes the **32-bit address of the task_struct structure** a useful means for the kernel to identify processes

####3.2.2.1. Process descriptors handling
+ For each process, Linux packs two different data structures in a single per-process memory area: a small data structure linked to the process descriptor, namely the thread_info structure, and the Kernel Mode process stack.
+ the **thread_info structure** and the **task_struct structure** are mutually linked by means of the fields task and thread_info, respectively.
+ The kernel uses the **alloc_thread_info** and **free_thread_info** macros to allocate and release the memory area storing a thread_info structure and a kernel stack.

####3.2.2.2. Identifying the current process
+ To get the process descriptor pointer of the process currently running on a CPU, the kernel makes use of the **current** macro, which is essentially equivalent to *current_thread_info( )->task*
+ On multiprocessor systems, it was necessary to define current as an arrayone element for each available CPU.

####3.2.2.3. Doubly linked lists

####3.2.2.4. The process list
+ a list that links together all existing process descriptors

####3.2.2.5. The lists of TASK_RUNNING processes
+ Earlier Linux versions put all runnable processes in the same list called *runqueue*
+ Linux 2.6 implements the runqueue differently. The aim is to allow the scheduler to select the best runnable process **in constant time**, independently of the number of runnable processes
+ This is a classic example of making a data structures more complex to improve performance: to make scheduler operations more efficient, **the runqueue list has been split into *140* different lists!**
+ The enqueue_task(p,array) function inserts a process descriptor into a runqueue list
+ the dequeue_task(p,array) function removes a process descriptor from a runqueue list.

###3.2.3. Relationships Among Processes
####3.2.3.1. The pidhash table and chained lists
+ Scanning the process list sequentially and checking the pid fields of the process descriptors is feasible but rather **inefficient**. To speed up the search, **four hash tables** have been introduced

###3.2.4. How Processes Are Organized
+ grouping processes in other states
+ Processes in a TASK_STOPPED, EXIT_ZOMBIE, or EXIT_DEAD state are not linked in specific lists
+ Processes in a TASK_INTERRUPTIBLE or TASK_UNINTERRUPTIBLE state are subdivided into many classes(wait queues)

####3.2.4.1. Wait queues
+ Wait queues have several uses in the kernel, particularly for interrupt handling, process synchronization, and timing.
+ there are two kinds of sleeping processes: exclusive processes are selectively woken up by the kernel, while nonexclusive processes are always woken up by the kernel when the event occurs.
+ the func field of a wait queue element is used to specify how the processes sleeping in the wait queue should be woken up

####3.2.4.2. Handling wait queues
+ Because all nonexclusive processes are always at the beginning of the doubly linked list and all exclusive processes are at the end, the function always wakes the nonexclusive processes and then wakes one exclusive process, if any exists.

###3.2.5. Process Resource Limits
+ Each process has an associated set of resource limits , which specify the amount of system resources it can use

##3.3. Process Switch
+ the kernel must be able to suspend and resume

###3.3.1. Hardware Context
+ a set of data loaded into registers before process resume the execution is called **hardware context**.
+ linux 2.6 use software support to perform a process switch instead of using hardware support in old version for reasons below.
  + step-by-step switch allows better control over the validity of data being loaded. possible to check the value of **ds and es** registers while the hardware way does not
  + not impossible to optimize a hardware context, while there is room for software.

###3.3.2. Task State Segment
+ in x86 arch, hardware context is stored in TSS, it's forced to set up a TSS for each distinct CPU for two reasons:
+ when x86 CPU switch from user mode to kernel mode, it **fetches the address of kernel mode stack from the TSS**
+ CPU may need to access an I/O permmission bitmap stored in TSS when user mode process attempt to access an I/O port.

####3.3.2.1. The thread field
+ hardware context of process being replaced must be stored somewhere but TSS(linux use a single TSS for each processor instead of one for every process)
+ the context stored in thread of type *thread_struct*

###3.3.3. Performing the Process Switch
+ how the kernel performs a process switch???
+ every process switch consists of two steps:
  + 1.Switching the Page Global Directory to install a new address space;
  + 2.Switching the Kernel Mode stack and the hardware context, which provides all the information needed by the kernel to execute the new process, including the CPU registers.
####3.3.3.1. The switch_to macro
####3.3.3.2. The _ _switch_to ( ) function

###3.3.4. Saving and Loading the FPU, MMX, and XMM Registers
####3.3.4.1. Saving the FPU registers
####3.3.4.2. Loading the FPU registers
####3.3.4.3. Using the FPU, MMX, and SSE/SSE2 units in Kernel Mode
