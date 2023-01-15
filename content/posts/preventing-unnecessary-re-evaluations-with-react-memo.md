---
title: "Preventing Unnecessary Re-Evaluations With React.memo()"
date: 2023-01-15T14:03:03-05:00
draft: false
tags:
  - JavaScript
  - React
---

Using `memo` will cause React to skip rendering a component if its props have not changed.
This can improve performance.

Following code sample uses `memo` to keep the `Todos` component from needlessly re-rendering by 
wrapping the `Todos` component export in `memo`:

**Todos.js**

```jsx
import { memo } from "react";

const Todos = ({ todos }) => {
  console.log("child render");
  return (
    <>
      <h2>My Todos</h2>
      {todos.map((todo, index) => {
        return <p key={index}>{todo}</p>;
      })}
    </>
  );
};

export default memo(Todos);
```

> It's important to note that `React.memo` only checks for prop changes. If a function component wrapped in `React.memo` has a `useState`, `useReducer` or `useContext` Hook in its implementation, it will still re-render when state or context change.

