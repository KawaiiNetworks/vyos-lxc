# VyOS-LXC
VyOS rootfs for LXC. Most functions are working properly.

## Usage

Download rootfs from https://github.com/KawaiiNetworks/vyos-lxc/releases , or build it.

PVE console mode: console, features: fuse mknod nesting

kernel module should be inserted in host kernel
for NAT: `modprobe nft_nat`

## Build

install squashfs-tools and distrobuilder

```bash
apt install -y squashfs-tools
go install -v -x github.com/lxc/distrobuilder/distrobuilder@latest
mkdir vyos-lxc & cd vyos-lxc
```

### Download VyOS ISO

``` bash
wget -O vyos.iso https://github.com/vyos/vyos-nightly-build/releases/download/1.5-rolling-202412160007/vyos-1.5-rolling-202412160007-generic-amd64.iso
```

### Extract filesystem.squashfs

```bash
mkdir rootfs
mount -o loop vyos.iso rootfs
unsquashfs -f -d source/ rootfs/live/filesystem.squashfs
```

### Modify Files

```bash
touch source/boot/grub/grub.cfg

vim source/etc/os-release # set ID=unmanaged

vim source/usr/lib/python3/dist-packages/vyos/ifconfig/ethernet.py
# surround above lines with try-except expression
# self._write_sysfs(f'/sys/class/net/{self.ifname}/queues/rx-{i}/rps_cpus', rps_cpus)
# self._write_sysfs(f'/sys/class/net/{self.ifname}/queues/rx-{i}/rps_flow_cnt', rfs_flow)

vim source/usr/lib/python3/dist-packages/vyos/system/image.py
# add func is_running_as_lxc
# def is_running_as_lxc() -> bool:
#     return True

vim source/usr/libexec/vyos/system/grub_update.py
#    if image.is_running_as_lxc():
#        exit(0)

vim source/usr/libexec/vyos/conf_mode/system_option.py
# add many try
# with open('/proc/sys/kernel/panic', 'w') as f:
#     f.write(timeout)
# cmd('loadkeys {keyboard_layout}'.format(**options))
# cmd('udevadm control --reload-rules')
#    for module in modules:
#        if module in modules_enabled:
#            check_kmod(module)
#            write_file(kernel_dynamic_debug, f'module {module} +p')
#        else:
#            write_file(kernel_dynamic_debug, f'module {module} -p')

vim source/usr/libexec/vyos/conf_mode/container.py
# --memory-swap {memory}m

vim source/usr/share/vyos/config.boot.default # delete system console

mv source/opt/vyatta/etc/config source/
cd source/opt/vyatta/etc
ln -s ../../../config config
```

### Build rootfs.tar.xz

```bash
distrobuilder pack-lxc vyos.yaml source
```


