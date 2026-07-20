---
title: "Inside a Running Process: Memory, the PCB, and Scheduling"
date: 2026-05-19T12:00:00Z
draft: false
tags:
  - Operating Systems
  - Computer Science
  - Fundamentals
---

The [first post](/posts/how-computers-run-programs/) followed the chain from disk to CPU. This one goes deeper into what actually happens once a program is running: how its memory is laid out, how the OS keeps track of it, and how many programs share the machine at once.

---

## From passive to active

A program sitting on the hard disk is a passive thing. Just saved instructions, doing nothing. The moment you run it, it becomes an active process living in RAM.

When that happens, two things are set up:

1. The OS carves out a slice of memory in RAM for the process. Its code, data, and space for variables load into that slice.
2. The OS creates a small record for it in kernel space called the PCB (more on that below).

So launching a program is really this:

```
Storage (passive program file)
       |
       v  copy into a fresh user-space slice of RAM
       v  create a PCB in kernel space
       |
       v
RAM (active process, tracked by the OS)
```

---

## How RAM is divided, logically

At the top level, RAM splits into two areas.

**Kernel space** is one shared region for the whole system. It holds the OS core and its bookkeeping. Only the OS can touch it.

**User space** is divided into slices, one per process. Each slice is private to its process and isolated from the others.

```
+---------------------------------------------------+
|                  KERNEL SPACE                     |   shared, protected
|         (OS core, PCBs, drivers, tables)          |
+---------------------------------------------------+
|  Process A slice  |  Process B slice  |  ...      |   user space,
|  (private)        |  (private)        |           |   one slice per process
+---------------------------------------------------+
```

The OS code that loads into RAM at boot lives in kernel space. Booting copies the kernel from storage into that protected region, and the CPU runs it from there whenever the OS itself needs to act.

---

## The five segments of a process

Inside a single process's user-space slice, memory is divided by purpose and lifetime into five classic segments:

- **Text** is the code, the program's actual instructions. Read-only.
- **Data** holds global and static variables that were given a starting value.
- **BSS** holds global and static variables that start at zero.
- **Heap** is dynamic memory the program requests at runtime. It grows as needed and lives until the program frees it.
- **Stack** holds local variables and function call frames. It grows and shrinks automatically as functions are called and return.

A useful picture: the stack grows down from the top and the heap grows up from the bottom, so they share the free middle space between them.

```
STACK   (grows down)   local variables, function frames
   free space in the middle
HEAP    (grows up)     malloc / calloc memory
BSS                    globals/statics starting at zero
DATA                   globals/statics with a starting value
TEXT                   the code
```

---

## Where each kind of variable lives

This is the practical payoff of the segment idea.

**Local variables**, declared inside a function, live on the stack. They are created when the function is called and destroyed automatically when it returns. Each function call gets its own block on the stack called a stack frame, holding its locals. When the function finishes, its frame is removed and its locals vanish instantly. That is why a local does not survive after its function returns.

**Global and static variables** live in the data or BSS region. They last the entire run of the program, not just one function call. The difference between global and static is visibility, not location: global is reachable everywhere, static is restricted to one function or file, but both persist the whole run.

**Dynamically allocated memory**, from `malloc` or `calloc`, lives on the heap. A subtle but important point: the pointer variable itself is a local on the stack, but the block it points to is on the heap. The stack holds the address, the heap holds the actual data.

```
STACK:  [ int* p ] ------+
                         |
HEAP:                    +--> [ actual block of bytes ]
```

---

## malloc and calloc

Both are C functions that grab memory from the heap while the program runs. This is dynamic allocation: you ask for the memory when you need it, at runtime, rather than fixing the size when you write the code.

`malloc` allocates a block and leaves it uninitialized, so it contains leftover garbage until you fill it. It takes one argument, the number of bytes.

`calloc` allocates a block and sets every byte to zero, giving you a clean start. It takes two arguments, the number of items and the size of each, and multiplies them for you.

