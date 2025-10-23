---
title: 'Why `sudo echo "..." > file` Doesn’t Work — and How to Fix It'
date: 2025-10-22T15:33:37-05:00
draft: false
tags:
  - cloudstack
  - kvm
---

Tried this command:

```bash
echo "/export  *(rw,async,no_root_squash,no_subtree_check)" > /etc/exports
```

But the file didn’t get updated.

---

## Core Issue: Redirection (`>`) Happens Before `sudo`

When executing

```bash
sudo echo "line" > /etc/file
```

Only the `echo` runs with `sudo`. So this fails silently (or leaves the file empty).

---

## Correct Approach: Use `sudo tee`

```bash
echo "/export  *(rw,async,no_root_squash,no_subtree_check)" | sudo tee /etc/exports > /dev/null
```

This works because:

- `echo` runs normally
- Output is piped (`|`) to `sudo tee`
- `tee` runs with root privileges and writes the file

---

## IMPORTANT: Append Instead of Overwrite?

Use `tee -a`:

```bash
echo "another line" | sudo tee -a /etc/exports > /dev/null
```

---

## What I Did (and Why It Went Wrong)

Ran:

```bash
echo "/export ..." > sudo tee /etc/exports > /dev/null
```

This:

- Created a **file named `sudo`** in your current directory (with the NFS export line in it)
- Did **not** invoke the `sudo` command

---

## Summary

| Goal                  | Command                                                  |
|-----------------------|----------------------------------------------------------|
| Overwrite root file   | `echo "..." | sudo tee /file > /dev/null`               |
| Append to root file   | `echo "..." | sudo tee -a /file > /dev/null`            |