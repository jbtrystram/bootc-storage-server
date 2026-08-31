# Confexts

The day-2 configuration of this machine is tracked through git using systemd-confexts.

## 01-base

The base config: users, hostname, IP addresses, SSH config.

## 02-storage

Everything related to disks : mountpoints, mount units, mergerFS and snapraid configuration.
This is where we leverage mergerfs + snapraid to do the magic.
See https://easyhtpc.com/media-servers/mergerfs-snapraid-guide/

The main design choice here is :
I want to be able to pool my disks, and get some parity, but not on all data
(linux ISOs don't need to be backed up).

Let's say we have 4 disks :
- 4TB mounted to `/mnt/disks/4tb-1` 
- 4TB mounted to `/mnt/disks/4tb-2` 
- 8TB mounted to `/mnt/disks/8tb` 
- 16TB mounted to `/mnt/disks/16tb` 

So we set up two mergerFS pools:
`/mnt/safe` that merges `/mnt/disks/4tb-1` and `/mnt/disks/4tb-2`.
`mnt/tank` that merges all disks.

Snapraid is set to use `/mnt/disks/4tb-1` and `/mnt/disks/4tb-2` as data disks
and `/mnt/disks/16-tb` as the parity store.
Paths that need to be saved for parity are added manually with [include](https://www.snapraid.it/manual#sec7-8) rules.
This way we have up to 8TB of redunded space without reserving a whole disk to it until it grows.

This allows us to make use of most of our space :)

## 03-shares

TODO
NFS and samba shares.
