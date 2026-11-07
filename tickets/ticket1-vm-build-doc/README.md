# Ticket 1. “New developer starts Monday, needs a test machine”

## 1. What was asked

```text
From: Aigerim T. (Office Manager)
Subject: test machine for new dev

Hi, we have someone starting on Monday and he needs a test computer
set up. Nothing special, just so he can try things without breaking
anything. Can you write down how to do it so I can do the next one
myself? Thanks
```

## 2. What was actually needed

The request describes one test computer but the assignment actually requires two virtual machines: one Linux guest and one Windows guest

**The original request did not specify:**
which operating systems and hypervisor should be used, how much RAM, CPU and disk space should be allocated, whether the guests need internet access, whether the Linux and Windows guests need to communicate with each other, which network mode should be used, if recovery snapshots are required, which installation images and architectures should be used.

| What I would ask | Assumption or decision |
|---|---|
| which OS versions and applications are required? | use the available Ubuntu server 26.04.1 LTS ARM64 and Windows 11 enterprise evaluation x64 for small development tests |
| Which physical hosts and resources are available? | Use VMware fusion on the Mac and Workstation pro on the PC. calculate each guest's allocation from its own host |
| Is internet and access between the guests required? | Both are required. Bridge the two guests to the same local router because they run on separate physical computers |
| Is a Windows licence or authorised image available? | Reuse the existing enterprise evaluation VM. evaluation is time limited and is not a permanent licence |
| How should failed experiments be recovered? | Keep a clean-install snapshot for each guest. |


## 3. What I did

1. Inspected the existing **lab01-ubuntu** VM in Fusion and **lab02-windows** VM in Workstation, and recorded both hosts resources.
2. Confirmed Ubuntu's **Ubuntu 64-bit Arm** profile, **4 GiB RAM**, **2 vCPUs**, **25 GiB growable split NVMe disk**, and **ubuntu-26.04.1-live-server-arm64.iso** installation media. Windows retains **4 GiB RAM**, **2 vCPUs**, its **64 GiB growable single-file disk**
3. Used bridged adapters. I used bridged connection instead of NAT because it can bring together both of VMs that located on a different devices. Fusion uses **Bridged Networking - Autodetect** with its network adapter connected. Both guests use gateway **192.168.31.*** 
4. Checked guest identities and integration tools, verified DNS and HTTPS from each guest, and sent 4 ping requests in each direction. Both directions returned four replies with 0 packet loss
5. Exported the original txt results and assembled png for each of stages in BUILD.md
6. Recorded **clean-install** snapshots for both guests

### Commands used

#### Ubuntu - VMware tools

```bash
sudo apt update
sudo apt install -y open-vm-tools
systemctl is-active open-vm-tools
```
#### Windows - guest information
```bash
hostname
Get-CimInstance Win32_OperatingSystem | Select-Object Caption,Version,BuildNumber
Get-CimInstance Win32_ComputerSystem | Select-Object TotalPhysicalMemory,NumberOfLogicalProcessors
Get-Volume -DriveLetter C | Select-Object DriveLetter,Size,SizeRemaining
Get-Service VMTools
```
#### Windows - Ubuntu network test
```bash
$linuxIP = Read-Host "Enter the Ubuntu IPv4 address"
ping -n 4 $linuxIP
```
#### Ubuntu - Windows network test
```bash
read -r -p 'Enter the Windows IPv4 address: ' WINDOWS_IP
ping -c 4 "$WINDOWS_IP"
```
#### Ubuntu internet test
```bash
ping -c 4 ubuntu.com
```
#### Windows internet test
```bash
ping ubuntu.com
```

### Decisions and reasons

| Decision | Reason |
|---|---|
| ubuntu: **4 GiB RAM, 2 vCPUs, 25 GiB disk** | enough for a small Ubuntu Server test VM while leaving resources available for macOS. |
| windows: **4 GiB RAM, 2 vCPUs, 64 GiB disk** | Enough for Windows testing without using too much of the PC's available resources. |
| ubuntu **ARM64** image | The ubuntu VM runs on an apple silicon Mac, so ARM64 matches the host architecture |
| **bridged networking** | The two VMs run on different physical computers so bridged mode places both guests on the same local network and allows them to communicate |
| **open-vm-tools** | Provides VMware guest integration through Ubuntu's package manager |
| **clean-install snapshots** | Provide a recovery point after both VMs are installed |

## 4. Evidence


| PNG file | what it proves |
|---|---|
|[01-new-vm-wizard.png](evidence/01-new-vm-wizard.png) | Workstation typical wizard used as the Windows rebuild reference. |
| [02-disk-settings.png](evidence/02-disk-settings.png) | Fusion 25 GB labelled disk, 4096 MB RAM and two processor cores |
| [03-network-adapter.png](evidence/03-network-adapter.png) | Connected Fusion adapter using bridged networking and autodetect |
| [08-linux-installation-media.png](evidence/08-linux-installation-media.png) | Connected CD/DVD with ubuntu-26.04.1-live-server-arm64.iso selected. |
| [09-linux-installation.png](evidence/09-linux-installation.png) | Installed Ubuntu console  |
| [10-linux-guest-tools.png](evidence/10-linux-guest-tools.png) | architecture check and active tools|
| [11-windows-definition.png](evidence/11-windows-definition.png) | Windows 11 x64 profile |
| [12-windows-hardware.png](evidence/12-windows-hardware.png) | Existing Windows memory and processor configuration. |
| [13-windows-installation.png](evidence/13-windows-installation.png) | Installed Windows VM |
| [14-windows-guest-tools.png](evidence/14-windows-guest-tools.png) | Windows build running VM tools |
| [15-guest-network.png](evidence/15-guest-network.png) | Windows bridged adapter selection |
| [16-internet-tests.png](evidence/16-internet-tests.png) | DNS and successful HTTPS results from both guests |
| [17-ping-between-guests.png](evidence/17-ping-between-guests.png) | Four replies in each direction with zero loss at the recorded test addresses |
| [06-snapshot-list.png](evidence/06-snapshot-list.png) | Both guests clean-install snapshot lists |

| TXT file | what it proves |
|---|---|
| [00-host-resources.txt](evidence/00-host-resources.txt) | PC host measurements used for the Windows budget |
| [00-mac-host-resources.txt](evidence/00-mac-host-resources.txt) | Mac version, RAM, logical CPU count and data volume capacity/availability |
| [04-guest-uname.txt](evidence/04-guest-uname.txt) | Linux identity, kernel, memory, filesystem, network and tools output |
| [05-ping-between-guests.txt](evidence/05-ping-between-guests.txt) | Combined original results for both successful ping directions |
| [25-linux-internet.txt](evidence/25-linux-internet.txt) | Ubuntu DNS and HTTPS status 200 with exit 0 |
| [26-windows-internet.txt](evidence/26-windows-internet.txt) | Windows DNS and HTTP/1.1 200 OK with exit 0 |

## 5. What I would do differently

I would record both host systems and the guest architecture before choosing the installation media and network settings. This would help avoid planning the wrong Ubuntu architecture for the available host

I would also check all captured commands before saving the evidence and record the test date, time, and current DHCP addresses together. This would make later IP address or system changes easier to define from the original test results