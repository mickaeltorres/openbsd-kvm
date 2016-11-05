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
 - ...

Compile the kernel with the patch:

  cd /usr/src/sys
  patch < /path/to/kernel_patch
  cd arch/amd64/conf 
  config GENERIC.MP
  cd ../compile/GENERIC.MP 
  make -jX
  make install

Reboot, and compile the qemu port with the patch:

  cp /path/to/qemu_patch /usr/ports/emulators/qemu/patches/patch-kvm
  cd /usr/ports/emulators/qemu
add --enable-kvm \ in CONFIGURE_ARGS= in Makefile
  make -jX
  make install
