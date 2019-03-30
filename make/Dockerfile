FROM gcc:7.4.0 AS build-env
WORKDIR /build
RUN wget -nv http://ftp.jaist.ac.jp/pub/GNU/binutils/binutils-2.16.1.tar.bz2
RUN tar jxf binutils-2.16.1.tar.bz2
RUN rm -rf binutils-2.16.1/gprof/*.m
WORKDIR /build/binutils-2.16.1/build
ENV PATH $PATH:/usr/local/sh-tools/bin
ENV CFLAGS "-fno-stack-protector -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=0"
RUN ../configure --target=sh-elf --prefix=/usr/local/sh-tools
RUN make -s -j8
RUN make -s install
WORKDIR /build
RUN wget -nv http://ftp.tsukuba.wide.ad.jp/software/gcc/releases/gcc-3.4.5/gcc-3.4.5.tar.bz2
RUN tar jxf gcc-3.4.5.tar.bz2
RUN sed -i -e "s/O_CREAT/O_CREAT, 0755/g" gcc-3.4.5/gcc/collect2.c
WORKDIR /build
RUN wget -nv ftp://sourceware.org/pub/newlib/newlib-1.14.0.tar.gz
RUN tar zxf newlib-1.14.0.tar.gz
WORKDIR /build/gcc-3.4.5
RUN ln -s ../newlib-1.14.0/newlib ../newlib-1.14.0/libgloss ../newlib-1.14.0/COPYING.NEWLIB .
WORKDIR /build/gcc-3.4.5/build
ENV CFLAGS ""
RUN ../configure --target=sh-elf --enable-languages=c --prefix=/usr/local/sh-tools -with-newlib -with-headers=/build/newlib-1.14.0/newlib/libc/include
RUN make -s -j8
RUN make -s install

FROM alpine:3.9

ENV LANG=C.UTF-8

# Here we install GNU libc (aka glibc) and set C.UTF-8 locale as default.

RUN ALPINE_GLIBC_BASE_URL="https://github.com/sgerrand/alpine-pkg-glibc/releases/download" && \
    ALPINE_GLIBC_PACKAGE_VERSION="2.29-r0" && \
    ALPINE_GLIBC_BASE_PACKAGE_FILENAME="glibc-$ALPINE_GLIBC_PACKAGE_VERSION.apk" && \
    ALPINE_GLIBC_BIN_PACKAGE_FILENAME="glibc-bin-$ALPINE_GLIBC_PACKAGE_VERSION.apk" && \
    ALPINE_GLIBC_I18N_PACKAGE_FILENAME="glibc-i18n-$ALPINE_GLIBC_PACKAGE_VERSION.apk" && \
    apk add --no-cache --virtual=.build-dependencies wget ca-certificates && \
    echo \
        "-----BEGIN PUBLIC KEY-----\
        MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEApZ2u1KJKUu/fW4A25y9m\
        y70AGEa/J3Wi5ibNVGNn1gT1r0VfgeWd0pUybS4UmcHdiNzxJPgoWQhV2SSW1JYu\
        tOqKZF5QSN6X937PTUpNBjUvLtTQ1ve1fp39uf/lEXPpFpOPL88LKnDBgbh7wkCp\
        m2KzLVGChf83MS0ShL6G9EQIAUxLm99VpgRjwqTQ/KfzGtpke1wqws4au0Ab4qPY\
        KXvMLSPLUp7cfulWvhmZSegr5AdhNw5KNizPqCJT8ZrGvgHypXyiFvvAH5YRtSsc\
        Zvo9GI2e2MaZyo9/lvb+LbLEJZKEQckqRj4P26gmASrZEPStwc+yqy1ShHLA0j6m\
        1QIDAQAB\
        -----END PUBLIC KEY-----" | sed 's/   */\n/g' > "/etc/apk/keys/sgerrand.rsa.pub" && \
    wget \
        "$ALPINE_GLIBC_BASE_URL/$ALPINE_GLIBC_PACKAGE_VERSION/$ALPINE_GLIBC_BASE_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_BASE_URL/$ALPINE_GLIBC_PACKAGE_VERSION/$ALPINE_GLIBC_BIN_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_BASE_URL/$ALPINE_GLIBC_PACKAGE_VERSION/$ALPINE_GLIBC_I18N_PACKAGE_FILENAME" && \
    apk add --no-cache \
        "$ALPINE_GLIBC_BASE_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_BIN_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_I18N_PACKAGE_FILENAME" && \
    \
    rm "/etc/apk/keys/sgerrand.rsa.pub" && \
    /usr/glibc-compat/bin/localedef --force --inputfile POSIX --charmap UTF-8 "$LANG" || true && \
    echo "export LANG=$LANG" > /etc/profile.d/locale.sh && \
    \
    apk del glibc-i18n && \
    \
    rm "/root/.wget-hsts" && \
    apk del .build-dependencies && \
    rm \
        "$ALPINE_GLIBC_BASE_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_BIN_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_I18N_PACKAGE_FILENAME"
COPY --from=build-env /usr/local/sh-tools /usr/local/sh-tools
ENV PATH $PATH:/usr/local/sh-tools/bin
RUN apk add --no-cache make
WORKDIR /workspace
ENTRYPOINT ["make"]
