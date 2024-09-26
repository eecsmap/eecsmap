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

# sync origin to github

```
git branch -avv
git fetch --all --prune
git checkout riscv
git rebase origin/riscv
git push github riscv
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

# GDB tips
Some from the .gdbinit file under the root of the project.
```
set confirm off
set architecture riscv:rv64
target remote 127.0.0.1:25501
symbol-file kernel/kernel
set disassemble-next-line auto
set riscv use-compressed-breakpoints yes
```

# project meta

```
engineer@xv6-labs-2024 % find kernel -name '*.S' -o -name '*.c' -o -name '*.h' | xargs wc -l | sort -n
       5 kernel/fcntl.h
       9 kernel/spinlock.h
      10 kernel/sleeplock.h
      10 kernel/types.h
      11 kernel/stat.h
      12 kernel/buf.h
      15 kernel/param.h
      21 kernel/entry.S
      22 kernel/syscall.h
      40 kernel/file.h
      42 kernel/elf.h
      42 kernel/swtch.S
      45 kernel/main.c
      47 kernel/plic.c
      55 kernel/sleeplock.c
      59 kernel/memlayout.h
      60 kernel/fs.h
      64 kernel/kernelvec.S
      66 kernel/start.c
      82 kernel/kalloc.c
      93 kernel/sysproc.c
      96 kernel/virtio.h
     107 kernel/proc.h
     107 kernel/string.c
     110 kernel/spinlock.c
     130 kernel/pipe.c
     147 kernel/syscall.c
     151 kernel/trampoline.S
     153 kernel/bio.c
     166 kernel/exec.c
     178 kernel/printf.c
     182 kernel/file.c
     189 kernel/defs.h
     191 kernel/uart.c
     192 kernel/console.c
     218 kernel/trap.c
     236 kernel/log.c
     327 kernel/virtio_disk.c
     378 kernel/riscv.h
     451 kernel/vm.c
     505 kernel/sysfile.c
     695 kernel/proc.c
     697 kernel/fs.c
    6416 total
engineer@xv6-labs-2024 % find user -name '*.S' -o -name '*.c' -o -name '*.h' | xargs wc -l | sort -n 
      14 user/zombie.c
      15 user/ln.c
      17 user/kill.c
      19 user/echo.c
      23 user/mkdir.c
      23 user/rm.c
      28 user/initcode.S
      43 user/cat.c
      43 user/user.h
      49 user/stressfs.c
      54 user/init.c
      55 user/wc.c
      56 user/forktest.c
      87 user/ls.c
      90 user/umalloc.c
     107 user/grep.c
     147 user/ulib.c
     157 user/printf.c
     351 user/grind.c
     494 user/sh.c
    3118 user/usertests.c
    4990 total
```

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

Let's look through them one by one.

```
0x1000: 0x00000297
auipc   t0,0x0
        0x00000297
        U
        AUIPC
        opcode  :0000000_00000_00000_000_00101_0010111  0x17
        rd      :0000000_00000_00000_000_00101_0010111  x5 t0
        funct3  :0000000_00000_00000_000_00101_0010111  0x0
        rs1     :0000000_00000_00000_000_00101_0010111  x0 zero
        rs2     :0000000_00000_00000_000_00101_0010111  x0 zero
        funct7  :0000000_00000_00000_000_00101_0010111  0x00
        I imm   :0000000_00000_00000_000_00101_0010111  0
        S imm   :0000000_00000_00000_000_00101_0010111  5
        B imm   :0000000_00000_00000_000_00101_0010111  2052
        U imm   :0000000_00000_00000_000_00101_0010111  0
        J imm   :0_0000000000_0_00000000_00101_0010111  0
```
The opcode lowest two digits are always 11.
The opcode is `10111`, stands for `AUPIC`, meaning: add upper immediate to `pc`
```
# according to the riscv spec
rd = pc + signed_extended_64bit(U_immediate_20bits << 12)
```
Here we get the $PC value 0x1000 and keep it in register `t0`

```
(gdb) stepi
0x0000000000001004 in ?? ()
=> 0x0000000000001004:  02828613                addi    a2,t0,40
(gdb) info register
ra             0x0      0x0
sp             0x0      0x0
gp             0x0      0x0
tp             0x0      0x0
t0             0x1000   4096
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
pc             0x1004   0x1004
```

Next:

```
0x1004: 0x02828613
addi    a2,t0,40
        0x02828613
        I
        OP_IMM
        opcode  :0000001_01000_00101_000_01100_0010011  0x13
        rd      :0000001_01000_00101_000_01100_0010011  x12a2
        funct3  :0000001_01000_00101_000_01100_0010011  0x0
        rs1     :0000001_01000_00101_000_01100_0010011  x5 t0
        rs2     :0000001_01000_00101_000_01100_0010011  x8 s0
        funct7  :0000001_01000_00101_000_01100_0010011  0x01
        I imm   :0000001_01000_00101_000_01100_0010011  40
        S imm   :0000001_01000_00101_000_01100_0010011  44
        B imm   :0000001_01000_00101_000_01100_0010011  44
        U imm   :0000001_01000_00101_000_01100_0010011  42106880
        J imm   :0_0000010100_0_00101000_01100_0010011  163880
```
It add the immediate value 40(decimal) on the `t0` value, and put the result `0x1028` into register `a2`.

```
(gdb) x/i $pc
=> 0x1004:      addi    a2,t0,40
(gdb) stepi
0x0000000000001008 in ?? ()
=> 0x0000000000001008:  f1402573                csrr    a0,mhartid
(gdb) info reg
ra             0x0      0x0
sp             0x0      0x0
gp             0x0      0x0
tp             0x0      0x0
t0             0x1000   4096
t1             0x0      0
t2             0x0      0
fp             0x0      0x0
s1             0x0      0
a0             0x0      0
a1             0x0      0
a2             0x1028   4136
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
pc             0x1008   0x1008
```

Next:
```
(gdb) info all-registers
...
mhartid        0x0      0
mconfigptr     0x0      0
priv           0x3      prv:3 [Machine]
```

```
(gdb) x/i $pc
=> 0x1008:      csrr    a0,mhartid
(gdb) stepi
0x000000000000100c in ?? ()
=> 0x000000000000100c:  0202b583                ld      a1,32(t0)
(gdb) info reg
ra             0x0      0x0
sp             0x0      0x0
gp             0x0      0x0
tp             0x0      0x0
t0             0x1000   4096
t1             0x0      0
t2             0x0      0
fp             0x0      0x0
s1             0x0      0
a0             0x0      0
a1             0x0      0
a2             0x1028   4136
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
pc             0x100c   0x100c
```

```
f1402573
csrrs
        0xf1402573
        UNDEFINED
        SYSTEM
        opcode  :1111000_10100_00000_010_01010_1110011  0x73
        rd      :1111000_10100_00000_010_01010_1110011  x10a0
        funct3  :1111000_10100_00000_010_01010_1110011  0x2
        rs1     :1111000_10100_00000_010_01010_1110011  x0 zero
        rs2     :1111000_10100_00000_010_01010_1110011  x20s4
        funct7  :1111000_10100_00000_010_01010_1110011  0x78
        I imm   :1111000_10100_00000_010_01010_1110011  -236
        S imm   :1111000_10100_00000_010_01010_1110011  -246
        B imm   :1111000_10100_00000_010_01010_1110011  -2294
        U imm   :1111000_10100_00000_010_01010_1110011  -247455744
        J imm   :1_1110001010_0_00000010_01010_1110011  -1038572
```



