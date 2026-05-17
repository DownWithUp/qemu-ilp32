# QEMU Changes

QEMU was cloned under:

```text
qemu/
```

The custom linux-user emulator is:

```text
qemu/build/qemu-aarch64_ilp32
```

### New Target

Added:

```text
qemu/configs/targets/aarch64_ilp32-linux-user.mak
```

This defines a linux-user target based on AArch64 but built with `TARGET_ABI32`,
so QEMU has a separate executable for ILP32 user binaries:

```text
qemu-aarch64_ilp32
```

### ELF Loader Support

Modified:

```text
qemu/linux-user/aarch64/target_elf.h
```

Key behavior:

- reports `ELFCLASS32` when `TARGET_ABI32` is enabled;
- keeps the machine type as AArch64;
- disables the AArch64 VDSO path for ABI32 for now.

### Address Space and `mmap`

Modified:

```text
qemu/linux-user/aarch64/target_mman.h
qemu/target/arm/cpu-param.h
```

Key behavior:

- uses a 32-bit virtual address shape for the AArch64 ILP32 target;
- keeps mappings in a range suitable for ILP32 user pointers.

### Linux-User Syscall ABI Fixes

Modified:

```text
qemu/linux-user/qemu.h
qemu/linux-user/syscall.c
qemu/linux-user/syscall_defs.h
```

Key behavior:

- avoids treating AArch64 ILP32 as legacy ARM OABI;
- avoids old ARM-only UID/stat assumptions for this target;
- adds an AArch64 ILP32 `target_stat` layout path;

### Usage
* Apply the patch with `git -C ~/fresh_qemu/qemu am qemu-ilp32/qemu-aarch64-ilp32.patch`
* Build with: `./configure --target-list=aarch64_ilp32-linux-user --disable-docs` then `ninja -C build qemu-aarch64_ilp32`
* Run the output binary at: `qemu/build/qemu-aarch64_ilp32 `