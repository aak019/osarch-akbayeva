## **Task 1**

# 1. 
Yes, the laptop can run both guests at the same time

| Component | Memory |
|---|---|
| Windows 11 Home host (idle) | 4–6 GB |
| Ubuntu guest | 2 GB |
| Windows guest | 4 GB |
| Virtualization overhead (estimated) | ~0.5 GB |
| Total | 10.5–12.5 GB of 16 GB |

The laptop has about 3.5–5.5 GB left. The Ryzen 7 7735HS has 8 cores and 16 threads which is enough for two small VMs. The 512 GB NVMe SSD is also suitable if there is enough free space for both guests

# 2. 
RAM would be the first priority for running more VMs, but the 16 GB DDR5-4800 is soldered to the motherboard so it cannot be replaced or expanded
The next possible upgrade is the NVMe SSD if the laptop model allows replacement. 
A larger SSD would give more space for VM files, but it would not solve a RAM shortage. The processor is also not normally replaceable in this laptop

# 3. 
| Component	| Card B | My host |
|---|---|---|
| CPU: | Ryzen 7 7735HS, 8 cores, 16 threads | Apple M5, 10 cores, 10 logical CPUs |
| RAM: |	16 GB DDR5-4800	| 16 GB LPDDR5 unified memory |
| Disk: |	512 GB NVMe SSD	| 500.28 GB Apple NVMe SSD |
| Architecture: |	x86-64 | ARM64 |
| OS: |	Windows 11 Home |	macOS |

Card B is better for running standard x86-64 Windows and Ubuntu guests because its architecture matches those operating systems. It also has 16 CPU threads. My mac has 10 CPU cores and the same amount of RAM. Both machines have fast NVMe storage with similar capacity. The M5 uses ARM64, so my VMware fusion guests need ARM64 operating systems for native virtualization and that will be working without problem

Neither machine has a clear RAM capacity advantage. CPU performance can't be determined from the core and thread counts alone


## Task 2
**Alternative and the trade-off**

I choose GPT because it is a modern partition table supported by Ubuntu. MBR would also work for a 1 GB disk but GPT is more suitable for modern systems, also I choose the ext4 because it supports Linux permissions and journaling. FAT32 would be more compatible with windows but it does not provide the same Linux filesystem features. I used UUID instead of /dev/nvme0n2p1 in /etc/fstab because linux can change device names after a reboot

**Verification**
I rebooted Ubuntu:

```bash
$ sudo reboot
```

After reboot i checked whether the disk was still mounted:

```bash
$ findmnt /data
TARGET SOURCE         FSTYPE OPTIONS
/data  /dev/nvme0n4p1 ext4   rw,relatime
```

From lsblk -f, the data partition showed:

```text
NAME        FSTYPE FSVER UUID                                 MOUNTPOINTS
nvme0n4p1   ext4   1.0   52129d9c-51d5-4257-a08e-3c064df54409 /data
```

```bash
$ df -h /data
Filesystem      Size  Used Avail Use% Mounted on
/dev/nvme0n4p1  988M  276K  921M   1% /data
```

After reboot the device name changed from /dev/nvme0n2p1 to /dev/nvme0n4p1 but the filesystem UUID remained the same. The disk was still mounted at /data with ext4 and 921 MB of available space. This confirms that the permanent mount works after a reboot

## **Task 3**

**1. Symptom**
Ubuntu cannot ping the Windows VM at 10.0.2.4 (E1 shows 100% packet loss). However people are still working on windows so the machine is running. The problem is likely with incoming ping requests

**2. Diagnosis path**
First E1 shows that Ubuntu cannot reach Windows by ping but this does not prove windows is down. E4 shows that Windows can successfully ping Ubuntu at 10.0.2.15 so communication works in one direction
E2 shows that Windows has IP address 10.0.2.4/24 and E3 shows that Ubuntu has 10.0.2.15/24 with its interface UP. Both machines are on the same subnet
E5 confirms that both VMs use the same NAT Network, so their network settings are correct. E6 shows that all Windows Firewall profiles are enabled.
E7 shows that the rule "Allow ping in (ICMPv4)" is missing. This suggests that Windows Firewall is blocking incoming ping requests.
E8 shows that Windows was restored from the clean-install snapshot on November 13, 2026. This explains why the firewall rule configured in Lab 2 may have disappeared.

**3. Ignored evidence**
The default gateway in E2 is not important because both VMs are on the same subnet. The IPv6 address in E3 is also not relevant to this IPv4 ping problem.

**4. Missing evidence**
Need to run this command in windows powerShell
`Get-NetFirewallRule -DisplayName "*Echo Request*" | Select-Object DisplayName, Enabled, Direction, Action`

expect to find no enabled inbound allow rule for ICMPv4 Echo Requests. This would support the firewall diagnosis

**5. Root cause**
The most likely cause is that Windows Firewall blocks incoming ICMPv4 Echo Requests. Restoring the clean-install snapshot probably removed the firewall rule created in Lab 2. Windows itself is working, but Ubuntu monitoring incorrectly reports it as down.

**6. Fix**
open powerShell as administrator on windows and run:

`New-NetFirewallRule -DisplayName "Allow ping in (ICMPv4)" -Direction Inbound -Protocol ICMPv4 -IcmpType 8 -Action Allow`

check the rule:

`Get-NetFirewallRule -DisplayName "Allow ping in (ICMPv4)"`
Expected result: Enabled is True, Direction is Inbound, and Action is Allow.

**7. Verification**
from ubuntu run:

`ping -c 3 10.0.2.4`

Expected result: 3 replies and 0% packet loss

From Windows, run:
`ping -n 2 10.0.2.15`

Expected result: 2 replies and 0% packet loss

After rebooting windows repeat the ping from Ubuntu. It should still show 0% packet loss because the firewall rule is saved. To prevent this problem create new snapshot after configuring firewall or document the rule so it can be restored after using an older snapshot
