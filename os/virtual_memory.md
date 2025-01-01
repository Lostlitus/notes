# Virtual memory

VM(Virtual memory) is a key part of modern systems. It's designed to provide an
abstraction to better manage main memory. It has three important capebilities:

1. Act as a sophisticated cache policy between main memory and disk.
2. Simplify memory management by providing a uniform address space layout.
3. Avoid processes from memory accessing corruption.

VM is actually really complex if we dive into its design details. It's
important to state again that VM is designed to manage main memory, and its
complexity comes from the three goals listed above. VM could be greatly
simplified if any of three is not required.  Keep these three goals in mind
when trying to understand VM's architecture.

## Why these three goals

Main memory is organized as an array of M contiguous bytes, and each byte has a
unique *physical address (PA)*. Main memory's PAs together form *physical
address space*.

Early OS let program use PA to access main memory directly, and here comes
three problems:

1. Program needs to write their own policy to swap between main memory and
   disk, which is always complex and error-prone.
2. PCs' size of main memory is not same. Program needs to make sure it can run
   well under different physical address space.
3. A program can access other running program's main memory parts, which leads
   to corruption.

These problems is hard to solve in a elegant way in process level. So OS
designs a unified abstraction with three corresponding goals. That is, VM.

## Clean abstraction

Like main memory, VM is designed to have *virtual address(VA)* and *virtual
address space*. Instaed of directly accessing, program now accesses main memory
via VA indriectly. There is a hardware called *memory management unit(MMU)* in
CPU to translate VA to PA.  The mapping relation is reocrded in a *page table*
data structure stored in main memory. VM is implemented with the combination of
hardware and software.

Although each OS has its own design of VM, they actually all provide rather
same and clean abstractions:

1. Memory management can be done by invoking a few simple system calls.
2. Memory layout of each process is idential, no matter the underlying
   hardware.
3. Only available PAs are mapped, so corruption is avoid logically.

From user's perspective, problems of directly using PA is sloved, and now
interacting with main memory is quite simple.

## VM as a cache policy

### Caching between main memory and disk

Modern system is built with a hierarchy of storage devices, from bottom to top.
And the way upper-level device interacts with the lower-level one is using
caching. Each cache has its policy. The one between main memory and disk is VM.
The minimum cache block unit is called *page*. In main memory's side, it is
*physical page*. In disk's side, it is *virtual page*.  And in VM, it is VP as
well.(it seems unfair to main memory, but the cached is called VM, so...)

When a process is started, VM is created by OS. VP is logically partitioned
into three disjoint subsets:

1. Cached. VP that is currently cached in main memory.
2. Uncached. VP that is currently not cached in main memory.
3. Unallocated. VP that is not mapped to any content in disk.

### Anonymous memory

Not all VPs in VM come from disk. For the volatile data(stack, heap and so on),
VM allocates VPs in main memory first and then maps it to disk. In main
memory's side, the PPs storing the data belongs to *anonymous memory*. In
disk's side, VPs are mapped to *swap space*. There has a few points to note
about anonymous memory:

- In the cases like out of memory happens, VM desides to write VPs to disk.
  This action is called *swapping*. Swapping occurs only when needed, since
  write to disk is slow.
- Anonymous memory is not persistent. The swap space is cleaned when the
  process stopped.

### Page table

The mapping relation between VPs and PPs is stored in a *page table*(one per
progress). A page table is made up with an array of *page table entries(PTEs)*.
A PTE has a valid bit and address field. The two part together indicates the
state of the VP:

1. Cached. Valid bit set and address field stores main memory address.
2. Uncached. Valid bit not set and address field stores disk address.
3. Unallocated. Valid bit not set and address field is null.

Once the program specifies a VA, VM finds VA's PTE from PT, and then operates
depending on PTE's state:

1. Cached. Return data from memory via address field.
2. Uncached. Fetch data from disk to main memory via address field. Update PTE
   and return data.
3. Unallocated. Trigger page fault.

## VM simplifies memory management

Since VM provides a standard view of memory. Process memory image now shares a
similar format. Such uniformity simplifies everything:

- Unified memory layout greatly reduces programer's mental load. Mastering one
  is much easier than many.
- Memory management is delegated to VM system. Now most memory operations are
  handled by VM and VM's optimization is always better. Besides, VM provides
  some interfaces to manually control VM as well, which is enough for almost
  all scenarios.

