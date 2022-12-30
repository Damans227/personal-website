---
title: "let and const keywords in ES6"
date: 2022-12-25T09:05:15-05:00
draft: false
tags:
  - JavaScript
  - ES6
---

`let` and `const` are new keywords introduced in `ES6` which enable us to change the way we work with variables and constants. Let and const are both block scoped. Prior to `ES6`, we had only function scope and global scope. Block scope restricts variable access to blocks, i.e. constructs with curly braces, such as `if...else`, `for loops`, etc. As an example:

In the past, we would declare variables using var as follows:

```javascript
if(true){
  var age = 23
}
console.log(age);
```

But with `ES6`, if we now use let to declare a variable inside a block of code, it fails with a legitimate error: 

```javascript
if(true){
  let age = 23
}
console.log(age);
```

Code block above fails with an error such as: 

```
"error"
"ReferenceError: age is not defined
    at yozibopigo.js:9:38
    at https://static.jsbin.com/js/prod/runner-4.1.8.min.js:1:13924
    at https://static.jsbin.com/js/prod/runner-4.1.8.min.js:1:10866"

```

You can read more about scopes [here](https://developer.mozilla.org/en-US/docs/Glossary/Scope).

 **Question:** Why wasn't just `var` modified to be block-scoped? 

 > It wasn't done so due to the backward compatibility reasons. If `var` was changed to be block scoped a lot of existing code would go useless. 

The `const` keyword is a new block-scoped keyword that is used to define constants. Thus, once the value of a constant has been defined, it cannot be changed later on. The following code, for example, would result in an error:
 
```javascript
const AGE = 23
AGE = 27
```

```
"TypeError: Assignment to constant variable.
    at yozibopigo.js:2:5
    at https://static.jsbin.com/js/prod/runner-4.1.8.min.js:1:13924
    at https://static.jsbin.com/js/prod/runner-4.1.8.min.js:1:10866"
```

> It is important to keep in mind that when a constant is defined, it does not hold the value but merely refers to the value being defined. Which is why, when a refernce type such as an array or an object is defined as a constant, they can be altered without an issue. 

Since, `const` holds a reference type as explained above, following code block works just fine: 

```javascript
const AGES = [23, 24, 25]
AGES.push(26)
```

