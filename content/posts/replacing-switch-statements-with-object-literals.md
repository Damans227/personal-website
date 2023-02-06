---
title: "Replacing Switch Statements With Object Literals"
date: 2023-02-03T19:35:44-05:00
draft: false
tags:
  - JavaScript
  - ES6
---

A typical `switch` statement in javascript looks like this example below: 

```javascript
var type = 'coke';
var drink;
switch(type) {
case 'coke':
  drink = 'Coke';
  break;
case 'pepsi':
  drink = 'Pepsi';
  break;
default:
  drink = 'Unknown drink!';
}
console.log(drink); // 'Coke'
```

It’s similar to `if` and `else` statements below:

```javascript
function getDrink (type) {
  if (type === 'coke') {
    type = 'Coke';
  } else if (type === 'pepsi') {
    type = 'Pepsi';
  } else if (type === 'mountain dew') {
    type = 'Mountain Dew';
  } else if (type === 'lemonade') {
    type = 'Lemonade';
  } else if (type === 'fanta') {
    type = 'Fanta';
  } else {
    // acts as our "default"
    type = 'Unknown drink!';
  }
  return 'You\'ve picked a ' + type;
}
```

## Problems with switch

Problem with using a `switch` statement like above is that, this implementation is too loose, there is room for error, plus it’s a very verbose syntax to keep repeating yourself.

Basically, from its procedural control flow to its non-standard-looking way it handles code blocks, `switch` statement ahs a lot of issues. Syntactically, it’s not one of JavaScript’s best, nor is its design. We’re forced to manually add `break;` statements within each `case`, which can lead to difficult debugging and nested errors further down the case should we forget!

## Solution: Object Literal lookups

We use Objects all the time, either as constructors or literals. Often, we use them for Object lookup purposes, to get values from Object properties. So why not use an **Object literal** to replace `switch` ?

As the number of “cases” increases, the performance of the object (hash table) gets better than the average cost of the switch (the order of the cases matter). The object approach is a hash table lookup, and the switch has to evaluate each case until it hits a match and a break.

So, let's re-write the above example using a simple Object literal that returns a `String` value only:

```javascript
function getDrink (type) {
  var drinks = {
    'coke': 'Coke',
    'pepsi': 'Pepsi',
    'lemonade': 'Lemonade',
    'default': 'Default item'
  };
  return 'The drink I chose was ' + (drinks[type] || drinks['default']);
}

var drink = getDrink('coke');
console.log(drink);
```
We’ve saved a few lines of code from the switch, and the data is a lot cleaner in presentation 😃 
