| What | Value | Where I got it |
|---|---|---|
| CPU Model | Apple M5 | system_profiler SPHardwareDataType \| grep -E 'Chip:\|Total Number of Cores:\|System Firmware Version:' |
| CPU Core Count | 10 | sysctl hw.physicalcpu hw.logicalcpu kern.hv_support |
| CPU Thread Count | 10 | sysctl hw.physicalcpu hw.logicalcpu kern.hv_support |
| Total Memory | 16 GB | system_profiler SPMemoryDataType |
| Memory Type | LPDDR5 | system_profiler SPMemoryDataType |
| Memory Modules | Unified, on-package memory; no replaceable RAM modules; physical package count not reported | system_profiler SPMemoryDataType |
| Memory Speed | Not reported by command | system_profiler SPMemoryDataType |
| Disk Model | APPLE SSD AP0512Z | system_profiler SPNVMeDataType \| grep -E 'NVMExpress:\|Model:\|Capacity:' |
| Disk Type | NVMe SSD (Apple Fabric) | system_profiler SPNVMeDataType \| grep -E 'NVMExpress:\|Model:\|Capacity:' |
| Disk Capacity | 500.28 GB | system_profiler SPNVMeDataType \| grep -E 'NVMExpress:\|Model:\|Capacity:' |
| Free Disk Space | 344 GiB | df -h ~ |
| Firmware Type | Apple Silicon boot firmware | system_profiler SPHardwareDataType \| grep -E 'Chip:\|Total Number of Cores:\|System Firmware Version:' |
| Firmware Version | 20457.1.29 | system_profiler SPHardwareDataType \| grep -i firmware |
| Firmware Date | Not reported by command | system_profiler SPHardwareDataType \| grep -i firmware |
| Hardware Virtualization | Supported (1) | sysctl hw.physicalcpu hw.logicalcpu kern.hv_support |

## What Did Not Work

The memory command showed 16 GB of LPDDR5 memory but did not report the number of physical memory packages or the memory speed. My MacBook uses unified, on-package memory instead of conventional replaceable RAM modules.

The firmware command showed version 20457.1.29 but did not provide its date.
