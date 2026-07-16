---
context_type: entry
---
2026-07-15 Wk 29 Wed - 13:34 +03:00
Parent: [[lan/2026/main/entry/000 Explorations/000 Explorations]]

Spawned by: [[lan/2026/main/entry/000 Explorations/task/000 Get my bn6f modified mgba fork up for use]]

Spawned in: [[lan/2026/main/entry/000 Explorations/task/000 Get my bn6f modified mgba fork up for use#^spawn-entry-eabf22|^spawn-entry-eabf22]]

# Journal

## No libSDL3 or libmgba

2026-07-13 Wk 29 Mon - 08:59 +03:00

```sh
# in /home/lan/data/roms/goldensun
/home/lan/src/forked/gh/LanHikari22/mgba-emu/branches/mgba@exp-re-logs/build-plucky/qt/mgba-qt goldeun.gba

# out
build-plucky/qt/mgba-qt: error while loading shared libraries: libSDL3.so.0: cannot open shared object file: No such file or directory
```

```sh
sudo apt-get install libsdl3-dev
```

```
/home/lan/src/forked/gh/LanHikari22/mgba-emu/branches/mgba@exp-re-logs/build-plucky/qt/mgba-qt: error while loading shared libraries: libmgba.so.0.11: cannot open shared object file: No such file or directory
```

Use

```sh
LD_LIBRARY_PATH=/home/lan/src/forked/gh/LanHikari22/mgba-emu/branches/mgba@exp-re-logs/build-plucky/install/usr/local/lib /home/lan/src/forked/gh/LanHikari22/mgba-emu/branches/mgba@exp-re-logs/build-plucky/install/usr/local/bin/mgba
```

## Bad Symbols from ea_to_sym_filter

2026-07-13 Wk 29 Mon - 10:03 +03:00

```sh
# in /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main
make
objdump -t goldensun.elf | sort -u | grep -E "^0[23689]" | perl -p -e 's/^(\w{8}) (\w).{6} \S+\t(\w{8}) (\S+)$$/\1 \2 \3 \4/g' > goldensun.sym
```

```sh
# in /home/lan/data/roms/goldensun
cat mgba_logs.log | python3 ~/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.py ea_to_sym_filter /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main/goldensun.sym --comment-original > mgba_logs_sym.log
```

We get entries like

```
LAN - GBALoad32 PC=/*80003CC*/ +0C, addr=/*80003F8*/ +04, val=/*3007FA0*/
LAN - BL PC=/*8002E32*/ +31, PC2=/*8004858*/  R0=/*3007EF4*/ +4F4 R1=/*3000000*/  R2=/*85001E00*/ 85001E00 R3=/*40000D4*/ 40000D4 R4=0 R5=0
```

2026-07-13 Wk 29 Mon - 10:07 +03:00

It shouldn't silently fail like this. It should uphold invariants or crash, or at least signify unfound symbols somehow more explicitly.

Let's get a small test file:

```
# in /home/lan/src/cloned/winlenovo/data/roms/goldensun/a

LAN - BL PC=80FA722, PC2=8006864 R0=3007ED0 R1=2003050 R2=50003EC R3=0 R4=80F9439 R5=2003050
LAN - GBALoad32 PC=80F91EA, addr=80F9234, val=2003000
LAN - GBALoad8 PC=80F91EC, addr=2003000, val=0
LAN - COND1 BR PC=80F91F2, PC2=80F920E R0=1 R1=2003000 R2=0 R3=0 R4=80F91E9 R5=A4
LAN - COND1 BR PC=80F91F2, PC2=80F920E R0=1 R1=2003000 R2=0 R3=0 R4=80F91E9 R5=A4
```

Cleaning up the code organization a bit.

Silent fallback:

```python
# in ~/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.py > fn AppEaToSymFilter::get_ea_symbol_or_shifted_or_default
if ea not in ea_to_sym_ord_map:
	return ea_token
```

So this happens when we promote it to raising an exception:

```
Exception: Could not find symbol for (ea 050003ec) (ea_token 50003EC)
```

Let's just report `NOT_FOUND:{ea_token}` for `05xxxxxx`.

Now it doesn't fail but we still reproduce the issue.

```sh
# in /home/lan/src/cloned/winlenovo/data/roms/goldensun
cat a | python3 ~/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.py ea_to_sym_filter /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main/goldensun.sym --comment-original > b

# out
LAN - BL PC=/*80FA722*/ +81, PC2=/*8006864*/  R0=/*3007ED0*/ +4D0 R1=/*/*2003050*/ */  R2=/*50003EC*/ NOT_FOUND:50003EC R3=0 R4=/*80F9439*/  R5=/*/*2003050*/ */ 
LAN - GBALoad32 PC=/*80F91EA*/ +1, addr=/*80F9234*/ , val=/*2003000*/ 
LAN - GBALoad8 PC=/*80F91EC*/ +03, addr=/*2003000*/ , val=0
LAN - COND1 BR PC=/*80F91F2*/ +09, PC2=/*80F920E*/ +25 R0=1 R1=/*2003000*/  R2=0 R3=0 R4=/*80F91E9*/  R5=A4
LAN - COND1 BR PC=/*80F91F2*/ +09, PC2=/*80F920E*/ +25 R0=1 R1=/*2003000*/  R2=0 R3=0 R4=/*80F91E9*/  R5=A4
```

When parsing

```
0200024c g       *ABS*	00000000 ewram_200024c
```

we instead get

```
...
(ea2 0200024c) (sym2 )
(ea2 02000380) (sym2 )
(ea2 02000402) (sym2 )
(ea2 0200042c) (sym2 )
...
(ea2 087f2f14) (sym2 )
(ea2 087f6e64) (sym2 )
(ea2 087fa4ec) (sym2 )
(ea2 087fb4a8) (sym2 )
...
```

via 

```python
# in ~/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.py > fn get_ea_to_sym_ord_map
for (ea2, sym2) in syms:
	print(f'(ea2 {ea2:08x}) (sym2 {sym2})')
```

So there is a problem with parsing the syms file, but this should still not propagate empty strings, so we're having it call it explicitly `ERR_EMPTY_STR`.

```sh
# in /home/lan/src/cloned/winlenovo/data/roms/goldensun
cat a | python3 ~/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.py ea_to_sym_filter /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main/goldensun.sym --comment-original > b

# out
LAN - BL PC=/*80FA722*/ ERR_EMPTY_STR+81, PC2=/*8006864*/ ERR_EMPTY_STR R0=/*3007ED0*/ ERR_EMPTY_STR+4D0 R1=/*/*2003050*/ ERR_EMPTY_STR*/ ERR_EMPTY_STR R2=/*50003EC*/ NOT_FOUND:50003EC R3=0 R4=/*80F9439*/ ERR_EMPTY_STR R5=/*/*2003050*/ ERR_EMPTY_STR*/ ERR_EMPTY_STR
LAN - GBALoad32 PC=/*80F91EA*/ ERR_EMPTY_STR+1, addr=/*80F9234*/ ERR_EMPTY_STR, val=/*2003000*/ ERR_EMPTY_STR
LAN - GBALoad8 PC=/*80F91EC*/ ERR_EMPTY_STR+03, addr=/*2003000*/ ERR_EMPTY_STR, val=0
LAN - COND1 BR PC=/*80F91F2*/ ERR_EMPTY_STR+09, PC2=/*80F920E*/ ERR_EMPTY_STR+25 R0=1 R1=/*2003000*/ ERR_EMPTY_STR R2=0 R3=0 R4=/*80F91E9*/ ERR_EMPTY_STR R5=A4
LAN - COND1 BR PC=/*80F91F2*/ ERR_EMPTY_STR+09, PC2=/*80F920E*/ ERR_EMPTY_STR+25 R0=1 R1=/*2003000*/ ERR_EMPTY_STR R2=0 R3=0 R4=/*80F91E9*/ ERR_EMPTY_STR R5=A4
```

Much more explicit.

```python
# in ~/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.py > fn read_sym_file
mut_out.append((int(tokens[0], 16), tokens[3].strip()))
```

It doesn't check if this is the right thing, or if it's even getting anything back. Let's fix this. In our case now, sym is not entry 3 but 4. But even then, why the empty string? It should be parsed as `00000000`.

This is because we also do not remove # in /home/lan/src/cloned/winlenovo/data/roms/goldensun
cat mgba_logs.log | python3 ~/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.py ea_to_sym_filter /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main/goldensun.sym --comment-original > mgba_logs_sym.log
extra space padding:

```python
# in ~/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.py > fn read_sym_file
tokens = line.split(' ')
```

Checking `~/src/cloned/gh/dism-exe/bn6f/bn6f.sym`, there is no such extra spacing!

```
02000070 g 00000000 byte_2000070
02000090 g 00000000 unk_2000090
```

Anyway in both cases it's always the first *and last* entries. So let's make use of that. (though we may have to generalize more some time since there is no guarantees it has to be like this) We're also throwing an exception if we get an empty string.


```sh
# in /home/lan/src/cloned/winlenovo/data/roms/goldensun
cat a | python3 ~/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.py ea_to_sym_filter /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main/goldensun.sym --comment-original > b

# out
LAN - BL PC=/*80FA722*/ SoundInit+81, PC2=/*8006864*/ .gcc2_compiled. R0=/*3007ED0*/ iwram_3007a00+4D0 R1=/*/*2003050*/ gSoundInfo*/ gSoundInfo R2=/*50003EC*/ NOT_FOUND:50003EC R3=0 R4=/*80F9439*/ InitSoundEngine R5=/*/*2003050*/ gSoundInfo*/ gSoundInfo
LAN - GBALoad32 PC=/*80F91EA*/ UpdateMusicSettings+1, addr=/*80F9234*/ $d, val=/*2003000*/ ewram_2003000
LAN - GBALoad8 PC=/*80F91EC*/ UpdateMusicSettings+03, addr=/*2003000*/ ewram_2003000, val=0
LAN - COND1 BR PC=/*80F91F2*/ UpdateMusicSettings+09, PC2=/*80F920E*/ UpdateMusicSettings+25 R0=1 R1=/*2003000*/ ewram_2003000 R2=0 R3=0 R4=/*80F91E9*/ UpdateMusicSettings R5=A4
LAN - COND1 BR PC=/*80F91F2*/ UpdateMusicSettings+09, PC2=/*80F920E*/ UpdateMusicSettings+25 R0=1 R1=/*2003000*/ ewram_2003000 R2=0 R3=0 R4=/*80F91E9*/ UpdateMusicSettings R5=A4
```

This is better but there are some strange things here like `PC2=/*8006864*/ .gcc2_compiled.` and `R1=/*/*2003050*/ gSoundInfo*/ gSoundInfo`.

```sh
# in ~/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.py > fn AppEaToSymFilter::process_line
if ctx.comment_original:
	mut_sym = f'/*{ea_token}*/ {mut_sym}'
```

`R1=/*/*2003050*/ gSoundInfo*/ gSoundInfo` is not because of

```python
# in ~/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.py > fn AppEaToSymFilter::process_line
print(f'(ea_token {ea_token}) (ea {ea:08x})')
mut_sym = cls.get_ea_symbol_or_shifted_or_default(ea_token, ea + ctx.shift, ctx)
```

neither before nor after. We just get this which has no nested comments:

```
(ea_token 2003050) (ea 02003050)
```

We also confirm that the effect happens here because swapping `/*` for `/?` shows that happening in a nested way

```python
# in ~/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.py > fn AppEaToSymFilter::process_line
if ctx.comment_original:
	mut_sym = f'/*{ea_token}*/ {mut_sym}'
```

```sh
# in /home/lan/src/cloned/winlenovo/data/roms/goldensun
cat a | python3 ~/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.py ea_to_sym_filter /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main/goldensun.sym --comment-original > b

# out (relevant)
R0=/?3007ED0?/ iwram_3007a00+4D0 R1=/?/?2003050?/ gSoundInfo?/ gSoundInfo 
```

It's likely this code, which does a full replace:

```sh
# in ~/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.py > fn AppEaToSymFilter::process_line
if '_' + ea_token in mut_out:
	mut_out = mut_out.replace('_' + ea_token, '<<<PLACEHOLDER>>>')
	mut_out = mut_out.replace(ea_token, mut_sym)
	mut_out = mut_out.replace('<<<PLACEHOLDER>>>', '_' + ea_token)
else:
	mut_out = mut_out.replace(ea_token, mut_sym)

```

--/ 2026-07-13 Wk 29 Mon - 12:00 +03:00

```python
# in ~/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.py > fn AppEaToSymFilter::process_line
for (ea_token, ea) in ea_tokens_and_parsed:
	mut_sym = cls.get_ea_symbol_or_shifted_or_default(ea_token, ea + ctx.shift, ctx)
	print(f'(ea_token {ea_token}) (ea {ea:08x}) (mut_sym {mut_sym})')
```

```sh
# in /home/lan/src/cloned/winlenovo/data/roms/goldensun
cat a | python3 ~/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.py ea_to_sym_filter /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main/goldensun.sym --comment-original > b

# out (relevant)
(ea_token 8006864) (ea 08006864) (mut_sym .gcc2_compiled.)
(ea_token 2003050) (ea 02003050) (mut_sym gSoundInfo)
```

--/ 2026-07-13 Wk 29 Mon - 12:00 +03:00

```python
# in ~/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.py > fn AppEaToSymFilter::process_line
if '_' + ea_token in mut_out:
	print(f'a0 (mut_out {mut_out})')
	mut_out = mut_out.replace('_' + ea_token, '<<<PLACEHOLDER>>>')
	print(f'a1 (mut_out {mut_out})')
	mut_out = mut_out.replace(ea_token, mut_sym)
	print(f'a2 (mut_out {mut_out})')
	mut_out = mut_out.replace('<<<PLACEHOLDER>>>', '_' + ea_token)
	print(f'a3 (mut_out {mut_out})')
else:
	print(f'b0 (mut_out {mut_out})')
	mut_out = mut_out.replace(ea_token, mut_sym)
	print(f'b1 (mut_out {mut_out})')
```

```sh
# in /home/lan/src/cloned/winlenovo/data/roms/goldensun
cat a | python3 ~/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.py ea_to_sym_filter /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main/goldensun.sym --comment-original > b

# out (relevant)
(ea_token 2003050) (ea 02003050) (mut_sym gSoundInfo)
b0 (mut_out LAN - BL PC=/*80FA722*/ SoundInit+81, PC2=/*8006864*/ .gcc2_compiled. R0=/*3007ED0*/ iwram_3007a00+4D0 R1=2003050 R2=50003EC R3=0 R4=80F9439 R5=2003050)
b1 (mut_out LAN - BL PC=/*80FA722*/ SoundInit+81, PC2=/*8006864*/ .gcc2_compiled. R0=/*3007ED0*/ iwram_3007a00+4D0 R1=/*2003050*/ gSoundInfo R2=50003EC R3=0 R4=80F9439 R5=/*2003050*/ gSoundInfo)

(ea_token 2003050) (ea 02003050) (mut_sym gSoundInfo)
b0 (mut_out LAN - BL PC=/*80FA722*/ SoundInit+81, PC2=/*8006864*/ .gcc2_compiled. R0=/*3007ED0*/ iwram_3007a00+4D0 R1=/*2003050*/ gSoundInfo R2=/*50003EC*/ NOT_FOUND:50003EC R3=0 R4=/*80F9439*/ InitSoundEngine R5=/*2003050*/ gSoundInfo)
b1 (mut_out LAN - BL PC=/*80FA722*/ SoundInit+81, PC2=/*8006864*/ .gcc2_compiled. R0=/*3007ED0*/ iwram_3007a00+4D0 R1=/*/*2003050*/ gSoundInfo*/ gSoundInfo R2=/*50003EC*/ NOT_FOUND:50003EC R3=0 R4=/*80F9439*/ InitSoundEngine R5=/*/*2003050*/ gSoundInfo*/ gSoundInfo)
```

It repeated twice.

--/

It's both in R5 and in R2. This is a bug in how we handled this. Let's fix it.

Let's keep track of previous replacements and not repeat them. That fixes that issue. `.gcc2_compiled.` is the weird name of a symbol, so that's a non-issue.

```sh
# in /home/lan/src/cloned/winlenovo/data/roms/goldensun
cat a | python3 ~/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.py ea_to_sym_filter /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main/goldensun.sym --comment-original > b

# out
LAN - BL PC=/*80FA722*/ SoundInit+81, PC2=/*8006864*/ .gcc2_compiled. R0=/*3007ED0*/ iwram_3007a00+4D0 R1=/*2003050*/ gSoundInfo R2=/*50003EC*/ NOT_FOUND:50003EC R3=0 R4=/*80F9439*/ InitSoundEngine R5=2003050
LAN - GBALoad32 PC=/*80F91EA*/ UpdateMusicSettings+1, addr=/*80F9234*/ $d, val=/*2003000*/ ewram_2003000
LAN - GBALoad8 PC=/*80F91EC*/ UpdateMusicSettings+03, addr=/*2003000*/ ewram_2003000, val=0
LAN - COND1 BR PC=/*80F91F2*/ UpdateMusicSettings+09, PC2=/*80F920E*/ UpdateMusicSettings+25 R0=1 R1=/*2003000*/ ewram_2003000 R2=0 R3=0 R4=/*80F91E9*/ UpdateMusicSettings R5=A4
LAN - COND1 BR PC=/*80F91F2*/ UpdateMusicSettings+09, PC2=/*80F920E*/ UpdateMusicSettings+25 R0=1 R1=/*2003000*/ ewram_2003000 R2=0 R3=0 R4=/*80F91E9*/ UpdateMusicSettings R5=A4
```

This looks good.

2026-07-13 Wk 29 Mon - 12:35 +03:00

On the full, we fail even for

```sh
# in /home/lan/src/cloned/winlenovo/data/roms/goldensun
cat mgba_logs.log | python3 ~/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.py ea_to_sym_filter /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main/goldensun.sym --comment-original > mgba_logs_sym.log

# out
Exception: Could not find symbol for (ea 02844010) (ea_token 2844010)
```

Better only do it for ROM.

```sh
# in /home/lan/src/cloned/winlenovo/data/roms/goldensun
cat mgba_logs.log | python3 ~/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.py ea_to_sym_filter /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main/goldensun.sym --comment-original > mgba_logs_sym.log

# out
Exception: Could not find symbol for (ea 08888888) (ea_token 88888888)
```

...Or maybe not at all. Just make it explicit it's not found.

2026-07-13 Wk 29 Mon - 14:11 +03:00

```sym
# in ~/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main/goldensun.sym

080fb768 l       rom_f9000	00000000 $t
080fb77a l       rom_f9000	00000000 $d
080fb77c l       rom_f9000	00000000 $t
080fb78e l       rom_f9000	00000000 $d
080fb790 l       rom_f9000	00000000 $t
```

There are also a lot of these noise symbols getting in the way (they duplicate other symbols and are put at the end). Let's filter them out while building the sym file:

```sh
# in /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main
objdump -t goldensun.elf | sort -u | grep -E "^0[23689]" | perl -p -e 's/^(\w{8}) (\w).{6} \S+\t(\w{8}) (\S+)$$/\1 \2 \3 \4/g' | grep -v '\$' > goldensun.sym
```

OK

## symbol_to_ea does not check exact match for dupl

2026-07-14 Wk 29 Tue - 01:11 +03:00

```python
# in /home/lan/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.py > fn symbol_to_ea
entry = os_system(f"cat {sym_file} | grep {symbol}")

entry_lines = entry.splitlines()

if len(entry_lines) != 1:
	raise Exception(f"Expected 1 entry for symbol {symbol}: {entry_lines}")
```

This fails on `_Func_80b0958` and `Func_80b0958`, but it shouldn't. There's exactly one `Func_80b0958`. Ensure we do further filtering by exact match on the sym field.

```python
# in /home/lan/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.py > fn symbol_to_ea

# Filter the lines by exact match 
def filtered_by_exact_match(lines: List[str]):
	mut_out = []

	for line in lines:
		tokens = line.split(" ")
		if len(tokens) == 0:
			continue

		sym = tokens[-1]

		if symbol == sym:
			mut_out.append(line)

	return mut_out

entry_lines_1 = filtered_by_exact_match(entry_lines)
```

It still grabs the stub instead (`_Func_80b0958`):

```sh
# in /home/lan/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.sh
echo start_addr_hex: $start_addr_hex
echo end_addr_hex: $end_addr_hex
```

```sh
# in /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main
~/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.sh goldensun.gba /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main/goldensun.sym "None" "symbol" Func_80b08b8 Func_80b0958

# out (relevant)
start_addr_hex: 0x80B0021
end_addr_hex: 0x80B0029
```

Needed to update the use here too: 

```diff
# in /home/lan/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.py > fn symbol_to_ea
-entry_tokens = entry.split(" ")
+entry_tokens = entry_lines_1[0].split(" ")
```

2026-07-14 Wk 29 Tue - 03:46 +03:00

From `/home/lan/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.py > fn app_compute_pool_usage`,

```
Exception: dump_ea 0x80B08B9 must be aligned by 2
```

Seems to not handle thumb eas? Let's try to handle if it's thumb in `dump_code.py > fn calc_pool32_ea`.

2026-07-14 Wk 29 Tue - 05:15 +03:00

```sh
# in /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main
~/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.sh goldensun.gba /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main/goldensun.sym "None" hex:0x80b08b8 hex:0x80b08c8

# out (relevant)
   0:   b5e0            push    {r5, r6, r7, lr}
   2:   4657            mov     r7, sl
   4:   464e            mov     r6, r9
```

Added a way to specify whether each start/end is hex/sym. When we use the address `0x80b08b8` we get correct disassembly with a push lr. Using the symbol here also doesn't handle thumb:

```sh
# in /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main
~/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.sh goldensun.gba /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main/goldensun.sym "None" sym:Func_80b08b8 hex:0x80b08c8

# out (relevant)
start_addr_hex: 0x80B08B9
   0:   57b5            ldrsb   r5, [r6, r6]
   2:   4e46            ldr     r6, [pc, #280]  ; (0x11c)
   4:   4546            cmp     r6, r8
```

These are also outputs with most transformations disabled so we handle this key issue first. This is a problem that is happening right from the use of `dd` without taking into account thumb. Which makes it a problem from the output we get from `~/src/cloned/gh/dism-exe/bn6f/tools/misc_scripts/dump_code/dump_code.py > fn get_symbol_boundary `

2026-07-14 Wk 29 Tue - 08:51 +03:00

We can't get the local labels to display because the `goldensun.sym` does not expose them:

```s
cmp r7, #0
beq ERR_LBL_NOT_FOUND:0x80b094a
```

We also get

```
Exception: unused values from shift 0x88 and ea 0x80B0940
```

right after the pool:

```
# generated
        bl DecompressLZ16_ROM_End
        ldrh r5, [r7, #6]
        mov r2, r8
        add r5, r5, r0
        strh r5, [r2, #8]
        strb r5, [r2, #0x14]
        b ERR_LBL_NOT_FOUND:0x80b0940
        .pool
```

```
# in repo
    bl  __divsi3
    ldrh  r5, [r7, #6]
    mov r2, r8
    add r5, r0
    strh  r5, [r2, #8]
    strb  r5, [r2, #0x14]
    b .Lb0940

    .pool_aligned

  .Lb0940:
```

```
# in goldensun.sym
080022ec l       rom_1b70	00000000 DecompressLZ16_ROM_End
080022ed g     F rom_1b70	00000008 __divsi3
```
