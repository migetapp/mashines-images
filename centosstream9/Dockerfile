# Copyright (c) 2026 Miget
# SPDX-License-Identifier: MIT
FROM quay.io/centos/centos:stream9@sha256:0996d37c69b3a8c33042932d415ad7bf5a8122270264e835e5c0153615aef0e4

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
		cloud-init \
	&& dnf clean all \
	&& rm -rf /var/cache/dnf \
	&& rm -f /etc/ssh/ssh_host_* \
	&& systemctl enable sshd.service
COPY sshd-mashine.conf /etc/ssh/sshd_config.d/10-mashine.conf
RUN chmod 0644 /etc/ssh/sshd_config.d/10-mashine.conf

COPY mashine-volumes /usr/local/sbin/mashine-volumes
COPY mashine-volumes.service /usr/lib/systemd/system/mashine-volumes.service
COPY 90-mashine-hotplug.rules /usr/lib/udev/rules.d/90-mashine-hotplug.rules
RUN chmod 0755 /usr/local/sbin/mashine-volumes \
	&& systemctl enable mashine-volumes.service

COPY mashine-init /usr/local/sbin/mashine-init
RUN chmod 0755 /usr/local/sbin/mashine-init
COPY mashine-netconfig /usr/local/sbin/mashine-netconfig
COPY mashine-netconfig.service /usr/lib/systemd/system/mashine-netconfig.service
COPY mashine-netconfig.timer /usr/lib/systemd/system/mashine-netconfig.timer
RUN chmod 0755 /usr/local/sbin/mashine-netconfig \
	&& chmod 0644 /usr/lib/systemd/system/mashine-netconfig.service /usr/lib/systemd/system/mashine-netconfig.timer \
	&& systemctl enable mashine-netconfig.timer
COPY cloud-init-mashine.cfg /etc/cloud/cloud.cfg.d/90-mashine.cfg
RUN chmod 0644 /etc/cloud/cloud.cfg.d/90-mashine.cfg

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
