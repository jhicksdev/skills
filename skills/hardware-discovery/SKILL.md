---
name: hardware-discovery
description: >-
  Discover and report the hardware specs of the machine the agent is running on.
  Use this whenever the user asks about system specs, hardware capabilities,
  compatibility checks ("can I run X"), performance troubleshooting, upgrade
  planning, or anything where understanding the physical machine matters.
  This includes questions about CPU, RAM, GPU, storage, PCI devices, kernel
  config, platform architecture, and thermal state. Also use it proactively
  when the task would benefit from knowing the hardware constraints (e.g.,
  compiling with the right -march, checking CUDA support, sizing a model).
---

# Hardware Discovery

When this skill triggers, your job is to discover the hardware of the machine
you're running on and present a structured, readable report. Do this *before*
answering any hardware-dependent question — get the facts first.

## Approach

### 1. Run detection commands in parallel

Always run OS-appropriate commands. On **Linux** the standard set is:

| Category    | Commands                                                                 |
|-------------|--------------------------------------------------------------------------|
| System      | `uname -a`, `cat /etc/os-release`, `hostname`, `uptime -p`               |
| CPU         | `lscpu`, `cat /proc/cpuinfo`                                             |
| Memory      | `free -h`, `cat /proc/meminfo` (just total/available/swap)               |
| Storage     | `lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT,MODEL`, `df -h /`            |
| GPU         | `lspci -nn \| grep -iE 'vga\|3d\|display'`, `glxinfo -B`, `nvidia-smi`  |
| PCI/USB     | `lspci -nn`, `lsusb` (summarize; don't dump all 50 entries)              |
| Network     | `ip addr show` or `ip link show` (brief summary of interfaces)           |
| Thermal     | `cat /sys/class/thermal/thermal_zone*/temp 2>/dev/null`                 |
| Kernel      | `lsmod \| head -20`, `cat /proc/sys/kernel/*` (only if relevant)         |

On **macOS** use `system_profiler SPHardwareDataType`, `sysctl -a`,
`system_profiler SPDisplaysDataType`, `diskutil list`, `sw_vers`.

On **Windows** use `systeminfo`, `wmic`, or PowerShell
`Get-CimInstance Win32_ComputerSystem` etc.

Don't spam the user with every possible value. Run the commands, parse the
output in your head, and present a synthesized report.

### 2. Handle failures gracefully

Some commands need root (`dmidecode`, certain `sysfs` reads). If a command
fails, note that in the report and move on. Don't guess values.

Some tools may not be installed (`nvidia-smi`, `sensors`, `glxinfo`). That's
fine — just skip them and say "not available" if the user would reasonably
expect them.

### 3. Tailor the depth to the question

- **"What's my hardware?"** → Full overview
- **"Can I run X?"** → Focus on the specific constraint (GPU for CUDA, RAM for
  LLMs, CPU cores for compilation)
- **"Why is my system slow?"** → Focus on RAM pressure, thermal throttling,
  disk I/O, swap usage
- **"I want to upgrade"** → Focus on the relevant component, its current
  specs, and compatibility constraints

### 4. Report structure

Present the findings in this format. Don't skip sections — if a value couldn't
be determined, mark it as such.

```
# Hardware Report — <hostname>
<timestamp>

## System
OS:       <distro> <version> (<kernel>)
Platform: <arch>
Uptime:   <uptime>

## CPU
Model:      <vendor> <model>
Cores:      <physical> physical / <logical> logical
Frequency:  <min>–<max> MHz
Cache:      L1i <n> · L1d <n> · L2 <n> · L3 <n>
Features:   <key instruction-set extensions relevant to the user's context>

## Memory
Total:     <amount>
Available: <amount>
Swap:      <amount>

## Storage
<device>: <size> (<type>, <model>)
└─ <partition>: <size> <fstype> <mountpoint>

## GPU
Model:  <vendor> <model>
Driver: <driver info>

## Network
<interface>: <description> (<speed if known>)

## Observations
<1–3 sentence analysis: what stands out about this machine's capabilities
and limitations for the user's likely use case>
```

### 5. Tie the observations to the user's context

The "Observations" section is where you connect the raw data to what the user
actually cares about. If they asked about LLM inference, note the RAM/VRAM
situation. If they asked about compilation, note core count and cache. If
they didn't specify, make reasonable inferences from the conversation.

## What NOT to do

- Don't dump raw command output into the response. The report should be
  curated, not a copy-paste of `/proc/cpuinfo`.
- Don't fabricate values when commands fail. Say "could not determine" or
  "requires root access".
- Don't run expensive benchmarks (stress tests, disk benchmarks) without
  explicit user request.
- Don't install packages to gather info unless the user asks you to.
  Work with what's already on the system.
