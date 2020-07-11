---
published: true
title: "Clocks, Timers and Virtualization"
date: 2020-05-12T00:00:00.000Z
categories: ["article"]
tags: ["virtualization", "clocks"]
---

# Clocks and Timers

---

A system might have two kinds of clocks available to use in the system, a Wall-Clock Timer and a Monotonic Clock.

- Wall-Clock Timer

    This represents the time as it will be on a wall-clock. Usually, wall-clock time is counted as the number of time units since January 1, 1970 00:00:00 UTC or `EPOCH`. This clock can go forward or backwards in time as desired by synchronization algorithms (NTP) or the user to represent the current time, due to this reason they are not recommended for measuring durations. CLOCK_REALTIME is a Wall-Clock timer in Linux.

- Monotonic Clock

    A Monotonic Clock is a constantly increasing timer which guarantees that it will never go backwards in time. The frequency rate of the timer might not be constant and be adjusted by synchronization algorithms, which means that one measured second might actually be a different amount of time from another measured second. CLOCK_MONOTONIC is a Monotonic Clock in Linux. The absolute value of the timer is not useful as it based on counting time from an arbitrary point in the past, but rather monotonic clocks are useful for measuring durations with high certainty. Monotonic Clocks often offer a higher resolution than Wall-Clock Timers. 

{{< image src="/img/hardware_clock.png" alt="Hardware Clock" position="center">}}

Hardware Clocks at an abstract level comprise of an oscillator set to a frequency which could be fixed or set by the operating system at boot. There are various types of hardware timers available in a system: Programmable Interval Timer (PIT), CMOS RTC, per-processor advance programmable interrupt controller (APIC), Advanced Configuration and Power Interface (ACPI) Timer, Timestamp Counter, High Precision Event Timer (HPET).

Big words, will be discussed in this section. 

- Tick Counting

    It involves setting up a hardware device to send "ticks" at a fixed interval known to the OS, the OS keeps track of the time based on the received ticks. Example: A fixed interval of 100 times per second can provide a minimum resolution of 0.01s. The hardware device in this case can be the Programmable Interval Timer.

- Tickless Counting

    It involves a hardware counter keeping the count of time units passed since system boot. The operating system only reads the counter when required. The CPU does not spend any time maintaining this timer and provides higher resolution. The X86 Timestamp Counter (TSC) is a Tickless counter which can be read using the `RDTSC` or `RDTSCP` instruction.

Initially, the computer reads latest time from the battery-powered CMOS Real Time Clock to initialize other clocks in the system. 

## Hardware Timekeeping Devices

