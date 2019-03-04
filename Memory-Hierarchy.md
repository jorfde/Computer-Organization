# On Memory Hierarchy


## 1. Cache memory   

###  Basic concepts
Ideally, memory should be both large and fast. However, the larger the slower and the faster, the smaller.

Thus, memory needs to have a speed hierarchy:
`` CPU registers > Cache > Main memory > disks``

Higher levels of this hierarchy contain code/data with higher probability of being accessed. The same holds true for lower levels (but inversely). 

This is possible thanks to the **principle of locality.**

* **Temporal locality:** loops
* **Spatial locality:** elements in an array

Cache memory us a very small and fast memory whose goal is to reduce the average access time to memory by taking advantage of locality.

* **Unified caches** contain both data and instructions
* **Split caches** contain a cache for data and another for instructions

Transfers between cache and main memory occur in blocks, which are typically sets of 4 or 8 words.   
Blocks are used to exploit locality because when there is a read miss, the full block containing the missing word is brought to cache and copied to a cache slot or line



![](/assets/11.png)

* **Thit:** time to access data in cache
* **Tmiss:** time to access data otherwise \\( \space Thit << Tmiss\\)
* **Hit rate  H = (Naccesses - Nmisses) / Naccesses**
* **Miss rate = 1 - H**
* **Average access time = H * Thit + (1-H) * Tmiss**


### Mapping schemes 

Blocks can be distributed in cache according to several policies, which are outlined in its **mapping sheme.**  

This way, we have different kinds of mapping schemes:

* **Direct mapping:** A memory block can be found only in one line.
* **Fully associative mapping:** A memory block can be found only in any cache line.
* **Set associative mapping:** A memory block can be found only in a subset of all cache lines.


#### Direct Mapping

Please review slides 20-22

Accesses to cache may show miss-and-replacement situations. In a direct-mapped cache there is only one possible slot per block, hence the replacement algorithm is straightforward

Please review the direct mapping implementation in slide 24

* **Amount of control information = #lines * (1 valid bit + #tagBits)**
* **Amount of data = cacheSize(B) + amountControlInfo(B)**
* **Percent of control information = amountControlInfo(B) 7 amountData(B)**

The bad thing about direct mapping is that you may have 2 blocks in memory competing for the same cache line: *conflict misses*.

Imagine we wanted to add all the elements of two arrays.
Cache can only hold one, so we have constant misses and replacing of the contents of cache from A to B. 
This means a hit rate of 0%.

#### Fully Associative Mapping

Even though it is the most flexible mapping policy, full associativity has practical limitations.

* More storage for control data required, since tags are larger
* The need to search the entire cache for a block
* High hardware cost

But, on the positive side

* There are no conflict misses in a fully asssociative mapped cache
* Misses only occur at startup and due to lack of free lines 

Please review the full associative mapping implementation in slide 31


#### Set Associative Mapping

In an n-way set-associative cache, lines are grouped in sets, each of n lines for the corresponding blocks.

All mapping schemes are variations of set-associativity: 

* Direct mapping is  1-way set associative
* Fully associative mapping is n-way set associative, for a cache of n lines.


#### EXERCISE 1

```
Lines = Sets * Ways
Size = Lines * Block Size
Address = Tag + Set + Offset

(despeja para lines, sets, ways y block size)


Offset = number of bits N to address Block Size (2^n)
Set = number of bits N to address Sets (2^n)


```

![](/assets/22.png)

#### EXERCISE 2

```
I-cache
---------- n accesses = n of instructions executed
---------- n misses = n accesses / n words in a block -- (in MIPS 1 word = 4 bytes)

D-cache
---------- n accesses = n of memory instructions executed (lw/sw)
---------- n misses = n accesses / n data words in a block -- (byte = *4, half = *2, word = *1)

Hit rate = (accesses - misses) / accesses

```
A program performs some operation over a vector (they give you the program). The program runs on a processor with one level of dual cache (I-Cache and D-Cache memories). Each cache has a capacity of 64 KB. Both caches use direct mapping and a block size of 8 Bytes. Fill the following table for the given sizes of Vector.


![](/assets/33.png)


Misses can be of one of these types:

