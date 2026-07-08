---
status: todo
---
# Resolution

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
- $\to$ `/home/lan/src/cloned/gh/Coaltergeist/goldensun-decomp/Makefile > data/strings/strings.txt`
	- |--- `/home/lan/src/cloned/gh/Coaltergeist/goldensun-decomp/tools/unpack_strings.c`
	- |--- `/home/lan/src/cloned/gh/Coaltergeist/goldensun-decomp/tools/pack_strings.c`