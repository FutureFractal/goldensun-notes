---
context_type: investigation
status: wontdo
---

Parent: [[lan/2026/main/investigation/000 How do we track common game strings?/000 How do we track common game strings?]]

Spawned by: [[lan/2026/main/investigation/000 How do we track common game strings?/000 How do we track common game strings?]]

Spawned in: [[lan/2026/main/investigation/000 How do we track common game strings?/000 How do we track common game strings?#^spawn-invst-45064e|^spawn-invst-45064e]]

# Objective

Resolution should include a program that applies the same decompression algorithm.

# Won't do

We already have tooling that applies the modified huffman decompression algorithm. See `/home/lan/src/cloned/gh/Coaltergeist/goldensun-decomp/tools/unpack_strings.c`

# Resolution

It should be similar to what we have in `/home/lan/src/cloned/gh/Coaltergeist/goldensun-decomp/tools/unpack_strings.c` in the case of text.

WONTDO

# Journal

2026-07-08 Wk 28 Wed - 10:38 +03:00

In communications, Beyond (developer of gsmagic) pointed to https://github.com/romhack/GoldenSunCompression/blob/master/gs-huffman/gs-huffman.hs as a project that is able to decompress the same assets. The README there explains that text in golden sun is compressed with a modified Huffman algorithm.

