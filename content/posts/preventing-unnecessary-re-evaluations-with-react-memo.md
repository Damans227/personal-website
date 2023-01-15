---
title: "Preventing Unnecessary Re-Evaluations With React.memo()"
date: 2023-01-15T14:03:03-05:00
draft: false
tags:
  - JavaScript
  - React
---
## What is `React.memo()` ? 

Using `memo` will cause React to skip rendering a component if its props have not changed.
This can improve performance.

## When to use `React.memo()` ? 

We can use `React.memo` if React component:

- Will always render the same thing given the same props (i.e, if we have to make a network call to fetch some data and there’s a chance the data might not be the same, do not use it).

- Renders expensively (i.e, it takes at least 100ms to render).

- Renders often.

If our component does not meet the above requirements, then we might not need to memoize it as using React.memo, in some cases, make performance worse as it’s more code for the client to parse.

## Example: 

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

