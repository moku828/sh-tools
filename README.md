# sh-tools
Toolchain for SH7262(make, qemu, gdb) and TOPPERS sample

## make
    docker build -t sh-tools/make .
    docker run --rm -it -vC:\Users\basas\Downloads\sh7262_bootrom-sh2a_isa_test:/workspace sh-tools/make
    docker run --rm -it -vC:\test\sh7262_an\task:/workspace --entrypoint=/usr/local/sh-tools/bin/sh-elf-nm sh-tools/make task.elf

## toppers_sample
    docker build -t sh-tools/toppers_sample .
    docker run --rm -it sh-tools/toppers_sample
    docker cp CONTAINER:/asp C:\test\toppers_sample
    docker run --rm -it -vC:\test\toppers_sample\asp:/asp sh-tools/toppers_sample ../configure -T apsh2a_gcc
    docker run --rm -it -vC:\test\toppers_sample\asp:/asp sh-tools/toppers_sample make depend
    docker run --rm -it -vC:\test\toppers_sample\asp:/asp sh-tools/toppers_sample make
    docker run --rm -it -vC:\test\toppers_sample\asp:/asp sh-tools/toppers_sample make asp.bin
    docker run --rm -it -vC:\test\toppers_sample\asp:/asp sh-tools/toppers_sample sh-elf-nm asp | grep \s__kernel_start$
HJ-LINK/USB Debugger for SH7262起動
Download program...からasp選んでPC:1C0004E4(__kernel_startのアドレス)に書き換えてRun

## qemu
    docker build -t sh-tools/qemu .
    docker run --rm -it -vC:\Users\basas\Downloads\sh7262_bootrom-sh2a_isa_test:/workspace -p 1201:1201 -p 1202:1202 --name qemu00 sh-tools/qemu -monitor telnet:0.0.0.0:1201,server,nowait -serial tcp::1202,server,nodelay -drive file=SPIROM.IMG,if=mtd,format=raw -s -S

## gdb
    docker build -t sh-tools/gdb .
    docker run --rm -it -vC:\Users\basas\Downloads\sh7262_bootrom-sh2a_isa_test:/workspace --link qemu00:qemu00 sh-tools/gdb bootrom.elf
    (gdb) target remote qemu00:1234
Note: locate SPIROM.IMG onto C:\Users\basas\Downloads\sh7262_bootrom-sh2a_isa_test

## telnet
port 1201:QEMU monitor  
port 1202:IRQ  
type 'irq0' on 1202  
Note: no echo  
