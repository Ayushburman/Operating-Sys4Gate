# OS for GATE CSE: Mastery Cheat Sheet

Formula-first and trap-aware. High-yield order: Synchronization, Scheduling, Deadlock, Paging/TLB, Page replacement, Disk, File system.

## 1. Basics and Processes

- Mode bit separates kernel and user mode. System call = trap into kernel. Privileged: I/O, set timer, disable interrupts, MMU registers.
- Process states: New, Ready, Running, Waiting, Terminated. Running → Ready = preemption. Waiting → Ready = I/O done.
- Schedulers: long-term (degree of multiprogramming), short-term (CPU), medium (swapping). Dispatcher performs the context switch.
- Thread shares code, data, heap, open files. Own: stack, registers, PC. User-level threads: fast, kernel unaware, one block stalls all. Kernel-level threads: slower, true parallelism.
- fork() returns 0 in child, child PID in parent, -1 on failure. n sequential unconditional forks = 2^n processes total, 2^n − 1 new. Loop of k forks = 2^k total.
- Zombie: finished, parent has not called wait(). Orphan: parent died, init adopts it.
- IPC: pipes, message queues, shared memory (fastest, needs sync), sockets.

## 2. CPU Scheduling

TAT = CT − AT. WT = TAT − BT. RT = first CPU time − AT. Throughput = jobs / time.

| Algorithm | Preemptive | Key facts |
| --- | --- | --- |
| FCFS | No | Convoy effect |
| SJF | No | Min avg WT among non-preemptive; starvation |
| SRTF | Yes | Min avg WT overall; starvation |
| Round Robin | Yes | Quantum q; q → ∞ = FCFS; tiny q = overhead; no starvation |
| Priority | Either | Starvation, fixed by aging |
| HRRN | No | Ratio = (W + S) / S; no starvation |
| MLQ / MLFQ | Yes | Fixed queues vs feedback between queues |

- RR efficiency = q / (q + s), s = context switch time.
- CPU utilization, n processes, I/O wait fraction p: 1 − p^n.
- Burst prediction: τ(n+1) = α·t(n) + (1 − α)·τ(n).
- Traps: in RR, a new arrival joins the queue before the preempted process (standard). Watch idle gaps in the Gantt chart. Apply the stated tie-break.

## 3. Synchronization

Critical section needs: Mutual Exclusion, Progress, Bounded Waiting.

| Solution | ME | Progress | Bounded wait |
| --- | --- | --- | --- |
| Lock variable | No | Yes | No |
| Strict alternation (turn) | Yes | No | Yes |
| Peterson | Yes | Yes | Yes |
| Test-and-Set / Swap | Yes | Yes | No |

Peterson (process i, other j): flag\[i\] = true; turn = j; while (flag\[j\] && turn == j); CS; flag\[i\] = false.

Semaphores:

- wait/P: S−−; if S < 0, block. signal/V: S++; if S ≤ 0, wake one.
- Counting semaphore init n, after x waits and y signals: S = n − x + y. If S < 0, |S| processes are blocked.
- Mutex has ownership; a binary semaphore does not.

Classics:

- Producer-Consumer: mutex = 1, empty = N, full = 0. Producer: P(empty), P(mutex), add, V(mutex), V(full). Swapping P(empty) and P(mutex) deadlocks.
- Readers-Writers: reader preference starves writers; writer preference starves readers.
- Dining philosophers (n): naive left-then-right deadlocks. Fixes: seat at most n − 1, asymmetric pickup, pick both atomically. Max eating at once = ⌊n/2⌋.
- Spinlock: busy wait, only for short CS on multiprocessors. Priority inversion fixed by priority inheritance.

## 4. Deadlock

Four conditions, all required: Mutual exclusion, Hold and wait, No preemption, Circular wait.

- Prevention: break one condition. Avoidance: Banker's (stay in safe state). Detection: RAG or wait-for graph, then recover. Ostrich: ignore.
- RAG: no cycle = no deadlock. Cycle + single instance per resource = deadlock. Cycle + multiple instances = maybe.
- Deadlock ⊂ Unsafe. Safe = no deadlock. Unsafe does not always mean deadlock.
- Banker's: Need = Max − Allocation. Safety check: repeatedly pick a process with Need ≤ Work, then Work += its Allocation. All finish = safe; that order is a safe sequence.
- Min resources to guarantee deadlock freedom: R = Σ(m\_i − 1) + 1. For n processes each needing m: R = n(m − 1) + 1. Max n for given R: ⌊(R − 1) / (m − 1)⌋.
- Example: 3 processes each needing 2 → R = 3(1) + 1 = 4.

