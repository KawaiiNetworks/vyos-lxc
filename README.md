# VyOS-LXC
VyOS rootfs for LXC. Most functions are working properly.

## Build

```bash
apt install -y squashfs-tools
go install -v -x github.com/lxc/distrobuilder/distrobuilder@latest
mkdir vyos-lxc & cd vyos-lxc
```

### Download VyOS ISO

``` bash
wget https://github.com/vyos/vyos-nightly-build/releases/download/1.5-rolling-202412160007/vyos-1.5-rolling-202412160007-generic-amd64.iso -O vyos.iso
```

### Extract filesystem.squashfs

```bash
mkdir rootfs
mount -o loop vyos.iso rootfs
unsquashfs -f -d source/ rootfs/live/filesystem.squashfs
```

### Modify Files

```bash
mkdir source/config
touch source/boot/grub/grub.cfg
vim source/etc/os-release # set ID=unmanaged
vim source/usr/lib/python3/dist-packages/vyos/ifconfig/ethernet.py # try _write_sysfs
vim source/usr/lib/python3/dist-packages/vyos/system/image.py # add func is_running_as_lxc
vim source/usr/libexec/vyos/system/grub_update.py # if is_running_as_lxc then exit(0)
vim source/usr/libexec/vyos/conf_mode/system_option.py # add many try
vim source/usr/libexec/vyos/conf_mode/container.py # --memory-swap {memory}m
vim source/source/usr/share/vyos/config.boot.default # delete system console
```

### Build rootfs.tar.xz

```bash
distrobuilder pack-lxc vyos.yaml source
```

## Usage

PVE mode console, func: fuse mknod nesting

kernel module should be inserted in host kernel
for NAT: `modprobe nft_nat`
