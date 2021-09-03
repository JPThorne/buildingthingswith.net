---
layout: default
title: "A TDD exercise"
date: 2021-09-03
categories: tdd
tags: tdd
# permalink: /a-tdd-exercise.html
---

## What is TDD?

Test Driven Development (TDD) is a way of coding whereby we write our tests before we write our implementation class.

## Process

In more detail, the process of using TDD is to:

- understand the requirement we need to fulfill, it's test-cases etc.
- write a failing test (because the code for it to pass does not yet exist or is wrong).
- write the code that makes the test pass (this code does not necessarily have to be super-neat and performant).
- if needed, once the tests all pass, refactor to improve the code-quality.

In this way we only write code that is inherently tested, and testable, & so our code-coverage is 100% & our test-cases are addressed, as best they can be in a unit-test context.

## Example exercise

Scenario: we want to build a component that can tell us if there is a problem in an environment in which a temperature-sensor is placed. If the temperature in this environment is consistently too high or too low, human intervention may be required. 

## Pros, Cons & Considerations of TDD

It must be remembered that a unit-test does not constitute an integration or end-to-end test, so of course we cannot assume our applcation is working as intended without testing these things as well. However, if our business logic or algorithm can be 100% encoded without external dependencies, then we can say this component of the application is working as intended. We should be mindful that, depending on the feature, it can be difficult to define a set of test-cases that completely cover everything we actually want out of that feature. 

TDD can aid us in not writing code that we don't need, e.g. adhering to the principle of [YAGNI](https://martinfowler.com/bliki/Yagni.html)

## References

- [Wikipedia on TDD](https://en.wikipedia.org/wiki/Test-driven_development)

[Home]({{ site.url }})
