---
title: "How a Computer Actually Runs Your Programs"
date: 2026-05-19T10:00:00Z
draft: false
tags:
  - Operating Systems
  - Computer Science
  - Fundamentals
---

Every time you open an app, a chain of hardware kicks into motion behind the scenes. This post follows that chain from the disk all the way to the CPU, one idea building on the last.

---

## 1. Why RAM is faster than a hard disk

RAM is electronic. Data is fetched by sending electrical signals, almost instantly. A hard disk drive (HDD) is mechanical. It has spinning platters and a moving arm that must physically travel to the right spot before it can read. Waiting for physical parts to move is slow.

HDDs still work this mechanical way today. But most modern devices now use SSDs (solid state drives), which have no moving parts and are electronic like RAM. SSDs are much faster than HDDs, but still slower than RAM.

One point worth being clear about: HDDs did not "evolve into" SSDs. They are two separate technologies. SSDs largely replaced HDDs, but an HDD is still an HDD and still spins.

**What each is made of:**

```
RAM  = DRAM chips        (electronic, forgets everything when power is off)
SSD  = flash / NAND chips (electronic, keeps data with no power)
HDD  = spinning platters  (mechanical, keeps data with no power)
```

RAM and SSD are cousins. Both are silicon and have no moving parts. The key difference is what they are for. RAM is temporary and built purely for speed. SSD and HDD are permanent storage, which adds overhead that slows them down.

---

## 2. The speed hierarchy

Everything in a computer is arranged fastest to slowest. Each fast layer is fed by the slower layer below it.

```
FASTEST
  Registers   <- inside the CPU, the value being worked on right now
  L1 cache    <- tiny, fastest cache, closest to the core
  L2 cache    <- bigger, slightly slower
  L3 cache    <- larger, shared across cores
  RAM         <- main working memory
  SSD         <- permanent storage, electronic
  HDD         <- permanent storage, mechanical
SLOWEST
```

**Rough real numbers:**

```
RAM  : ~100 nanoseconds   ~20 to 50 GB/s
SSD  : ~100 microseconds  ~3 to 7 GB/s     (about 1,000x slower to respond than RAM)
HDD  : ~10 milliseconds   ~0.1 to 0.2 GB/s (about 100,000x slower than RAM)
```

To feel the scale, imagine RAM's response took 1 second:

```
RAM : 1 second
SSD : ~15 minutes
HDD : ~1 day
```

This gap is the reason for almost everything that follows.

---

## 3. The CPU can only work out of RAM

The CPU can never run a program directly from the hard disk or SSD. It only works with data in RAM.

```
Program sitting on disk  --(you open it)-->  copied into RAM  -->  CPU runs it from RAM
```

The CPU only ever talks to RAM and its own tiny cache. Storage is too slow to feed the CPU directly. If the CPU had to wait on storage, it would sit idle most of the time and waste its speed.

This is both a design choice (matching speeds) and a wiring reality. The CPU is physically wired to RAM through the memory bus using direct memory addresses. Storage connects through a different, slower pathway and does not use those addresses. So RAM is a required middleman.

Analogy: the hard disk is a filing cabinet, RAM is your desk, the CPU is you. You do not read documents inside the cabinet. You pull them onto your desk first, then work. That copying step is also why programs take a moment to load.

One note on physical distance. RAM is only a few inches from the CPU and an SSD maybe a few inches to a foot. At those distances travel time is basically nothing. So distance is not the real reason RAM is faster. The real reasons are the connection type (a fast dedicated memory bus versus a slower storage pathway) and how RAM is built (for pure speed).

---

## 4. Registers, cache, buffers

These are all small fast memories, but they do different jobs.

**Registers** are the smallest, fastest storage of all. Tiny slots inside the CPU holding the exact values being worked on this instant. If the CPU adds 5 and 3, both numbers sit in registers, the addition happens, the result lands in a register. They are the CPU's hands.

**Cache** keeps recently or frequently used data close so the CPU can reuse it fast without going back to slower RAM. It comes in levels L1, L2, L3.