- [PIT](https://en.wikipedia.org/wiki/Programmable_interval_timer)

    A hardware device which sends a signal when a certain programmed count is reached. Depending on one-shot or periodic modes, it might send the signal only once or keep sending it periodically. Oldschool timekeeping device, oscillates at 1.193182 MHz, 16 bit counter. It consists of three different timers used historically for different purposes like RAM Refresh, PC Speaker Tone Generation, etc. The Intel 8253 is a classic example of a PIT.

- CMOS RTC

    It's a battery-backed clock which stores the wall-clock time to the nearest second. Also, CMOS RTC contains a timer which does tick counting by generating interrupts in the frequency range of 2Hz-8192Hz. It can also provide per second interrupts, imitating a real wall-clock.

- Local APIC Timer

    A per-processor integrated onto the processor itself (whereas the PIT is an external device). It oscillates at the same frequency as the CPU which can vary from machine to machine. Similar to the PIT timer, it can generate interrupts to the CPU in One-Shot or Periodic modes with time intervals configured by the operating system/user. The OS sets a current count on the APIC timer and the circuit counts down until zero is hit. Depending on the bus frequency of the system, the CPU decrements this value. In new CPUs, a TSC-Deadline mode interacts with the CPU TSC and generates an interrupt when a certain TSC value is hit. It is also unreliable as there isn't a reliable way to determine CPU frequency.

- ACPI Timer

    Required as per the ACPI spec, A 24-bit timer oscillating at three times the PIT frequency used for power management in the system.

- [TSC](https://en.wikipedia.org/wiki/Time_Stamp_Counter)

    A tickless counter available on modern x86 Processors which provides apparent time in the system. It is the highest resolution clock available on the system. **It is a 64-bit timer oscillating at the CPUs frequency and counts CPU cycles**. Unlike the PIT or APIC, it does not generate any interrupts. Instead, it is read when required. **A Model-specific Register exposes the TSC** which is accessible in both user mode and kernel mode by default.

    **In newer CPUs, a constant rate TSC is available which oscillates independently of the current CPU frequency**, counting the passage of time rather than CPU clock cycles. 

    **The TSC is known to be unreliable due to rate of tick flaws, multi-core issues, variable CPU frequency etc**. In multi-core systems, the TSC might not be exactly synchronized across each CPU, which can lead to incorrect measurements [5]. Due to out of order execution, the CPU might execute RDTSC earlier than expected. This issue can be solved by serializing execution and running the CPUID instruction before the RDTSC instruction.

    **Due to its high resolution, it is used by side-channel exploitation techniques like Meltdown and Spectre** to measure instruction execution times accurately. [5] highlights various issues with the TSC. 

- HPET

    It is an external hardware timer available on some newer systems. An HPET could be 32-or-64bit and can provider higher resolution than the PIT or CMOS RTC. Due to lax specifications, HPET implementations are not guaranteed to have a high resolution or low drift. It is designed to replace the PIT and CMOS in newer systems.

# Virtualized Timekeeping

---

Hypervisors like VMWare ESXi or Workstation virtualize all the above described timers for the virtual machine, though there might be trade-offs involved.

- The virtualized PIT provides all the modes and three channels of the PIT timer, however, the sound generation timer might not correctly generate the frequency or duration for the requested sound. The virtualized PIT might not be able to generate high frequencies of requested interrupts, ex. 1000hz default on many Linux distros.
- The virtualized CMOS RTC in VMWare emulates all functionalities (TOD, etc) of the RTC implemented as an offset of the host operating systems software clock. If the host clock's time is change, the guest's clock might also be changed. 
In some systems, this can trigger an unexpected feedback loop where if the virtual machine is the time server for the host, they might go back and forth changing their clocks trying to synchronize with each other.
- The virtualized APIC timer, ACPI timer and the TSC both present apparent time matching each other.

I have not been able to find similar implementation information for KVM or other hypervisors.  

### TSC Virtualization

### KVM [5]

In both Intel VT-x and AMD SVM, the TSC can be fully virtualized by trapping the `RDTSC`, `RDMSR`, `WRMSR` and `RDTSCP` instructions [5]. Also, it is possible to pass through the host TSC to the virtual machine. Both trapping and passthrough are implemented in KVM [5].

Issues surrounding TSC synchronization arise from the fact that the CPUs frequency can be altered by the system for different reasons. This can change the rate of TSC per-processor. The Inter-CPU drift is even further amplified in the case of inter-socket drift in the case of multi-socket systems [7]. 

A program reading the TSC from one core might get a different incorrect value of the TSC when reading it from another core. To mitigate this issue solutions include reading the processor ID using the CPUID instruction first and indexing into the TSC array to fetch the specific processors TSC or using the RDTSCP instruction which fetches the processor ID and the TSC value. 

Apart from solutions in software, newer x86 CPUs also include an invariant TSC which operates independently of the CPUs current frequency, guaranteeing the rate of the TSC. This can even be used a wall-clock source.

### VMWare [1]

VMWare provides an exactly synchronized view of the TSC to the virtual machine [1], either backed by the physical TSC itself, or an emulated virtual TSC if the physical TSC turns out to be unsynchronized at the trade-off of increasing execution times of the `RDTSC` instruction. 

### Hyper-V [3]

Hyper-V, similar to VMWare's products, provides virtualized timer services based on a reference time source in the host. According to the Top-Level Functional Specifications (v6.0b), a per-virtual machine reference counter, four synthetic timers per-vCPU, one virtualized APIC timer per vCPU, two timer assists, and a partition reference time enlightenment based on support for an invariant TSC is available to the guest.

- Reference Counter: A strictly monotonic constant rate timer seen by all virtual processors unaffected by processor or bus frequency. It runs at the same rate for all VMs, but reference counters between VM are not synchronized (no same absolute value). This counter counts as long as at least one vCPU in the VM is not suspended. This timer is exposed to the guest via a Model-specific Register (MSR).
- Synthetic Timers: Virtualized interrupt-generataing one-shot or periodic timers. These timers are exposes to the guest via Model-specific Registers.
- Partition Reference Time Enlightenment: An invariant time source for the virtual machine which does not require an intercept into the hypervisor. This is available only if a constant rate TSC is available on the host, as is the case with newer CPUs. 
The invariant TSC solves the issues surrounding the older TSC implementation as its frequency remains constant irrespective of the CPU frequency, as was the case with older TSC implementation. This enlightened reference counter is exposed as a virtual reference TSC page in the guest physical addresses.

{{< image src="/img/hyperv_clocksource.png" alt="HyperV Clocksource" position="center">}}

In the WSL2 Kernel, the only available clocksource is based on this partition wide TSC enlightened clocksource.

### VirtualBox [8]

By default VirtualBox exposes the time sources available to the guest synchronized to the host's monotonic clock. However, there is also a special configuration available to make the virtual guest TSC reflect the actual time spent by the CPU executing the guest. Depending on the hardware TSC, it might be unreliable to use the TSC for measurements.

# Clocks in Linux

---

Latest linux kernels exposes the clocks described in the above sections via the Clocksource framework to expose different clock sources via a generic API [4]. In a bare-metal Linux system, the clocksources might consist of the TSC, ACPI Power Management timer, the HPET etc. 

```bash
# Enumerate your available clocksources.
$ cat /sys/devices/system/clocksource/clocksource0/available_clocksource
tsc hpet acpi_pm

# Get current clocksource
$ cat /sys/devices/system/clocksource/clocksource0/current_clocksource
tsc
```

User-mode time related functions like `gettimeofday`, `clock_gettime` are backed by these clock sources. Using the `clock_sources` system-call, various different types of clocks in the systems can be accessed by a user-mode program.

```c
#include <time.h>

int clock_getres(clockid_t clk_id, struct timespec *res);
int clock_gettime(clockid_t clk_id, struct timespec *tp);
int clock_settime(clockid_t clk_id, const struct timespec *tp);

Time system calls in Linux.
```

Available Clock IDs could include:

- `CLOCK_REALTIME`
- `CLOCK_REALTIME_COARSE`
- `CLOCK_MONOTONIC`
- `CLOCK_MONOTONIC_COARSE`
- `CLOCK_MONOTONIC_RAW`
- `CLOCK_BOOTTIME`
- `CLOCK_PROCESS_CPUTIME_ID`
- `CLOCK_THREAD_CPUTIME_ID`

Run `man clock_gettime` in your favourite shell to learn more about `clock_gettime`. The current clocksource is used as the basis for all these timers.

{{< image src="/img/clock_gettime_man.png" alt="clock_gettime manpage" position="center">}}

As their name suggests, these different clock implementation has different characteristics. The REALTIME clocks give wall-clock time while the MONOTONIC clocks give apparent time.

```c
#include <time.h>
#include <sys/time.h>
#include <stdio.h>

int main(int argc, char **argv)
{
    struct timespec elapsed_from_boot;

    clock_gettime(CLOCK_MONOTONIC, &elapsed_from_boot);

    printf("%d - seconds elapsed from boot\n", elapsed_from_boot.tv_sec);

    return 0;
}

# Reading the system monotonic clock in C.
```

# Clocks in Windows

---

### System Time

For the current time-of-day and date, Windows uses the CMOS RTC at boot to initialize the *System Time* [10]. This time can be obtained by using the [`GetSystemTimeAsFileTime`](https://docs.microsoft.com/en-us/windows/win32/api/sysinfoapi/nf-sysinfoapi-getsystemtimeasfiletime?redirectedfrom=MSDN) function. System Time in windows can be adjusted by the user.

### Interrupt Time

A monotonic clock, *Interrupt Time*[11] is maintained by windows which is updated at 100ns intervals. This timer cannot be changed by the user and is a tick counter (updated via interrupts). The `QueryInterruptTime` and `QueryInterruptTimePrecise` can be used to access the Interrupt Time with a resolution ranging from 0.5ms to 15.625ms.

### High-Resolution Timestamps

To acquire higher-resolution timestamps in Windows[9], The `QueryPerformanceCounter` and `QueryPerformanceFrequency` can be combined to create a timer based on a high-resolution performance counter. `QueryPerformanceFrequency` returns the frequency at which counting is happening. `QueryPerformanceCounter` returns the counter value, dividing the difference between two counter values with the counter frequency can be used to measure duration values at a high resolution. This is the highest resolution timer available in windows. The API itself is not based on reading a performance counter, but rather is an abstraction on the available timers in the system, like the TSC or HPET.

Older versions of Windows implemented the high-resolution performance counter using either the non-invariant TSC which could cause synchronization problems across cores (in XP and 2000) or the HPET (in Vista). Starting from Windows 7, the performance counter is implemented via the constant rate TSC (using the `RDTSC`/`RDTSCP` instruction). Microsoft does not recommend using `RDTSC`/`RDTSCP` due to issues surrounding non-invariant TSCs and to leverage the abstraction and consistency provided by `QueryPerformanceCounter`.

### Windows Time

A low-resolution timer *Windows Time*[12] is also available which counts the number of milliseconds elapsed since boot. It is accessible via the `GetTick` and `GetTick64` function. It is primarily present for backwards compatibility with 16-bit Windows applications.

# Timers used by various languages

---

There are so many timers available in a system! PIT, HPET, RTC, APIC, ACPI, TSC! Wall Clocks, Monotonic Clocks! Let's take a look at which timer is used by the standard library of various languages.

### Golang

The `time` module[13] in Go provides access to both a wall-clock reading and monotonic clock reading. Time-telling operations use the wall-clock reading and time-measuring operations use the monotonic clock reading. 

```go
// time.Now() gives a monotonic clock reading!
start = time.Now()
do_something()
end = time.Now()

// Uses monotonic clock reading as end.Sub(start) is a time-measuring operation.
elapsed = end.Sub(start)

// Falls back to wall-clock reading as end.AddDate(1, 1, 1) is a time-telling operation.
next_year = end.AddDate(1, 1, 1)

// Check time godoc for more information!
// https://golang.org/pkg/time/
```

However, the Golang docs does not specify which APIs does Golang use internally for retrieving wall-clock or monotonic time.

The `time` modules [source code](https://github.com/golang/go/blob/master/src/time/time.go#L1051) reveals `time.Now` on two runtime specific functions `now`  for the wall-clock and `runtimeNano` for the monotonic clock. These functions are individually implemented for each runtime supported by Golang.

{{< image src="/img/go_time_common.png" alt="Go Time Common" position="center">}}

[Source Code](https://github.com/golang/go/blob/master/src/time/time.go#L1051)

The Windows Golang runtime uses `QueryPerformanceCounter` API to implement `runtimeNano` for monotonic clock-reading (which in turn uses the invariant TSC) and the `GetSystemTimeAsFileTime` API to implement  `now()` for wall-clock time reading. 

{{< image src="/img/go_time_windows.png" alt="Go Time Windows" position="center">}}

[Source Code](https://github.com/golang/go/blob/master/src/runtime/os_windows.go#L452)

On the Linux and AMD64 runtime, `clock_gettime` is used to fetch both wall-clock reading and monotonic clock reading, with `gettimeofday` as the fallback function. `CLOCK_REALTIME` is used for wall-clock readings and `CLOCK_MONOTNIC` for the monotonic-clock readings. The actual hardware device is decided by the current clocksource used by the system, which in most cases should either be the HPET or the invariant TSC. Source code for [monotonic clock](https://github.com/golang/go/blob/master/src/runtime/sys_linux_amd64.s#L268) and [wall-clock](https://github.com/golang/go/blob/master/src/runtime/sys_linux_amd64.s#L268).

### Rust

Rust's documentation conveniently documents the source of time used by the timers present in its standard library.

The monotonic clock in Rust `std::time::Instant`[14] uses `QueryPerformanceCounter` on Windows and `CLOCK_MONOTONIC` on Linux.

{{< image src="/img/rust_instant.png" alt="Rust Monotonic Clock" position="center">}}

Wall-clock time is fetched using `std::time::SystemTime`[15] which uses `GetSystemTimeAsFileTime` on Windows and `CLOCK_REALTIME` on Linux.

{{< image src="/img/rust_system.png" alt="Rust System Clock" position="center">}}

### Python 3.8

Python provides `time.get_clock_info` in its time library to get implementation-specific information of the five clocks exposed by the `time` library [16].  We focus on three: `monotonic`, `perf_counter` and `time`.

{{< image src="/img/python_clocks.png" alt="Python Clocks" position="center">}}

Clock implementations in Python 3.8.2

### [`monotonic`](https://docs.python.org/3/library/time.html#time.monotonic)

It returns the value of a monotonic clock, which might not necessarily be the highest resolution clock available in the system.  There are two functions [`time.monotonic`](https://docs.python.org/3/library/time.html#time.perf_counter) and [`time.monotonic_ns`](https://docs.python.org/3/library/time.html#time.monotonic_ns) available to access this clock which returns the value of the clock as fractional seconds.

On Windows, this clock is implemented using `GetTickCount64` which is a low-resolution monotonic clock ([Windows Time](https://docs.microsoft.com/en-us/windows/win32/sysinfo/windows-time) as discussed above).

On Linux, this clock is implemented using `CLOCK_MONOTONIC` .

### [`perf_counter`](https://docs.python.org/3/library/time.html#time.perf_counter)

It returns the value of a monotonic clock which is the highest resolution clock available in the system. There are two functions [`time.perf_counter`](https://docs.python.org/3/library/time.html#time.perf_counter) and [`time.perf_counter_ns`](https://docs.python.org/3/library/time.html#time.perf_counter_ns) available to access this clock which returns the value of the clock as fractional seconds.

On Windows, this clock is implemented using `QueryPerformanceCounter` which in turn uses the invariant TSC internally.

### [`time`](https://docs.python.org/3/library/time.html#time.time)

It returns the value of a wall clock as a floating-point number which represents the time since epoch in seconds. 

On Windows, this is implemented using `GetSystemTimeAsFileTime` which is the *System Time* as discussed in the previous sections.

On Linux, this is implemented using `CLOCK_REALTIME` .

### Node.js

On [Windows](https://github.com/nodejs/node/blob/master/deps/v8/src/base/platform/time.cc#L575), Node.js `QueryPerformanceCounter` for its monotonic clock with fall-backs to HPET or ACPI PM timer in case an invariant TSC is not present and `CLOCK_MONOTONIC` on Linux. It uses `GetSystemTimeAsFileTime` for wall-clock time readings on Windows. 

# References

1. [Timekeeping  in VMWare Virtual Machines](https://www.vmware.com/content/dam/digitalmarketing/vmware/en/pdf/techpaper/Timekeeping-In-VirtualMachines.pdf)
2. [Clocks, Timers and Counters](https://wiki.osdev.org/Main_Page)
3. [Hypervisor Top-Level Functional Specfiications](https://github.com/MicrosoftDocs/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v6.0b.pdf)
4. [Timers and time management in the Linux Kernel](https://0xax.gitbooks.io/linux-insides/content/Timers/linux-timers-1.html)
5. [Linux Kernel Timekeeping](https://www.kernel.org/doc/Documentation/virtual/kvm/timekeeping.txt)
6. [Measuring Latency in Linux](http://btorpey.github.io/blog/2014/02/18/clock-sources-in-linux/)
7. [A Quest Against Time](https://www.linux-kvm.org/images/6/6a/2010-forum-time-keeping.pdf)
8. [VirtualBox Advanced Topics: Fine-tuning timers](https://www.virtualbox.org/manual/ch09.html#fine-tune-timers)
9. [Acquiring high-resolution time stamps](https://docs.microsoft.com/en-us/windows/win32/sysinfo/acquiring-high-resolution-time-stamps)
10. [System Time](https://docs.microsoft.com/en-us/windows/win32/sysinfo/system-time)
11. [Interrupt Time](https://docs.microsoft.com/en-us/windows/win32/sysinfo/interrupt-time)
12. [Windows Time](https://docs.microsoft.com/en-us/windows/win32/sysinfo/windows-time)
13. [time - Golang](https://golang.org/pkg/time/)
14. [std::time::Instant](https://doc.rust-lang.org/std/time/struct.Instant.html)
15. [std::time::SystemTime](https://doc.rust-lang.org/std/time/struct.SystemTime.html)
16. [time.get_clock_info](https://docs.python.org/3/library/time.html#time.get_clock_info)

# TODO

- [Measuring clock resolution.](http://btorpey.github.io/blog/2014/02/18/clock-sources-in-linux/)
- Dive into `hyperv_clocksource_tsc_page` in WSL2 and measure clock resolution.
