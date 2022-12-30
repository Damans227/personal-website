---
title: "Use Unary Operator to Convert Non Numbers to Numbers"
date: 2022-12-28T10:18:26-05:00
draft: false
tags:
  - JavaScript
---

The *unary plus (+)* operator precedes its operand and evaluates to its operand but attempts to convert it into a number, if it isn't already.

>Although *unary negation (-)* also can convert non-numbers, unary plus is the fastest and preferred way of converting something into a number, because it does not perform any other operations on the number. It can convert string representations of integers and floats, as well as the non-string values `true, false, and null`. Integers in both decimal and hexadecimal (0x-prefixed) formats are supported. Negative numbers are supported (though not for hex). Using the operator on BigInt values throws a TypeError. If it cannot parse a particular value, it will evaluate to NaN.

## Examples

### Usage with numbers

```javascript

const x = 1;
const y = -1;

console.log(+x);
// 1
console.log(+y);
// -1

```

### Usage with non-numbers

```javascript

+true  // 1
+false // 0
+null  // 0
+function (val) { return val; } // NaN
+1n    // throws TypeError: Cannot convert BigInt value to number

```

For more details read [MDN Docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Unary_plus). 