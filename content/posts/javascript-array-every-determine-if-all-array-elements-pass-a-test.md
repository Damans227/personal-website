---
title: "Javascript Array every: Determine if All Array Elements Pass a Test"
date: 2023-02-10T06:42:09-05:00
draft: false
tags:
  - JavaScript
  - ES5
---

The Javascript `Array.every()` method tests whether all the elements of an array satisfy the given condition (a callback passed in by the user). 

> In essence, the `every()` method executes the **callback function** for each element of the array and returns `true` if all elements return `true` or `false` if **only one element** returns `false`.

```javascript
const isBelowThreshold = (currentValue) => currentValue < 40;

const array1 = [1, 30, 39, 29, 10, 13];

console.log(array1.every(isBelowThreshold));
// Expected output: true
```

You can read more about `every()` method [here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/every). 