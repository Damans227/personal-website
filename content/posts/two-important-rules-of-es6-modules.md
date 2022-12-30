---
title: "Two Important Rules of ES6 Modules"
date: 2022-12-26T11:06:26-05:00
draft: false
tags:
  - JavaScript
  - ES6
---

There are two important Rules, which you need to understand if you're working with `ES6` Modules:

1. Modules are _always_ in **Strict Mode** (no need to define `"use strict"`).
2. Modules _don't_ have a **shared, global Scope**. Instead each Module has *its own Scope*.