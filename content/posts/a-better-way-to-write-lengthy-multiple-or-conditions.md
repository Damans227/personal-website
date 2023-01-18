---
title: "A Better Way to Write Lengthy Multiple or Conditions"
date: 2023-01-18T15:33:37-05:00
draft: false
tags:
  - JavaScript
---

When working with JavaScript, we deal a lot with conditionals, here is a quick and easy way to write better / cleaner multiple or conditionals.

Let’s take a look at the example below:

```javascript
// condition
function test(fruit) {
  if (fruit == 'apple' || fruit == 'strawberry') {
    console.log('red');
  }
}
```

Initially, the example above appears to be a good one. However, what if we received more red fruits, such as **cherries** and **cranberries**? Will the statement be extended with more `||` ?

By using `Array.includes` [(Array.includes)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/includes), we can rewrite the conditional above.

```javascript
function test(fruit) {
  // extract conditions to array
  const redFruits = ['apple', 'strawberry', 'cherry', 'cranberries'];

  if (redFruits.includes(fruit)) {
    console.log('red');
  }
}
```

We extract the `red fruits` (conditions) to an array. By doing this, the code looks tidier.