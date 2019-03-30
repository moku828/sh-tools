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

FROM ubuntu:16.04
RUN apt-get update \
 && apt-get install -y --no-install-recommends \
 wget \
 ca-certificates \
 make \
 perl \
 libc6-i386 \
 lib32stdc++6 \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*
WORKDIR /
RUN wget -nv https://www.toppers.jp/download.cgi/asp_apsh2a_gcc-20110809.tar.gz \
 && tar zxf asp_apsh2a_gcc-20110809.tar.gz \
 && rm asp_apsh2a_gcc-20110809.tar.gz
WORKDIR /asp/cfg/cfg
RUN wget -nv https://www.toppers.jp/download.cgi/cfg-linux-static-1_9_6.gz \
 && gzip -d cfg-linux-static-1_9_6.gz \
 && mv cfg-linux-static-1_9_6 cfg \
 && chmod 755 cfg
WORKDIR /asp
RUN sed -i "s/require \"getopt.pl\";/use Getopt::Std;/g" configure utils/makedep \
 && sed -i "s/Getopt(/getopt(/g" configure utils/makedep \
 && sed -i "s/do //g" configure utils/makedep \
 && sed -i "s/-m2a-nofpu//g" arch/sh12a_gcc/Makefile.sh2a
COPY --from=build-env /usr/local/sh-tools /usr/local/sh-tools
ENV PATH $PATH:/usr/local/sh-tools/bin
WORKDIR /asp/obj
RUN ../configure -T apsh2a_gcc \
 && make depend \
 && make \
 && make asp.bin
