# Copyright (c) 2026 Miget
# SPDX-License-Identifier: MIT
#
# rockylinux/rockylinux, not library/rockylinux: the Docker Official image was
# deprecated.
FROM rockylinux/rockylinux:8@sha256:e8a49c5403b687db05d4d67333fa45808fbe74f36e683cec7abb1f7d0f2338c6

# No host keys baked, ever: every machine is a copy of this filesystem.
# Short list -- this base already has systemd, journalctl, curl, less, ping, tar,
# gzip and vi.
RUN dnf -y upgrade \
	&& dnf -y install \
		procps-ng iproute nano \
		e2fsprogs \
		openssh-server \
	&& dnf clean all \
	&& rm -rf /var/cache/dnf \
	&& rm -f /etc/ssh/ssh_host_* \
	&& systemctl enable sshd.service
COPY sshd-mashine.conf /etc/ssh/sshd_config.d/10-mashine.conf
# EL8's sshd_config has no Include line and no sshd_config.d, so the drop-in
# would be read by nothing -- while that file ships an uncommented
# `PermitRootLogin yes`. Include itself works: Red Hat backported it into
# openssh 8.0p1. Line 1 because sshd takes the first value it obtains. Check
# with `sshd -T`.
RUN chmod 0644 /etc/ssh/sshd_config.d/10-mashine.conf \
	&& sed -i '1i Include /etc/ssh/sshd_config.d/*.conf' /etc/ssh/sshd_config

COPY mashine-volumes /usr/local/sbin/mashine-volumes
COPY mashine-volumes.service /usr/lib/systemd/system/mashine-volumes.service
RUN chmod 0755 /usr/local/sbin/mashine-volumes \
	&& systemctl enable mashine-volumes.service

COPY mashine-init /usr/local/sbin/mashine-init
RUN chmod 0755 /usr/local/sbin/mashine-init

# This base ships systemd, the console and kernel-filesystem masks and a
# multi-user default already, like almalinux:9. Only firstboot is left, and it
# prompts for a root password on a console a machine does not have.
RUN systemctl mask systemd-firstboot.service

CMD ["/usr/local/sbin/mashine-init"]
