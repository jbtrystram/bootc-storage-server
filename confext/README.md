# Confexts

The day-2 configuration of this machine is tracked through git using systemd-confexts.

## 01-storage

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

### btrfs subvolumes as mergerFS branches

Each disk is formatted with btrfs. Rather than passing the whole disk to mergerFS,
every disk gets a `tank` subvolume, mounted at `/mnt/disks/<disk>/tank`.
This subvolume is used as a mergerFS branch. For disks that store parity data we use a
`safe` subvolume, mounted at `/mnt/disks/<disk>/safe`.

This keeps the `safe` and `tank` pools from overlapping on disks that
participate in both. Each pool only ever sees its
own subvolume.

Subvolumes of the same disk have no space quota, so they compete for the same
free space. There is no reserved/isolated capacity for either pool.

So we set up two mergerFS pools:
- `/mnt/pools/safe` pooling only `/mnt/disks/8tb/safe`.
- `/mnt/pools/tank` pooling the `tank` subvolume of every disk.

### Snapraid

Snapraid is set to use `/mnt/disks/8tb/safe` as a data disk
and `/mnt/disks/16tb` as the parity store. This give us redundancy for the
`safe` subvolume.

Finally, we have a 2TB ssd mounted to `/mnt/flash` for transient and write intensive data:
databases, partial downloads and so on. This will preserve the lifespan of the hard drives.
It isn't part of any mergerFS pool or of the snapraid array.


This way we have up to 8TB of redunded space without reserving a whole disk for it until it grows,
allowing us to make use of most of our space :)


### Adding a new disk

1. Format the disk: `mkfs.btrfs -L <label> /dev/<device>`
2. Mount it temporarily and create the `tank` subvolume (and `safe` too, if this
   disk should also join the safe pool):
   ```
   mount /dev/<device> /mnt/tmp
   btrfs subvolume create /mnt/tmp/tank
   umount /mnt/tmp
   ```
3. Add a `mnt-disks-<disk>-tank.mount` unit under `01-storage/etc/systemd/system/`,
   using the disk's PARTUUID (`/dev/disk/by-partuuid/...`), mounted at
   `/mnt/disks/<disk>/tank` with `Options=subvol=tank`.
4. Add `/mnt/disks/<disk>/tank` to `mnt-tank.service`'s `RequiresMountsFor=` and
   to the `mergerfs` branch list in its `ExecStart=`.

## 02-shares

TODO
NFS and samba shares.

# Install

To install the confext, first clone the repo under a disk that is available early (i.e. reachable by `local-fs.target`):
```
git clone $this-repo /var/config
sudo mkdir /var/lib/confexts
sudo ln -s /var/config/confexts/01-storage /var/lib/confexts/:q
```
