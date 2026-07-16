---
context_type: entry
---

Parent: [[lan/2026/main/entry/000 Explorations/000 Explorations]]

Spawned by: [[lan/2026/main/entry/000 Explorations/task/000 Get my bn6f modified mgba fork up for use]]

Spawned in: [[lan/2026/main/entry/000 Explorations/task/000 Get my bn6f modified mgba fork up for use#^spawn-entry-999edc|^spawn-entry-999edc]]

# Journal

## Errata 1

2026-07-13 Wk 29 Mon - 08:37 +03:00

I wrote,

(quote)

2026-07-13 Wk 29 Mon - 08:29 +03:00

Let's build. I found this in my history:

```sh
# in /home/lan/src/forked/gh/LanHikari22/mgba-emu/branches/mgba@exp-re-logs/
docker run --rm -it -v ${PWD}:/home/lan/src/forked/gh/LanHikari22/mgba-emu/branches/mgba@exp-re-logs/ mgba/ubuntu:plucky
```

(/quote)

But actually that doesn't do it. It gives a permission denied for creating a folder:

```
Status: Downloaded newer image for mgba/ubuntu:plucky
mkdir: cannot create directory 'build-plucky': Permission denied
```

But from the README we can see the correct command should be

```
docker run --rm -it -v ${PWD}:/home/mgba/src mgba/ubuntu:plucky
```

Which also was in my history!

