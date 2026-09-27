# Copyright (c) 2026 Miget
# SPDX-License-Identifier: MIT
FROM almalinux:9@sha256:d2515c769e7b73f95c4fde38c0a505336ff38f14990c0b7253b77060a049a743

# No host keys baked, ever: every machine is a copy of this filesystem.
# The upgrade means the pinned digest alone no longer decides the contents --
# publish a rebuild as a new revision.
RUN dnf -y upgrade \
	&& dnf -y install \
		procps-ng iproute iputils less tar gzip vim-minimal nano \
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

# Prompts for locale and a root password on a console a machine does not have.
RUN systemctl mask systemd-firstboot.service

CMD ["/usr/local/sbin/mashine-init"]
