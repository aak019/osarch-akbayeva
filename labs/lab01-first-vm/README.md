# Lab 1 — Your first virtual machine

Pair: Adiyana / Driver: Adiyana
Machine: MacBook Air and macOS, guest Ubuntu Server 26.04.1 LTS
Date: 25.10.2026

## What I did
1. Installed VMware Fusion 26H1u1. License: https://knowledge.broadcom.com/external/article/368667/download-and-license-vmware-desktop-hyper.html
2. Created VM lab01-ubuntu: 2 vCPU, 4 GB RAM, 25 GB dynamic disk, NAT.
3. Installed Ubuntu Server, took snapshot clean-install.
4. Broke it: sudo sed -i 's/dhcp4: true/dhcp4: false/' /etc/netplan/*.yaml
sudo netplan apply
5. Rolled back to clean-install: ip -br a ping -c 3 ubuntu.com

## Resource justification
I assigned 2 CPU cores out of 10 and 4 GB of RAM out of 16 GB to leave enough resources for macOS. I also chose a 25 GB dynamically allocated disk to save storage space and NAT networking to give Ubuntu internet access through my Mac.

## Result
The VM boots; the break-and-rollback cycle is shown in the evidence below.

## What did not work the first time
At first, the VM could not boot from the Ubuntu ISO and showed No Media. The CD/DVD drive was not reading the ISO correctly. I added the CD/DVD drive again, connected the Ubuntu Server ARM64 ISO, and then the installer started correctly.

## Evidence
- [evidence/vm-settings.png](evidence/vm-settings.png) — CPU, memory and disk as configured
- [evidence/guest-release.txt](evidence/guest-release.txt) — Ubuntu Server is running in the guest
- [evidence/broken.png](evidence/broken.png) — the guest in its broken state
- [evidence/snapshot-list.png](evidence/snapshot-list.png) — the snapshot, after the rollback