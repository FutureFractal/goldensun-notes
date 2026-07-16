---
context_type: entry
---

Parent: [[lan/2026/main/entry/001 Inbox/001 Inbox]]

Spawned by: [[lan/2026/main/entry/001 Inbox/entry/000 Spawns for Index]]

Spawned in: [[lan/2026/main/entry/001 Inbox/entry/000 Spawns for Index#^spawn-entry-c28945|^spawn-entry-c28945]]

---

# What?

As I work through the repository I inevitably encounter mysteries that are out of scope of my current investigation. We inline them as questions and add them to the inlined question inbox. This may allow me to answer them later when I have more knowledge, or for others to get a sense for my current confusion.

# Status

- [[004 Inlined Question Inbox Done]]

# Inbox

## Inbox

- [ ] From [[000 How do we track common game strings?]],

> - order of these files currently unclear, like with `_c_c` and `rom_1fe2c.o` being between `rom_73812.o` and `rom_73864.o`. Why is that?

## Offered a possible explanation

- [ ] From [[000 How do we track common game strings?]],

> So `strings.txt` is a build artifact, always sourced by decompression from `baserom.gba` and then subsequently divided into different parts that are recompressed into `strings.o`. This makes sense for the initial pass building of the repo, but I wonder about modifying the text. Would this allow ROM shifting?