# How to Build?

The GNU C Library cannot be compiled in the source directory.  You must build it in a separate build directory.  For example, if you have unpacked the GNU C Library sources in `/src/gnu/glibc-VERSION`, create a directory `/src/gnu/glibc-build` to put the object files in.

## Steps
The steps to build Glibc for Occlum are as follows:

1. Install the required modules
```
sudo apt-get update
sudo apt-get install gawk bison -y
```

2. Configure, build and install Glibc
```
cd <glibc-build-folder>
unset LD_LIBRARY_PATH
CFLAGS="-O2 -g" <glibc-VERSION>/configure \
--prefix=<glibc-install-folder> --with-tls --without-selinux --enable-stack-protector=strong --disable-nscd
make
make install
```
When completed, the resulting Glibc can be found in `<glibc-install-folder>` directory.

# System calls and the vector registers

A system call through the entry of Occlum (`OCCLUM_SYSCALL`, see
`sysdeps/unix/sysv/linux/x86_64/occlum_syscall.h`) preserves the general
purpose registers and the flags, but not the vector registers (xmm, ymm, zmm,
and the mask registers k): the LibOS uses them and saves only the general
purpose registers of the user. Linux preserves them across a system call. So the
asm statements of the system calls that the compiler sees (`internal_syscall0`
to `internal_syscall6` in `sysdeps/unix/sysv/linux/x86_64/sysdep.h`, and
`TLS_INIT_TP`) declare the vector registers as clobbered with
`REGISTERS_CLOBBERED_BY_SYSCALL`, and the compiler never keeps a value in one of
them across a system call. The assembly functions (`syscall`, `clone`, `vfork`,
...) are called like functions, which do not preserve the vector registers
anyway. The instruction `syscall` itself (emulated by the LibOS), exceptions and
signals do preserve all of the user's state.
