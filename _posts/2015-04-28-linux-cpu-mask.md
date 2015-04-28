---
layout: post
title: 'Linux CPU mask'
date: 2015-04-28 23:00
comments: true
tags: [cpumask, linux, kernel]
categories: linux
---

In Linux kernel, we can see some CPU masks are set during SMP bootup procedure. According to
`include/linux/cpumask.h`, there are different kinds of masks,

| cpu masks         |  condition |
|-------------------|-------------------------------------|
| cpu\_possible\_mask | bit 'cpu' set iff cpu is populatable |
| cpu\_present\_mask  | bit 'cpu' set iff cpu is populated   |
| cpu\_online\_mask   | bit 'cpu' set iff cpu available to scheduler |
| cpu\_active\_mask   | bit 'cpu' set iff cpu available to migration |

According to the comment, when CPU Hotplug is not enabled, `present` mask is the same as `possible` one,
likewise for the `online` and `active` mask.

Until now, we can roughly see that `possible` and `present` mask might means that how many CPUs we have
, as well as `online` and `active` might be represented as usable resources in kernel.

# cpu\_possible\_mask and cpu\_present\_mask

It's quite clear to get their differences from comments of `cpumask.h`.

```
If HOTPLUG is enabled, then cpu_possible_mask is forced to have
all NR_CPUS bits set, otherwise it is just the set of CPUs that
ACPI reports present at boot.

If HOTPLUG is enabled, then cpu_present_mask varies dynamically,
depending on what ACPI reports as currently plugged in, otherwise
cpu_present_mask is just a copy of cpu_possible_mask.
```

however it should be architecture dependent. In ARM architecture, there are two ways to mark
a CPU as `possible`, in either device tree or `smp_init_cpus()` machine-implemented operation.
In ARM64, `present` mask will be added in `smp_prepare_cpus()`.

In summary, `possible` mask is set statically and `present` mask means that CPU has left
waiting loop in bootloader.

# cpu\_online\_mask and cpu\_active\_mask

It's quite ambigious to get their differences from boot procedure. They are just enabled at the same time when booting.

According to the implementation of [`set_cpu_online()`](http://lxr.free-electrons.com/source/kernel/cpu.c?v=3.19#L769)

```
void set_cpu_online(unsigned int cpu, bool online)
{
        if (online) {
                cpumask_set_cpu(cpu, to_cpumask(cpu_online_bits));
                cpumask_set_cpu(cpu, to_cpumask(cpu_active_bits));
        } else {
                cpumask_clear_cpu(cpu, to_cpumask(cpu_online_bits));
        }
}
```

`cpu_online_bits` and `cpu_active_bits` are enabled at the same time. It will be more clear to see
their difference from the procedure of removing CPU in Hotplug.

From [1], we can see the procedure of CPU going offline,

```
CPUs going offline have four notification phases: CPU DOWN PREPARE (which might
run on any CPU), CPU DYING (which runs with interrupts disabled on the offlining
CPU while all other CPUs are spinning waiting), CPU DEAD (which runs on some other
CPU after the CPU has gone offline), and CPU POST DEAD (which runs on some
other CPU after some of the CPU-hotplug locks have been dropped). The CPU UP
PREPARE and CPU DOWN PREPARE notifiers are permitted to “fail”, in other words,
to refuse to allow the hotplug operation to proceed.
```

In [`sched_cpu_inactive()`](http://lxr.free-electrons.com/source/kernel/sched/core.c?v=3.19#L5357)

```
static int sched_cpu_inactive(struct notifier_block *nfb,
                                        unsigned long action, void *hcpu)
{
        unsigned long flags;
        long cpu = (long)hcpu;
        struct dl_bw *dl_b;

        switch (action & ~CPU_TASKS_FROZEN) {
        case CPU_DOWN_PREPARE:
                set_cpu_active(cpu, false);
```

And `sched_cpu_inactive()` is a notification callback of CPU Hotplug. Therefore
CPU is marked as inactive when we trying to put it into offline. At this time,
`cpu_online_mask` is still set, but nevertheless scheduler should not migrate any
task to it.


# Reference

1.  Thomas Gleixner, Paul E. McKenney, Vincent Guittot (2012).
    ["Cleaning Up Linux’s CPU Hotplug For Real Time and Energy Management"](http://www2.rdrop.com/~paulmck/realtime/paper/hotplug-ecrts.2012.06.11a.pdf)
2.  Linux-3.19 source and documentation
