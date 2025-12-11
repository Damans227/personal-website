---
title: "Logical grammar used in linux vi editor"
date: 2025-12-11T12:00:00Z
draft: false
tags:
  - vi
  - linux
  - comptia
---

# Vi Command Patterns

Vi has a logical grammar once you see it. Let me break down the patterns:

## The Core Grammar: Operator + Motion

Most vi commands follow this structure:

**[count] + operator + motion**

**Operators** (what to do):
- `d` = delete
- `c` = change (delete + enter insert mode)
- `y` = yank (copy)

**Motions** (where/how far):
- `w` = word forward
- `b` = word backward
- `$` = end of line
- `0` = beginning of line
- `G` = end of file
- `gg` = beginning of file

**Combine them:**

| Command | Meaning |
|---------|---------|
| `dw` | delete word |
| `d$` | delete to end of line |
| `dG` | delete to end of file |
| `cw` | change word |
| `c$` | change to end of line |
| `y$` | yank to end of line |

## Double Letter = Whole Line

When you double an operator, it acts on the entire current line:

- `dd` = delete line
- `yy` = yank line
- `cc` = change line

## Uppercase = Stronger/Bigger Version

Uppercase letters are often shortcuts for "to end of line" or a more powerful variant:

| Lowercase | Uppercase | Pattern |
|-----------|-----------|---------|
| `d` (needs motion) | `D` = `d$` | delete to end of line |
| `c` (needs motion) | `C` = `c$` | change to end of line |
| `o` (open line below) | `O` (open line above) | opposite direction |
| `p` (paste after) | `P` (paste before) | opposite position |
| `a` (append after cursor) | `A` (append at end of line) | bigger scope |
| `i` (insert at cursor) | `I` (insert at line beginning) | bigger scope |
| `n` (next search forward) | `N` (next search backward) | opposite direction |
| `g` (prefix key) | `G` (go to last line) | end of file |

## Counts Multiply Everything

Put a number before any command to repeat it:

- `5j` = move down 5 lines
- `3dd` = delete 3 lines
- `4yy` = yank 4 lines
- `2dw` = delete 2 words

## Mnemonics — The Letters Mean Something

- `d` = **d**elete
- `c` = **c**hange
- `y` = **y**ank (copy)
- `p` = **p**ut (paste)
- `w` = **w**ord
- `b` = **b**ack
- `r` = **r**eplace
- `u` = **u**ndo
- `o` = **o**pen line
- `a` = **a**ppend
- `i` = **i**nsert
- `x` = think of it as crossing out a character
- `J` = **J**oin lines
- `G` = **G**o to line

## Search Pattern

- `/pattern` = search forward (slash points right/down)
- `?pattern` = search backward (question = going back to ask again)
- `n` = next match (same direction)
- `N` = next match (opposite direction)

## Scrolling — Directional Letters

- `Ctrl+d` = **d**own half page
- `Ctrl+u` = **u**p half page
- `Ctrl+f` = **f**orward full page
- `Ctrl+b` = **b**ack full page