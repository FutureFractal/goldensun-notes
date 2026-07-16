---
context_type: task
status: todo
---

Parent: [[lan/2026/main/entry/000 Explorations/000 Explorations]]

Spawned by: [[lan/2026/main/entry/000 Explorations/entry/005 Explore tracing by gdb and augmented mgba]]

Spawned in: [[lan/2026/main/entry/000 Explorations/entry/005 Explore tracing by gdb and augmented mgba#^spawn-task-584892|^spawn-task-584892]]

Issues: [[007 Issues for Get my bn6f modified mgba fork up for use]]

Errata: [[006 Errata for Get my bn6f modified mgba fork up for use]]

# Journal


## Setting the fork branch up

2026-07-13 Wk 29 Mon - 07:54 +03:00

Before for mmbn6 ROM shifting, I experimented with generating logs from a modified mgba directly. I still have it in `/home/lan/src/cloned/gh/mgba-emu/mgba` but it's not released. Let's do that via a fork.

--/ 2026-07-13 Wk 29 Mon - 07:58 +03:00

From deltatraced `001 SB reopen PR for customizable title without make fmt`:

```sh
# in /home/lan/src/cloned/gh/LanHikari22/forked/silverbulletmd/branches/silverbullet@title-text-config
git format-patch -1 a86350da

# out
0001-Add-option-to-strip-title-full-path-to-name-via-serv.patch

# Later ...

# in /home/lan/src/cloned/gh/LanHikari22/forked/silverbulletmd/branches/silverbullet@title-text-config
git apply --verbose ~/tmp/0001-Add-option-to-strip-title-full-path-to-name-via-serv.patch
```

--/

```sh
# in /home/lan/src/cloned/gh/mgba-emu/mgba
git commit

# out
[master 45d2fef09] custom mgba logging for re work
```

```sh
# in /home/lan/src/cloned/gh/mgba-emu/mgba
git format-patch -1 45d2fef09

# out
0001-custom-mgba-logging-for-re-work.patch
```

```sh
# in /home/lan/src/cloned/gh/mgba-emu/mgba
mkdir -p ~/data/patches
mv 0001-custom-mgba-logging-for-re-work.patch ~/data/patches
git reset --hard 583eb8eb9

# Fork the project in github -- already forked: 3112 commits behind! At 6900d13 from 7 years ago, and the project is at 5974 commits now.
mkdir -p ~/src/forked/gh/LanHikari22/mgba-emu/branches/
cd ~/src/forked/gh/LanHikari22/mgba-emu/branches/
git clone git@github.com:LanHikari22/mgba.git mgba@exp-re-logs
cd mgba@exp-re-logs/

```

Configure upstream remote. Add:

```sh
# in /home/lan/src/forked/gh/LanHikari22/mgba-emu/branches/mgba@exp-re-logs/.git/config
[remote "upstream"]
	url = git@github.com:mgba-emu/mgba.git
	fetch = +refs/heads/*:refs/remotes/origin/*
```

Update my fork.

```sh
# in /home/lan/src/forked/gh/LanHikari22/mgba-emu/branches/mgba@exp-re-logs/
git pull upstream master
git push origin master
```

--/ 2026-07-13 Wk 29 Mon - 08:23 +03:00 | Updated branch name `re-logs` to `exp-re-logs`. Roughly, experimental branches I do not expect to merge back into main, or my own `fork-main`.
--/

Now let's apply my logging patch in a new branch.

```sh
# in /home/lan/src/forked/gh/LanHikari22/mgba-emu/branches/mgba@exp-re-logs/
git switch -c exp-re-logs
git apply --verbose ~/data/patches/0001-custom-mgba-logging-for-re-work.patch
```

add and commit, then we have it:

```sh
# in /home/lan/src/forked/gh/LanHikari22/mgba-emu/branches/mgba@exp-re-logs/
git commit

# out
[exp-re-logs fb0513035] custom mgba logging for re work
```

## Using it!

2026-07-13 Wk 29 Mon - 08:29 +03:00

Spawn [[lan/2026/main/entry/000 Explorations/entry/006 Errata for Get my bn6f modified mgba fork up for use]] ^spawn-entry-999edc

Spawn [[lan/2026/main/entry/000 Explorations/entry/007 Issues for Get my bn6f modified mgba fork up for use]] ^spawn-entry-eabf22

[[006 Errata for Get my bn6f modified mgba fork up for use#Errata 1]]

Let's build.

```sh
# in /home/lan/src/forked/gh/LanHikari22/mgba-emu/branches/mgba@exp-re-logs/
docker run --rm -it -v ${PWD}:/home/mgba/src mgba/ubuntu:plucky
```

The binary is in `build-plucky/qt/mgba-qt`: `/home/lan/src/forked/gh/LanHikari22/mgba-emu/branches/mgba@exp-re-logs/build-plucky/qt/mgba-qt`

We can run the game and see what we get.

[[007 Issues for Get my bn6f modified mgba fork up for use#No libSDL3 or libmgba]]

```sh
# in /home/lan/data/roms/goldensun
LD_LIBRARY_PATH=/home/lan/src/forked/gh/LanHikari22/mgba-emu/branches/mgba@exp-re-logs/build-plucky/install/us    raise Exception(f"No symbol found for 0x{n_as_label:X}")
r/local/lib /home/lan/src/forked/gh/LanHikari22/mgba-emu/branches/mgba@exp-re-logs/build-plucky/install/usr/local/bin/mgba goldeun.gba | tee mgba_logs
```

Just cancel some text in the starter menu, enter the name `ABCDE`, and quit.

Don't keep those around for long!

```sh
# in /home/lan/data/roms/goldensun
ls -alh mgba_logs
-rw-rw-r-- 1 lan lan 544M Jul 13 09:08 mgba_logs
```

But yeah they log things like gba stores and reads, bls and bxs:

```
LAN - GBAStore16 PC=8002E1C, addr=4000204, val=4014
LAN - GBALoad32 PC=8002E26, addr=8002EA8, val=40000D4
LAN - BL PC=8002F16, PC2=8002F40 R0=2 R1=30000E0 R2=1001 R3=3001F58 R4=3000000 R5=0
LAN - BX PC=8002F2A, rm=#0:8002E4C
```

Not convenient to be reading all these raw addresses though. Would be better to map them to their symbols.

--/ 2026-07-13 Wk 29 Mon - 09:19 +03:00

From `note-repo dism-exe-notes > 006 Attempt to modify mgba to get information on save corruption gunner issue`,

```sh
cat a | python3 ~/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.py ea_to_sym_filter bn6f.sym --shift -2 > b.log
```

$\to$ 

```python
# in /home/lan/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.py > commit 37de3e85

    sp = subparsers.add_parser('ea_to_sym_filter', 
                               help='Replaces any ea with a symbol. And adds a +N if it is between symbols')
    sp.add_argument('sym_file',
                help='The sym file is used to convert an ea to its corresponding symbol')
    sp.add_argument('--shift',
                default='0',
                help='An amount to shift the eas parsed by')
    sp.add_argument('--file',
                help='Read from a file instead of directly on stdin')
    sp.add_argument('--skip-after',
                help='substring to skip converting values after')
    sp.add_argument('--comment-original',
                action='store_true',
                    help='Keep the original filtered as comment: /*orig*/ new')

def app_ea_to_sym_filter(args: argparse.Namespace):
    sym_file = args.sym_file
    shift = int(args.shift, 10)
    file = args.file
    skip_after = args.skip_after
    comment_original = args.comment_original
	# ...
```

We need a `*.sym` file to use it:

```makefile
# in /home/lan/src/cloned/gh/dism-exe/bn6f/Makefile

OBJDUMP := tools/binutils/bin/arm-none-eabi-objdump

syms: $(SYM)

$(SYM): $(ELF)
	$(OBJDUMP) -t $< | sort -u | grep -E "^0[23689]" | perl -p -e 's/^(\w{8}) (\w).{6} \S+\t(\w{8}) (\S+)$$/\1 \2 \3 \4/g' > $@
```

--/

[[007 Issues for Get my bn6f modified mgba fork up for use#Bad Symbols from ea_to_sym_filter]]


[[007 Issues for Get my bn6f modified mgba fork up for use#Bad Symbols from ea_to_sym_filter]]

```sh
# in /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main
make
objdump -t goldensun.elf | sort -u | grep -E "^0[23689]" | perl -p -e 's/^(\w{8}) (\w).{6} \S+\t(\w{8}) (\S+)$$/\1 \2 \3 \4/g' | grep -v '\$' > goldensun.sym
```

```sh
# in /home/lan/data/roms/goldensun
cat mgba_logs.log | python3 ~/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.py ea_to_sym_filter /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main/goldensun.sym --comment-original > mgba_logs_sym.log
```

This gives a lot of data but it is structured and we can filter it.

## Frequency Map

Let's build a frequency map of functions calls.

```sh
# in /home/lan/data/roms/goldensun
cat mgba_logs_sym.log \
| grep 'BL\|BX' \

# only include PC=...
| cut -d ' ' -f5- \
| cut -d ',' -f1 \

```

```sh
# in /home/lan/data/roms/goldensun/cmd.sh

#!/bin/sh

echo "type,caller,called"

# Ex: LAN - BL PC=/*8002E32*/ AgbMain+31, PC2=/*8004858*/ .gcc2_compiled. R0 [...]
regex="LAN - BL PC=[^ ]* *\([^,]*\), *PC2=[^ ]* *\([^ ]*\).*"
output1="$(cat mgba_logs_sym.log \
  | grep -e "$regex" \
  | sed "s|$regex|BL,\1,\2|g" \
)"

echo "$output1"

# Ex: LAN - BX PC=/*3000714*/ Func_8000dc8+BB, rm=#1:/*300071C*/ Func_8000dc8+C3
regex="LAN - BX PC=[^ ]* *\([^,]*\), *rm=[^ ]* *\([^ ]*\).*"
output2="$(cat mgba_logs_sym.log \
  | grep -e "$regex" \
  | sed "s|$regex|BX,\1,\2|g" \
)"

echo "$output2"
```

--/ 2026-07-13 Wk 29 Mon - 15:12 +03:00 | Had some issues there with `\s` not being recognized.
--/

Now we can view the data with visidata:

```sh
# in /home/lan/data/roms/goldensun
./cmd.sh | vd -f csv
```

We can just press `F` on the called column for a frequency analysis on it.

2026-07-13 Wk 29 Mon - 15:37 +03:00

We found `UI_NameEntry` through this! `Func_80b08b8+9D -> UI_NameEntry+24D` had 742 counts.

How can we inspect these counts?

```
LAN - BX PC=/*80B0956*/ Func_80b08b8+9D, rm=#0:/*8020E26*/ UI_NameEntry+24D
```

> **aside** The new visidata space modal filter for commands is cool! You can tab and it gives numbers to the options so you only have to type the number!

2026-07-13 Wk 29 Mon - 16:23 +03:00

Modifying `dump_code.sh` to see if we can use it for goldensun too. I used it to dump some extra code for bn6f.

```sh
addr=0x80B0956
shift=0x9D
dd skip=$(python3 -c "print($addr - 0x8000000 + $shift)") \
   count=10 \
   if=goldensun.gba \
   of=a.bin \
   bs=1 \
   2> /dev/null
```

```sh
~/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.sh a.bin /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main/goldensun.sym "None" 0x80B0956 0x80B0966
```

> **aside** `fzf-vim` (https://github.com/junegunn/fzf.vim/): (alt+/ wrap text) (ctrl+/ hide preview) (alt+a select-all) (alt+d de-select all) (ctrl-w delete-last-word) (ctrl-v open-horizontal-split) (ctrl-x open-vertical-split) (ctrl-t open-in-new-tab)

