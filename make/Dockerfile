FROM gcc:5.4.0 AS build-env
ENV PATH $PATH:/usr/local/sh-tools/bin
ENV LD_LIBRARY_PATH /usr/local/gmp/lib:/usr/local/mpfr/lib:/usr/local/mpc/lib:$LD_LIBRARY_PATH
RUN apt-get update
RUN apt-get install -y texinfo
WORKDIR /build
RUN wget -nv https://ftp.gnu.org/gnu/binutils/binutils-2.30.tar.xz
RUN tar Jxf binutils-2.30.tar.xz
WORKDIR /build/binutils-2.30/build
RUN ../configure --target=sh-elf --prefix=/usr/local/sh-tools
RUN make -s -j8
RUN make -s install
WORKDIR /build
RUN wget -nv https://gmplib.org/download/gmp/gmp-6.1.2.tar.xz
RUN tar Jxf gmp-6.1.2.tar.xz
WORKDIR /build/gmp-6.1.2/build
RUN ../configure --prefix=/usr/local/gmp
RUN make -s -j8
RUN make install
WORKDIR /build
RUN wget -nv https://www.mpfr.org/mpfr-current/mpfr-4.0.2.tar.xz
RUN tar Jxf mpfr-4.0.2.tar.xz
WORKDIR /build/mpfr-4.0.2/build
RUN ../configure --with-gmp=/usr/local/gmp --prefix=/usr/local/mpfr
RUN make -s -j8
RUN make install
WORKDIR /build
RUN wget -nv https://ftp.gnu.org/gnu/mpc/mpc-1.1.0.tar.gz
RUN tar zxf mpc-1.1.0.tar.gz
WORKDIR /build/mpc-1.1.0/build
RUN ../configure --with-gmp=/usr/local/gmp --with-mpfr=/usr/local/mpfr --prefix=/usr/local/mpc
RUN make -s -j8
RUN make install
WORKDIR /build
RUN wget -nv http://ftp.tsukuba.wide.ad.jp/software/gcc/releases/gcc-7.3.0/gcc-7.3.0.tar.xz
RUN tar Jxf gcc-7.3.0.tar.xz
WORKDIR /build/gcc-7.3.0/build_step1
RUN ../configure --target=sh-elf --prefix=/usr/local/sh-tools --with-gmp=/usr/local/gmp --with-mpfr=/usr/local/mpfr --with-mpc=/usr/local/mpc --enable-languages=c --disable-libssp
RUN make -s -j8
RUN make -s install
WORKDIR /build
RUN wget -nv ftp://sourceware.org/pub/newlib/newlib-3.0.0.tar.gz
RUN tar zxf newlib-3.0.0.tar.gz
WORKDIR /build/newlib-3.0.0/build
RUN ../configure --target=sh-elf --prefix=/usr/local/sh-tools
RUN make -s -j8
RUN make install
WORKDIR /build/gcc-7.3.0/build_step2
RUN ../configure --target=sh-elf --prefix=/usr/local/sh-tools --with-gmp=/usr/local/gmp --with-mpfr=/usr/local/mpfr --with-mpc=/usr/local/mpc --enable-languages=c --disable-libssp -with-newlib -with-headers=/build/newlib-3.0.0/newlib/libc/include
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
COPY --from=build-env /usr/local/gmp /usr/local/gmp
COPY --from=build-env /usr/local/mpfr /usr/local/mpfr
COPY --from=build-env /usr/local/mpc /usr/local/mpc
ENV PATH $PATH:/usr/local/sh-tools/bin
ENV LD_LIBRARY_PATH /usr/local/gmp/lib:/usr/local/mpfr/lib:/usr/local/mpc/lib:$LD_LIBRARY_PATH
RUN apk add --no-cache make
WORKDIR /workspace
ENTRYPOINT ["make"]
