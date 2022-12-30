---
title: "Arrow Functions and Default Arguments in ES6"
date: 2022-12-25T20:25:38-05:00
draft: false
tags:
  - JavaScript
  - ES6
---

In `ES6` there were two major additions regarding functions: 

1. (Fat) Arrow Functions( `() => {}` ) 
2. and default arguments( `doSmth(arg = 1)` ).

`Fat Arrow Functions` allow you to use a shorter syntax to create functions:

```javascript
const fn = (arg1, arg2) => return arg1 + arg2
```

You may leave out the parenthesis around the arguments if only one argument is passed. If no argument is passed, empty parentheses are required. The function body may be written inline and without curly braces if you’re only writing a return statement (return then also may be left out).

> **Important Note:** Besides the shorter syntax, fat arrow functions also change the behavior of the **this** keyword. Inside a fat arrow function, the **this** keyword will always refer to the context of the function instead of the caller of the function.

`Default parameters` allow you specify default values for parameters passed to functions.

```javascript
function fn(arg1 = 'default string', arg2 = 23) { return arg1+ arg2}
```

More info on [Arrow Functions](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions) and [Default Paramaters](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Default_parameters).
