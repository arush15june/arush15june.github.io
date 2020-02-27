---
published: true
title: "Understanding x86 Virtualization"
date: 2019-11-11T00:00:00.000Z
categories: ["article"]
tags: ["x86", "virtualization", "vmware", "virtualbox", "kvm", "qemu", "systems", "operating-systems"]
---
<base href="{{ .Site.BaseURL }}">
In the late 1990s when internet consumption was booming and microprocessors were growing by Moore's law, A need to build large scale systems was arising. Traditionally, a single computer system runs a single operating system. Instead of adding more power to a single machine running applications, businesses like IBM and Microsoft started building reliable and performant distributed systems using unreliable commodity hardware. Datacenters expanded in capacity comprising thousands of physical machines serving applications on the internet. With rising user demand and underutilization of resources by single users on dedicated machines, virtualization came up as a solution to provision independent virtual machines running on a single physical system. VMWare arose to the technological forefront of virtualization with their state of the art server and desktop virtualization offerings.

Virtualization enables a computer system to run multiple operating systems simultaneously and isolated from each other. x86 is the Instruction Set Architecture found in most PCs and Servers today, we will take a look at how x86 hardware is virtualized by applications like VMWare Workstation and QEMU.

![Virtualized Ubuntu 16 running via VMWare on Windows 10](/img/Ubuntu16VMWare.jpg)
*Virtualized Ubuntu 19 running via VMWare on Windows 10*

## **Advantages of Virtualization**

Developers can have multiple virtual machines running various environments required for development and testing. VirtualBox and VMWare can be used to run virtual machines on a regular laptop, A developer on windows can develop Linux applications as if they were running it natively. Vagrant can be used to automate the process of setting up virtual machines and share them as declarative Vagrantfiles.

Server virtualization enables datacenters to efficiently utilize hardware resources by running multiple operating systems provisioned to many users running on a single machine. Amazon EC2 is the cloud compute service on Amazon Web Services, They use a modified Xen Server based system to provision and manage virtual machines across their datacenters efficiently.

## **Virtualization of Hardware**

Various hardware components constitute a computer, the job of virtualization software is to emulate the hardware identically in software. This emulated computer system is identical to the one running the host and exposes identical interfaces. Virtualization Software executes the code for a guest operating system as it might be running on actual hardware and thus be able to run multiple operating systems on the same machine. The difficult problem is to emulate all the aspects of the hardware identically and efficiently.

The CPU, Memory and Device I/O are the major components needed to be virtualized by virtualization software. This emulation poses several technical challenges in performance, isolation, and security.

Important terms we'll encounter on the way:

- Hypervisor: It is an application that manages many virtual machines running on the system. It is either executed as a separate OS on the bare metal called a Type 1 Hypervisor. Otherwise, it can run inside another operating system as an application called a Type 2 Hypervisor. Microsoft's Hyper-V, The Xen Project are examples of Type 1 hypervisors. VirtualBox, VMWare Workstation, QEMU are examples of Type 2 hypervisors.

- Host Operating System: The operating system via which the Virtual Machines are run. For Type 1 Hypervisors, as in Hyper-V, the hypervisor itself is the Host OS which schedules the virtual machines and allocates memory. For Type 2 hypervisors, the OS on which the hypervisor applications run is the Host OS.

- Guest Operating System: The operating system that uses virtualized hardware. It can be either Fully Virtualized or Para Virtualized. An enlightened guest OS knows that its a virtualized system which can improve performance.

- Virtual Machine Monitor: VMM is the application that virtualizes hardware for a specific virtual machine and executes the guest OS with the virtualized hardware.

- Full Virtualization: The guest OS is presented an identical CPU and hardware as the original host. This is difficult to achieve on x86 without hardware support as some components like the memory management unit is difficult to simulate in software.

- Para Virtualization: The code of the guest operating systems is modified. Aspects like Memory Management, CPU Interrupts, Device I/O systems, etc are modified internally. The interfaces for user applications don't change but the kernel uses these modified interfaces to interact with the hypervisor to access certain functions of the system. This can improve virtualization performance.

![Organization of Microsoft’s Hyper-V, a Type-1 Hypervisor](/img/hyper-v-type-1.png)
*Organization of Microsoft’s Hyper-V, a Type-1 Hypervisor*

Full Virtualization does not require modification of the Guest Operating Systems while Para Virtualization requires modification of the OS kernel to support the modified interfaces.

Emulating the CPU requires the software to present to the virtual machine an identical CPU which can execute all the instructions present in the instruction set. As the virtualized guest OS also runs x86, the programs to be virtualized can be directly executed by the processor and scheduled by the host OS. Challenges arise in the direct execution of code. If given complete access to the CPU and Memory it can modify the memory of the host itself.

x86 contains various sensitive privileged and unprivileged instructions. The virtualization system should prevent the Guest OS to have complete control over the CPU and thus control which instructions can be directly executed by the guest.

The Popek-Goldberg virtualization requirements set up a framework to analyze CPUs on their ability to virtualize efficiently. The x86 architecture without hardware assistance does not fulfil the requirements of effective virtualization due to the presence of many critical privileged and unprivileged instructions. Though, in 2005, with the introduction of hardware-assisted virtualization extensions (Intel VT-x, AMD-V), x86 processors were able to fulfil the Popek-Goldberg requirements.

PS. Just like how a regular computer boots via a bootloader, similarly, a virtual machine starts by executing bootloader code to load the installed guest OS.

![Organization of VirtualBox, a Type-2 Hypervisor](/img/VirtualBox-architecture-Bob-Netherton-2010.png)
*Organization of VirtualBox, a Type-2 Hypervisor*

