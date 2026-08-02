# Copyright (c) 2026 Miget
# SPDX-License-Identifier: MIT
#
# rockylinux/rockylinux, not library/rockylinux: the Docker Official image was
# deprecated and never got a 10.
FROM rockylinux/rockylinux:10@sha256:827d37bc128288ccf160ee318bb3cb92d591164cb217e92f8bc61e3982ae1834

# No host keys baked, ever: every machine is a copy of this filesystem.
# systemd because this base has no system manager for mashine-init to exec --
# CentOS Stream 10 is the same EL release and ships one.
RUN dnf -y upgrade \
	&& dnf -y install \
		systemd \
		procps-ng iproute nano \
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

# No console, no screen, no hardware of its own. Nothing was pre-masked because
# nothing installed systemd until we did, and a fresh systemd defaults to
# graphical.target. EL10 ships no systemd-remount-fs unit to mask.
RUN systemctl mask \
		systemd-firstboot.service \
		getty.target \
		console-getty.service \
		dev-hugepages.mount \
		sys-fs-fuse-connections.mount \
	&& systemctl set-default multi-user.target

CMD ["/usr/local/sbin/mashine-init"]
