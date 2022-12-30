---
title: "Destructuring an Array or an Object in ES6"
date: 2022-12-25T20:47:19-05:00
draft: false
tags:
  - JavaScript
  - ES6
---

`Destructuring` is a cool new feature which allows you to easily extract values from an object or an array.

> It is important to note, that destructuring for arrays is position based, whereas in case of the objects it is based on the names of the keys. 

### Array:

```javascript 
let numbers = [1, 2, 3, 4, 5];

let [a, b] = numbers;// a => 1, b => 2
```

### Object:

```javascript 
let person = {
    name: 'Max',
    age: 27
    };
    
let {name, age} = person; // Notice the {} instead of []
```

More information may be [found here.](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)
