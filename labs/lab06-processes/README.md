# Lab 6 — Live processes and memory pressure

Pair: Adiyana Akbayeva (solo) · Date: 22 November 2026

Guests: Ubuntu Server 26.04.1 LTS (4 GiB for the process capture) and Windows 11 Enterprise Evaluation (4 GiB)

## Part 1 — One process, three tools

I chose Python’s HTTP server on port 8000 because the same Python module runs on both guests. The Windows command was `python -m http.server 8000` and the Ubuntu command was `python3 -m http.server 8000`. The two Windows tools show the same child process, PID 1268. The Ubuntu server has its own PID, 1844.

| | Task Manager | Process Explorer | htop / ps |
| --- | --- | --- | --- |
| PID | 1268 | 1268 | 1844 |
| Parent (name and PID) | Not shown | `python.exe` (8108) | `-bash` (1812) |
| “Memory” as shown by default | Private working set: 13,576 KiB | Working Set: 25,456 KiB | RES: 22,420 KiB |
| Private memory | Private working set: 13,576 KiB; Commit size: 16,812 KiB | WS Private: 13,544 KiB; Private Bytes: 16,812 KiB | Private RSS not shown |
| Working set / RES | 25,456 KiB | 25,456 KiB | 22,420 KiB |
| Virtual size | Not shown | Not captured | VIRT: 35,612 KiB |
| Handles / file descriptors | 137 handles | 137 handles | 4 FDs on the repeat run |
| Threads | 1 | 1 | 1 on the repeat run |

The Ubuntu screenshot also shows SHR at 9,544 KiB, CPU at 0.7%, and MEM at 0.6%. I repeated the command-line checks on an isolated copy of the Ubuntu guest. The new server instance had PID 745, so the thread and descriptor counts in the table refer to that instance.

```text
$ ps -o pid,ppid,user,rss,vsz,nlwp,cmd -p 745
    PID    PPID USER       RSS    VSZ NLWP CMD
    745     728 root     22448  33704    1 python3 -m http.server 8000
$ ls /proc/745/fd | wc -l
4
$ sudo ls -l /proc/745/fd | head
total 0
lr-x------  1 root root 64 Nov 22 22:53 0 -> /dev/null
l-wx------  1 root root 64 Nov 22 22:53 1 -> /dev/hvc0
l-wx------  1 root root 64 Nov 22 22:53 2 -> /dev/hvc0
lrwx------+ 1 root root 64 Nov 22 22:53 3 -> socket:[15168]
```

### Where the tools disagree

The two Windows tools agree on the 25,456 KiB working set and the 16,812 KiB of private committed memory. Task Manager calls the committed figure **Commit size** and Process Explorer calls it **Private Bytes**. The private working set differs by 32 KiB between the screenshots. Both tools refresh live, so that small difference can come from separate refreshes.

The Windows private working set counts resident memory private to the process. Ubuntu’s RES covers all resident pages for the server, including shared pages. The figures therefore describe different sets of pages. Ubuntu’s VIRT is the size of the process’s virtual address space, including pages that are outside RAM.

The Linux process had three standard streams and a listening socket. The Process Explorer handle pane also shows Windows object types such as desktops and object directories, so the Windows handle count covers more than the objects represented by those four Linux descriptors.

## Part 2 — Memory pressure

The Ubuntu guest had 2,096 MiB assigned in VMware Fusion for the experiment. Ubuntu reported 1.6 GiB of RAM and 3.8 GiB of swap. The recovery snapshot was named `before-lab6`. All times are shown in Almaty time.

```bash
free -h
swapon --show
cd ~ && mkdir -p lab06 && cd lab06
{ echo "=== BEFORE $(TZ=Asia/Almaty date '+%F %H:%M') ==="; free -h; } > memory-pressure.txt
while true; do echo "=== $(TZ=Asia/Almaty date +%H:%M) ==="; free -h; sleep 3; done | tee -a memory-pressure.txt
```

In a second terminal, I increased the load in this order. After the last run, I stopped the recording loop with Ctrl+C and appended the final `free -h`.

```bash
stress-ng --vm 2 --vm-bytes 70% --timeout 60s
stress-ng --vm 2 --vm-bytes 90% --timeout 60s
stress-ng --vm 2 --vm-bytes 120% --timeout 90s
{ echo "=== AFTER $(TZ=Asia/Almaty date '+%F %H:%M') ==="; free -h; } >> memory-pressure.txt
```

