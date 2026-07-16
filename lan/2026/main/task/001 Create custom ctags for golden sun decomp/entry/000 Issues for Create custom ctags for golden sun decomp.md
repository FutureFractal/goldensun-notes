---
context_type: entry
---

Parent: [[001 Create custom ctags for golden sun decomp]]

Spawned by: [[001 Create custom ctags for golden sun decomp]]

Spawned in: [[001 Create custom ctags for golden sun decomp#^spawn-entry-d6a886|^spawn-entry-d6a886]]

# Journal

## x No option map for ctags

2026-07-15 Wk 29 Wed - 22:43 +03:00

For a given `.opt.ctags`

```
--langdef=pp
--map-pp=+.pp
--kinddef-asm=thumb_func_start
```

This yields an error:

```sh
# in /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main
ctags --options=.opt.ctags -R **/*.s **/*.c

# out (error)
ctags: Unknown option: --map-pp
```

2026-07-15 Wk 29 Wed - 22:46 +03:00

https://github.com/universal-ctags/ctags mentions that Universal ctags `u-ctags` is currently maintained. Install it:

```sh
mkdir -p ~/src/cloned/gh/universal-ctags/
cd ~/src/cloned/gh/universal-ctags/
git clone git@github.com:universal-ctags/ctags.git
cd ctags
./autogen.sh
./configure
make
sudo make install
```

Check it is installed with `ctags --version`

```sh
# before
Exuberant Ctags 5.9~svn20110310, Copyright (C) 1996-2009 Darren Hiebert
  Addresses: <dhiebert@users.sourceforge.net>, http://ctags.sourceforge.net
  Optional compiled features: +wildcards, +regex
  
# after
Universal Ctags 6.2.0(p6.2.20260705.0), Copyright (C) 2015-2025 Universal Ctags Team
Universal Ctags is derived from Exuberant Ctags.
Exuberant Ctags 5.8, Copyright (C) 1996-2009 Darren Hiebert
  Compiled: Jul 15 2026, 22:50:45
  URL: https://ctags.io/
  Output version: 1.1
  Optional compiled features: +wildcards, +regex, +iconv, +option-directory, +xpath, +packcc, +optscript, +pcre2
```

This should resolve the unknown option problem.