## **Emulating the CPU**

To emulate the CPU is to execute the instructions present in the Guest OS program. This can be done by directly executing the memory containing these instructions. We need to make sure that the guest OS is not able to manipulate the system outside of regions of its memory and it cannot modify sensitive parts of the host system like the segment descriptors, MMU registers, etc. A situation which allows this is a vulnerability and is often called a VM Escape, allowing the guest OS to escape the isolation of the virtual machine.

### **Binary Translation**

The first virtualization software were based on Binary Translation or BT, It trapped the execution of the instructions from the guest OS and translated them as required. If it required execution of sensitive instructions, it will convert the binary data to use a different instruction in actual execution and return the data as defined in the Virtual Machine Monitor. An example of this is the `cpuid` instruction in x86 which returns information about the CPU itself. This instruction when trapped and identified by the binary translator could be modified to return a different value

Even though most of the non-privileged instructions were directly executed by the CPU as IDENT translation, the extra overhead of the translation layer due to high amount of context switching between the guest and the host for translating and executing the instructions lead to performance degradation in binary translation based virtualization.

Another problem that binary translation poses is that BT based virtualization runs in ring 0 of an Intel processor, thus it is required that the kernel of the guest OS run in a ring with lesser privileges than the hypervisor. Xen used the Ring 0 to run the Xen hypervisor, and the Ring 1 to run the guest kernel with Ring 3 running userland code. This challenge was overcome using hardware virtualization which introduced a ring -1 a higher privileged mode where the hypervisor can run allowing the guest OS to run unmodified on Rings 0 and 3.

### **Hardware Assisted Virtualization**

To overcome the lack of performance in binary translation, CPU makers added virtualization support to the hardware which provided various features like hardware isolation of virtual machines, hardware paging and memory management for individual virtual machines. This enables a virtual machine to run at near-native speeds. Intel VT-x introduces new instructions to x86 enabling virtualization support in the hardware.

This hardware support brings the concepts of hosts and guests to the hardware enabling the CPU to virtualize its components like the MMU, TLB's, etc to each virtual machine automatically. This makes full virtualization possible and virtual machines can execute at near-native performance. The amount of context switching between the guest to host OS decreases and most of the guest OS executes directly on the CPU rather than via the host OS. Only specific instructions like `in` or `out` used for interacting with external devices are forwarded to the VMM or host to be handled. Using this interface, the system can expose devices from the host to the guest.

With hardware virtualization, the hardware can virtualize the virtualization extensions themselves allowing recursive virtualization. The nested virtual machines run at the same level as the first guest OS rather than being executed inside the guest OS.

Hypervisors like Hyper-V run a simple OS on the bare metal CPU and use the hardware virtualization layer to launch virtual machines. One of the Virtual Machines managed by the hypervisor is allowed to manage the hypervisor underneath using Hypercalls, which are similar to system calls to the kernel.

## **Emulating the memory**

An operating system uses virtual memory to create address spaces and processes. Physical memory is divided into contiguous blocks of memory called pages. Address Spaces is a contiguous piece of memory for each process. They can be as large as 128 terabytes. Even though the physical machine does not have terabytes of physical memory, A process can traverse through the whole address space just like contiguous physical memory. Even though 32bit systems couldn't address 4GB of physical memory, processes in 32bit operating systems can have address spaces as large as 4GB. This is called Virtual Memory. Operating Systems use pages and page tables to represent the address space of the process. Implementing virtual memory systems in software is inefficient thus many CPUs come with an inbuilt Memory Management Unit which gives hardware assistance in creating virtual memory systems by providing hardware page tables and translation lookaside buffers.

![page tables.jpg](/img/page tables.jpg)

The x86 architecture has supported virtual memory since the 80386 with an MMU consisting of a TLB and a hardware page table walker. The walker fills the TLB by traversing hierarchical page tables, in physical memory.

The Translation Lookaside Buffer (TLB) caches page table entries resolving to physical addresses. The hardware page table walker will speculatively cache entries as pages are accessed. As the TLB fills up with entries performance of the system increases as it less frequently incurs the penalty of a `page fault` where the system walks the page table structures filling the page table entry in the TLB. In Intel CPUs, The TLBs are flushed of all entries when it executes the `invlpg` instruction or a process context switch.

The VMM virtualizes the MMU of the CPU in the software using the virtual memory mechanisms of the host OS. The Guest OS uses the MMU to create virtual memory systems just as it will on a physical machine and the VMM managed MMU keeps the memory of the guest OS isolated from the host OS implicitly providing security. Virtualizing the MMU in software is a difficult task, it involves using the host's virtual memory mechanisms to build a TLB in software. Software TLBs can be built using Shadow Page Tables. It was inefficient and one of the reasons why software virtualization is not performant.

![shadow-page-table.jpg](/img/shadow-page-table.jpg)

Hardware virtualization extensions virtualize the hardware MMU, Intel VT-x provides this in the form of the Extended Page Table (EPT). Intel x86 CPU's provides the %CR3 register which contains the pointer to the hardware page table and hardware instructions which can walk the page table and read the required page. EPT virtualizes the %CR3 register via the EPTPTR for each virtual machine and provides individual hardware walked page tables.

### References
[1] [The evolution of an x86 virtual machine monitor](http://course.ece.cmu.edu/~ece845/docs/vmware-evolution.pdf)

[2] [Xen and the Art of Virtualization](http://courses.cs.vt.edu/~cs5204/fall14-butt/lectures/xen.pdf)

[3] [From Kernel to VMM](https://www.youtube.com/watch?v=FSw8Ff1SFLM)

[4] [OSTEP](http://pages.cs.wisc.edu/~remzi/OSTEP/)