---
title: "sed"
permalink: /bash/sed/
excerpt: "sed"
toc: true
sidebar:
  nav: "docs"
---

## Sed

> **S**tream **ed**itor

Sed performs text transformations on streams.

A stream is data that travels from:
- One process to another through a pipe.
- One file to another as a redirect.
- One device to another.

So...for example stdout, stdin, stderr are streams.

### Usage

The most common use of Sed is as a cmd-line version of find-and-replace.

#### Substitution

>`sed s/search-pattern/replacement/flags`

*example*

I have the following text in `manager.txt`:

```
Dwight is the assistant regional manager.
```

I would like to replace `assistant` with `assistant to the`. I can do this by using the substitution syntax and by providing the file as input.

`sed 's/assistant/assistant to the/' manager.txt`

By default Sed prints the result to stdout.

*Note*
If the search pattern isn't found, sed returns the text unaltered.

*example*

I have the following text in `witcher.txt`:

```
The Witcher uses Axxi to kill monsters.
The Witcher uses axxi to break doors and axxi to break barrels.
```

I want to replace all instances of `A/axxi` with `igni`, so...

`sed 's/axxi/igni/ witcher.txt`

But this returns:

```
The Witcher uses Axxi to kill monsters.
The Witcher uses igni to break doors and axxi to break barrels.
```

This is for a couple of reasons:
- Search patterns are case-sensitive. To make a search case-insensitive we need to use the `i/I` flag.
- `sed` runs the replacement on each line, but only for the first instance. To replace all instances on a line we need to use the `g` flag.

The command becomes:

`sed 's/axxi/igni/ig witcher.txt`

*Note*
You can replace a specific instance of a search pattern by using an integer number as a flag.

...

`-r, --regexp-extended`

> The only difference between basic and extended regular expressions is in the behavior of a few characters: ‘?’, ‘+’, parentheses, and braces (‘{}’). While basic regular expressions require these to be escaped if you want them to behave as special characters, when using extended regular expressions you must escape them if you want them to match a literal character.
>[source](https://www.gnu.org/software/sed/manual/html_node/Extended-regexps.html)

