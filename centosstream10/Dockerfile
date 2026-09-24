# Copyright (c) 2026 Miget
# SPDX-License-Identifier: MIT
FROM quay.io/centos/centos:stream10@sha256:fd6b1e14d330489aac4624aeb6269428163b6fd557cfb572fd1098c73109e84b

# No host keys baked, ever: every machine is a copy of this filesystem.
# Shorter list than AlmaLinux's -- this base already has curl (the full one),
# tar, gzip and vi.
RUN dnf -y upgrade \
	&& dnf -y install \
		procps-ng iproute iputils less nano \
		e2fsprogs \
		openssh-server \
		cloud-init \
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
COPY cloud-init-mashine.cfg /etc/cloud/cloud.cfg.d/90-mashine.cfg
RUN chmod 0644 /etc/cloud/cloud.cfg.d/90-mashine.cfg

# This base ships systemd, the console and kernel-filesystem masks and a
# multi-user default already, like almalinux:9 -- Rocky's EL10 base ships none
# of it. Only firstboot is left, and it prompts for a root password on a console
# a machine does not have.
RUN systemctl mask systemd-firstboot.service

CMD ["/usr/local/sbin/mashine-init"]
