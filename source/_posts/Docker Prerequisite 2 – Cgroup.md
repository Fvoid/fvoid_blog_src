---

title: Docker Source Code Analysis
date: 2020-06-06 09:07:21
tags:

---

# Docker Prerequisite 2 – Cgroup

## 1. Purpose

Control Group (**cgroups**) allow processes to be grouped hierarchically and the specific details of this hierarchy are one area. The cgroup then will create each individual subsystem. The subsystem is like a `resource controller`.

Each subsystem can:

- store some arbitrary state data in each cgroup
- present a number of attribute files for each cgroup in the cgroup filesystem that can be used to view or modify this state data, or any other state details
- accept or reject a request to attach a process to a given cgroup
- accept or reject a request to create a new group as a child of an existing one
- be notified when any process in some cgroup forks or exits

## 2. Subsystem types

### 2.1 Identity Control – network related (net_cl & net_prio)

#### 2.1.1 net_cl

Goal:

- The `net_cls` subsystem tags network packets with a class identifier (classid) that allows the Linux traffic controller (**tc**) to identify packets originating from a particular cgroup.

Usages:

- net_cl.classid
  - it contains a single value that indicates a traffic control handle.
  - can be used with `iptables` to selectively filter packets based on which cgroup owns the originating socket.
  - can also [be used](https://www.kernel.org/doc/Documentation/cgroups/net_cls.txt) for packet classification during network scheduling. The packet classifier can make decisions based on the cgroup and other details, and these decisions can affect various scheduling details, including setting the priority of each message.

#### 2.1.2 net_prio

Goal:

- The Network Priority (`net_prio`) subsystem provides a way to dynamically set the priority of network traffic per each network interface for applications within various cgroups

Usage:

- net_prio.prioidx
  - is used purely to set the priority of network packets, so when used it overrides the priority set with the `SO_PRIORITY` socket option or by any other means. A similar effect could be achieved using `sk_classid` and the packet classifier.

### 2.2 devices

Goal:

- The `devices` subsystem allows or denies access to devices by tasks in a cgroup.

Usage:

- Each group can either allow or deny all access by default, and then have a list of exceptions where access is denied or allowed. The access is a sequence of one or more of the following letters:
  - r – allow to read
  - w – allow to write
  - m – allow to create

### 2.3 freezer

Goal:

- it is to suspends or resumes tasks in a cgroup
- The `freezer.state` is only available in non-root cgroups.
- It has three states:
  - `FROZEN` – tasks in the cgroup are suspended
  - `FREEZING` – the system is in the process of suspending tasks in the cgroup
  - `THAWED` – tasks in th cgroup have resumed.

Usage:

- To suspend a specific process:
  - 1. move that process to a cgroup in a hierarchy which has the `freezer` subsystem attached to it.
    2. freeze that particular cgroup to suspend the process contained in it.

### 2.4 perf_event

Goal:

- When the `perf_event` subsystem is attached to a hierarchy, all cgroups in that hierarchy can be used to group processes and threads which can then be monitored with the **perf** tool, as opposed to monitoring each process or thread separately or per-CPU.

### 2.5 cpuset

Goal:

- It assigns individual CPUs and memory nodes to cgroups.

Usage:

- cpuset.memory_migrate

contains a flag (`0` or `1`) that specifies whether a page in memory should migrate to a new node if the values in `cpuset.mems` change. By default, memory migration is disabled (`0`) and pages stay on the node to which they were originally allocated, even if the node is no longer among the nodes specified in `cpuset.mems`. If enabled (`1`), the system migrates pages to memory nodes within the new parameters specified by `cpuset.mems`, maintaining their relative placement if possible — for example, pages on the second node on the list originally specified by `cpuset.mems` are allocated to the second node on the new list specified by `cpuset.mems`, if the place is available.

- cpuset.cpu_exclusive

contains a flag (`0` or `1`) that specifies whether cpusets other than this one and its parents and children can share the CPUs specified for this cpuset. By default (`0`), CPUs are not allocated exclusively to one cpuset.

cpuset.mem_exclusive

contains a flag (`0` or `1`) that specifies whether other cpusets can share the memory nodes specified for the cpuset. By default (`0`), memory nodes are not allocated exclusively to one cpuset. Reserving memory nodes for the exclusive use of a cpuset (`1`) is functionally the same as enabling a memory hardwall with the `cpuset.mem_hardwall` parameter.

- cpuset.mem_hardwall

contains a flag (`0` or `1`) that specifies whether kernel allocations of memory page and buffer data should be restricted to the memory nodes specified for the cpuset. By default (`0`), page and buffer data is shared across processes belonging to multiple users. With a hardwall enabled (`1`), each tasks’ user allocation can be kept separate.

cpuset.memory_pressure

a read-only file that contains a running average of the *memory pressure* created by the processes in the cpuset. The value in this pseudofile is automatically updated when `cpuset.memory_pressure_enabled` is enabled, otherwise, the pseudofile contains the value `0`.

- cpuset.memory_pressure_enabled

contains a flag (`0` or `1`) that specifies whether the system should compute the *memory pressure* created by the processes in the cgroup. Computed values are output to `cpuset.memory_pressure` and represent the rate at which processes attempt to free in-use memory, reported as an integer value of attempts to reclaim memory per second, multiplied by 1000.

- cpuset.memory_spread_page

contains a flag (`0` or `1`) that specifies whether file system buffers should be spread evenly across the memory nodes allocated to the cpuset. By default (`0`), no attempt is made to spread memory pages for these buffers evenly, and buffers are placed on the same node on which the process that created them is running.

- cpuset.memory_spread_slab

contains a flag (`0` or `1`) that specifies whether kernel slab caches for file input/output operations should be spread evenly across the cpuset. By default (`0`), no attempt is made to spread kernel slab caches evenly, and slab caches are placed on the same node on which the process that created them is running.

- cpuset.sched_load_balance

contains a flag (`0` or `1`) that specifies whether the kernel will balance loads across the CPUs in the cpuset. By default (`1`), the kernel balances loads by moving processes from overloaded CPUs to less heavily used CPUs.

Note, however, that setting this flag in a cgroup has no effect if load balancing is enabled in any parent cgroup, as load balancing is already being carried out at a higher level. Therefore, to disable load balancing in a cgroup, disable load balancing also in each of its parents in the hierarchy. In this case, you should also consider whether load balancing should be enabled for any siblings of the cgroup in question.

- cpuset.sched_relax_domain_level

contains an integer between `-1` and a small positive value, which represents the width of the range of CPUs across which the kernel should attempt to balance loads. This value is meaningless if `cpuset.sched_load_balance` is disabled.
