---
title: "From Power Button to Desktop: UEFI, Bootloaders, and the Kernel"
date: 2026-05-19T14:00:00Z
draft: false
tags:
  - Operating Systems
  - Computer Science
  - Fundamentals
---

The earlier posts, [how computers run programs](/posts/how-computers-run-programs/) and [inside a running process](/posts/inside-a-running-process/), started with a program already running. This one goes further back, to the moment you press the power button, and follows the chain that has to happen before any of that machinery exists.

---

## What firmware is

Firmware is software. The difference from ordinary software is where it lives and what it does.

Regular software, your browser or the OS, sits on your hard disk or SSD. Firmware is stored on a small chip on the hardware itself, so it is there even with no disk attached.

Its job is to control the hardware it lives on. It is the lowest-level code, the layer that makes a piece of hardware actually work and lets higher software talk to it.

The name captures the idea. It is "firm" because it sits between hardware, which is fixed, and software, which is easily changed. It rarely changes, but it can be updated, which is exactly what a firmware update is.

---

## Firmware is spread across the machine

Firmware is not in one place. Most meaningful components carry some.

- **Motherboard: UEFI.** Stored on a chip on the board. This is the one people usually mean by "firmware." It runs at power-on and drives the boot process.
- **CPU: microcode.** The CPU's own low-level firmware, baked in. It translates the instructions the CPU receives into the actual internal steps the chip performs. It can be patched, which is how some CPU bugs and security flaws get fixed without replacing the chip.
- **Other components.** SSDs, GPUs, network cards, keyboards, printers. Each has firmware controlling how that piece behaves.

The order at power-on: the CPU's microcode is active the instant the chip gets power, since it is part of how the CPU functions at all. Then the motherboard's UEFI runs and boots the machine.

```
+-----------------------------+
|         Motherboard         |
|  +-----------+              |
|  |  UEFI     |  <-- runs the boot
|  |  chip     |
|  +-----------+
|                             |
|  CPU (microcode inside)     |
|  SSD (firmware inside)      |
|  GPU (firmware inside)      |
|  NIC (firmware inside)      |
+-----------------------------+
```

---

## What UEFI is

UEFI is the motherboard's firmware. It runs when you press the power button, before the OS exists.

Its job is to initialize the hardware, then find the OS on storage and hand control over to it.

It replaced the older BIOS. The main differences:

- Supports drives larger than 2 TB, which BIOS could not.
- Startup is faster.
- Has **Secure Boot**, which checks the OS is signed and untampered before loading it.
- The settings screen is graphical with mouse support, rather than the old text-only BIOS menus.

UEFI also needs a small dedicated partition on the disk, the **EFI System Partition**, where bootloaders live. That is where it looks to find what to start.

---

## UEFI runs outside the process world

This is worth pausing on, because it breaks the model from the earlier posts.

At power-on there is nothing in RAM. RAM is empty and not even initialized yet. So UEFI cannot be loaded into RAM to run, because there is no usable RAM and nothing available to do the loading. It executes directly from its chip on the motherboard.

That is precisely why firmware sits on a chip. It is the code that has to run when nothing else is available.

There is also no OS at that point, so there is no kernel space, no user space, no PCBs, and no scheduler. All of that is created later by the OS. UEFI runs before any of that machinery exists.

One nuance: partway through, once UEFI has initialized RAM, it does copy parts of itself into RAM to run faster. But this is still before the OS, so it is not a process. It is just firmware using memory it has only now switched on.

So UEFI runs from its chip, outside the process world entirely. The process, RAM, and scheduler model only starts existing once the kernel is loaded and running.

---

## What UEFI actually does

It works in stages, each preparing the machine a bit more.

```
1. POST                Check essential hardware (CPU, RAM, basic components).
                       Beeps or an error if something critical fails.

2. Initialize hardware Turn on and configure RAM, storage controllers, USB, graphics.
                       This is the step that makes RAM actually usable.

3. Detect devices      Find drives, keyboard, mouse, network.

4. Read settings       Boot order, Secure Boot on/off, other options.
                       Stored on the board, so they survive a power-off.

5. Find bootloader     Look at the EFI System Partition, follow the boot order,
                       locate the bootloader file.

6. Verify (if Secure   Check the bootloader is properly signed and untampered.
   Boot is on)

7. Hand off            Load the bootloader into RAM and give it control.
                       UEFI's job is finished.
```

In one line: UEFI wakes up the hardware, gets RAM working, finds the OS starter on disk, and passes control to it.

---

## The bootloader

The bootloader is a small program whose only job is to load the OS kernel into RAM and start it.

UEFI cannot do this itself. UEFI knows hardware, not operating systems. It does not understand Linux or Windows kernel formats, where they sit, or what settings they need. So each OS ships its own bootloader that does know, and UEFI only needs to find and start that.

The EFI System Partition is a small partition on the drive, formatted so UEFI can read it, holding these bootloader files. UEFI looks there because it is a standard location every OS agrees to use.

Common examples are **GRUB** for Linux and **Windows Boot Manager** for Windows.

What the bootloader does once running:

- Finds the kernel file on disk.
- Loads it into RAM, into what becomes kernel space.
- Passes it settings, such as which drive holds the root filesystem and any boot options.
- Hands over control.