| 19 November | Before (15:53) | Least available (15:57) | Peak swap (15:58) | After (16:01) |
| --- | --- | --- | --- | --- |
| Available | 1.2 GiB | 26 MiB | 86 MiB | 1.4 GiB |
| Swap used | 0 B | 87 MiB | 1.4 GiB | 73 MiB |

The three runs started at 15:54, 15:56 and 15:57. Swap first registered use at 15:56, during the 90% run. The recorder continued sampling roughly every three seconds throughout the run. Available RAM returned to 1.4 GiB at 15:58, just after the 120% run reached its 90-second timeout. The final sample still showed 73 MiB in swap.

### What the kernel did

For the original run, I searched the journal for OOM messages in the interval covered by the recording.

```bash
sudo env TZ=Asia/Almaty journalctl -k --since "2026-11-19 15:53" --until "2026-11-19 16:02" |
    grep -i -E "out of memory|oom-kill|killed process"
```

The search returned no matching lines. At peak swap use, `free -h` still reported 2.4 GiB of unused swap.

On 22 November I retried on an isolated copy of the Ubuntu guest with 1,536 MiB assigned RAM and a 128 MiB swap file. Ubuntu reported 1.4 GiB of RAM and 127 MiB of usable swap. I recorded `free -h` every 0.2 seconds to capture the short drop. I ran the load in a separate control group without a memory limit so the recorder could continue if a worker was killed.

```bash
stress-ng --vm 2 --vm-bytes 150% --oomable --timeout 90s
journalctl -k -b --since "30 min ago" --no-pager -o short-iso |
    grep -i -E -B3 -A8 "out of memory|oom-kill"
```

| 22 November | Before (22:53) | Worst (22:53) | After (22:55) |
| --- | --- | --- | --- |
| Available | 1.2 GiB | 22 MiB | 1.2 GiB |
| Swap used | 0 B | 127 MiB | 36 KiB |

```text
22:53 lab01-akbayeva kernel: Out of memory: Killed process 864 (stress-ng-vm) total-vm:824592kB, anon-rss:683776kB, file-rss:644kB, shmem-rss:64kB, UID:0 pgtables:1548kB oom_score_adj:1000
```

At 22:53, the kernel recorded `global_oom` and killed worker 864. The memory sample at the OOM shows 22 MiB available and swap full. Worker 864 held about 668 MiB of anonymous resident memory and had an OOM score adjustment of 1000, which made it a preferred victim. Within the same minute, available RAM was back to 508 MiB. The collector kept running, although one sample interval stretched to about 1.07 seconds at 22:54. The parent `stress-ng` command finished at 22:55.

After the memory-pressure test, I restored the original Ubuntu guest’s RAM allocation to 4 GiB (4096 MiB). I shut down the copy after its final sample.

### Windows

```powershell
Get-WinEvent -FilterHashtable @{LogName='System'; ProviderName='Microsoft-Windows-Resource-Exhaustion-Detector'} -MaxEvents 5 |
    Format-List TimeCreated, Id, Message
```

PowerShell returned `NoMatchingEventsFound`.

```text
Get-WinEvent : No events were found that match the specified selection criteria.
```

Windows records event 2004 when it detects low virtual memory. The event names the largest consumers of committed memory. If the system reaches its commit limit, further allocations can fail.

I restored the Windows VM’s RAM allocation to 4 GB (4096 MB), as shown in VMware Workstation.

## What did not work the first time

On the first retry, the recorder and the memory workers ran in the same service. The service stopped when a worker was killed, before the after sample was saved. Running the workers in a separate control group kept the recorder alive for the complete retry.

## Evidence

- [task-manager.png](evidence/task-manager.png) — Windows Details tab with the Python process and its memory, handle and thread counts.
- [process-explorer.png](evidence/process-explorer.png) — the same Windows process in the tree, with its counters and open handles.
- [htop.png](evidence/htop.png) — the Ubuntu HTTP server command, its parent and its memory figures.
- [memory-pressure.txt](evidence/memory-pressure.txt) — time-stamped `free -h` output from before, during and after both recorded runs.
- [kernel-log-extract.txt](evidence/kernel-log-extract.txt) — the retry’s journal query, OOM decision and worker PID.
- [windows-event-log.png](evidence/windows-event-log.png) — the PowerShell query and its result.
- [windows-ram-restored.png](evidence/windows-ram-restored.png) — VMware Workstation showing the Windows VM’s 4 GB RAM allocation.
