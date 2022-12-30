---
title: "Using Composition to Contain Children Components"
date: 2022-12-30T13:52:04-05:00
draft: false
tags:
  - JavaScript
  - React
---

Some components don’t know their children ahead of time. This is especially common for components like _Sidebar_ or _Dialog_ that represent generic “boxes”.

For such components, we can use the special _children_ prop to pass children elements directly into their output:

```javascript
function FancyBorder(props) {
  return (
    <div className={'FancyBorder FancyBorder-' + props.color}>
      {props.children}
    </div>
  );
}
```

This lets other components pass arbitrary children to them by nesting the JSX:

```javascript
function WelcomeDialog() {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        Welcome
      </h1>
      <p className="Dialog-message">
        Thank you for visiting our spacecraft!
      </p>
    </FancyBorder>
  );
}
```