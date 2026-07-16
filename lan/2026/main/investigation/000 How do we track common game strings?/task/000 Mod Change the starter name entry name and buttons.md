---
context_type: task
status: todo
---

Parent: [lan/2026/main/investigation/000 How do we track common game strings?/000 How do we track common game strings?](../000%20How%20do%20we%20track%20common%20game%20strings%3F.md)

Spawned by: [lan/2026/main/investigation/000 How do we track common game strings?/000 How do we track common game strings?](../000%20How%20do%20we%20track%20common%20game%20strings%3F.md)

Spawned in: [<a name="spawn-task-adabc8" />^spawn-task-adabc8](../000%20How%20do%20we%20track%20common%20game%20strings%3F.md#spawn-task-adabc8)

# Objective

When we boot the game, it gives us a name entry UI. It has a default value "Isaac" and two buttons "DEL" and "END".

Change those to "caasI", "LED", and "DNE".

# Resolution

# Journal

2026-07-09 Wk 28 Thu - 07:00 +03:00

2026-07-13 Wk 29 Mon - 03:38 +03:00

Even if we change `Isaac > Ivaac` in `~/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main/data/strings/strings.txt`, it simply gets recreated on a new build.

Let's try to comment out the building of `data/strings/strings.txt`:

````diff
# in /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main/Makefile
-data/strings/strings.txt: baserom.gba tools/unpack_strings
-       mkdir -p $(dir $@)
-       tools/unpack_strings -r $< -o $@
+#data/strings/strings.txt: baserom.gba tools/unpack_strings
+#      mkdir -p $(dir $@)
+#      tools/unpack_strings -r $< -o $@
````

Running `make compare-rom` on this does generate a modified ROM `goldensun.gba: FAILED`.

Changing only the first instance of Isaac to Ivaac did not change it in the starter name entry widget. Let's try to update all of them.

Even updating all of them to `Ivaac` in strings.txt does not change it.
