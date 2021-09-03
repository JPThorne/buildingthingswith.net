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

Scenario: we want to build a component that can tell us if there is a problem in an environment in which a temperature sensor is placed. If the temperature in this environment is consistently too high or too low, human intervention may be required.

To solve for this, we will build a small console app that can read in the following values: `deviceId`, `temperature`, `dateTime`. This app will check some business rules to say whether the inputs are problematic, and if so, compare the inputs with existing stored values. If sufficient data exists (based on another busines rule) the app will indicate that there is a problem. 

### Prerequisites

You need to have installed some version of the [.NET SDK](https://dotnet.microsoft.com/download), in this case we are using `net5.0`.

### Creating initial solution

As an example, the following commands can be used to create a simple solution (or just manually do it via Visual Studio (VS)). 

```
dotnet new sln -n temperature-alert-tdd
dotnet new console -n TemperatureAlert.CLI -o ./src/TemperatureAlert.CLI
dotnet new classlib -n TemperatureAlert.Domain -o ./src/TemperatureAlert.Domain
dotnet new classlib -n TemperatureAlert.Domain.Abstractions -o ./src/TemperatureAlert.Domain.Abstractions
dotnet new classlib -n TemperatureAlert.Repository -o ./src/TemperatureAlert.Repository
dotnet new xunit -n TemperatureAlert.Domain.Tests -o ./tests/TemperatureAlert.Domain.Tests

dotnet sln add ./src/TemperatureAlert.CLI/TemperatureAlert.CLI.csproj
dotnet sln add ./src/TemperatureAlert.Domain/TemperatureAlert.Domain.csproj
dotnet sln add ./src/TemperatureAlert.Domain.Abstractions/TemperatureAlert.Domain.Abstractions.csproj
dotnet sln add ./src/TemperatureAlert.Repository/TemperatureAlert.Repository.csproj
dotnet sln add ./tests/TemperatureAlert.Domain.Tests/TemperatureAlert.Domain.Tests.csproj
```

Now we are able to run commands such as (or just run via VS)

```
dotnet test
```
```
dotnet run --project ./src/TemperatureAlert.CLI/TemperatureAlert.CLI.csproj
```
```
cd \src\TemperatureAlert.CLI\bin\Debug\net5.0 && TemperatureAlert.CLI.exe
```

Now, we could leave this part till later and only add project references when the test requires it, but, I know I'm going to need them and want to make sure the expected solution structure is adhered to so I'll add them now. When done you should have something that looks a little like the below image.

![Initial Solution Structure](/images/tdd/initial-solution-structure.png)

## Pros, Cons & Considerations of TDD

It must be remembered that a unit-test does not constitute an integration or end-to-end test, so of course we cannot assume our applcation is working as intended without testing these things as well. However, if our business logic or algorithm can be 100% encoded without external dependencies, then we can say this component of the application is working as intended. We should be mindful that, depending on the feature, it can be difficult to define a set of test-cases that completely cover everything we actually want out of that feature. 

TDD can aid us in not writing code that we don't need, e.g. adhering to the principle of [YAGNI](https://martinfowler.com/bliki/Yagni.html)

## References

- [Wikipedia on TDD](https://en.wikipedia.org/wiki/Test-driven_development)

[Home]({{ site.url }})
