---
title: "UseEffect Usecase: Run Side Effect Only Once After Initial Render"
date: 2023-01-10T12:08:17-05:00
draft: false
tags:
  - JavaScript
  - React
---

## Side Effect Runs Only Once After Initial Render

You may want to run the side effect just once after the initial render. A typical case will be fetching data making an API call, and storing the response in the state variable after the initial render. You do not want to make this API call again.

You can pass an empty array as the second argument to the `useEffect` hook to tackle this use case.

```jsx
useEffect(() => {
  // Side Effect
}, []);
```