# Copyright (c) 2026 Miget
# SPDX-License-Identifier: MIT
FROM debian:13@sha256:fac46bff2e02f51425b6e33b0e1169f55dfb053d83511ca28aa50c09fd5ed7a4

ENV DEBIAN_FRONTEND=noninteractive

# openssh-server's postinst runs ssh-keygen -A, so the rm must be in this layer:
# a published image carries every layer it was built from. e2fsprogs because
# trixie's base image dropped it; bookworm's still carries it. ssh.socket is
# disabled as a guard -- Debian does not enable it, Ubuntu does.
RUN apt-get update \
	&& apt-get -y upgrade \
	&& apt-get -y install --no-install-recommends \
		systemd \
		openssh-server \
		iproute2 iputils-ping less nano curl \
		e2fsprogs \
	&& rm -rf /var/lib/apt/lists/* \
	&& rm -f /etc/ssh/ssh_host_* \
	&& systemctl disable ssh.socket \
	&& systemctl enable ssh.service
COPY sshd-mashine.conf /etc/ssh/sshd_config.d/10-mashine.conf
# Trixie's own sshd-keygen.service is not a substitute: it is
# ConditionFirstBoot=yes, and installing systemd writes /etc/machine-id during
# the build, so it would never fire on a machine. The drop-in's empty
# ExecStartPre= reset is required because a drop-in appends.
COPY sshd-keygen.conf /usr/lib/systemd/system/ssh.service.d/10-mashine-keygen.conf
RUN chmod 0644 /etc/ssh/sshd_config.d/10-mashine.conf \
	/usr/lib/systemd/system/ssh.service.d/10-mashine-keygen.conf

COPY mashine-volumes /usr/local/sbin/mashine-volumes
COPY mashine-volumes.service /usr/lib/systemd/system/mashine-volumes.service
RUN chmod 0755 /usr/local/sbin/mashine-volumes \
	&& systemctl enable mashine-volumes.service

COPY mashine-init /usr/local/sbin/mashine-init
RUN chmod 0755 /usr/local/sbin/mashine-init

# No console, no screen, no hardware of its own -- and no kernel modules: the
# guest has no /lib/modules, so libkmod cannot even initialise and
# systemd-modules-load can only fail. It is masked here and not on the
# RHEL-family images because Debian ships a dangling
# /etc/modules-load.d/modules.conf, which is enough to satisfy the unit's
# ConditionDirectoryNotEmpty; on EL that directory is empty and the unit is
# skipped instead. Without this the machine boots degraded.
RUN systemctl mask \
		systemd-firstboot.service \
		getty.target \
		console-getty.service \
		systemd-remount-fs.service \
		systemd-modules-load.service \
		dev-hugepages.mount \
		sys-fs-fuse-connections.mount \
	&& systemctl set-default multi-user.target

CMD ["/usr/local/sbin/mashine-init"]
