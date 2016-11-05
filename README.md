# openbsd-kvm
Quick and dirty port of the KVM API to OpenBSD, plus small changes to the qemu port to use it

Has been tested on OpenBSD 6.0-stable on the following CPUs:
 - amd a8-7600
 - intel i7-4600u
 - intel i7-4770k

The following guests have been tested:
 - OpenBSD 6.0 amd64
 - FreeBSD 11.0-RC3 amd64
 - Debian 8.6.0 amd64

Known caveats:
 - only one vCPU supported
 - quick and dirty code :D
 - the IO/MMIO instruction emulation is, at best, a hack
 - sometimes to install debian on intel CPU, it is needed to kill a process
 - text and VGA console doesn't refresh well, the culprit would be, IO/MMIO emulation and/or the KVM_GET_DIRTY_LOG ioctl
 - ...

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

Adapt as you see fit, FreeBSD is known to panic with -M q35, so remove this for that type of guests