If you have multiple operating systems installed, this is also the stage that shows the menu asking which one to boot.

---

## Everything runs from RAM

Every stage in the chain gets loaded into RAM to run.

```
UEFI  --loads into RAM-->  Bootloader
                                 |
                                 v  loads into RAM
                              Kernel
                                 |
                                 v  starts processes
                          User-space processes
                          (each in its own RAM slice)
```

This follows the rule from the first post: the CPU can only execute code that is in RAM, so anything that needs to run has to be copied there first.

The one exception is UEFI itself, which ran from its motherboard chip because RAM was not initialized yet. Once UEFI switches RAM on, everything after it follows the normal rule.

**What happens to the bootloader afterward.** Nothing actively unloads it. The kernel simply takes over and treats that memory as free. The bootloader's bytes may still be sitting there, but they are abandoned and get overwritten as the kernel allocates memory. There is no cleanup step. They are just no longer referenced.

The pattern across the whole chain is worth noticing: each stage exists only to start the next one, then gets out of the way. UEFI knows hardware but not operating systems. The bootloader knows how to start one specific kernel but nothing else. The kernel then runs everything. Each stage hands off to something that knows more than it does.

---

## Who loads the OS

No single thing loads the whole OS. It happens in two stages.

1. The **bootloader** loads only the kernel into RAM and starts it. Then it is done.
2. The **kernel** takes over and brings up everything else. It initializes memory management, sets up the scheduler, starts drivers, and then launches the first user-space process, which starts all the remaining services, the login screen, and the desktop.

This connects back to something from the first post: only the essential parts of the OS live in RAM. Other pieces stay on disk and get pulled in on demand.

The full chain:

```
UEFI  ->  Bootloader  ->  Kernel  ->  First user-space process  ->  Rest of the OS
```

---

## The kernel is software, but not a process

The kernel is software, the core part of the OS. It sits in a layered picture:

```
Firmware (UEFI, microcode)  ->  lives on chips in the hardware
Kernel                      ->  software loaded from disk into RAM at boot
Your programs               ->  software loaded into RAM when you run them
```

What separates the kernel from ordinary software is privilege, not its nature. It runs in privileged mode with full access to hardware and all memory. Your programs run restricted in user space and must ask the kernel through system calls to do anything privileged. Same category as your browser, very different powers.

But although the kernel's code and data sit in kernel space in RAM, calling it a process is not right. The kernel is the thing that creates and manages processes. It defines what a process is, gives each one memory, creates PCBs, and schedules them. It sits above that system rather than being an entry in it.

The concrete differences:

- **No PCB of its own.** Every process has one. The kernel does not, because it is the thing that writes PCBs.
- **Not scheduled.** The scheduler decides which process runs next. The kernel is the scheduler. Nothing schedules it.
- **No user-space slice.** Processes get isolated user-space memory. The kernel lives in kernel space with full privileges.

So when does kernel code actually run? Not as a separate scheduled thing. It runs when something triggers it: a system call from a program, a hardware interrupt, or a timer tick. The CPU switches into privileged mode, runs kernel code, then returns to whatever process it was running.

One footnote: some kernels have internal helper threads for background jobs, and those are scheduled somewhat like processes. But the kernel proper is not one.

---

## Kernel space is smaller than you would think

It is tempting to assume the OS lives in kernel space and your programs live in user space. That is not the split.

- **Kernel space holds the core only.** Memory management, the scheduler, drivers, system call handling. The privileged machinery.
- **User space holds everything else,** including most of what you would call the OS. Background services, the login screen, the desktop, the file manager, the settings app. All of these are ordinary processes with their own PCBs, scheduled like your programs.

```
KERNEL SPACE   The privileged core only: memory manager, scheduler,
               drivers, system call handling.

USER SPACE     Everything else, including most of "the OS":
               login screen, desktop, file manager, settings, your apps.
```

The dividing line is privilege, not whether something belongs to the OS. Code goes in kernel space only if it genuinely needs unrestricted hardware and memory access. Everything else is kept out, so that a fault stays contained.

That is why your desktop can crash and restart without taking the machine down, but a kernel fault brings down the whole system.

---

## The whole chain

Power hits the board. The CPU comes alive, its microcode active as part of how the chip functions. The CPU immediately starts executing UEFI from the motherboard chip.

UEFI runs its self test, initializes the hardware including RAM, detects devices, reads its settings, and finds the bootloader on the EFI System Partition. If Secure Boot is on, it verifies the bootloader is signed. Then it loads the bootloader into RAM and hands over.

The bootloader finds the kernel on disk, loads it into kernel space in RAM, passes it its settings, and hands over. Its own memory is left behind to be reclaimed.

The kernel takes over. It sets up memory management, the scheduler, and drivers in kernel space. Then it launches the first user-space process, which brings up the rest of the OS: services, the login screen, the desktop, all as ordinary user-space processes.

From that point the machinery from the earlier posts is in place. Any program you run is loaded from disk into its own user-space slice, gets a PCB in kernel space, and is scheduled onto the CPU alongside everything else.

Each stage in that chain knows only enough to start the next one, and then steps aside.

---
