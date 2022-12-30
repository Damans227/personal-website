---
title: "Modules and Classes in ES6"
date: 2022-12-26T12:57:57-05:00
draft: false
tags:
  - JavaScript
  - ES6
---

### Modules

*ES6* has added Modules to JavaScript. This means, that you may split up your code over multiple files, which of course is a good practice. This is common in *ES6* already, however you always require a module loader for that. 

To split up your code, you basically export variables, functions, objects, ... in one file and import it in another:

```javascript 
// export.js
export let myExportedVar = 42;

// import.js

import { myExportedVar } from './export.js'
```

### Classes

Classes are now also available via the class keyword. You may of course continue using other ways to create objects, but here’s the class-way:

```javascript 
class Person {
greet() {
    this.name = 'Max'; // this is how you set up properties!
    console.log('Hello!');
    }
}

let person = new Person();
person.greet(); // prints ‘Hello!’
```
