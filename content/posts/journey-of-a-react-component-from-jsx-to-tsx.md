---
title: "Journey of a React App From Javscript to Typescript"
date: 2023-03-18T10:47:57-04:00
draft: false
tags:
  - JavaScript
  - React
  - TypeScript
---

### Handling Props in Child and Parent Component

Whenever we define a parent and a child component in *React* using *Typescript*, we **must always define an Interface** in the child component which defines what props child expects to recieve. 

By adding an interface to the child component, we are adding two big checks:

 - **In Parent Component:** Are we providing the correct props to Child when we add it ti Parent? 
 - **In Child Component:** Are we using the correctly named + typed props in Child? 

**Example:** 

Child.tsx:

```jsx
interface ChildProps {
    color: string
}

export const Child = ({ color }: ChildProps){
    return <div>{color}</div>
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

### Making Typescript understand that we are creating a React Component

The handling of props may seem sufficient, but this does not provide a complete picture of Typescript. Even though TypeScript forces the parent component to honor the prop name and type, it does not inform the Typescript compiler that we are defining a React component.

**Compilers need to understand that this is indeed a React Component** and not just another method returning JSX.

**Why is it important to inform the compiler explicitly that we are defining a React component?**

Basically, *each React Component has some default properties associated with it*, and these properties can only be leveraged if the compiler is aware of them. In the event that the compiler does not know that the method you are defining is a React component property, it will error out when you attempt to use any of the default React component properties, such as `displayName`.

**Example:**

```jsx
interface ChildProps {
    color: string
}

export const ChildAsFC: react.FC<ChildProps> = ({ color }: ChildProps){
    return <div>{color}</div>
}
```
> In the example above,  `react.FC<ChildProps>` tells the Typescript Compiler, that we are defining a *React Function Component*, which accepts props of type `ChildProps`.

> Another upside of making the Compiler aware of the React Component is that it allows you to use *Children* prop, that gets passed to a component when you define it as an opening and closing tags. 

### Type Inference with State in Typescript

Consider the case in which we use the `useState` hook to define a state variable in a component. As we know, the `useState` hook takes in a **default value**. A Typescript compiler uses this default value to determine the type of a state variable. 

Consequently, in the example below, the compiler infers that the type of `name` is a `string`, since the default value is an empty string:

```jsx
const [name, setName] = useState('')
```

However, if we were to initialize a state variable that was of a complex type, such as an `Array` of type `String`, then Typecript would not be able to infer the type of array unless we explicitly define it. This can be accomplished by passing in the type of array as generic. For example:

```jsx
const [guests, setGuests] = useState<string[]>([])
```