# tool-sun-path

A single-HTML-file calculator for the AF_UNIX `sun_path` budget.

[**Use it: truffle.ghostwright.dev/public/tools/sun-path/**](https://truffle.ghostwright.dev/public/tools/sun-path/)

## What it does

Type or paste a Unix domain socket path. The tool tells you how
many bytes it occupies, how many are left under the kernel limit,
and whether `bind(2)` will return `EINVAL`. Linux and Solaris
give you 108 bytes; macOS, FreeBSD, NetBSD, OpenBSD give you
104. Windows AF_UNIX matches Linux.

Per-segment breakdown shows the byte cost of each path component
and the running total, so when you compose a path from a fixed
prefix plus a per-pid socket name plus an IPC suffix, you can see
exactly which segment pushes you over.

## Why

The `sockaddr_un` struct in `<sys/un.h>` reserves a fixed-size
character array for the path. The kernel checks length at
`bind()` time and rejects anything longer with `EINVAL`. Code
that composes socket paths from a tempdir prefix plus a
tool-specific subdirectory plus a per-pid suffix can overflow
this limit before touching the filesystem.

I needed this calculator while reading [pnpm
#12222](https://github.com/pnpm/pnpm/issues/12222), where a
Docker-as-root install hits the limit because the pnpm-store
TMPDIR prefix plus `node_modules/.tmp/` plus the `tsx` IPC
socket suffix add up to ~75 bytes of fixed overhead before
counting `$HOME`. With a 32-byte store root, the budget is gone
before the per-package content hash even lands.

## Shape

- One HTML file. Inline CSS, inline JS, no build, no npm, no
  CDN. Save it; it keeps working offline.
- URL hash state: `?path=...&platform=...` round-trips, so a
  shared link reproduces the calculation.
- UTF-8 byte length, not character count. Multi-byte characters
  in paths cost more than their visual width.
- Light and dark theme via `prefers-color-scheme`.
- Mobile-tested at 375x667.

## Presets

Common Unix domain socket paths so you can see how much budget
real-world sockets use:

- X11 socket
- D-Bus session and system bus
- systemd notify
- Docker daemon
- ssh-agent
- pnpm-store + tsx (the worked example that motivated the tool)
- `$XDG_RUNTIME_DIR` app socket

## License

MIT.
