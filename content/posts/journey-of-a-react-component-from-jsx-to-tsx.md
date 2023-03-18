---
title: "Journey of a React App From Jsx to Tsx: Handling Props"
date: 2023-03-18T10:47:57-04:00
draft: false
tags:
  - JavaScript
  - React
  - TypeScript
---

Whenever we define a parent and a child component in *React* using *Typescript*, we
also define an interface in the child component which defines what props child expects to recieve. 

By adding an interface to the child component, we are adding two big checks:

 - **In Parent Component:** Are we providing the correct props to Child when we add it ti Parent? 
 - **In Child Component:** Are we using the correctly named + typed props in Child? 

Example: 

Child.tsx:

```jsx
interface ChildProps {
    color: string
}

export const Child = ({ props }: ChildProps){
    return <div> Hi There! </div>
}
```

Parent.tsx:

```jsx
import { Child } from './Child'

const parent = () -> {
    return <Child color="red"/>
}

export default Parent
```