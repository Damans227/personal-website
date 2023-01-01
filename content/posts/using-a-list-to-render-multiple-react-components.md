---
title: "Using a List to Render Multiple React Components"
date: 2023-01-01T17:50:31-05:00
draft: false
tags:
  - JavaScript
  - React
---

You can build collections of elements and include them in JSX using curly braces `{}`.

Below, we loop through the `numbers` array using the JavaScript `map()` function. We return a `<li>` element for each item. Finally, we assign the resulting array of elements to `listItems`:

```javascript
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li>{number}</li>
);
```

Then, we can include the entire `listItems` array inside a `<ul>` element:

```javascript
<ul>{listItems}</ul>
```

This code displays a bullet list of numbers between 1 and 5.

Usually you would render lists inside a component.

We can refactor the previous example into a component that accepts an array of `numbers` and outputs a list of elements.

```javascript
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <li>{number}</li>
  );
  return (
    <ul>{listItems}</ul>
  );
}
```

> When you run this code, you’ll be given a warning that a key should be provided for list items. A _key_ is a special string attribute you need to include when creating lists of elements. We’ll discuss why it’s important in the next section.

Let’s assign a `key` to our list items inside `numbers.map()` and fix the missing key issue.

```javascript
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <li key={number.toString()}>
      {number}
    </li>
  );
  return (
    <ul>{listItems}</ul>
  );
}
```

Keys help React identify which items have changed, are added, or are removed. Keys should be given to the elements inside the array to give the elements a stable identity.


