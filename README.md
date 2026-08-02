# mashines-images

Root filesystems for [mashines.dev](https://mashines.dev) machines.

A machine is a VM guest running its own system manager, not a container, so
these images carry things a container image has no reason to: an init that keeps
the machine's log stream open, an sshd, and a way to format and mount the disks
attached to it.

Each directory is one distro. The files at the top are shared by all of them.

## Build

The build context is the repository root:

```
docker build -f almalinux9/Dockerfile -t mashine-almalinux:9 .
```

Nothing outside this repository and the distro's own package mirrors is needed.

The build upgrades the distro's packages, so what lands in the image depends on
when it was built and not only on the base digest the Dockerfile pins. A rebuild
is a new revision, never a republished one.

## Publish

`.github/workflows/build.yaml` is run by hand and pushes
`docker.io/miget/<image>:<version>-<revision>`. A revision is published once and
never moved, so a rebuild is a new revision rather than a replacement.

It needs two repository secrets: `DOCKERHUB_USERNAME`, and `DOCKERHUB_TOKEN`, an
access token that can push to the `miget` namespace.

## Guest contract

Two variables reach the guest. An image that reads them can be used as a machine
image; one that ignores them will boot, and will silently do nothing with the
machine's keys or disks.

**`MACHINE_SSH_KEYS`** — authorized public keys, **newline**-separated. Install
them as root's `authorized_keys` before sshd starts.

**`MACHINE_VOLUMES`** — attached disks, **comma**-separated `<device>:<mountpoint>`
pairs. The devices arrive raw and unformatted; the guest owns the filesystem as
well as the mount. mashines.dev still sends this one under its older name,
`MIGET_MACHINE_VOLUMES`, so `mashine-volumes` reads that too and prefers the
name above. An image that reads only `MACHINE_VOLUMES` today will boot and
mount nothing.

The separators differ on purpose: an SSH key's option list can contain commas.

`MACHINE_SSH_KEYS` can only be read by PID 1. A systemd service is started with
a clean environment rather than the one the guest was given, and the
`/proc/1/environ` workaround `mashine-volumes` uses cannot carry a value with
newlines in it — `tr '\0' '\n'` makes an embedded newline look exactly like the
separator between variables.

## Host keys

SSH host keys are generated on the machine's first boot and are never baked into
an image. Every machine starts from a copy of its image's filesystem, so a key
generated at build time would be the same key on every machine — and, in a
published image, one anyone can read.

An image that installs an SSH server must therefore ship no host keys and leave
generating them to first boot. `almalinux9/Dockerfile` does this by deleting any
key the package brought and letting `sshd-keygen@.service` make them.

## Licence

MIT.
