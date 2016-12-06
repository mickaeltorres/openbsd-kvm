# openbsd-kvm
Quick and dirty port of the KVM API to OpenBSD, plus small changes to the qemu port to use it


## This code is completely un-finished and un-polished, use at your own risks


Has been tested on OpenBSD 6.0-stable on the following CPUs:
 - amd a8-7600
 - intel i7-4600u
 - intel i7-4770k

The following guests have been tested:
 - OpenBSD 6.0 amd64: installs / works
 - FreeBSD 11.0-RC3 amd64: installs / works
 - Debian 8.6.0 amd64: installs / works
 - TinyCore 7.6 i386 non-PAE: installs / works
 - Debian 8.6.0 i386 PAE: installs / works
 - Windows 10 (1607 x64): iso boots, install starts, then BSOD

Known caveats:
 - ~~only one vCPU supported~~ very crude smp without mutexes / expect a lot of race conditions when running -smp > 1
 - quick and dirty code :D
 - ~~the IO/MMIO instruction emulation is, at best, a hack~~ getting better
 - ~~sometimes to install debian on intel CPU, it is needed to kill a modprobe btrfs process~~ fixed by disabling AVX for guests until we support xsave/xrstor
 - ~~text and VGA console doesn't refresh well, the culprit would be, IO/MMIO emulation and/or the KVM_GET_DIRTY_LOG ioctl~~ fixed with dirty log patch
 - ~~FreeBSD 11 doesn't work with Q35 chipset~~ instruction enmulation fixes have resolved this
 - when -smp > 1, the vm may hang or crash in bios or in the guest, smp support is very preliminary
 - max number of vcpu is hardcoded at 16, and it is the max limit KVM_CAP_MAX_VCPUS is returning, KVM_CAP_NR_VCPUS returns the number of used cores on the host

Compile the kernel with the patch:

```
cd /usr/src/sys
patch < /path/to/kernel_patch
cd arch/amd64/conf 
config GENERIC.MP
cd ../compile/GENERIC.MP 
make -jX
make install
cp /usr/src/sys/arch/amd64/include/kvm.h /usr/include/machine/kvm.h
```

Reboot, and compile the qemu port with the patch:

```
cp /path/to/qemu_patch /usr/ports/emulators/qemu/patches/patch-kvm
cd /usr/ports/emulators/qemu
add --enable-kvm \ in CONFIGURE_ARGS= in Makefile
make -jX
make install
```

Create the device nodes:

```
cd /dev
mknod kvm c 97 0
mknod kvm_vm c 97 1
mknod kvm_vcpu c 97 2
```

You should be able to start, and install guests like this:

```
qemu-system-x86_64 -enable-kvm -cpu host -boot d -cdrom iso -m 512 -netdev user,id=vionet -device virtio-net,netdev=vionet -drive file=disk,if=virtio,format=raw -M q35
```

Adapt as you see fit
