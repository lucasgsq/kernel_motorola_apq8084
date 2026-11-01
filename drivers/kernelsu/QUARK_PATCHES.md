# Local Quark adaptations

Original hashes in QUARK_SOURCE.json refer to the pinned upstream sources.

- kernel_includes.h: remove the unused generated/compile.h include. Linux 3.10
  generates it through init/version.o rather than the generic prepare target;
  the driver does not use any symbol from it. Removing the include also makes
  a standalone driver build independent of init compilation order.
