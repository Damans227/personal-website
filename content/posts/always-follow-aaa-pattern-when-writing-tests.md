---
title: "Always Follow AAA Pattern When Writing Tests"
date: 2022-12-27T16:41:40-05:00
draft: false
tags:
  - JavaScript
  - Testing
---

Following is a very simple unit test which verifies if the `add()` function works correctly. While the test works, but it does not follow the **AAA(Arrange, Act, Assert)** pattern.

```javascript
import {it, expect} from 'vitest';
import { add } from './math';

it('should summarize all number values in an array', ()=>{
    const result = add([1,2,3])
    expect(result).toBe(6)
});
```
> *Question: What is AAA (Arrange, Act, Assert) pattern in testing?*
>
> *AAA pattern allows you to write unit tests which are more readable and maintainable in long term.*

- Arrange Phase: is where you define the testing enviornment and values. 
- Act phase: is where you run the actual code / function that should be tested.
- Assert Phase: is where you evaluate the produced value. 

Example of the test above when formatted using AAA pattern: 

```javascript
import { it, expect } from 'vitest';
import { add } from './math';

it('should summarize all number values in an array', () => {
    //Arrange
    const numbers = [1, 2];
    const expectedValue = numbers.reduce((prevValue, curValue) => prevValue + curValue, 0)

    //Act
    const result = add(numbers)

    //Assert
    expect(result).toBe(expectedValue)
});

```