`malloc` is slightly faster because it skips the zeroing. `calloc` is safer when you want a clean zeroed block, and handy for arrays. Both hand back a pointer to memory on the heap, and both come with a duty: heap memory does not free itself, so you must free it when done, or the program leaks memory.

---

## The Process Control Block (PCB)

The PCB is the OS's ID card for a process. It is a small record the OS keeps about each running process, holding its status and key details:

- Process ID
- State: running, waiting, or ready
- Where it is in execution
- Saved register values
- Where its memory lives
- Open files

One PCB per process. It is not the process's memory. It does not hold the stack or heap. It is the OS's notecard about the process, used to track it and to pause and resume it correctly.

The PCB lives in kernel space, the shared protected region, not in the process's own user-space slice. That is deliberate. A process must not be able to reach in and edit its own PCB, changing its priority or tampering with its saved state, so the OS keeps every PCB walled off where only the OS can touch it.

So the relationship is:

```
KERNEL SPACE                 USER SPACE
+-------------+
| PCB for A   | -----points to----> [ Process A's memory slice ]
+-------------+
| PCB for B   | -----points to----> [ Process B's memory slice ]
+-------------+
| PCB for C   | -----points to----> [ Process C's memory slice ]
+-------------+
```

The OS reads and updates each PCB to manage its process, while the real code and data sit out in the user-space slice.

---

## Context switching

The PCB is what makes sharing one core among many processes possible.

When the OS pauses a process, it saves that process's current state, its registers and where it was in execution, into the process's PCB. It then picks another process and loads that one's state from its PCB, and the CPU continues it as if it had never stopped.

```
Process A running on CPU
       |
       v  save A's state -> PCB for A
       v  load B's state <- PCB for B
       |
       v
Process B running on CPU (from exactly where it was paused)
```

This save-into-one-PCB then load-from-another-PCB step is called a **context switch**. Without a place to store each process's progress, the CPU could never pause one process and resume another correctly. The PCB is that place.

---

## The three schedulers

The OS uses schedulers to decide which programs run and when.

```
LONG-TERM scheduler    (slow, rare)
   Decides which programs get ADMITTED into the system at all,
   loaded from disk into RAM. Controls how many are in play.

SHORT-TERM scheduler   (fast, constant, many times a second)
   Decides which of the in-RAM processes gets the CPU RIGHT NOW.
   Also called the CPU scheduler. This is the one doing the juggling.

MEDIUM-TERM scheduler
   Temporarily parks a process out of RAM onto disk when RAM is
   crowded, then brings it back later. This is called swapping.
```

In one line each: long-term controls entry, short-term controls the CPU right now, medium-term controls temporary parking.

The short-term scheduler works through the PCBs. It scans them to find processes that are ready, picks one based on its rules, and has the chosen process's state loaded from its PCB onto the CPU. A helper step called the **dispatcher** does the actual handoff.

---

## Scheduling algorithms

These are the common strategies the short-term scheduler can use to pick who runs next.

- **First Come First Serve** runs processes in the order they arrive. Simple, but a long process at the front makes everyone wait.
- **Shortest Job First** runs the shortest process next. Efficient on average, but long processes can keep getting pushed back.
- **Priority Scheduling** gives each process a priority and runs the highest first. The risk is that low-priority processes may wait forever, called **starvation**.
- **Round Robin** gives each process a small fixed time slice in turn, then sends it to the back of the line. Fair and responsive, the classic choice for sharing.
- **Multilevel Queue** groups processes into separate queues, such as system tasks versus user tasks, each with its own rules.
- **Multilevel Feedback Queue** is like the above, but processes can move between queues based on their behavior. Flexible and widely used in practice.

Two cross-cutting ideas run through all of these:

**Preemptive versus non-preemptive.** Preemptive means the OS can interrupt a running process to give the CPU to another, as Round Robin and most modern systems do. Non-preemptive means a process keeps the CPU until it finishes or waits, as in First Come First Serve.

The trade-off they all balance is fairness, responsiveness, throughput, and avoiding starvation. No single algorithm wins on all counts, which is why real systems use hybrids.

---

