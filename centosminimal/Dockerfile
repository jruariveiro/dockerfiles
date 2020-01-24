FROM centos:7 as builder

LABEL maintainer="Jorge Rua <jorge.ruariveiro@servicenow.com>"
LABEL description="CentOS7 Minimal Image"

ENV centos_root=/centos_image/rootfs
ENV BUSYBOX_VERSION 1.31.0
ENV BUSYBOX_URL https://busybox.net/downloads/busybox-${BUSYBOX_VERSION}.tar.bz2

RUN set -x \
    && yum -y update \
    && mkdir -p ${centos_root} \
    && rpm --root ${centos_root} --initdb \
    && yum reinstall --downloadonly --downloaddir . centos-release \
    && rpm --root ${centos_root} -ivh centos-release*.rpm \
    && rpm --root ${centos_root} --import ${centos_root}/etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7 \
    && yum install -y --installroot=${centos_root} \
       --setopt=tsflags='nodocs' \
       --setopt=override_install_langs=en_US.utf8 \
        glibc libtool-ltdl ca-certificates

# Prepare RPM build for Busybox
RUN set -x \
    && yum group install -y "Development Tools" \ 
    && yum install -y glibc-static wget \
    && mkdir -p /root/rpmbuild/{BUILD,BUILDROOT,RPMS,SOURCES,SPECS,SRPMS} \
    && wget -q -O /root/rpmbuild/SOURCES/busybox-${BUSYBOX_VERSION}.tar.bz2 ${BUSYBOX_URL}

# Deploy spec file and config
COPY files/busybox-config /root/rpmbuild/SOURCES/
COPY files/busybox.spec /root/rpmbuild/SPECS/

# Build busybox RPM
RUN set -x \
    && chown -R root: /root/rpmbuild \ 
    && cd /root/rpmbuild \ 
    && rpmbuild -v -bb --clean SPECS/busybox.spec

# Install busybox
RUN set -x \
    && rpm --root ${centos_root} -ivh /root/rpmbuild/RPMS/x86_64/busybox-static-${BUSYBOX_VERSION}-1.x86_64.rpm \ 
    && chroot ${centos_root} /usr/bin/busybox.static --install

# Reduce size of locale-archive
RUN set -x \
    && rm -rf ${centos_root}/var/cache/yum \
    && echo "localedef --list-archive | grep -v -i ^en | xargs localedef --delete-from-archive" > ${centos_root}/fixme.sh \
    && echo "mv /usr/lib/locale/locale-archive /usr/lib/locale/locale-archive.tmpl" >> ${centos_root}/fixme.sh \
    && echo "build-locale-archive" >> ${centos_root}/fixme.sh \
    && chmod 755 ${centos_root}/fixme.sh \ 
    && chroot ${centos_root} /fixme.sh \
    && rm ${centos_root}/fixme.sh

# Copy the resulting OS 
FROM scratch

COPY --from=builder /centos_image/rootfs /

CMD ["bash"] 
