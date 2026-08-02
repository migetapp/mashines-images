# Copyright (c) 2026 Miget
# SPDX-License-Identifier: MIT
FROM quay.io/centos/centos:stream9@sha256:be972f1929b214fd8d1fe6204678bb9d2d1f3ede997554fcf7e50eed3ff0d970

# No host keys baked, ever: every machine is a copy of this filesystem.
# The longest list here, and the same EL release as AlmaLinux 9 -- what a base
# carries is the vendor's decision, not the release's. No system manager at all,
# so systemd is what mashine-init execs, and it brings the util-linux that
# mashine-volumes' blkid and mountpoint come from.
RUN dnf -y upgrade \
	&& dnf -y install \
		systemd \
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

# No console, no screen, no hardware of its own. Nothing was pre-masked because
# nothing installed systemd until we did, and a fresh systemd defaults to
# graphical.target. EL9 ships no systemd-remount-fs unit to mask.
RUN systemctl mask \
		systemd-firstboot.service \
		getty.target \
		console-getty.service \
		dev-hugepages.mount \
		sys-fs-fuse-connections.mount \
	&& systemctl set-default multi-user.target

CMD ["/usr/local/sbin/mashine-init"]
