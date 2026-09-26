# proot

User-space implementation of `chroot`, `mount --bind`, and
`binfmt_misc`: run programs with an arbitrary directory as their root
file system, relocate files, and execute binaries built for another CPU
architecture through QEMU user-mode — all without privileges or setup.
Technically, PRoot traces its child processes with `ptrace` (plus
`seccomp` filtering where available) and rewrites their system calls.

This is the [Termux](https://termux.dev) fork of
[proot-me/PRoot](https://github.com/proot-me/PRoot), maintained for
running Linux distributions on Android.  Termux users normally consume
it through [proot-distro](https://github.com/termux/proot-distro).

## Differences from upstream

* `-l`/`--link2symlink`: emulate hard links with symlinks when SELinux
  forbids `link(2)`
* `-0`/`-i`: extended fake-root support that persists permissions in
  `.proot-meta-file.*` meta files
* `-H`: hide `.proot*` helper files from directory listings
* `-p`: remap privileged (<1024) localhost ports to unprivileged ones
* `-L`: report correct sizes/inodes for emulated links
* `--sysvipc`: emulate System V IPC (shmget/semget/msgget/...) per
  instance, since Android lacks it
* `--ashmem-memfd` (Android only): emulate `memfd_create` through ashmem
* `--kill-on-exit`: kill all traced processes when the command exits
* Ongoing syscall coverage and seccomp/ptrace correctness fixes for
  Android and modern kernels (see `doc/proot/changelog.txt`)

## Building

Requirements: a C compiler, GNU make, and `libtalloc` development files
(e.g. `apt install libtalloc-dev`; Termux provides it automatically).

    make -C src        # produces src/proot
    make -C src install PREFIX=/usr/local   # optional

## Testing

    make -C tests check

## Usage

    proot -S ~/rootfs            # shell into a guest rootfs as fake root
    proot -R ~/rootfs cmd args   # run a command with host config bound
    proot --help                 # full option list

## Documentation

* `doc/MANUAL.md` — full manual (generated, checked in)
* `doc/proot.1` — man page (generated, checked in)
* `doc/proot.md` — prose source; option text comes from
  `src/cli/proot.h`, the same table that produces `proot --help`
* Regenerate with `make -C doc`; verify freshness with
  `make -C doc check`
* `doc/articles/` — historical upstream articles (circa 2012, kept for
  reference)
* `doc/proot/changelog.txt` — upstream history plus fork summary
* `doc/proot/roadmap.txt` — upstream roadmap, historical

## License

GPL v2 or later — see `COPYING`.  Upstream PRoot was written by Cédric
Vincent at STMicroelectronics; this fork is maintained by the Termux
contributors.

## Security

PRoot is **not** a security boundary: guests act with your full host
privileges.  To report vulnerabilities in this fork, see `SECURITY.md`.
