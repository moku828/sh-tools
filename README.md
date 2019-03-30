# sh-tools
Toolchain for SH7262(make, qemu, gdb) and TOPPERS sample

## make
    docker build -t sh-tools/make .
    docker run --rm -it -vC:\Users\basas\Downloads\sh7262_bootrom-sh2a_isa_test:/workspace sh-tools/make

## toppers_sample
    docker build -t sh-tools/toppers_sample .
    docker run --rm -it sh-tools/toppers_sample

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
