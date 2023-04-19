# Setting up ZFS on Proxmox

ALl forms of ZFS can be setup directly on proxmox. more options can be configured via CLI

[Proxmox KB for ZFS](https://pve.proxmox.com/wiki/ZFS_on_Linux)

## ZFS Administration
---
This section gives you some usage examples for common tasks. ZFS itself is really powerful and provides many options. The main commands to manage ZFS are zfs and zpool. Both commands come with great manual pages, which can be read with:

```
# man zpool
# man zfs
```
### Create a new zpool
To create a new pool, at least one disk is needed. The ashift should have the same sector-size (2 power of ashift) or larger as the underlying disk.
```
# zpool create -f -o ashift=12 <pool> <device>
```
To activate compression (see section [Compression in ZFS](https://pve.proxmox.com/wiki/ZFS_on_Linux#zfs_compression)):

```
# zfs set compression=lz4 <pool>
```
### Create a new pool with RAID-0
Minimum 1 disk
```
# zpool create -f -o ashift=12 <pool> <device1> <device2>
```

### Create a new pool with RAID-1
Minimum 2 disks
```
# zpool create -f -o ashift=12 <pool> mirror <device1> <device2>
```



### Create a new pool with RAID-10
Minimum 4 disks
```
# zpool create -f -o ashift=12 <pool> mirror <device1> <device2> mirror <device3> <device4>
```
### Create a new pool with RAIDZ-1
Minimum 3 disks

```
# zpool create -f -o ashift=12 <pool> raidz1 <device1> <device2> <device3>
```
### Create a new pool with RAIDZ-2
Minimum 4 disks
```
# zpool create -f -o ashift=12 <pool> raidz2 <device1> <device2> <device3> <device4>
```

### Create a new pool with cache (L2ARC)
It is possible to use a dedicated cache drive partition to increase the performance (use SSD).

As <device> it is possible to use more devices, like it’s shown in "Create a new pool with RAID*".
```
# zpool create -f -o ashift=12 <pool> <device> cache <cache_device>
```

### Create a new pool with log (ZIL)
It is possible to use a dedicated cache drive partition to increase the performance(SSD).

As <device> it is possible to use more devices, like it’s shown in "Create a new pool with RAID*".
```
# zpool create -f -o ashift=12 <pool> <device> log <log_device>
```

### Add cache and log to an existing pool
If you have a pool without cache and log. First partition the SSD in 2 partition with parted or gdisk

Important	Always use GPT partition tables.
The maximum size of a log device should be about half the size of physical memory, so this is usually quite small. The rest of the SSD can be used as cache.
```
# zpool add -f <pool> log <device-part1> cache <device-part2>
```
### Changing a failed device
```
# zpool replace -f <pool> <old device> <new device>
```
### Changing a failed bootable device
Depending on how Proxmox VE was installed it is either using systemd-boot or grub through proxmox-boot-tool [1] or plain grub as bootloader (see Host Bootloader). You can check by running:
```
# proxmox-boot-tool status
```

