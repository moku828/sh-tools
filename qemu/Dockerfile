FROM gcc:7.4.0 AS build-env
WORKDIR /build
RUN git clone https://github.com/moku828/qemu
WORKDIR /build/qemu
RUN git pull && git checkout 7cd4cec9dd73eaefc502dac6f4dabdbd1a452baa
WORKDIR /build/qemu/build
RUN ../configure --disable-bsd-user --disable-guest-agent --disable-strip --disable-werror --disable-gcrypt --disable-debug-info --disable-debug-tcg --disable-docs --disable-tcg-interpreter --enable-attr --disable-brlapi --disable-linux-aio --disable-bzip2 --disable-bluez --disable-cap-ng --disable-curl --disable-fdt --disable-glusterfs --disable-gnutls --disable-nettle --disable-gtk --disable-rdma --disable-libiscsi --disable-vnc-jpeg --disable-kvm --disable-lzo --disable-curses --disable-libnfs --disable-numa --disable-opengl --disable-vnc-png --disable-rbd --disable-vnc-sasl --disable-sdl --disable-seccomp --disable-smartcard --disable-snappy --disable-spice --disable-libssh2 --disable-libusb --disable-usb-redir --disable-vde --disable-vhost-net --disable-virglrenderer --disable-virtfs --enable-vnc --disable-vte --disable-xen --disable-xen-pci-passthrough --disable-xfsctl --disable-linux-user --enable-system --disable-blobs --disable-tools --target-list=sh4eb-softmmu --static --disable-pie --prefix=/usr/local/sh-tools
RUN make -j8
RUN make -s install

FROM ubuntu:18.04
COPY --from=build-env /usr/local/sh-tools /usr/local/sh-tools
ENV PATH $PATH:/usr/local/sh-tools/bin
WORKDIR /workspace
ENTRYPOINT ["qemu-system-sh4eb"]
EXPOSE 1234
