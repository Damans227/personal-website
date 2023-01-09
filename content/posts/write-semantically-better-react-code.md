---
title: "Write Semantically Better React Code"
date: 2023-01-09T18:05:12-05:00
draft: false
tags:
  - JavaScript
  - React
---

## Using Fragments to solve the problem of `</div>` Soup

When writing a component in React, you can't have more than one root JSX element. So if you return a value or if you store a value in a variable, that value must only be *exactly one JSX element* not two or three or four side by side adjacent elements. Due to which, you have to wrap a `<div>` around all the elements in the return statement, this technically make the return statement to return only one value as a whole.

> Note: This doesn't have to be a `<div>` but can also be ANY element like a custom component as a wrapper.

However with a wrapping `<div>` or generally any wrapping element, a new problem arises, which is called *div soup*.

Example:

```jsx
<html>
..
  <div id="component-placeholder">
    <div class="DeadSimpleComponent">
            <div class="DeadSimpleComponent__time">10:23:12</div>
            <div class="DeadSimpleComponent__date">MONDAY, 2 MARCH 2015</div>
    </div>
</div>
..
</html>
```

Now of course it's not a good practice even the things are working fine with this approach. You're rendering too many `HTML` elements and ultimately that also makes your application *slower*.

## Using `Refs` and `Portals` to write imperative code WHEN necessary

