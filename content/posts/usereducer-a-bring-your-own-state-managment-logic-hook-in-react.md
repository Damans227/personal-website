---
title: "useReducer - A Bring Your Own State Managment Logic Hook in React"
date: 2023-01-17T08:16:52-05:00
draft: false
tags:
  - JavaScript
  - React
---

In addition to the `useState` Hook, there is the `useReducer` Hook in React. It allows for the creation of custom state logic.

`useReducer` may be useful if you have to track multiple pieces of state involving complex logic.

The `useReducer` Hook accepts two arguments.

## Syntax: 

```jsx
useReducer(<reducer>, <initialState>)
```

The `reducer` function contains your custom state logic and the `initialState` can be a simple value but generally will contain an object.

The `useReducer` Hook returns the current `state` and a `dispatch` method.

Here is an example of `useReducer` in a counter app:

```jsx
import { useReducer } from "react";
import ReactDOM from "react-dom/client";

const initialTodos = [
  {
    id: 1,
    title: "Todo 1",
    complete: false,
  },
  {
    id: 2,
    title: "Todo 2",
    complete: false,
  },
];

const reducer = (state, action) => {
  switch (action.type) {
    case "COMPLETE":
      return state.map((todo) => {
        if (todo.id === action.id) {
          return { ...todo, complete: !todo.complete };
        } else {
          return todo;
        }
      });
    default:
      return state;
  }
};

function Todos() {
  const [todos, dispatch] = useReducer(reducer, initialTodos);

  const handleComplete = (todo) => {
    dispatch({ type: "COMPLETE", id: todo.id });
  };

  return (
    <>
      {todos.map((todo) => (
        <div key={todo.id}>
          <label>
            <input
              type="checkbox"
              checked={todo.complete}
              onChange={() => handleComplete(todo)}
            />
            {todo.title}
          </label>
        </div>
      ))}
    </>
  );
}

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<Todos />);
```

