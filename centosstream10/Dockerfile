# Copyright (c) 2026 Miget
# SPDX-License-Identifier: MIT
FROM quay.io/centos/centos:stream10@sha256:32916e884cc02e70fc57a4b7c8f6a5e0e4b168e7df49fd0a350e56867d88aa80

# No host keys baked, ever: every machine is a copy of this filesystem.
# Shorter list than AlmaLinux's -- this base already has curl (the full one),
# tar, gzip and vi.
RUN dnf -y upgrade \
	&& dnf -y install \
		procps-ng iproute iputils less nano \
		e2fsprogs \
		openssh-server \
	&& dnf clean all \
	&& rm -rf /var/cache/dnf \
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

# This base ships systemd, the console and kernel-filesystem masks and a
# multi-user default already, like almalinux:9 -- Rocky's EL10 base ships none
# of it. Only firstboot is left, and it prompts for a root password on a console
# a machine does not have.
RUN systemctl mask systemd-firstboot.service

CMD ["/usr/local/sbin/mashine-init"]