## Running many programs at once: three distinctions

These three terms sound alike but mean different things.

- **Multiprogramming** is one core with many separate programs taking turns on it. While one waits, another runs. Only one runs at any instant, but the switching is fast enough to feel simultaneous.
- **Multiprocessing** is many cores, so several separate programs actually run at the same instant.
- **Multithreading** is one single program split into parts, called threads, that run together, either taking turns on one core or truly at once on many.

The clean split:

```
Multiprogramming  : several separate PROGRAMS taking turns on one core
Multiprocessing   : several separate PROGRAMS running at once on many cores
Multithreading    : ONE program split into parts that run together
```

The barista version used throughout this series:

- **Multiprogramming** is one barista juggling several orders, switching to pour an iced coffee while the hot coffee brews, so nothing sits idle.
- **Multiprocessing** is many baristas, each making a separate drink at the same time.
- **Multithreading** is one order broken into steps worked in an overlapping way, whether by one barista switching or several sharing the job.

---

## What decides concurrency versus parallelism

Concurrency means many things in progress over the same period, taking turns. Parallelism means many things running at the same instant.

The distinction matters because they depend on different things:

- **Concurrency** comes from how the program is structured, splitting work into parts that can progress independently. A single core can do this by taking turns.
- **Parallelism** needs multiple cores. No amount of clever software gives you true parallelism on a single core. The hardware sets the ceiling.

So a single-core machine can do multiprogramming and concurrency, but never real parallel work. Add more cores and several of those turns can happen at the same instant.

---

## Who controls threading

Threading is mostly the program's own doing. Every program starts with one thread by default, the main thread. It becomes multithreaded only if the code deliberately creates more threads. If it never does, it stays single-threaded for its whole run.

This matters: if a program was not written to use threads, the OS will not add threading to it on its own, even when the opportunity is obvious. Only the program knows which parts of its work are safe to split without corrupting shared data. The OS just sees a running process and will not guess.

What the OS does do is spread separate programs across cores. So many single-threaded programs still fill the cores between them, but any one unthreaded program stays on a single thread and leaves other cores unused by it.

The division of labor:

```
Program author  : decides to split into threads, and how
OS              : decides when those threads run
Hardware        : decides whether they can truly run at the same instant
```

One tie-back to memory. Threads live inside one process. They share that process's heap and code, but each thread gets its own stack. That shared memory is exactly what lets threads cooperate on the same data, something separate processes cannot easily do, since each process is isolated in its own slice.

```
Process
+---------------------------------------------+
|  Shared: TEXT (code), DATA, BSS, HEAP       |
|                                             |
|  Thread 1 stack   Thread 2 stack   ...      |
+---------------------------------------------+
```

---

## Single-threaded versus multithreaded languages

The label describes what a language lets you do by default.

A **multithreaded language** gives you built-in tools to create real threads that can run across cores. Java, C, C++, C#, Go, and Rust are examples.

A **single-threaded language** runs your code on one thread. JavaScript is the classic case: its core runs on one thread and handles many things concurrently through other mechanisms, not by running your code on several threads at once.

One nuance: even among multithreaded languages, they differ in how truly parallel those threads can be. A few allow you to write threads but stop them from running on multiple cores at the same instant, so you get concurrency but not real parallelism. So it is less a clean two-way split and more a spectrum of how much real threading a language allows.

---

## Putting it together

A computer starts and the OS core loads into kernel space in RAM. Now the machine has an operating system running.

You launch a program living on disk as a passive entity. On execution it becomes active. The OS gives it a private user-space slice in RAM, where its code, data, heap, and stack live, and creates a PCB for it in kernel space that points to that slice.

This keeps happening for many programs at once. The scheduler reads the PCBs to decide which process gets the CPU, saving and restoring each one's state through its PCB as it switches between them. On a single core they take turns. Across many cores they genuinely run in parallel. And any individual program can further split itself into threads if it was written to.

That is the full life of a running program: from a passive file on disk, to a tracked, scheduled, memory-managed process sharing the machine with everything else you have open.

---
