---
title: "Tracing a Function Call in Javascript Using Console"
date: 2023-01-03T14:38:04-05:00
draft: false
tags:
  - JavaScript
---

In `JavaScript` we're often working in the context of deeply nested functions and objects. When debugging our code, it may be necessary to traverse through the stack trace of our code. 

In order to do that, we can use `console.trace()` in the function that we would expect to be at the _top of the call stack_. Using `console.trace()` we can see exactly what happened before our function was pushed on to the top of the call stack. 

For Example: 

```javascript
getClusterControls (items = [], featureContentGetter) {
    console.trace("who called upon me?")

    const filteredItems = items.reduce((filteredItems, item) => {
      if (item.attributes.neType === NodeTypes.AGGREGATED_NETWORK_ELEMENT) {
       ...
      }
      return controls
    }
  }
```

Will produce the following output in the console of the browser running our code: 

```javascript

map.js:493 who called upon me?

getClusterControls	@	map.js:493
getPodListControls	@	map.js:562
getControlsForPod	@	map.js:375
(anonymous)	@	component.js:73

```
