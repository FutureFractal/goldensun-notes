---
context_type: issue
status: todo
---

Parent: [[000 setup Coaltergeist decomp proj]]

Spawned by: [[000 Issues for building camelot-gcc]]

Spawned in: [[000 Issues for building camelot-gcc#^spawn-issue-bccbd7|^spawn-issue-bccbd7]]

# Journal

2026-07-13 Wk 29 Mon - 02:22 +03:00

The error was:

```
arm-none-eabi-ld -r -T stage1.ld  -Map stage1.map -o stage1.o
arm-none-eabi-ld:message.sym:12: ignoring invalid character `#' in script
arm-none-eabi-ld:message.sym:12: syntax error
```

Just going to clone and rebuild everything. The author does not reproduce my problem.

Also parallel builds with `make -j$(nproc)` are currently broken but this is a known issue by the author.

2026-07-13 Wk 29 Mon - 02:24 +03:00

```sh
# in /home/lan/src/cloned/gh/Coaltergeist
git clone git@github.com:Coaltergeist/camelot-gcc.git
git clone git@github.com:Coaltergeist/goldensun-decomp.git
cd camelot-gcc
./build.sh all && ./install.sh ../goldensun-decomp all
cd ../goldensun-decomp
# Provide baserom.gba
make
```

Still the same problem

```
arm-none-eabi-as -mcpu=arm7tdmi -Iinclude -MD asm/rom_185000/rom_185008_c.d -o asm/rom_185000/rom_185008_c.o asm/rom_185000/rom_185008_c.s
include/macros.inc: Assembler messages:
include/macros.inc: Warning: end of file not at end of a line; newline inserted
arm-none-eabi-ld -r -T stage1.ld  -Map stage1.map -o stage1.o
arm-none-eabi-ld:message.sym:13: ignoring invalid character `#' in script
arm-none-eabi-ld:message.sym:13: syntax error
make: *** [Makefile:50: stage1.o] Error 1
rm tools/unpack_strings.o tools/pack_strings.o
```

2026-07-13 Wk 29 Mon - 02:47 +03:00

```diff
# in /home/lan/src/cloned/gh/Coaltergeist/goldensun-decomp/message.sym
-# shiftable __MessageID IDs (named by value; pending semantic names)
+/* shiftable __MessageID IDs (named by value; pending semantic names) */
```

Suggested by [SBird](https://github.com/SBird1337). This allows us to build and get an OK:

```sh
# in /home/lan/src/cloned/gh/Coaltergeist/goldensun-decomp
make compare-rom

# out (relevant)
goldensun.gba: OK
```

This is apparently because of `ld` version differences. Mine was:

```sh
ld --version

# out
GNU ld (GNU Binutils for Ubuntu) 2.44
Copyright (C) 2025 Free Software Foundation, Inc.
This program is free software; you may redistribute it under the terms of
the GNU General Public License version 3 or (at your option) a later version.
This program has absolutely no warranty.
```

2026-07-13 Wk 29 Mon - 02:50 +03:00

Let's create a PR with this fix and credit [SBird](https://github.com/SBird1337) with suggesting a solution.

```sh
mkdir -p ~/src/forked/gh/LanHikari22/Coaltergeist
cd ~/src/forked/gh/LanHikari22/Coaltergeist
# Fork it on github
git clone git@github.com:LanHikari22/goldensun-decomp.git
cd goldensun-decomp
git switch -c fix-ld-nonstandard-comment-syntax
cd ~/src/cloned/gh/Coaltergeist/camelot-gcc
./install.sh ~/src/forked/gh/LanHikari22/Coaltergeist/goldensun-decomp all
cd ~/src/forked/gh/LanHikari22/Coaltergeist/goldensun-decomp
```

Add in the edit

```diff
# in /home/lan/src/forked/gh/LanHikari22/Coaltergeist/goldensun-decomp/message.sym
-# shiftable __MessageID IDs (named by value; pending semantic names)
+/* shiftable __MessageID IDs (named by value; pending semantic names) */
```

Then provide `baserom.gba` and build:

```sh
# in /home/lan/src/forked/gh/LanHikari22/Coaltergeist/goldensun-decomp
make compare-rom

# out (relevant)
goldensun.gba: OK
```

```sh
# in /home/lan/src/forked/gh/LanHikari22/Coaltergeist/goldensun-decomp
git commit

# out
[fix-ld-nonstandard-comment-syntax d760653b] swap hash ldscript comment for higher build compat
```

--/ 2026-07-13 Wk 29 Mon - 03:04 +03:00 | Shortened commit name
- before: `[fix-ld-nonstandard-comment-syntax d760653b] swap hash ldscript comment for higher build compat`
- after: `swap hash ldscript comment for higher build compat`
--/

```sh
# in /home/lan/src/forked/gh/LanHikari22/Coaltergeist/goldensun-decomp
git push origin fix-ld-nonstandard-comment-syntax
# Open PR on github
```

https://github.com/Coaltergeist/goldensun-decomp/pull/13

Until it's merged, let's just have it in our `fork-main` branch:

```sh
# in /home/lan/src/forked/gh/LanHikari22/Coaltergeist/goldensun-decomp > branch fix-ld-nonstandard-comment-syntax
git switch -c fork-main
git push origin fork-main
mkdir -p ~/src/forked/gh/LanHikari22/Coaltergeist/branches/
cp -r . ~/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main
```