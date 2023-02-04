---
title: "Computed Property Names in Javascript"
date: 2023-02-03T19:49:23-05:00
draft: false
tags:
  - JavaScript
  - ES6
---

Computed Property Names is an ES6 feature which allows the names of object properties in JavaScript object literal notation to be determined dynamically, i.e. computed.

JavaScript objects are really dictionaries, so it was always possible to dynamically create a string and use it as a key with the syntax `object[‘property’] = value`.

However, ES6 Computed Property Names allow us to use dynamically generated names within object literals. Example:

```javascript
const myPropertyName = 'c'

const myObject = {
  a: 5,
  b: 10,
  [myPropertyName]: 15
} 

console.log(myObject.c) // prints 15
```

To stress that expressions can be used directly as computed property names, another example:

```javascript
const fieldNumber = 3

const myObject = {
  field1: 5,
  field2: 10,
  ['field' + fieldNumber]: 15
}

console.log(myObject.field3) // prints 15
```