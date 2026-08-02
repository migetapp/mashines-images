# Copyright (c) 2026 Miget
#
# SPDX-License-Identifier: MIT
#
# AlmaLinux 9. Built from the repository root, which is the build context:
#
#     docker build -f almalinux9/Dockerfile -t mashine-almalinux:9 .
#
# Pinned by digest. `almalinux:9` names a different filesystem every few weeks,
# and a machine image is published under a version that never moves -- so a tag
# here would make that version a fiction. Bumping the digest is what a rebuild
# (a CVE, a new minor) looks like.
FROM almalinux:9@sha256:d2515c769e7b73f95c4fde38c0a505336ff38f14990c0b7253b77060a049a743

# A machine is a box someone logs into, and the base image is built for
# containers: no ps, no ip, no editor but vi, and none of mkfs.ext4, e2fsck or
# resize2fs that mashine-volumes needs. curl is already here as curl-minimal --
# installing curl over it is a conflict, not an upgrade.
#
# One transaction: dnf downloads repo metadata per invocation, and a clean in a
# later layer does not shrink an earlier one.
#
# Upgrading means the pinned digest no longer decides what ends up in the
# image: two builds weeks apart differ. Publish a rebuild as a new revision.
#
# NO HOST KEYS ARE BAKED HERE, and none ever may be. Every machine starts from
# a copy of this filesystem, so a key generated at build time would be the same
# key on all of them -- one machine's operator could impersonate every other --
# and in a published image it would be a key anyone can read.
#
# Nothing has to be added to get that, only nothing taken away: openssh-server
# ships no keys, and sshd-keygen@.service (ConditionFileNotEmpty=|!/etc/ssh/
# ssh_host_%i_key, pulled in by sshd-keygen.target, which sshd.service Wants and
# runs After) generates them on first start, onto the machine's own durable
# disk. The rm is a guard on that invariant rather than a fix: it makes a future
# package that DID bake a key fail loudly instead of shipping it everywhere.
RUN dnf -y upgrade \
	&& dnf -y install \
		procps-ng iproute iputils less tar gzip vim-minimal nano \
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

# systemd-firstboot's ConditionFirstBoot=yes holds on a freshly cloned rootfs --
# /etc/machine-id is empty -- and it then prompts for locale, timezone and a
# root password on a console a machine does not have. Masked rather than
# removed: the package still owns the file.
RUN systemctl mask systemd-firstboot.service

CMD ["/usr/local/sbin/mashine-init"]
