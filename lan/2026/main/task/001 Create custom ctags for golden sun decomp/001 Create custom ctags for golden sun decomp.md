
Issues encountered: [[000 Issues for Create custom ctags for golden sun decomp]]

# Journal

2026-07-15 Wk 29 Wed - 21:25 +03:00

Added to [[001 Inbox]]

2026-07-15 Wk 29 Wed - 22:42 +03:00

Spawn [[000 Issues for Create custom ctags for golden sun decomp]] ^spawn-entry-d6a886

2026-07-15 Wk 29 Wed - 22:58 +03:00

Install https://github.com/universal-ctags/ctags:

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

2026-07-15 Wk 29 Wed - 21:30 +03:00

```sh
ctags --help

# out (relevant)
  --options=file
       Specify file from which command line options should be read.
```

`Debug_StartGame` under `asm/rom_77000/rom_77320_a_a_b.s` is reachable directly by the tags file created through `ctags -R **/* .` but not 

```
asm/rom_8a000/rom_8a5f8_a_c_a.s:.thumb_func_start GameStart  @ 0x0808a8e4
```

https://docs.ctags.io/en/latest/optlib.html

2026-07-15 Wk 29 Wed - 23:10 +03:00

Let's test for this file:

```asm
# in /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main/foo.s

Hello:
  push {lr}
  pop {pc}

.thumb_func_start Bye
  push {lr}
  pop {pc}
.func_end Bye

.thumb_func_start GameStart  @ 0x0808a8e4
	push	{r5, r6, r7, lr}
	mov	r7, r11
  # ...
	bl	Func_808a5f8
	b	.L8a96e
.func_end GameStart
```

2026-07-16 Wk 29 Thu - 00:22 +03:00

When having the file as a `*.pp` I am able to detect the `thumb_func_start` symbols with this, where `.s` is replaced with `.pp`:

```sh
# in /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main/.opt.ctags
--langdef=asmgs
--map-asmgs=+.s
--kinddef-asmgs=f,fnmacro,functions
--regex-asmgs=/^.thumb_func_start[[:blank:]]*([a-zA-Z0-9_]*).*$/\1/f
```

With `.s` now we only get the `Hello`. It might be because of precedence and it is defaulting to its own asm rules.

https://docs.ctags.io/en/latest/optlib.html#defining-a-subparser

A subparser is what we want. We're relying on the rules included for asm, but with a subparser for the extra key choices made by the goldensun-decomp project.

It works!

```sh
# in /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main/.opt.ctags
--langdef=asmgs{base=asm}
--kinddef-asmgs=f,fnmacro,functions
--regex-asmgs=/^.thumb_func_start[[:blank:]]*([a-zA-Z0-9_]*).*$/\1/f

# Run `ctags --options=.opt.ctags foo.s`

# in /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main/tags
Bye	foo.s	/^.thumb_func_start Bye$/;"	r
GameStart	foo.s	/^.thumb_func_start GameStart  @ 0x0808a8e4$/;"	r
Hello	foo.s	/^Hello:$/;"	l
```

Vim recognizes this also with C-] for me!

2026-07-16 Wk 29 Thu - 00:57 +03:00

There's a particularity of this project, that there are many stubs. `_foo` pointing over to `foo`. It wouldn't make sense to an extern deceleration since there can be many of those, but we know that the true source is likely a `Foo` defined in asm. Unsure yet if this may happen between C files as I did not observe it. We are able to support routing `_Foo` to `Foo` just by repeating the regex and instead emitting an `_` variant:

```sh
# in /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main/.opt.ctags
--regex-asmgs=/^.thumb_func_start[[:blank:]]*([a-zA-Z0-9_]*).*$/\1/f
--regex-asmgs=/^.thumb_func_start[[:blank:]]*([a-zA-Z0-9_]*).*$/_\1/f
```

Although a cost of this choice is that we double the tags collected and likely include many false negatives.

2026-07-16 Wk 29 Thu - 01:03 +03:00

We also want to be able to go to where data is defined. For example this uses `gDebugMode`. And it may also be used with a `.word gDebugMode`. But where is it defined?

```asm
# in /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main/asm/rom_8a000/rom_8a5f8_a_c_a.s
	ldr	r3, =gDebugMode
```

We should be able to `C-]` to this. It's actually defined in `wram.sym`. And it's simply a symbol with a defined value. We confirm that we can vary this value and get a different position for it in a generated `goldensun.map`.

We are able to add handle for sym files with