## VM avoid memory corruption

Main memory stores various kind of runtime data. Accessing data should be well
controled, or main memory may be messed. VM achieves this with its two
characteristics:

1. Unified memory layout prevents programer from accessing other data
   accidently.
2. PTE level permission control.

Point 1 as software guarantee has been talked above. And point 2 as hardware
guarantee is quite simple as well. That is, adding permission bits to PTE. Once
an access occurs, CPU checks the PTE corresponding to the VP. If the type of
the accessing is not allowed(e.g. try to write a VP without write permission
bit set), CPU triggers an exception.

## VM design details

We've talked about big picture things about VM. Now here comes some design
details.

### Address translation

We know that given a VA, VM translates it(to main memory address or to disk
address depending on where the data is stored). Well, to implement it is quite
complex.

#### 1. Combine hardware and software

Modern OS supports VM, and so as modern CPU. A dedicated hardware on CPU chip
called *memory management unit(MMU)* is responsible for translating VA.
Together with VM software, VA is translated properly.

PT is stored in main memory, and a register called *page table base
register(PTBR)* in CPU points to current process's PT. MMU uses PTBR as start
address and higher bits part of VA(the lower bits is not used since CPU fetches
at least a page at once) as index to find the corresponding PTE.

Modern OS implements PT with an array of PTEs, and a PTE is represented with a
unsigned long type variable. A PTE has much more bitwise fields rather than
simply one valid field and one address field. These fields make sure that VM
system can manipulate data between main memory and disk in a correct way.

#### 2. Fetch data

Take Linux as an example. The VM system extracts present bit from PTE, if
set(cached VP), then some bits in PTE are extracted as *physical frame
number(PFN)* to generate PA and the corresponding data in main memory is
returned. If present bit not set, then other bits are used to distinguish if VP
is uncached or unallocated. For the former one, some bits in PTE are extracted
as disk location, OS traps to fetch data from disk to main memory and returns
data.  For the latter one, an exception is triggered.

#### 3. Cache PTE

In the case of VM system, an extra look up for PTE is unavoidable.  Although
PTE is small enough to stored in L1 cache, fetching it still costs a handful of
cycles.  To speed up translation, *translation lookaside buffer(TLB)* is
introduced in MMU as cache(set-associative) for PTEs above L1 cache.

#### 4. Match cache line via PA

OS uses address to matches data in cache line. Now we has VA and PA, which one
should we use? Well, we choose PA. That is, MMU translates VA to PA before
looking up the data in cache. Since distinct VPs may point to same PP,
translating PA first helps better utilize cache.

#### 5. Multi-level PT

Thus far, we've assumed that each process has only one PT. However, this
assumption is not realistic. Let's do some calculation.

```txt
32 bits OS with page in 4 KB, PTE in 4 byte:

how many PTEs:

(2 ^ 32) / (2 ^ 12) = 2 ^ 20

PT size:

(2 ^ 32) / (4 ^ 10) * 4B = 2 ^ 22 B = 4 MB


64 bits OS with page in 4 KB, PTE in 8 byte:

(2 ^ 64) / (4 ^ 10) * 8B = 2 ^ 55 B = 32 PB
```

As we can see, in 32 bits OS with 1024 running processes, PTs sum to cost 4 GB
is resident in main memory or disk. In the case of 64 bits OS, we cannot even
run one process.

So do we really needs this much size of space to describe processes? Obviously
not. A process only uses small part of the address space. Other PTEs in a PT
are in unallocated state, but they costs main memory to store. So here comes
the common approach.  Using a hierarchy of PTs and don't allocate memory for
unallocated PTEs.

In 32 bits OS, for example, we use the highest 10 bits as index for level 1 PT,
and its PTE points to a level 2 PT which then uses the next 10 bits as index to
the final PTE.  Now, only the level 1 PT and frequently used level 2 PTs need
to be resident in main memory. Other PTs are either not allocated.(so no PTE is
allocated) Both level 1 and level 2 PT cost 4KB storage of main memory. Let's
say one process uses 1 level 1 PT and 10 level 2 PTs, then the sum storage is
44KB, a quite reasonable size. In practice, 10 level 2 PTs can handle a process
with `10 * 2 ^ 10 * 2 ^ 12 = 40MB` memory used, also a quite reasonable size.

In 64 bits OS, PT with more hierarchy level is used.(for example 4 level in
Linux) So VM system works great here as well.
