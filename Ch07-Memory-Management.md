# chapter 7 | Memory management
**The principal operation of memory management is to bring processes into main
memory for execution by the processor.**
- In a uni-programming system, main memory is divided into two parts: one part for
the operating system (resident monitor, kernel) and one part for the program currently being executed.
- In a multiprogramming system, the “user” part of memory
must be further subdivided to accommodate multiple processes. The task of subdivision is carried out dynamically by the operating system and is known as `memory management`.

## Requirements for memory management
### 1. Relocation
- it would be quite limiting to specify that when it is next swapped back in,
it must be placed in the same main memory region as before. Instead, we may need
to relocate the process to a different area of memory.
- it is not possible for the programmer to
know in advance which other programs will be resident in main memory at the time
of execution of his or her program.
- we would like to be able to swap
active processes in and out of main memory to maximize processor utilization by
providing a large pool of ready processes to execute.

### 2. Protection
- Each process should be protected against unwanted interference by other
processes, whether accidental or intentional.
- so, programs in other processes
should not be able to reference memory locations in a process for reading or writing
purposes without permission.
- In one sense, satisfaction of the relocation requirement increases the difficulty of satisfying the protection requirement.

### 3. Sharing
- The memory management system must therefore allow controlled access to shared
areas of memory without compromising essential protection
- if a number of processes are
executing the same program, it is advantageous to allow each process to access the
same copy of the program rather than have its own separate copy. Processes that
are cooperating on some task may need to share access to the same data structure.

### 4. Logical Organization ( Segmentation mechanism satisfies that )
- main memory in a computer system is organized as a linear,
or one-dimensional, address space, consisting of a sequence of bytes or words. while, most programs are organized into
modules, some of which are un-modifiable (read only, execute only) and some of
which contain data that may be modified
- If the operating system and computer
hardware can effectively deal with user programs and data in the form of modules
of some sort, then a number of advantages can be realized :
    1. **Modules can be written and compiled independently**, with all references from
    one module to another resolved by the system at run time.
    2. With modest additional overhead, **different degrees of protection (read only,
    execute only) can be given to different modules.**
    3. It is possible to introduce mechanisms by which **modules can be shared among
    processes**. The advantage of providing sharing on a module level is that this
corresponds to the user’s way of viewing the problem, and hence it is easy for
the user to specify the sharing that is desired.

### 5. Physical Organization
- The responsibility for this
flow could be assigned to the individual programmer, but this is impractical and
undesirable for two reasons:
    1. The main memory available for a program plus its data may be insufficient. In
    that case, the programmer must engage in a practice known as overlaying. Even with the aid of compiler
    tools, **overlay programming wastes programmer time**.
    2. In a multiprogramming environment, the **programmer does not know at the
time of coding how much space will be available** or **where that space will be**.

>**Overlaying** : the program and data are organized in such a way that various modules
can be assigned the same region of memory, with a main program responsible
for switching the modules in and out as needed.

---
## Memory partitioning (Obsolete i.e. not used in modern OS)
### Technique 1 | Fixed Partitioning
#### case : fixed-size partition
- There are two difficulties with the use of equal-size fixed partitions:
    1. A program may be **too big to fit** into a partition. In this case, the programmer
must design the program with the use of overlays so that only a portion of the
program need be in main memory at any one time.     
        - When a module is needed but is not present, the user’s program must load that module into the program’s partition, overlaying whatever programs or data are there.
    2. Main memory utilization is extremely inefficient due to internal fragmentation.
> **Internal fragmentation**
is when there is wasted space
internal to a partition due to the fact that the block of data loaded is smaller
than the partition.

#### case : unequally sized partition
With unequal-size partitions, there are two possible ways to assign processes
to partitions.
##### Approach 1 :  1 queue per partition
![7.3 a](/imgs/7.3a.png)
- assign each process to **the smallest partition
within which it will fit**.
- Although this technique seems optimum from the point of view of an individual partition, it is not optimum from the point of view of the system as a whole.
- consider a case in which there are no processes with a
size between 12 and 16M at a certain point in time. In that case, the 16M partition
will remain unused, even though some smaller process could have been assigned to
it.

##### Approach 2 : 1 queue for all processes
![7.3 b](/imgs/7.3b.png)
- When it is time to load a process into main memory, **the smallest
available partition** that will hold the process is selected.
- If all partitions are occupied,
then a swapping decision must be made; considering factors, such as priority, and a preference for swapping out blocked
processes versus ready processes.  

