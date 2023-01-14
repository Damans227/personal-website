---
title: "Rules of Using React Hooks"
date: 2023-01-13T20:30:21-05:00
draft: false
tags:
  - JavaScript
  - React
---

## 1) Only Call Hooks at the Top Level

**Don’t call Hooks inside loops, conditions, or nested functions.** Instead, always use Hooks at the top level of your React function, before any early returns. 

By following this rule, you ensure that Hooks are called in the same order each time a component renders. That’s what allows React to correctly preserve the state of Hooks between multiple `useState` and `useEffect` calls.

## 2) Only Call Hooks from React Functions

**Don’t call Hooks from regular JavaScript functions.** Instead, you can:

- Call Hooks from React function components.
- Call Hooks from custom Hooks.

By following this rule, you ensure that all stateful logic in a component is clearly visible from its source code.

## 3) Unofficial rule for `useEffect()`

**ALWAYS** add everything you refer to inside `useEffect()` as a dependency. 