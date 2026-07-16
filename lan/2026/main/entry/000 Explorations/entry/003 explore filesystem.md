---
context_type: entry
---

Parent: [000 Explorations](../000%20Explorations.md)

Spawned by: [000 Explorations](../000%20Explorations.md)

Spawned in: [<a name="spawn-entry-4905e6" />^spawn-entry-4905e6](../000%20Explorations.md#spawn-entry-4905e6)

# Journal

2026-07-08 Wk 28 Wed - 11:53 +03:00

Let's try to see what the files are in the game.

* From [001 Golden Sun Master File Table Pointers](../../../wiki/001%20Wiki%20Entities/entry/001%20Golden%20Sun%20Master%20File%20Table%20Pointers.md), we expect the table to be at `0x08320000`.
* $\to$ `/home/lan/src/cloned/gh/Coaltergeist/goldensun-decomp/goldensun.map`: `rom_320000`
  * note
    * commit: `85283d89`
* $\to$ `/home/lan/src/cloned/gh/Coaltergeist/goldensun-decomp/include/file_table.h > gFileTable`
