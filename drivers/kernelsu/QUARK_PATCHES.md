# Local Quark adaptations

Original hashes in QUARK_SOURCE.json refer to the pinned upstream sources.

- kernel_includes.h and ksu.c: remove generated/compile.h and replace its
  UTS_MACHINE logging field with init_utsname()->machine. Linux 3.10 generates
  the header through init/version.o rather than the generic prepare target.
  Reading the initialized UTS identity removes the init build-order dependency
  without changing root behavior or the architecture shown in the log.