```sh
# in /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main/.opt.ctags
--langdef=gssym
--map-gssym=+.sym
--kinddef-gssym=v,var,variable
--regex-gssym=/^([a-zA-Z0-9_]*)[[:blank:]]*=.*/\1/v
```

This is for more than `wram.sym`. There's also `file_table.sym` and `message.sym`.

I changed my own generated `goldensun.sym` to `goldensun.rosym` for read-only symbol mapping. This serves a post-build purpose of providing a single file mapping many symbols from the binary, while the `*.sym` files are actually part of the build in this project.

2026-07-16 Wk 29 Thu - 02:12 +03:00

There's also some false positives coming from the parent parser as well. For example `gScript_888__0200c15c`

The parent parser is already able to handle this, but takes us to `.global gScript_888__0200c15c` instead, although the definition is elsewhere. Note `tags` does also record where the definition is, but there are multiple matches as a result and it picks the `.global` as a default when I do `Ctrl+]` with vim. This shouldn't be a big deal and we shouldn't have to interfere here. In the case of `gScript_888__0200c15c` It's also defined in the same file so it can be found in the file.

2026-07-16 Wk 29 Thu - 02:28 +03:00

Here is what we have so far:

```sh
# in /home/lan/src/forked/gh/LanHikari22/Coaltergeist/branches/goldensun-decomp@fork-main/.opt.ctags
--langdef=asmgs{base=asm}
--kinddef-asmgs=f,fnmacro,functions
--regex-asmgs=/^.thumb_func_start[[:blank:]]*([a-zA-Z0-9_]*).*$/\1/f
--regex-asmgs=/^.thumb_func_start[[:blank:]]*([a-zA-Z0-9_]*).*$/_\1/f

--langdef=gssym
--map-gssym=+.sym
--kinddef-gssym=v,var,sym global variable
--regex-gssym=/^([a-zA-Z0-9_]*)[[:blank:]]*=.*/\1/v
```

2026-07-16 Wk 29 Thu - 02:33 +03:00

Another issue is that autogenerated assembly corresponding to C is being favored for definitions that have C definitions, since they're duplicated with tags... I guess there really are multiple definitions of many things thanks to these autogenerated files. But anyway In my case with vim I can do something like `9-ctrl-]` to go to the last entry instead of the first, or you can inspect which from the tags you want to follow separately.

2026-07-16 Wk 29 Thu - 02:36 +03:00

Ok let's open an issue for this and then contribute a PR. I was wondering if I should open an issue since I seem to be the first, but I think it's fine: https://github.com/Coaltergeist/goldensun-decomp/issues/14. This should explain the rationale and desired change. Now let's open a PR.

Add upstream,


```sh
# in /home/lan/src/forked/gh/LanHikari22/Coaltergeist/goldensun-decomp/.git/config
[remote "upstream"]
	url = git@github.com:Coaltergeist/goldensun-decomp.git
	fetch = +refs/heads/*:refs/remotes/origin/*
```

```sh
# in /home/lan/src/forked/gh/LanHikari22/Coaltergeist/goldensun-decomp/.git/config > branch main
git pull upstream main
git switch -c add-ctag-opts
# Add .opt.ctags
# Add tags file to .gitignore
```

I also add a way to run it from the the makefile. I can do it on my own but this should standardize it:

```make
# Builds ctags using custom parsing on top of asm.
# Ensure https://github.com/universal-ctags/ctags is installed to use.
tags:
	ctags -R --options=.opt.ctags **/* *
```

--/ 2026-07-16 Wk 29 Thu - 03:11 +03:00 | Had some issues with this without the `-R` and also when doing `**/*.c **/*.s *.sym`  . But in this form it works.
--/

Make sure to remove the tags file on clean too:

```make
clean::
	-$(RM) $(ROM) $(OVERLAYS) $(ELFS) $(MAPS) tags
```

Renaming `.opt.ctag` to `.opts.ctag`.

2026-07-16 Wk 29 Thu - 03:18 +03:00

```sh
# in /home/lan/src/forked/gh/LanHikari22/Coaltergeist/goldensun-decomp > branch add-ctag-opts
git commit

# out
[add-ctag-opts a13b5c55] Added cust ctag parsing for asm funcs and syms
```

--/ 2026-07-16 Wk 29 Thu - 03:20 +03:00 | Amend: 
- `tag` rather than `tags` typo in .gitignore
- some spacing at the end of lines.
--/ 2026-07-16 Wk 29 Thu - 03:23 +03:00 | Amend:
- Description for `.opt.ctags` decision to duplicate with `_` at start for `.thumb_func_start` functions.
--/