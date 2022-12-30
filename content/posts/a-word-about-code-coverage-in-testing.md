---
title: "A Word About Code Coverage in Testing"
date: 2022-12-28T12:22:58-05:00
draft: false
tags:
  - JavaScript
  - Testing
---

An important aspect of testing is to achieve *good code coverage*. This means, that you want to write tests for the majority of your code (both code files and line of code).

There are tools that help you measure your code coverage - Testing library, _Vitest_ comes with a built-in functionality: https://vitest.dev/guide/features.html#coverage

> It is worth noting though, that the goal is not necessarily 100% coverage. There always can be some code that doesn't need any tests (e.g., because it merely calls other functions that are tested already).

In addition, achieving (close to) full code coverage also isn't any guarantee that you wrote good tests. You could cover 100% of your code with meaningless tests after all. Or you could be missing important tests (that should test important behaviors). The code would still technically be covered by tests in such scenarios.

_So don't see a high amount of code coverage as the ultimate goal!_