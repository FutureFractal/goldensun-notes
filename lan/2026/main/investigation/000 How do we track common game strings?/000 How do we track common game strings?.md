---
status: todo
---
2026-07-11 Wk 28 Sat - 05:55 +03:00# Resolution

TODO

# Journal

2026-07-08 Wk 28 Wed - 07:36 +03:00

So at the very beginning of the game we see the string "Isaac" in character type menu. There's also "DEL" and "END". 

Checking `Golden Sun 1: The Broken Seal Documentation documentation` from Golden Sun Hacking Community Server

There is also channel `GS1 Game Text Dump` $\to$  document `[GS1] Game Text`.

There is a section `PC Names` with the text `Isaac` saying `This is where default character names come from`., though not much more on that. No addresses or method.

There are some long strings like `Extracts water from a magic spring`. Can we figure out how to find this text?

There is an editor for golden sun called GSMagic by Beyond in [[000 Entity Golden Sun Community Hacking Server|GSCHS]].

Spawn [[lan/2026/main/investigation/000 How do we track common game strings?/investigation/000 How does gsmagic parse strings from the GS ROM?]] ^spawn-invst-a14068

Spawn [[001 Wontdo How does gsmagic decompress an asset?]] ^spawn-invst-45064e

2026-07-08 Wk 28 Wed - 18:33 +03:00

- `/home/lan/src/cloned/gh/Coaltergeist/goldensun-decomp/data/strings/strings.txt > commit 85283d89`. 
	- note
		- They are many strings_xx.bin there and reference to huffman trees for decompression.
- $\to$ `goldensun-decomp/Makefile > data/strings/strings.txt`
	- note
		- Makefile builds 
			- source `baserom.gba`
				- relates by: through `tools/unpack_strings`
			- $\to$ `data/strings/strings.txt` 
				- relates by: through `tools/pack_strings`
			- $\to$ `data/strings/strings.s` 
			- $\to$ `data/strings/strings.o`.
	- --------------------------------------------------------------------------------
	- |-- `goldensun-decomp/tools/unpack_strings.c`
	- $\to$ `goldensun-decomp/tools/unpack_strings.c > print_strings`
		- --------------------------------------------------------------------------------
		- |-- `goldensun-decomp/tools/unpack_strings.c > DATA_OFFSET (0x80736b8)`
		- $\to$ `goldensun-decomp/aliases.txt > gStringTable`
			- related by: address mentioned in
		- $\to$ `goldensun-decomp/goldensun.map > StringPointers`
			- related by: address mentioned in
		- $\to$ `goldensun-decomp/stage1.ld > data/strings/strings.o``
			- related by: built through
			- note
				- order of these files currently unclear, like with `_c_c` and `rom_1fe2c.o` being between `rom_73812.o` and `rom_73864.o`. Why is that?
					- Added to [[003 Inlined Question Inbox]]
	- --------------------------------------------------------------------------------
	- |-- `goldensun-decomp/tools/pack_strings.c`

2026-07-09 Wk 28 Thu - 05:38 +03:00

So `strings.txt` is a build artifact, always sourced by decompression from `baserom.gba` and then subsequently divided into different parts that are recompressed into `strings.o`. This makes sense for the initial pass building of the repo, but I wonder about modifying the text. Would this allow ROM shifting?

Added to [[003 Inlined Question Inbox]].

I imagine it should be possible to just bypass the recompilation process. We are already decoupled from the exact address. It only hardcodes the address when sourcing from `baserom.gba`.

2026-07-09 Wk 28 Thu - 07:00 +03:00

Spawn [[000 Mod Change the starter name entry name and buttons]] ^spawn-task-adabc8

2026-07-13 Wk 29 Mon - 02:22 +03:00

[[001 Issue building gcc_decomp ld message.sym has syntax error hash]]
