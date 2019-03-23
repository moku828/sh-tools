FROM gcc:7.4.0 AS build-env
WORKDIR /build
RUN wget -nv https://ftp.gnu.org/gnu/gdb/gdb-8.1.tar.xz
RUN tar Jxf gdb-8.1.tar.xz
WORKDIR /build/gdb-8.1/build
RUN ../configure --target=sh-elf --prefix=/usr/local/sh-tools
RUN make -s -j8
RUN apt-get update
RUN apt-get install -y texinfo
RUN make -s install

FROM ubuntu:18.04
RUN apt-get update \
 && apt-get -y install --no-install-recommends \
      libexpat1 \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*
COPY --from=build-env /usr/local/sh-tools/bin/sh-elf-gdb /usr/local/sh-tools/bin/
ENV PATH $PATH:/usr/local/sh-tools/bin
WORKDIR /workspace
ENTRYPOINT ["/usr/local/sh-tools/bin/sh-elf-gdb"]
CMD ["--help"]
