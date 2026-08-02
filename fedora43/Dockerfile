# Copyright (c) 2026 Miget
# SPDX-License-Identifier: MIT
FROM fedora:43@sha256:762d73ba1c455232b0272c5d445a34f36c4b9f421cbc05ce8102552325b6a222

# No host keys baked, ever: every machine is a copy of this filesystem.
# `dnf` here is dnf5: same subcommands and flags, but the cache is under
# /var/cache/libdnf5. systemd because this base has no system manager for
# mashine-init to exec.
RUN dnf -y upgrade \
	&& dnf -y install \
		systemd \
		procps-ng iproute iputils less nano \
		e2fsprogs \
		openssh-server \
	&& dnf clean all \
	&& rm -rf /var/cache/libdnf5 \
	&& rm -f /etc/ssh/ssh_host_* \
	&& systemctl enable sshd.service
COPY sshd-mashine.conf /etc/ssh/sshd_config.d/10-mashine.conf
RUN chmod 0644 /etc/ssh/sshd_config.d/10-mashine.conf

COPY mashine-volumes /usr/local/sbin/mashine-volumes
COPY mashine-volumes.service /usr/lib/systemd/system/mashine-volumes.service
RUN chmod 0755 /usr/local/sbin/mashine-volumes \
	&& systemctl enable mashine-volumes.service

COPY mashine-init /usr/local/sbin/mashine-init
RUN chmod 0755 /usr/local/sbin/mashine-init

# No console, no screen, no hardware of its own. Nothing was pre-masked because
# nothing installed systemd until we did, and a fresh systemd defaults to
# graphical.target. Fedora is the only one in this family that still ships
# systemd-remount-fs.
RUN systemctl mask \
		systemd-firstboot.service \
		getty.target \
		console-getty.service \
		systemd-remount-fs.service \
		dev-hugepages.mount \
		sys-fs-fuse-connections.mount \
	&& systemctl set-default multi-user.target

CMD ["/usr/local/sbin/mashine-init"]
