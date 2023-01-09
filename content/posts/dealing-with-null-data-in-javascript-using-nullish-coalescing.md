---
title: "Dealing With Null Data in Javascript Using Nullish Coalescing"
date: 2023-01-09T17:41:18-05:00
draft: false
tags:
  - JavaScript
---

The *nullish coalescing (??)* operator is a logical operator that returns its right-hand side operand when its left-hand side operand is `null` or `undefined`, and otherwise returns its left-hand side operand.

Earlier, when one wanted to assign a default value to a variable, a common pattern was to use the logical OR operator `(||)`:

```javascript
let foo;

// foo is never assigned any value so it is still undefined
const someDummyText = foo || "Hello!";
```
However, due to `||` being a boolean logical operator, the left-hand-side operand was coerced to a boolean for the evaluation and any falsy value (including `0, '', NaN, false`, etc.) was not returned.

This behavior may cause unexpected consequences if you consider `0, '', or NaN` as valid values.

```javascript
const count = 0;
const text = "";

const qty = count || 42;
const message = text || "hi!";
console.log(qty); // 42 and not 0
console.log(message); // "hi!" and not ""
```

> The nullish coalescing operator avoids this pitfall by only returning the second operand when the first one evaluates to either `null` or `undefined` (but no other falsy values):

```javascript
const myText = ""; // An empty string (which is also a falsy value)

const notFalsyText = myText || "Hello world";
console.log(notFalsyText); // Hello world

const preservingFalsy = myText ?? "Hi neighborhood";
console.log(preservingFalsy); // '' (as myText is neither undefined nor null)
```