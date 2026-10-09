
|         What            |           Value             |                 Where I got it   
| :---------------------- | :-------------------------: | --------------------------------------------------: |
| CPU Model               | Apple M5                    | system_profiler SPHardwareDataType                  |
| CPU Core Count          | 10                          | system_profiler SPHardwareDataType                  |
| CPU Thread Count        | 10                          | sysctl hw.physicalcpu hw.logicalcpu kern.hv_support |
| Total Memory            | 16 GB                       | system_profiler SPMemoryDataType                    |
| Memory Type             | LPDDR5                      | system_profiler SPMemoryDataType                    |
| Memory Modules          | Not reported                | system_profiler SPMemoryDataType                    |
| Memory Speed            | Not reported                | system_profiler SPMemoryDataType                    |
| Disk Model              | APPLE SSD AP0512Z           | system_profiler SPNVMeDataType                      |
| Disk Type               | NVMe SSD                    | system_profiler SPNVMeDataType                      |
| Disk Capacity           | 500.28 GB                   | system_profiler SPNVMeDataType                      |
| Free Disk Space         | 351 GiB                     | df -h ~                                             |
| Firmware Type           | Apple Silicon boot firmware | system_profiler SPHardwareDataType                  |
| Firmware Version        | 20457.1.29                  | system_profiler SPHardwareDataType                  |
| Firmware Date           | Not reported                | system_profiler SPHardwareDataType                  |
| Hardware Virtualization | Supported (1)               | sysctl hw.physicalcpu hw.logicalcpu kern.hv_support |


## What did not work:

The memory command showed 16 GB of LPDDR5 memory but did not report
the number of individual memory modules or the memory speed.

The firmware command showed the firmware version but did not provide
its date.