#### Pros and cons of fixed partitioning
- **Advantages**
    -  fixed-partitioning schemes are relatively
**simple** and require **minimal OS software and processing overhead**.
- **Disadvantages**
    - The number of partitions specified at system generation time **limits the number
    of active (not suspended) processes in the system**.
    - Because partition sizes are preset at system generation time, **small jobs will not
utilize partition space efficiently**. In an environment where the main storage
requirement of all jobs is known beforehand, this may be reasonable, but in
most cases, it is an inefficient technique.

### Technique 2 | Dynamic Partitioning
![7.4](/imgs/7.4.png)
- this method starts out well, but eventually due to swapping in and out it leads to a
situation in which there are a lot of small holes in memory. As time goes on, memory becomes more and more fragmented, and memory utilization declines. This
phenomenon is referred to as **external fragmentation**
- **overcoming external fragmentation : Compaction**
    - From
time to time, the OS shifts the processes so that they are contiguous and so that all of
the free memory is together in one block
    - **Cons** :
        - it is a time-consuming procedure
        -  wasteful of processor time.
        - the need for a dynamic relocation capability without invalidating the
memory references in the program

#### Placement algorithms (Clever choices to eliminate external fragmentation and compaction)
- **Best-fit** :  chooses the
block that is closest in size to the request.
- **First-fit** begins to scan memory from the beginning and chooses the first available block that is large enough.
- **Next-fit** begins
to scan memory from the location of the last placement, and chooses the next available block that is large enough

![7.5](/imgs/7.5.png)


- **Both fixed and dynamic partitioning schemes have drawbacks :**
    - fixed : limits the number of active processes and may use space inefficiently
if there is a poor match between available partition sizes and process sizes.
    - dynamic : more complex to maintain and includes the overhead of compaction.

### Buddy Systems
- repeatedly split partitions into halves(aka buddies) until the smallest block
`greater than or equal to` requested size is generated and allocated to the request
- Whenever a pair of buddies both become unallocated,
they are removed from that list and **coalesced** into a single block.

![](/imgs/7.6.png)

#### Relocation
A distinction is made among several
types of addresses.

**1. Logical or Virtual address** is a reference to a memory location independent of the current assignment of data to memory; a translation must be made to a
physical address before the memory access can be achieved.

**2. Relative address** is a
particular example of logical address, in which the address is expressed as a location
relative to some known point, usually a value in a processor register.
- Each such relative address
goes through two steps of manipulation by the processor.
    - First, the value in the base
register is added to the relative address to produce an absolute address.
    - Second, the
resulting address is compared to the value in the bounds register.
- If the address is
within bounds, then the instruction execution may proceed. Otherwise, an interrupt is
generated to the operating system, which must respond to the error in some fashion.


**3. Physical or Absolute address**, is an actual location in main memory.

[more on this](https://www.geeksforgeeks.org/logical-and-physical-address-in-operating-system/)

---
## Paging
- The main memory is partitioned into equal fixed-size
chunks that are relatively small, and each process is also divided into small
fixed-size chunks of the same size. Then the chunks of a process, known as **pages**,
could be assigned to available chunks of memory, known as **frames, or page frames**.
- The operating system
maintains a page table for each process. The page table shows the frame location for
each page of the process.
- With paging, the logical-to-physical
address translation is still done by processor hardware.
- Presented with a logical address (page number, offset), the processor uses the page table to produce a physical address (frame number, offset).

## Segmentation
-  the program and its
associated data are divided into a number of segments.
- It is not required that all segments of all programs be of the same length, although there is a maximum segment
length.
- As with paging, a logical address using segmentation consists of two parts, in
this case a segment number and an offset.
- **Compared to dynamic partitioning**
    - **same in** : use of unequal-size segments; suffers from external fragmentation & in the absence of an overlay scheme or the use of virtual
memory, it would be required that all of a program’s segments be loaded into memory for execution.
    - **The difference** : compared to dynamic partitioning, is that with
segmentation a program may occupy more than one partition, and these partitions
need not be contiguous.
- **Address translation**
    - *Extract the segment number* as the leftmost `n bits` of the logical address.
    - *Use the segment number as an index* into the process segment table to find the
starting physical address of the segment.
    - Compare the offset, expressed in the rightmost `m bits`, to the length of the segment. If the offset is greater than or equal to the length, the address is invalid.
    - The desired physical address is *the sum of the starting physical address of the
segment plus the offset*.
