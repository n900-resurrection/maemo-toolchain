# maemo-toolchain

Compiler wrappers for building programs for **Maemo 5 (Nokia N900: ARMv7 armel, glibc 2.5, kernel 2.6.28)** with the cross compiler of a modern
Ubuntu, without the old Maratona/Scratchbox SDK. They are used by [nokia-n900-apps](https://github.com/n900-resurrection/nokia-n900-apps) and run
**on the computer** (in its Ubuntu Workshop container), not on the phone; the programs they make run on the phone and under `qemu-user`.

| File | What |
|---|---|
| `maemo-gcc` | C: `arm-linux-gnueabi-gcc` with the Fremantle sysroot (`$MAEMO_SYSROOT`, default `/project/sysroot`), gold linker, no host glibc |
| `maemo-g++` | C++ (C++17, `<filesystem>`, threads, regex...) with the **static** `libstdc++` of the Ubuntu compiler; the system one is 4.2.1 |
| `maemo-compat.h` | `-include` shims for the glibc 2.5 headers (`O_CLOEXEC`...) |
| `maemo-endian.h` | `-include` for code that wants `<endian.h>` functions |
| `maemo-cxx-compat.{c,h}` | the symbols the Ubuntu `libstdc++.a` (built for glibc 2.39 and a 64-bit `time_t`) asks for and glibc 2.5 lacks: `secure_getenv`, `getentropy`, `aligned_alloc`, `pthread_cond_clockwait`, the `*l` math functions, and the translation of `__clock_gettime64`, `__nanosleep64`, `__fstat64_time64`... (a 112-byte `struct stat`) |
| `cxx-selftest.sh`, `tests/cxx/` | builds and runs a C++ test (threads, exceptions, `<filesystem>`, `<regex>`, time) under qemu-user, and on a phone with `DEVICE=n900` |

The Fremantle sysroot itself is made by `scripts/fetch_sysroot.py` of nokia-n900-apps (it is not part of this repository). Binaries need at most
`GLIBC_2.4` (checked in the self-test).

## Use

```bash
export MAEMO_SYSROOT=/path/to/sysroot            # default /project/sysroot
maemo-gcc -Os hello.c -o hello
maemo-g++ -std=gnu++17 -O2 prog.cpp -o prog      # libstdc++ is linked statically
MAEMO_DEPS=/dir1:/dir2 maemo-g++ ...             # prefixes whose include/ and lib/ are searched before the sysroot
./cxx-selftest.sh                                # inside the Workshop: runs tests/cxx under qemu-user
```

Not covered: `std::locale` other than "C"; `*64` functions that no program needed yet (a new one shows as `undefined reference` at link time: add it
to `maemo-cxx-compat.c`). Tested only on the N900 (kernel 2.6.28) and in qemu-user.

MIT licence.
