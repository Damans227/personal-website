---
title: "What Are Side Effects in React and How to Handle Them Effectively"
date: 2023-01-10T11:48:27-05:00
draft: false
tags:
  - JavaScript
  - React
---

In essence, a _react component_ consists of a function that executes from top to bottom and handles bringing something onto the screen or responding to user input, such as clicking. Whether you use state or other functions in such a function, everything in a component is concerned with bringing something to the screen. 

A side effect is anything else that occurs within an application. For example, sending an HTTP request or storing something in the browser's storage. These are the tasks that are not responsible for displaying anything on the screen. Not directly, at least. Therefore, such tasks should occur outside of the normal component evaluation and rendering cycle - especially since they might cause rendering to be delayed. It will impact performance if we do not do this and send an HTTP request every time a component is re-rendered (for example, every time a state changes). 

Furthermore, if an http request results in a state change that causes the component to re-render, then the component will go into an infinite loop.

The best way to handle such situations in react components is to use the `useEffect` hook. 

> Hooks are a new addition in React 16.8. They let you use state and other React features without writing a class.

The `useEffect` Hook lets you perform side effects in function components:

```jsx
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // Similar to componentDidMount and componentDidUpdate:
  useEffect(() => {
    // Update the document title using the browser API
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```