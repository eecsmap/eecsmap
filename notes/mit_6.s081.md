MIT 6.S081 was the old course number. The URL https://pdos.csail.mit.edu/6.S081 is valid yet the new course number is 6.1810.

For some historical reason, the following links are pointing to the same content at the time of writing.
* https://pdos.csail.mit.edu/6.S081
* https://pdos.csail.mit.edu/6.1810
* https://pdos.csail.mit.edu/6.828

# Setup

I am doing this course on my macbook pro (2021) with macOS 14.6.1

I already have both XCode Version 15.4 (15F31d) and homebrew installed.

Following the instructions of https://pdos.csail.mit.edu/6.828/2024/tools.html on macOS

```
$ brew tap riscv/riscv
$ brew install riscv-tools
```

Brew put the tools in folder /opt/homebrew/Cellar/riscv-gnu-toolchain/main.reinstall/bin, I add it into $PATH

```
echo export PATH=/opt/homebrew/Cellar/riscv-gnu-toolchain/main.reinstall/bin:\$PATH >> ~/.zprofile
```

```
$ qemu-system-riscv64 --version
QEMU emulator version 9.0.2
Copyright (c) 2003-2024 Fabrice Bellard and the QEMU Project developers
$ riscv64-unknown-elf-gcc --version
riscv64-unknown-elf-gcc (gc891d8dc2) 13.2.0
Copyright (C) 2023 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

For some reason gdb is not included in riscv-tools. And I don't want to build the tools from source. So I gave it a try using brew:

```
brew install riscv64-elf-gdb
```

```
$ riscv64-elf-gdb --version
GNU gdb (GDB) 15.1
Copyright (C) 2024 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
```

# prepare

```
git clone git://g.csail.mit.edu/xv6-labs-2024
cd xv6-labs-2024
git remote add github https://github.com/eecsmap/xv6-labs-2024.git
git config --global http.postBuffer 524288000
git push -u github
```

# verify
In one terminal
```
cd xv6-labs-2024
make qemu-gdb
```

In another terminal
```
cd xv6-labs-2024
riscv64-elf-gdb
```

![QQ_1724659055078](https://github.com/user-attachments/assets/69c9be93-738c-4103-a401-e9ff8edd4197)

To quit the qemu, press `ctrl-a` then `x`.

# Walk through the instructions

```
(gdb) x/16i $pc
=> 0x1000:      auipc   t0,0x0
   0x1004:      addi    a2,t0,40
   0x1008:      csrr    a0,mhartid
   0x100c:      ld      a1,32(t0)
   0x1010:      ld      t0,24(t0)
   0x1014:      jr      t0
   0x1018:      unimp
   0x101a:      .insn   2, 0x8000
   0x101c:      unimp
   0x101e:      unimp
   0x1020:      unimp
   0x1022:      .insn   2, 0x87e0
   0x1024:      unimp
   0x1026:      unimp
   0x1028:      fnmadd.s        ft6,ft4,fs4,fs1,unknown
   0x102c:      unimp
(gdb) info reg
ra             0x0      0x0
sp             0x0      0x0
gp             0x0      0x0
tp             0x0      0x0
t0             0x0      0
t1             0x0      0
t2             0x0      0
fp             0x0      0x0
s1             0x0      0
a0             0x0      0
a1             0x0      0
a2             0x0      0
a3             0x0      0
a4             0x0      0
a5             0x0      0
a6             0x0      0
a7             0x0      0
s2             0x0      0
s3             0x0      0
s4             0x0      0
s5             0x0      0
s6             0x0      0
s7             0x0      0
s8             0x0      0
s9             0x0      0
s10            0x0      0
s11            0x0      0
t3             0x0      0
t4             0x0      0
t5             0x0      0
t6             0x0      0
pc             0x1000   0x1000
```
As we can see, all registers are zero except `pc`.

Why do we start from pc 0x1000?

Let's look at the exact command executed when we run `make qemu` or `make qemu-gdb`

```
qemu-system-riscv64 -machine virt -bios none -kernel kernel/kernel -m 128M -smp 3 -nographic -global virtio-mmio.force-legacy=false -drive file=fs.img,if=none,format=raw,id=x0 -device virtio-blk-device,drive=x0,bus=virtio-mmio-bus.0 -S -gdb tcp::25501
```
We see it is using machine `virt`. And read the code related to this `virt` machine: https://github.com/qemu/qemu/blob/afaee42f777bc359db95f692804f7fc7e12c0c02/hw/riscv/virt.c#L72

We can tell it is the boot rom of the `virt` machine, and the very first data structure is the `reset vector`.
The content is defined in:

https://github.com/qemu/qemu/blob/afaee42f777bc359db95f692804f7fc7e12c0c02/hw/riscv/boot.c#L397

compare with:

```
(gdb) x/16x $pc
0x1000: 0x00000297      0x02828613      0xf1402573      0x0202b583
0x1010: 0x0182b283      0x00028067      0x80000000      0x00000000
0x1020: 0x87e00000      0x00000000      0x4942534f      0x00000000
0x1030: 0x00000002      0x00000000      0x80000000      0x00000000
```
