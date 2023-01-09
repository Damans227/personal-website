---
title: "Type Annotation vs Type Inference vs Type Assertion in Typescript"
date: 2023-01-09T17:51:17-05:00
draft: false
tags:
  - JavaScript
  - TypeScript
---

## Type Annotations

The following example declares variables with different data types, where the type is specified using `:Type` syntax after the name of the variable, parameter or property. :

```typescript
var age: number = 32; // number variable
var name: string = "John";// string variable
var isUpdated: boolean = true;// Boolean variable

function display(id:number, name:string)
{
    console.log("Id = " + id + ", Name = " + name);
}

var employee : { 
    id: number; 
    name: string; 
}; 

employee = { 
  id: 100, 
  name : "John"
}
```

These are type annotations. You cannot change the value using a different data type other than the declared data type of a variable. If you try to do so, TypeScript compiler will show an error.

## Type Inference 

Although _TypeScript_ is a typed language. However, it is not mandatory to specify the type of a variable. TypeScript infers types of variables when there is no explicit information available in the form of type annotations.

Types are inferred by TypeScript compiler when:

- Variables are initialized

- Default values are set for parameters

- Function return types are determined

For example,

```typescript
var a = "some text"
```

Here, since we are not explicitly defining `a: string` with a type annotation, TypeScript infers the type of the variable based on the value assigned to the variable. The value of a is a string and hence the type of a is inferred as string.

Consider the following example:

```typescript
var a = "some text";
var b = 123;
a = b; // Compiler Error: Type 'number' is not assignable to type 'string'
```

The above code shows an error because while inferring types, TypeScript inferred the type of variable `a` as string and variable `b` as number. When we try to assign `b` to `a`, the compiler complains saying a number type cannot be assigned to a string type.

## Type Assertion 

Type assertion allows you to set the type of a value and tell the compiler not to infer it. This is when you, as a programmer, might have a better understanding of the type of a variable than what TypeScript can infer on its own. 

Example:

```typescript
let code: any = 123; 
let employeeCode = <number> code; 
console.log(typeof(employeeCode)); //Output: number
```

In the above example, we have a variable `code` of type `any`. We assign the value of this variable to another variable called `employeeCode`. However, we know that `code` is of type number, even though it has been declared as 'any'. So, while assigning `code` to `employeeCode`, we have asserted that code is of type number in this case, and we are certain about it. Now, the type of `employeeCode` is number.