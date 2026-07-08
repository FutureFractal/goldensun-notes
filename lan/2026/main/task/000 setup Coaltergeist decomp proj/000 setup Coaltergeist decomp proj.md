---
status: done
---
# 1 Objective

- [x] Build an OK ROM.

# Related

 - [[000 Issues for building camelot-gcc]].

# 2 Journal

2026-05-25 Wk 22 Mon - 08:01 +03:00

My system is `Ubuntu 25.04`. Let's get to setting this up.

```sh
mkdir -p ~/src/cloned/gh/Coaltergeist
cd ~/src/cloned/gh/Coaltergeist
git clone git@github.com:Coaltergeist/goldensun-decomp.git
```

According to the README, we need a `baserom.gba` for `Golden Sun (2001)` matching the following checksum:

> * **goldensun.gba** `sha1: 5c4695205413df7db52b9a184815a07783999971` (USA)

2026-05-25 Wk 22 Mon - 08:12 +03:00

```sh
# in /home/lan/src/cloned/gh/Coaltergeist/goldensun-decomp
sha1sum baserom.gba

# out
5c4695205413df7db52b9a184815a07783999971  baserom.gba
```

We have it.

Following [INSTALL.md](https://github.com/Coaltergeist/goldensun-decomp/blob/main/INSTALL.md),

```sh
sudo apt update
sudo apt install build-essential binutils-arm-none-eabi python3 git \
                 bison flex texinfo
```

```sh
cd ~/src/cloned/gh/Coaltergeist
git clone https://github.com/Coaltergeist/camelot-gcc
cd camelot-gcc
chmod +x ./build-296.sh ./install-296.sh
./build-296.sh
```

2026-05-25 Wk 22 Mon - 09:20 +03:00

Spawn [[000 Issues for building camelot-gcc]] ^spawn-entry-cb3dec


---

Next Steps:

```sh
# in /home/lan/src/cloned/gh/Coaltergeist/camelot-gcc
./install-296.sh ~/src/cloned/gh/Coaltergeist/goldensun-decomp
cd ~/src/cloned/gh/Coaltergeist/goldensun-decomp
```