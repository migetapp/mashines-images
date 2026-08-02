# Copyright (c) 2026 Miget
# SPDX-License-Identifier: MIT
FROM ubuntu:24.04@sha256:4fbb8e6a8395de5a7550b33509421a2bafbc0aab6c06ba2cef9ebffbc7092d90

ENV DEBIAN_FRONTEND=noninteractive

# openssh-server's postinst runs ssh-keygen -A, so the rm must be in this layer:
# a published image carries every layer it was built from.
RUN apt-get update \
	&& apt-get -y upgrade \
	&& apt-get -y install --no-install-recommends \
		systemd \
		openssh-server \
		iproute2 iputils-ping less nano curl \
	&& rm -rf /var/lib/apt/lists/* \
	&& rm -f /etc/ssh/ssh_host_* \
	&& systemctl disable ssh.socket \
	&& systemctl enable ssh.service
COPY sshd-mashine.conf /etc/ssh/sshd_config.d/10-mashine.conf
# Ubuntu has no sshd-keygen@.service; cloud-init does it there, and a machine
# does not run cloud-init. The drop-in's empty ExecStartPre= reset is required
# because a drop-in appends.
COPY sshd-keygen.conf /usr/lib/systemd/system/ssh.service.d/10-mashine-keygen.conf
RUN chmod 0644 /etc/ssh/sshd_config.d/10-mashine.conf \
	/usr/lib/systemd/system/ssh.service.d/10-mashine-keygen.conf

COPY mashine-volumes /usr/local/sbin/mashine-volumes
COPY mashine-volumes.service /usr/lib/systemd/system/mashine-volumes.service
RUN chmod 0755 /usr/local/sbin/mashine-volumes \
	&& systemctl enable mashine-volumes.service

COPY mashine-init /usr/local/sbin/mashine-init
RUN chmod 0755 /usr/local/sbin/mashine-init

# No console, no screen, no hardware of its own.
RUN systemctl mask \
		systemd-firstboot.service \
		getty.target \
		console-getty.service \
		systemd-remount-fs.service \
		dev-hugepages.mount \
		sys-fs-fuse-connections.mount \
	&& systemctl set-default multi-user.target

CMD ["/usr/local/sbin/mashine-init"]
