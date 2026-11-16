# Graywatch downloads

Graywatch tells you where a Linux node is losing time: the disks, a FUSE
storage client such as Quobyte, memory, the network or the CPU. Run it for a
minute while the problem is happening and read a plain-text report. Nothing to
install, no reboot, no server.

This repository only holds ready-to-use builds. The newest build is always
here:

```text
https://github.com/mysysadmin/graywatch-releases/releases/latest/download/graywatch-portable-linux-x86_64.tar.gz
```

Older builds are listed under [Releases](../../releases).

## Quick start

On the affected node, while the problem is happening:

```sh
curl -fLO https://github.com/mysysadmin/graywatch-releases/releases/latest/download/graywatch-portable-linux-x86_64.tar.gz
curl -fLO https://github.com/mysysadmin/graywatch-releases/releases/latest/download/graywatch-portable-linux-x86_64.tar.gz.sha256
sha256sum -c graywatch-portable-linux-x86_64.tar.gz.sha256
tar -xzf graywatch-portable-linux-x86_64.tar.gz
cd graywatch-portable-*/
sudo ./collect-graywatch-support.sh
```

After 60 seconds it prints two paths: the report, and an archive with the
evidence behind it. Read the report with `less`, or send the archive to
whoever helps you. The archive contains process names, mount points and
kernel log lines, so review it before sending it on.

## Run it as a service

A one-off collection only sees its own minute. Installed as a service,
Graywatch keeps watching, records what happened before anyone noticed, and
collects by itself while a problem lasts:

```sh
sudo ./install-graywatch.sh
```

- The daemon runs unprivileged and keeps the last 30 minutes of what it
  measures in `/var/log/graywatch`, plus a small log of problem periods: when
  a disk, mount or resource went bad and when it recovered.
- When a problem lasts a minute, a full collection runs as root, at most once
  an hour, into `/var/lib/graywatch/captures` (newest 10 kept).
- Every collection on that node then includes the recorded minutes before it,
  and its report says when the problems began.

Settings are in `/etc/default/graywatch`. `GRAYWATCH_KEEP=7d` records all the
time instead of the last 30 minutes. `--no-capture` installs without automatic
collections; `--uninstall` removes it and keeps the recordings.

## What it needs

- Linux on x86_64. Run it as root: the eBPF measurements need it, and
  without them the report loses stuck FUSE requests, kernel stacks of
  blocked threads, per-device queue and service times and which
  connections retransmit.
- For eBPF: glibc 2.35 or later, or RHEL 9 and its rebuilds, and the libelf
  library, which standard installs include. Tested on Rocky Linux 9 and
  Ubuntu 22.04 and 24.04, with kernels 5.14 to 7.0.
- PSI (pressure stall information) is optional. It is usually off on RHEL,
  Rocky and AlmaLinux; the report then simply says what is missing.

It stops by itself after the collection and leaves only the report and the
archive behind.

## What the eBPF part does

Graywatch only measures. It loads tracing programs that read timings and
counters, and none of the eBPF features that can change the kernel, other
processes or network traffic. Nothing stays loaded after the collection.
`EBPF.txt` in each bundle lists every program, what it reads, and how to
check it on your node with `bpftool`.

## License

Graywatch is MIT licensed. Each bundle contains `LICENSE` and
`THIRD_PARTY_NOTICES` for the libraries linked into it.
