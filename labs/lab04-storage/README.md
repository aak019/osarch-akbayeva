# Lab 4 — Partition, mirror, fail, recover

Pair: Adiyana · 15.11.2026 · Hypervisor: VMware Fusion (Ubuntu) / VMware Workstation (Windows)

## Windows

```text
>> Get-Disk | Select Number, FriendlyName, PartitionStyle, Size

Number         : 0
FriendlyName   : VMware Virtual NVMe Disk
PartitionStyle : GPT
Size           : 68719476736

Number         : 1
FriendlyName   : VMware Virtual NVMe Disk
PartitionStyle : GPT
Size           : 1073741824

>> Get-Volume | Where FileSystemLabel -eq 'LAB04'

DriveLetter FriendlyName FileSystemType DriveType HealthStatus OperationalStatus SizeRemaining Size
----------- ------------ -------------- --------- ------------ ----------------- ------------- ----
E           LAB04        NTFS           Fixed     Healthy      OK                993.35 MB     1007.93 MB
```

I chose GPT because it works with UEFI systems and keeps backup copy of the partition table with checksums. MBR has limitations, such as a 2 TiB disk size limit

## Linux — layout before and after
### Before

```text
$ lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
NAME         SIZE TYPE FSTYPE  MOUNTPOINTS
sr0          2.9G rom  iso9660
nvme0n2        1G disk
nvme0n3       25G disk
├─nvme0n3p1    1G part vfat    /boot/efi
└─nvme0n3p2 23.9G part ext4    /
nvme0n1        1G disk
nvme0n4        1G disk
```

### After

```text
$ lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
NAME         SIZE TYPE  FSTYPE            MOUNTPOINTS
sr0          2.9G rom   iso9660
nvme0n2        1G disk
└─nvme0n2p1 1022M part  linux_raid_member
nvme0n3       25G disk
├─nvme0n3p1    1G part  vfat              /boot/efi
└─nvme0n3p2 23.9G part  ext4              /
nvme0n1        1G disk
└─nvme0n1p1 1022M part  linux_raid_member
  └─md0     1021M raid1 ext4              /mnt/mirror
nvme0n4        1G disk
└─nvme0n4p1 1022M part  linux_raid_member
  └─md0     1021M raid1 ext4              /mnt/mirror
```

## What I did

**Commands used:**

Creating partitions:

```bash
for d in nvme0n1 nvme0n2 nvme0n4; do
  sudo parted -s /dev/$d mklabel gpt mkpart primary 1MiB 100% set 1 raid on
done
lsblk
```

Creating RAID 1:

```bash
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/nvme0n1p1 /dev/nvme0n2p1
cat /proc/mdstat
```

Saving the healthy state:

```bash
{ echo '$ cat /proc/mdstat'; cat /proc/mdstat; echo; echo '$ sudo mdadm --detail /dev/md0'; sudo mdadm --detail /dev/md0; } > ~/lab04/evidence/array-healthy.txt
```

Formatting and mounting:

```bash
sudo mkfs.ext4 -L mirror /dev/md0
sudo mkdir -p /mnt/mirror
sudo mount /dev/md0 /mnt/mirror
sudo chown $USER: /mnt/mirror
```

Creating test files and saving checksums:

```bash
echo "Lab 4 proof file — Adiyana — $(date)" > /mnt/mirror/proof.txt
head -c 20M /dev/urandom > /mnt/mirror/random.bin
cd /mnt/mirror && { echo '$ sha256sum proof.txt random.bin'; sha256sum proof.txt random.bin; } > ~/lab04/evidence/checksum-before.txt; cd ~/lab04
```

Failing and removing the second disk:

```bash
sudo mdadm /dev/md0 --fail /dev/nvme0n2p1
sudo mdadm /dev/md0 --remove /dev/nvme0n2p1
```

Saving degraded state:

```bash
{ echo '$ cat /proc/mdstat'; cat /proc/mdstat; echo; echo '$ sudo mdadm --detail /dev/md0'; sudo mdadm --detail /dev/md0; } > ~/lab04/evidence/array-degraded.txt
```

Checking files after the failure:

```bash
cat /mnt/mirror/proof.txt
cd /mnt/mirror && { echo '$ sha256sum proof.txt random.bin'; sha256sum proof.txt random.bin; } > ~/lab04/evidence/checksum-after-failure.txt; cd ~/lab04
diff <(tail -n +2 evidence/checksum-before.txt) <(tail -n +2 evidence/checksum-after-failure.txt) && echo "checksums match"
```

Rebuilding the RAID:

```bash
sudo mdadm /dev/md0 --add /dev/nvme0n4p1
cat /proc/mdstat
```

Saving the rebuilt state:

```bash
{ echo '$ cat /proc/mdstat'; cat /proc/mdstat; echo; echo '$ sudo mdadm --detail /dev/md0'; sudo mdadm --detail /dev/md0; } > ~/lab04/evidence/array-rebuilt.txt
```

Saving RAID configuration and checking the layout:

```bash
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
sudo update-initramfs -u
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
```

## Three states

| State | What /proc/mdstat showed | File |
| --- | --- | --- |
| Healthy | [2/2] [UU] | [array-healthy.txt](evidence/array-healthy.txt) |
| Degraded | [2/1] [U_] | [array-degraded.txt](evidence/array-degraded.txt) |
| Rebuilt | [2/2] [UU], nvme0n4p1 replaced nvme0n2p1 | [array-rebuilt.txt](evidence/array-rebuilt.txt) |

Checksums before and after the failure: match

## RAID is not a backup
**Your colleague deleted the wrong file from the mirror an hour ago. Does RAID 1 help? Why?**

No, RAID 1 does not help when someone deletes a file by mistake. The file is deleted from both disks because RAID 1 mirrors the changes. Separate backup is needed to recover the deleted file

## What did not work the first time
After creating the partitions nvme0n1p1 didn't appear in the lsblk output, while the other two partitions were visible. I checked the disk and corrected the missing partition before creating RAID 1