* **Compulsory misses:** Occur on the first reference to a block of memory, as it hasn't been in cache yet
* **Capacity misses:** Occur when cache is not large enough to hold every block needed by a program
* **Conflict misses:** the cache is NOT full, the problem is with the configuration of the cache, not the capacity. This will never happen in a FA mapping because everyone can be everywhere, you can only have capacity misses in FA adressing


Reads can be of one of these types

* **Read hit:** what you’te looking for is in cache, so we get the data in cache access time.
* **Read miss:** go to the next level of cache. If cache has no more levels, go to main memory. Pay the cost of accessing slower memory.
	* For a DRAM: ``tRCD + tCL``
	* For a next-level cache, its access time


### Write policies

Writes are not so simple... When we get a write hit, if we write to cache only, then memory and cache may become inconsistent.

If a piece of data resides in 1+ places we might have a consistency problem. If the info is the cache of processor1 and in memory, processor 2 will look for it in memory because it cannot see the cache of processor1.

Thus, we have several policies for handling write hits:

* **Write through:** (the whole memory hierarchy). Data is written both to cache and memory (on next level). Penalty: every write access is also a memory access.

* **Write back:** Write only in cache, take note that the block has been modified  (dirty bit) and update it to memory when replacing  the next block in cache
	* This requires an additional control bit: the **dirty bit**, that marks the block as modified.

And several policies for handling write misses:

* **Write allocate:** treat the miss as a read miss and bring the offending block to cache

* **No Write allocate:** treat writes ass less frequent, so data is only written in next level; the written block is not brought to cache



![](/assets/44.png)

### Replacement algorithms 

**LRU:**   Each line (or way) in the set has an associated counter of n bits, with n = log2 (Number_Of_Ways). You know how it works. Review slide 52 for more detail on counters.

In the end, we need the following control information: 1 valid bit + _ tag bits + 1 dirty bit + N LRU bits.

All these bits must be replicated for all cache lines

### Multilevel caches

We introduce a second level of cache, larger but slower than L1. Now, the average accces time is:  

* **Average access time = H1 * Thit(L1) + (1-H)H2 * T(L2) + (1-H1)(1-H2)Tmiss**

Accesses to cache in a multilevel system may be **sequential** (an L1 miss is followed by a L2 access) or **parallel** (both L1 and L2 are accessed in parallel).


![](/assets/55.png)

### Examples
* Intel Pentium
* MIPS R2000


## 2. Virtual memory

###  Concept and motivation

VM extends the addressable memory my making use of secondary storage devices (hard disks). This is good because it

* Removes the burden on programmin with a limited amount of memory
* Allows efficient memory sharing between programmes 

With VM the disk serves as storage for all program's needs and only the active portions reside in physical memory.

### Virtual addressing 

Each program is compiled into its own address space, and the program's addressed are referred to as Virtual Addresses.  



### Address translation

Address translation is called *paging*, a VM page is called a *block* and a VM misss is called a *page fault.* Please note that all virtual addresses need to be translated to physical addresses.   

Translation is performed using a page table, so it's fast. Remember that a virtual address is structured into two fields: Virtual page number and offset.


![](/assets/66.png)

Thus, the design choices for VM systems are mostly morivated by the high cost of page faults, so:

* Pages should be large enough to make up for the high access time
* Reducing the fault rate is crucial
* Page faults can be handled in software
* Write through doesn’t work – VM uses write back

But, even with all of this, we must remember that the page table resides in main memory (containing physical page address, valid bit, dirty bit and permission bits for each entry), which gives rise to two problems:

* The page table may grow too large
* Every access to memory requires two accesses (translation + actual access)
	* This can be solved with translation caches! (TLB)

 
#### The TLB

The Translation Lookaside Buffer is possible thanks to locality and it effectiviley means that only a small portion of memory is in use at a particular time.


![](/assets/77.png)

![](/assets/88.png)


## 3. An example combining cache and VM 

### MIPS R2000 and the DECstation 3100

In MIPS R2000, the MMU is in charge of handling VM. On top of that, several instructions exist for managing the TLB in kernel mode, on top of special registers like *index* or *random*. 

* tlbr (TLB read)
* tlbwi (TLB write index)
* tlbwr (TLB write random – for random replacement))

It's interesting to see how Virtual Address Translation is performed on top of the TLB

<img src ="/assets/99.png" width="350" >

The DECstation 3100 also shows an interesting cache access model:

<img src ="/assets/100.png" width="350" >

