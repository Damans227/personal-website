---
title: "Remember Data Down Action Up in React"
date: 2022-12-30T14:04:04-05:00
draft: false
tags:
  - JavaScript
  - React
---

## “Data down, Action up”

This phrase helps developers understand the flow of information from parent to child component and vice versa.

The first and more simple concept, _“data down,”_ refers to the passing of data and/or functions via props from parent to child components. These props are passed down when a child component gets created. We pass data down to child components so they can render them on to the DOM.

> Data is sent down to the child from the parent component with the help of props, and 
data is sent back up to the parent from the child component with the help of callback functions. 