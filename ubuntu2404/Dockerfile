# Copyright (c) 2026 Miget
#
# SPDX-License-Identifier: MIT
#
# Ubuntu 24.04 LTS. Built from the repository root, which is the build context:
#
#     docker build -f ubuntu2404/Dockerfile -t mashine-ubuntu:24.04 .
#
# Pinned by digest, for the reason almalinux9 is: `ubuntu:24.04` names a
# different filesystem every few weeks, and a machine image is published under a
# version that never moves. Bumping the digest is what a rebuild looks like.
FROM ubuntu:24.04@sha256:4fbb8e6a8395de5a7550b33509421a2bafbc0aab6c06ba2cef9ebffbc7092d90

# apt asks questions on a terminal a build does not have, and a package that
# asks one waits until the build is killed.
ENV DEBIAN_FRONTEND=noninteractive

# systemd is the entry that is not a convenience: this base image has no system
# manager at all, so without it mashine-init has nothing to exec. almalinux:9
# ships one and needed nothing here.
#
# The rest is the box, and the list is shorter than AlmaLinux's because the two
# bases differ: ubuntu:24.04 already has ps, tar, gzip and all of mkfs.ext4,
# e2fsck and resize2fs, so no procps and no e2fsprogs. It has no ip, no ping, no
# pager, no editor and -- unlike almalinux:9, which brings curl-minimal -- no
# curl.
#
# One transaction, and the package lists deleted inside it: apt-get update
# writes ~40 MB under /var/lib/apt/lists, and a later layer that removes them
# does not shrink the earlier one that added them.
#
# The upgrade means the pinned digest no longer decides on its own what is in
# the image -- two builds weeks apart differ. Publish a rebuild as a new
# revision.
#
# NO HOST KEYS ARE BAKED HERE, and none ever may be. Every machine starts from a
# copy of this filesystem, so a key generated at build time would be the same
# key on all of them -- one machine's operator could impersonate every other --
# and in a published image it would be a key anyone can read.
#
# On AlmaLinux getting that takes nothing: openssh-server ships no keys. Ubuntu
# is the opposite and this is the difference to be careful about -- its
# openssh-server postinst runs `ssh-keygen -A` as it installs, so three key
# pairs exist the moment the package lands. The rm is load-bearing here, not the
# guard it is on AlmaLinux, and it has to run in THIS layer: a published image
# carries every layer it was built from, so an rm in a later one would delete
# the keys from the filesystem and leave them in the tarball.
#
# What puts them back on first boot is ubuntu2404/sshd-keygen.conf. Ubuntu has
# no sshd-keygen@.service, and its cloud images leave the job to cloud-init,
# which a machine does not run.
#
# sshd is a boot-time daemon here rather than the socket-activated one Ubuntu
# enables by default. Socket activation would accept the connection before the
# first key existed, and it hides a broken sshd until someone tries to log in --
# whereas a machine that failed to start one should say so in its own boot log.
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
COPY ubuntu2404/sshd-keygen.conf /usr/lib/systemd/system/ssh.service.d/10-mashine-keygen.conf
RUN chmod 0644 /etc/ssh/sshd_config.d/10-mashine.conf \
	/usr/lib/systemd/system/ssh.service.d/10-mashine-keygen.conf

COPY mashine-volumes /usr/local/sbin/mashine-volumes
COPY mashine-volumes.service /usr/lib/systemd/system/mashine-volumes.service
RUN chmod 0755 /usr/local/sbin/mashine-volumes \
	&& systemctl enable mashine-volumes.service

COPY mashine-init /usr/local/sbin/mashine-init
RUN chmod 0755 /usr/local/sbin/mashine-init

# What a machine has none of: a console, a screen, and hardware of its own.
#
# almalinux:9 arrives with this already done -- its own /etc/systemd/system
# carries /dev/null symlinks for the console and the kernel filesystems, and a
# default.target pointing at multi-user. ubuntu:24.04 has no systemd at all, so
# there is nothing pre-masked and every one of these is ours to decide.
#
# systemd-firstboot prompts for locale, timezone and a root password on a
# console, and its ConditionFirstBoot=yes holds on a freshly cloned rootfs,
# whose /etc/machine-id is empty. It is the one both distros need.
#
# getty.target is masked rather than getty@tty1.service disabled: the target is
# the thing that pulls a getty in, so masking it covers the tty1 the systemd
# package enables and anything else that later asks for one. console-getty
# likewise -- both would be condition-skipped for want of /dev/tty0 and
# /dev/console, but a machine's lack of a console is a fact about the product,
# not a lucky accident of the guest's /dev.
#
# systemd-remount-fs remounts / according to an /etc/fstab a machine does not
# have; its root is already writable when systemd starts. dev-hugepages and
# sys-fs-fuse-connections mount kernel filesystems the guest does not carry.
#
# Masked rather than removed, in every case: the packages still own the files.
RUN systemctl mask \
		systemd-firstboot.service \
		getty.target \
		console-getty.service \
		systemd-remount-fs.service \
		dev-hugepages.mount \
		sys-fs-fuse-connections.mount \
	&& systemctl set-default multi-user.target

CMD ["/usr/local/sbin/mashine-init"]
