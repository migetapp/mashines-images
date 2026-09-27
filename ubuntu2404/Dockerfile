# Copyright (c) 2026 Miget
# SPDX-License-Identifier: MIT
FROM ubuntu:24.04@sha256:4fbb8e6a8395de5a7550b33509421a2bafbc0aab6c06ba2cef9ebffbc7092d90

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
	&& apt-get -y upgrade \
	&& apt-get -y install --no-install-recommends \
		systemd \
		openssh-server \
		cloud-init lsb-release \
		iproute2 iputils-ping less nano curl \
	&& rm -rf /var/lib/apt/lists/* \
	&& rm -f /etc/ssh/ssh_host_* \
	&& systemctl disable ssh.socket \
	&& systemctl enable ssh.service

COPY sshd-mashine.conf /etc/ssh/sshd_config.d/10-mashine.conf
COPY sshd-keygen.conf /usr/lib/systemd/system/ssh.service.d/10-mashine-keygen.conf
RUN chmod 0644 /etc/ssh/sshd_config.d/10-mashine.conf \
	/usr/lib/systemd/system/ssh.service.d/10-mashine-keygen.conf

COPY mashine-volumes /usr/local/sbin/mashine-volumes
COPY mashine-volumes.service /usr/lib/systemd/system/mashine-volumes.service
COPY 90-mashine-hotplug.rules /usr/lib/udev/rules.d/90-mashine-hotplug.rules
RUN chmod 0755 /usr/local/sbin/mashine-volumes \
	&& systemctl enable mashine-volumes.service

COPY fstrim-timer-mashine.conf /usr/lib/systemd/system/fstrim.timer.d/10-mashine.conf
COPY fstrim-mashine.conf /usr/lib/systemd/system/fstrim.service.d/10-mashine.conf
RUN chmod 0644 /usr/lib/systemd/system/fstrim.timer.d/10-mashine.conf \
	/usr/lib/systemd/system/fstrim.service.d/10-mashine.conf \
	&& systemctl enable fstrim.timer

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
