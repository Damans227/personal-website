---
title: "What Is UseCallback Hook in React"
date: 2023-01-15T14:50:34-05:00
draft: false
tags:
  - JavaScript
  - React
---

The React `useCallback` Hook returns a memoized callback function. 

> Think of memoization as caching a value so that it does not need to be recalculated. This allows us to isolate resource intensive functions so that they will not automatically run on every render.

The `useCallback` Hook only runs when one of its dependencies update. This can improve performance.

## Why does this matter?

In JavaScript, functions display referential equality. This means they are only considered equal if they point to the same object.

Therefore if you were to declare two functions with identical implementations they would not be equal to each other.

For example:

```javascript
const firstFunction = function() {
    return 1 + 2; // same as secondFunction
}

const secondFunction = function() {
    return 1 + 2; // same as firstFunction
}

// Same implementation, different objects
firstFunction === secondFunction; // false

// Same objects!
firstFunction === firstFunction; // true
```

## When is this useful in React?

Typically `useCallback` is helpful when passing callback props to highly optimised child components.

For example, if a child component that accepts a callback function as a prop, relies on a referential equality check (eg: `React.memo()`). To prevent unnecessary re-renders when its props change, then it is important that any callback props do not change between renders.

To do this, the parent component can wrap the callback prop in `useCallback` and be guaranteed that it is passing the same function object down into the optimised child component.

```jsx
function ParentComponent() {
    const onHandleClick = useCallback(() => {
        // this will return the same function
        // instance between re-renders
    });

    return (
        <MemoizedSubComponent
            handleClick={onHandleClick}
        />
    );
}
```

> It’s worth noting that applying `useCallback` doesn’t memoize the *result of a function’s invocation*. Rather it memoizes the function object itself.
