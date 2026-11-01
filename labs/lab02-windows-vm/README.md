# Lab 2 — A Windows guest, and a network between them

Pair: No partner  
Driver first half: Akbayeva Adiyana  
Machine: Windows PC — Windows 11, 32 GB RAM, Intel 270k plus; Ubuntu Server VM on MacBook + Windows 11 Enterprise Evaluation VM on Windows PC 
Date: 01.11.2026

## What I did

1. Downloaded Windows 11 Enterprise Evaluation. Licence: https://www.microsoft.com/en-us/evalcenter/download-windows-11-enterprise

2. Created lab02-windows in VMware Workstation with:
   - 2 vCPU
   - 4 GB RAM
   - 64 GB virtual disk
   - UEFI + Secure Boot
   - vTPM

3. Network mode: Bridged on both virtual machines.
I used Bridged networking because my Mac is on Apple Silicon and I had my stationary PC for Windows VM. But I kept Ubuntu VM on my Mac. using NAT would place them on separate private networks on each host. Bridged networking puts both guests on the same local network, allowing them to communicate with each other and access the internet. So I used Bridged networking in order to make them see each other

4. The first ping from Ubuntu to Windows timed out because Windows Defender Firewall blocked incoming ICMP echo requests.
I fixed it by opening PowerShell as Administrator and creating this firewall rule:
New-NetFirewallRule -DisplayName "Allow ping in (ICMPv4)" -Protocol ICMPv4 -IcmpType 8 -Direction Inbound -Action Allow
I did not disable Windows Defender Firewall

5. For the memory overcommitment experiment, I used the Windows VM on my Windows PC. The host has 32 GB of physical RAM. I increased the Windows VM memory to 24 GB while the host itself was also using memory.
This created strong memory pressure and caused committed memory to exceed the amount of physical RAM available on the host. After the experiment, I shut down the VM and returned its memory to 4 GB.

## Network diagram

                         Internet
                            |
                      Wi-Fi Router
                            |
                     Bridged Network
                      /           \
                     /             \
            MacBook host       Windows PC host
                 |                  |
           VMware Fusion     VMware Workstation
                 |                  |
          Bridged Adapter     Bridged Adapter
                 |                  |
         lab01-ubuntu        lab02-windows
         192.168.31.75       192.168.31.148
                 \                  /
                  \________________/
                    Ping both ways


## Result

| | Before | After |
|---|---|---|
| Host RAM in use | 10.7 GB | 24.5 GB |
| Available | 21.0 GB | 7.2 GB |
| Committed | 16.5 / 33.7 GB | 45.5 / 46.0 GB |
| What I noticed | — | Memory usage increased significantly. Available RAM decreased and committed memory became very close to its limit. When I tried using my PC it was lagging on opening games |

## What did not work the first time

At first, Ubuntu could not ping the Windows VM and showed 100% packet loss. The cause was windows defender firewall blocking incoming ICMP echo requests.
I first tried to create the firewall rule in a normal PowerShell window, but windows returned Access is denied. I reopened PowerShell as Administrator and created the ICMPv4 inbound firewall rule.
After that, Ubuntu successfully pinged the Windows VM with 0% packet loss. Windows could also ping Ubuntu successfully, and both virtual machines could reach ubuntu.com

## Evidence

- [evidence/ipconfig-windows.txt](evidence/ipconfig-windows.txt) — Windows VM address
- [evidence/ip-addr-linux.txt](evidence/ip-addr-linux.txt) — Ubuntu VM address
- [evidence/ping-both-ways.txt](evidence/ping-both-ways.txt) — ping in both directions and internet connectivity
- [evidence/host-memory-before.png](evidence/host-memory-before.png) — Windows host memory before the experiment
- [evidence/host-memory-after.png](evidence/host-memory-after.png) — Windows host memory during the memory overcommitment experiment
