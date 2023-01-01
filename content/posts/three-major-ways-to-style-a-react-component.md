---
title: "Three Major Ways to Style a React Component"
date: 2023-01-01T18:03:59-05:00
draft: false
tags:
  - JavaScript
  - React
---

## Inline CSS

This is the CSS styling sent to the element directly using the _HTML or JSX_. You can include a JavaScript object for CSS in React components, although there are a few restrictions such as camel casing any property names which contain a hyphen.

### Example: 

```javascript 
import React from "react";

const spanStyles = {
  color: "#fff",
  borderColor: "#00f"
};

const Button = props => (
  <button style={{
    color: "#fff",
    borderColor: "#00f"
  }}>
    <span style={spanStyles}>Button Name</span>
  </button>
);
```

## Styled-Components 

[Styled-components](https://styled-components.com/) is a library that utilizes _tagged template literals_ — which contain actual CSS code between two backticks — to style your components. 

Props can be used to style styled components in the same way that they are passed to normal React components. Props are used instead of classes in CSS and set the properties dynamically.

### Example:

```javascript
import React from "react";
import styled, { css } from "styled-components";

const Button = styled.button`
  cursor: pointer;
  background: transparent;
  font-size: 16px;
  border-radius: 3px;
  color: palevioletred;
  margin: 0 1em;
  padding: 0.25em 1em;
  transition: 0.5s all ease-out;
  ${props =>
    props.primary &&
    css`
      background-color: white;
      color: green;
      border-color: green;
    `};
`;

export default Button;
```

## CSS Modules

CSS Modules enable you to make sure that all of the styles for a component are in one single place and apply only to that particular component. Their composition feature acts as a weapon to represent shared styles between states in your application. They are similar to mixins in Sass, which makes it possible to combine multiple groups of styles.

### Example: 

```javascript
import React from "react";
import style from "./panel.css";

const Panel = () => (
  <div className={style.panelDefault}>
    <div className={style.panelBody}>A Basic Panel</div>
  </div>
);

export default Panel;
```

```javascript
.panelDefault {
  border-color: #ddd;
}
.panelBody {
  padding: 15px;
}
```