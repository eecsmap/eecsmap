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

For some reason gdb is not included in riscv-tools. And I don't want to build the tools from source. So I gave it a try to brew:

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
riscv64-elf-gdb
```
