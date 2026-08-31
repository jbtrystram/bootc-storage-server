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

## Install

### bootc install
Boot the fedora CoreOS live iso then install with [bootc install](https://docs.fedoraproject.org/en-US/bootc/bare-metal/#_using_bootc_install).
E.g : 
```
podman run --rm --privileged --pid=host -v /dev:/dev -v /var/lib/containers:/var/lib/containers \
   -v ./:/mnt/home --security-opt label=type:unconfined_t quay.io/jbtrystram/storageserver \
     bootc install to-disk  \
     --root-ssh-authorized-keys /mnt/home/key \
     --karg ip=192.168.250.22::192.168.250.1:255.255.255.0:storage-server::none \
     --karg console=ttyS0 --karg console=tty0 --filesystem=xfs \
     /dev/sda
```

See https://docs.fedoraproject.org/en-US/bootc/bare-metal/#_using_bootc_install
Note that this does not work as there is not enough free RAM to pull the image on 4GB I guess ?

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
podman run --rm --privileged \
  --network=none \
  -v /var/lib/containers/storage:/var/lib/containers/storage \
  -v ./output:/output \
  -v .:/srv \
  ghcr.io/osbuild/image-builder-cli:latest \
    build qcow2 \
    --bootc-ref quay.io/jbtrystram/storageserver \
    --output-dir storage --blueprint /srv/blueprint.toml
```

# TODO

use partitions UUIDs for mounts
Write a SOP for volume and disk management operations.
Write NFS config confext.
Setup automated rebuilds of the image for updates