**Buffer** is temporary storage that bridges a speed gap during a transfer. Data waits there while a fast side and a slow side catch up.

The clean distinction:

```
Cache  = keep HOT data handy for fast reuse
Buffer = a waiting room to smooth out speed differences during transfer
```

The CPU's fast memory is about reusing hot data, so it is called cache, not buffer.

---

## 5. Buffers for I/O devices

I/O devices like a printer, disk, keyboard, or network card often deal with buffers in two places at once. It is not one or the other.

```
Device buffer (built into the hardware)   +   RAM buffer (created by the OS)
```

Data flows through both. Here is an example of reading from a disk:

```
Disk reads data
   -> disk's own buffer
   -> travels across
   -> OS buffer in RAM
   -> program / CPU uses it
```

Each buffer solves a speed gap at its own stage. Most devices have some buffer of their own, and the OS also keeps buffers in RAM to back them up. The two layers work together.

Another example: a printer has a buffer so the CPU can dump a whole document quickly and move on, while the printer prints slowly from its own buffer.

---

## 6. Program vs Process

This is one of the most important distinctions.

```
PROGRAM = a passive file sitting on disk. Just instructions. Does nothing.
PROCESS = a program that is running, loaded into RAM, actively executed by the CPU. It is alive.
```

- The passive entity (program) lives on the HDD or SSD.
- The active entity (process) lives in RAM.

The moment you launch it, it moves from passive on storage to active in RAM:

```
Storage (passive program)  --launch-->  RAM (active process)
```

One program can become many processes. Open the same browser three times: one program on disk, three separate processes in RAM, each with its own memory and its own live state. Typing in one does not affect the others.

Analogy: the recipe on paper is passive. Each act of cooking it is active. One recipe, many simultaneous cook alongs.

---

## 7. Process states: execution, waiting, I/O event

While a process runs, it moves through states.

```
EXECUTING (Running) : actively using the CPU right now
WAITING (Blocked)   : paused, needs something (usually I/O), not using the CPU
I/O EVENT           : the thing it was waiting for finishing, which wakes it up
```

How they flow:

```
Process is EXECUTING on the CPU
   -> it needs data from disk, so it enters WAITING and gives up the CPU
   -> meanwhile the CPU runs OTHER processes (no time wasted)
   -> the I/O EVENT completes
   -> the process is ready again and waits its turn
   -> back to EXECUTING
```

This is the whole point of the design. Slow I/O would otherwise freeze the CPU, so a process steps aside during I/O and frees the CPU to do other work. Nothing sits idle.

---

## 8. Schedulers

Schedulers are parts of the OS that decide which programs run and when.

```
LONG-TERM scheduler  (slow, rare)
   Decides which programs get ADMITTED into the system at all
   (loaded from storage into RAM). Controls how many are in play.
   Like a bouncer at the door.

SHORT-TERM scheduler (fast, constant)
   Decides which of the in-RAM processes gets the CPU RIGHT NOW.
   Runs many times per second. Also called the CPU scheduler.
   Like a waiter deciding which table to serve this second.

MEDIUM-TERM scheduler
   Temporarily removes a process from RAM (parks it on disk) when RAM
   is crowded, then brings it back later. This is called swapping.
   Like asking seated guests to wait in the lounge when it is packed.
```

---

## 9. Kernel space vs user space

When the OS loads into RAM, memory is divided into two zones.

```
KERNEL SPACE : reserved for the OS core (the kernel). Protected and privileged.
               Only the kernel can touch it. Holds kernel code and its live state.

USER SPACE   : where your normal programs run. Each program gets its own slice,
               isolated from other programs and from kernel space.
```

The reason is protection. A normal program cannot reach into kernel space and crash the whole system. If an app misbehaves, it is contained. When a program needs something only the kernel can do, like access storage or the network, it asks the kernel through a **system call**. It cannot just reach in.

"OS space in RAM" means kernel space: the protected region where the OS core does its privileged work.

---

## 10. Is the OS core stateless?