## 5. Memory Management

Fragmentation: variable partitions → external. Fixed partitions and paging → internal.

Placement: first fit (fast), next fit, best fit (smallest hole that fits; leaves tiny holes), worst fit (largest hole).

Paging:

- Page size = frame size = 2^d. Logical address = page number + d offset bits.
- \#pages = LAS / page size. #frames = PAS / page size.
- Page table size = #pages × PTE size. PTE ≥ frame bits + valid, dirty, reference, protection.
- Avg internal fragmentation per process = page size / 2.
- Multilevel: each table fits in one page, so entries per table = page size / PTE size. Levels = ⌈page-number bits / log2(entries per table)⌉.
- Example: 32-bit VA, 4 KB page, 4 B PTE: offset 12 bits, page number 20 bits, 1024 entries per table (10 bits) → 2 levels.
- Inverted page table: one entry per frame, size scales with physical memory, hashed lookup.
- Optimal page size ≈ √(2·s·e), s = avg process size, e = PTE size.

Access time (t = TLB time, m = memory time, h = TLB hit ratio):

- Single level: EMAT = h(t + m) + (1 − h)(t + 2m).
- k levels: TLB miss path = t + (k + 1)m. No TLB: (k + 1)m.
- Demand paging: EAT = (1 − p)·m + p·(page fault service time).

Segmentation: address (s, d). Check d < limit else trap; physical = base + d. External fragmentation, no internal.

Virtual memory:

- Page fault steps: trap, check validity, find free frame (or victim), read from disk, update page table, restart instruction.
- FIFO: Belady's anomaly possible. LRU and OPT: stack algorithms, no anomaly. Clock/Second chance: FIFO plus reference bit. OPT = minimum faults.
- Belady example: 1 2 3 4 1 2 5 1 2 3 4 5 under FIFO → 3 frames: 9 faults, 4 frames: 10 faults.
- Minimum faults = number of distinct pages (compulsory misses).
- Thrashing: low CPU use, heavy paging. Fix: working set model, page-fault-frequency control, reduce multiprogramming.
- Copy-on-write: fork shares pages until one side writes.

## 6. Disk and File Systems

Disk access time = seek + rotational latency + transfer.

- Avg rotational latency = half a rotation = 30 / RPM seconds. 7200 RPM → 4.17 ms.
- One rotation reads one track. Transfer time = data size / transfer rate.
- Capacity = surfaces × tracks × sectors × bytes per sector.

| Scheduling | Key facts |
| --- | --- |
| FCFS | Fair, long seeks |
| SSTF | Greedy, starvation possible |
| SCAN | Elevator; goes to disk end, then reverses |
| C-SCAN | Serve one direction, jump back; uniform wait |
| LOOK / C-LOOK | Like SCAN / C-SCAN but turn at last request, not disk end |

Trap: SCAN and C-SCAN travel to the physical end; LOOK variants do not. Check whether the question counts the return jump.

| Allocation | Key facts |
| --- | --- |
| Contiguous | Fast access; external fragmentation; hard to grow |
| Linked | No external fragmentation; slow random access; lost pointer hurts |
| FAT | Linked with table in memory; random access is fine |
| Indexed | Direct access; index block overhead |

- Unix inode: D direct + single + double + triple indirect. P = block size / pointer size. Max file size = (D + P + P² + P³) × block size.
- Hard link: same inode, link count, same file system only. Soft link: stores path, can dangle, crosses file systems.
- Free space: bit vector, linked list, grouping, counting.
- I/O: polling, interrupt-driven, DMA (CPU involved only at start and end; cycle stealing). Also spooling, buffering, caching.

## 7. Rapid-Fire Traps

- TLB miss is not a page fault. A page fault means the page is not in memory.
- Unsafe state is not deadlock.
- Belady's anomaly: FIFO only (among the standard algorithms).
- Bigger page: smaller page table, more internal fragmentation.
- SJF is optimal for avg WT among non-preemptive; SRTF is optimal overall.
- User-level thread blocking stalls the whole process; kernel-level does not.
- Counting semaphore with S < 0: |S| processes waiting.
- Thrashing is cured by lowering multiprogramming, not by adding more processes.

## 8. Study Loop

1. Read each section once, then solve PYQs topic by topic.
2. Hand-trace Gantt charts, page replacement tables and Banker's. Redo wrong ones after a week.
3. Daily: 5 minutes of formula recall from this sheet.
