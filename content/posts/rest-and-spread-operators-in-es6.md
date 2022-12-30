---
title: "Rest and Spread Operators in ES6"
date: 2022-12-25T20:39:45-05:00
draft: false
tags:
  - JavaScript
  - ES6
---

`ES6` added two important new operators: `Rest` and `Spread`. Technically they look the same `( ... => three dots)` but they are used in different places.

### Rest:

```javascript
function sumUp(start, ...toAdd) {}
```
Transforms a list of arguments `(1, 2, 3)` into an array `([1, 2, 3])` which may be used inside the function. This behavior is triggered when used inside of a function argument list.

### Spread:

```javascript 
let ids = [1, 2, 3, 4, 5, 6];
console.log(Math.max(...ids)); // prints 6
```

Transforms an array into a list of arguments. This behavior is triggered when used outside of a function argument list.