At startup a small program copies the essential parts of the OS (the **kernel**, the heart of the OS) from storage into RAM. The CPU runs it from there. This is booting.

Not the entire OS is loaded, just the core and whatever is currently needed. Other pieces like drivers, features, and tools are pulled into RAM on demand and dropped when not in use.

Now the subtle part. Whether the kernel is stateful or stateless depends on the timescale:

```
DURING a single run  -> STATEFUL
   The kernel constantly tracks changing info: running processes, memory use,
   open files, connected devices, what the CPU does next. Its CODE is mostly
   fixed once loaded, but its DATA changes every moment.

ACROSS reboots (normal shutdown) -> STATELESS
   The kernel is loaded clean from storage every boot and builds its state from
   scratch. Nothing from the previous run's live state is restored. Shut down and
   that state is gone. What is stored on disk is only the fixed code, the blank template.
```

One exception is **hibernation**. Instead of shutting down, the computer copies all of RAM (including the kernel's live state) onto disk, then reloads it at next boot, so you resume exactly where you left off.

```
Normal shutdown -> next boot is a fresh, blank state
Hibernate       -> state saved to disk, restored on next boot
```

---

## 11. Concurrency vs Parallelism

This is the core idea behind modern processors.

```
CONCURRENT : many processes IN PROGRESS over the same period, taking turns on one
             core. Interleaved, not at the same instant. Feels simultaneous.

PARALLEL   : many processes executing AT THE SAME INSTANT. Needs multiple cores.
```

- **Multiprogramming** means 1 core, many processes in RAM, run concurrently by rapid switching.
- **Multiprocessing** means many cores, genuinely parallel, plus concurrency on top since each core still juggles many.

### The coffee shop model

```
CONCURRENT (one barista = one core)
   While the hot coffee brews (an I/O wait), the barista does not stand idle.
   He switches to pouring an iced coffee. One person, one thing at a time, but by
   filling the waits he keeps several orders in progress. That is concurrency.

PARALLEL (many baristas = many cores)
   Several baristas, each working a separate order at the same instant. That is
   true parallelism, capped by how many baristas you have.
```

In the real world, the multi barista shop also has concurrency on top of parallelism. Each barista is individually just as clever, filling their own waits, and there are several of them at once. The two multiply, so far more orders get done. A 4 core CPU behaves the same: 4 in parallel, and each core concurrently juggling many.

---

## 12. Cores, threads, and hyper-threading

```
A CPU with 4 cores  -> 4 processes truly in PARALLEL (one per core, at one instant)
```

But the number of processes loaded in RAM is much higher, often hundreds. They are not all running at once. They take turns, and the short-term scheduler rotates them onto the cores fast enough that it feels like far more than 4 are running.

```
In parallel (truly at once) = number of cores
Loaded in RAM (taking turns) = many more, limited by RAM size
```

**Hyper-threading** is Intel's name for the trick. The general term is SMT, or Simultaneous Multithreading. It lets one physical core handle two processes at once. It exploits the same idle waits. While one process waits briefly, a second slips in and uses the gap. The core presents itself as two **logical cores**.

```
16 physical cores + hyper-threading -> shows as 32 logical cores, juggles 32 processes
```

But it is not as good as real cores. Two real cores are two full baristas. Hyper-threading is one barista working a bit more efficiently, roughly a 20 to 30 percent gain, not double.

**How many threads per core?**

```
2 per core  -> standard on mainstream Intel and AMD chips
4 or 8      -> specialized chips like IBM POWER (SMT4 / SMT8), servers, mainframes
```

Why not more? The trick relies on filling idle gaps. Two processes fill most of them. A third or fourth finds fewer gaps left, so the gain shrinks while complexity rises. For everyday CPUs, 2 is the sweet spot. Heavy server workloads with lots of waiting benefit from 4 or 8.

---

## 13. Intel Core tiers (i3 / i5 / i7 / i9)

These are Intel's mainstream consumer CPUs. The naming is a tier system, weakest to strongest.

```
i3 : entry-level    (browsing, email, light work)
i5 : mid-range      (common all-round sweet spot)
i7 : high-end       (gaming, video editing, demanding apps)
i9 : top-tier       (heavy editing, 3D, high-end gaming)
```

Going up the tiers generally means more cores, more threads, higher speeds, and more cache. A modern i7 or i9 uses everything above at once:

```
Many cores               -> real parallelism (multiprocessing)
Each core hyper-threaded -> 2 processes per core
Each core still juggling -> concurrency, switching during waits
```

Modern wrinkle: recent Intel chips mix two core types.

```
P-cores : Performance. Powerful, hyper-threaded, for heavy work.
E-cores : Efficient. Smaller, save power, for light background tasks.
```

So an i9 might be something like 8 P-cores plus 16 E-cores, with the chip routing work to the right type.

Notes: the number after the tier, like `i7-14700`, gives generation and model. Intel is renaming these to Core 5, Core 7, Core Ultra 7 and dropping the "i", same idea. AMD's Ryzen 3/5/7/9 line mirrors i3/i5/i7/i9.

---

## 14. Are all modern CPUs multi-core?

Yes. Essentially all modern CPUs are multi-core. Single-core CPUs are extinct in everyday devices. Phones, laptops, desktops, and tablets all ship with multiple cores.

```
Multi-core     = multiple cores on ONE chip (universal today)
Multiprocessor = multiple separate CPU chips in one machine (common in servers)
```

In casual use, "multiprocessing" covers both. The point is more than one core running things in parallel.

Why the shift happened: chipmakers hit a physical wall making a single core faster, mainly heat and power limits. So instead of one ever faster core, they went wider with more cores. That is why parallelism, concurrency, and scheduling now matter for every device.

---

## 15. CPU vs GPU

The GPU takes the parallel idea to the extreme, but for a different kind of work.

```
CPU : a few powerful cores (say 16). Each is smart and fast. Good at varied,
      sequential, complex tasks (running the OS, apps, logic).

GPU : thousands of small simple cores. Each is weak alone, but there are so many
      that they crunch huge amounts of SIMILAR work all at once. Massively parallel.
```

GPUs were built for graphics, where every pixel needs similar math and there are millions of them at once. Thousands of tiny cores each handle a pixel. People then realized the same pattern of "same math, massive data, all at once" fits other work:

- AI and machine learning. Training models is tons of identical math on huge data. This is why GPUs power the current AI boom.
- Video rendering, simulations, crypto.

Coffee analogy: a CPU is a few master baristas handling any complex custom order. A GPU is a thousand people each doing one identical simple step. Useless for a custom drink, unbeatable when you need 10,000 identical cups at once.

---

## 16. Stack and heap per process

Running programs need two kinds of memory: stack and heap.

```
STACK : temporary, short-lived data. Function calls, local variables.
        Grows and shrinks automatically as functions run and finish. Fast, orderly.

HEAP  : data the program allocates on demand, living as long as needed.
        Flexible, managed by the program.
```

Every process gets its own private stack and heap. They are not shared across processes. Three processes means three separate stacks and three separate heaps, all isolated. One process cannot see or touch another's. This is the same protection model as user space.

One refinement, for **threads** inside a single process:

```
Across processes         : each has its OWN stack AND OWN heap (fully isolated)
Threads within a process : each thread has its OWN stack, but they SHARE the one heap
```

The shared heap is how threads cooperate on common data, while separate stacks keep their individual function calls independent.

---

## Quick recap of the whole chain

```
HDD / SSD  (passive programs, permanent, slow)
    |  launch: copy into RAM
    v
RAM        (active processes, temporary, fast; kernel space + user space;
            each process has its own stack + heap)
    |  feed the CPU
    v
L3 -> L2 -> L1 cache   (keep hot data close)
    |
    v
Registers  (the exact values being worked on this instant)
    |
    v
CPU cores  (parallel across cores, concurrent within each core,
            hyper-threading = 2 processes per core)
```

Every layer exists to bridge the speed gap to the layer above it, and the whole system is arranged so the fast CPU never has to sit idle waiting on slow storage.

---
