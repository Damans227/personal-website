---
title: "Exploring the Core Redux Concepts"
date: 2023-01-26T20:20:10-05:00
draft: false
tags:
  - JavaScript
  - React
  - Redux
---

So, let's try to explore the **core redux concepts** by creating a simple counter application.

## Redux Store

First thing we need is a **redux store**. Let's create one.

```javascript
const store = redux.createdStore();
```

## Reducer

This store is responsible to hold our data. The data it holds is determined by the **reducer**. The reducer has one job, i.e. to spit out a new snapshot of the store when an action occurs. So, the next thing we want to add is a reducer function.

> Reducer function is always a pure function. That means, same input paramasters must always produce the same output and there should be no side effects inside of that function.

```javascript
const initialState = { counter: 0 };
const counterReducer = (state = initialState, action) => {
  return {
    counter: state.counter + 1,
  };
};
```

## Subscriber

Now, we have a store and a reducer. We need someone who will **subscribe** to this store for data.

```javascript
const counterSubscriber = () => {
  const latestState = store.getState(); // gives the latest snapshot of the store after it has changed
  console.log({ latestState });
};
```

We should make redux aware of our subscriber so whenever the state changes, the subscriber function is executed. We do that by calling subscribe method on the store.

```javascript
store.subscribe(counterSubscriber);
```

## Action

Now, we have a subscriber, reducer and the store. What we are missing is **action**.

```javascript
store.dispatch({ type: "increment" });
```

Since, we are dispatching an action of type increment, let's update our reducer to use that action type.

```javascript
const counterReducer = (state = initialState, action) => {
  if ((action.type === "increment")) {
    return {
      counter: state.counter + 1,
    };
  }
  return state;
};
```
