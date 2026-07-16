---
context_type: entry
---

Parent: [lan/2026/main/wiki/002 Wiki Ea Indexed/002 Wiki Ea Indexed](../002%20Wiki%20Ea%20Indexed.md)

Spawned by: [lan/2026/main/wikiproc/002 Wiki Proc Ea Indexed/entry/000 Spawns for Wiki Proc Ea Indexed](../../../wikiproc/002%20Wiki%20Proc%20Ea%20Indexed/entry/000%20Spawns%20for%20Wiki%20Proc%20Ea%20Indexed.md)

Spawned in: [<a name="spawn-entry-983919" />^spawn-entry-983919](../../../wikiproc/002%20Wiki%20Proc%20Ea%20Indexed/entry/000%20Spawns%20for%20Wiki%20Proc%20Ea%20Indexed.md#spawn-entry-983919)

# What?

This is for a structured experimental changes to the code base. We can disable some code and note what we observe when we build and run the ROM.

Do not expect the entries to be sorted. Just search by ea of the label which you can find in the `*.sym` file. If you don't have it, create it. For example:

````sh
# in /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main
make
objdump -t goldensun.elf | sort -u | grep -E "^0[23689]" | perl -p -e 's/^(\w{8}) (\w).{6} \S+\t(\w{8}) (\S+)$$/\1 \2 \3 \4/g' | grep -v '\$' > goldensun.sym
````

Every ea will appear twice as `positive {ea}` and `negative {ea}` to separate positive results (they expose something useful, rather than just being a failed attempt) and negative results which didn't lead anywhere.

# Positive Index

# Negative Index
