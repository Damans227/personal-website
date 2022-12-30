---
title: "Updating State That Depends on Previous State Using Callback"
date: 2022-12-30T13:45:31-05:00
draft: false
tags:
  - JavaScript
  - React
---

When the new state is calculated using the previous state, you can update the state with a callback:

```javascript
const [state, setState] = useState(initialState);
// changes state to `newState` and triggers re-rendering
setState(prevState => nextState);
// after re-render `state` becomes `newState`
```

Some more examples: 

```javascript
// Toggle a boolean
const [toggled, setToggled] = useState(false);
setToggled(toggled => !toggled);
// Increase a counter
const [count, setCount] = useState(0);
setCount(count => count + 1);
// Add an item to array
const [items, setItems] = useState([]);
setItems(items => [...items, 'New Item']);
```