# BOOTC Storage server

This is a custom bootc image to run the storage server on my homelab.

## Goals

Manage a multi-drives storage through the following tools:
 - Snapraid
 - MergerFS
 - Cockpit

The storage is then exported to other services through NFS shares.

I am not using FCOS for this as I need kernel-level tools to be installed (NFS)
and it make little sense to run mergerfs and snapraid in containers (though definitely doable).

Also, I'd like to use 45drives cockpit plugin to manage NFS shares so let's layer everything on top
of bootc.

## Build

Build the container !


## Deploy

### Image builder

Inject the ssh-key and default config through a blueprint and generate a qcow image :
```
[customizations.kernel]
append = "console=tty0 console=ttyS0,115200n8"

[[customizations.user]]
name = "root"
key = "ssh-ed25519 AAAA.... user@host"
```
Then run as root :
```
sudo podman run --rm --privileged \
  --network=none \
  -v /var/lib/containers/storage:/var/lib/containers/storage \
  -v ./output:/output \
  -v .:/srv \
  ghcr.io/osbuild/image-builder:latest \
    build qcow2 \
    --bootc-ref quay.io/jbtrystram/storageserver \
    --output-dir storage --blueprint /srv/blueprint.toml
```

### Proxmox

Attach the disk to the VM with : `qm disk import <VM_ID> /path/to/qcow local-lvm --format qcow2`
See https://vormox.com/blog/how-to-import-a-qcow2-or-vmdk-disk-image-into-a-vm-in-proxmox-ve-qm-importdisk-explained

# TODO

use partitions UUIDs for mounts
Write a SOP for volume and disk management operations.
Write NFS config confext.
Setup automated rebuilds of the image for updates
