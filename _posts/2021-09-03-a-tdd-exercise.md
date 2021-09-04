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

```sh
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

```sh
dotnet test
```

```sh
dotnet run --project ./src/TemperatureAlert.CLI/TemperatureAlert.CLI.csproj
```

```sh
cd .\src\TemperatureAlert.CLI\bin\Debug\net5.0 && TemperatureAlert.CLI.exe
```

Outputting the default

```sh
Hello World!
```

Now, we could leave this part till later and only add project references when the test requires it, but, I know I'm going to need them and want to make sure the expected solution structure is adhered to so I'll add them now. When done you should have something that looks like the below image. I've also gone ahead and removed all the boilerplate `Class1.cs`, `UnitTest1.cs` classes.

![Initial Solution Structure](/images/tdd/initial-solution-structure.png)

### Test-cases

We need a few test-cases if we are going to write some tests. Here are a few to get started.

```text
NOTE: Of course there would normally be a large amount of test cases, user-stories, acceptance-criteria etc. for something like this and we would naturally fit a component like this into a larger system, however, here we are going to just focus on the core of the program here.
```

- For a deviceId, if the temperature reading lies outside a range of acceptable values then store this anomaly in a record.
- For a deviceId, if (x) number of anomalous records exist, for a time range (y), create an alert.

Some business rules:

- We define "number of anomalous records (x) that should trigger an alert" as say, 5.
- Time range (y) as 10 minutes.

We can then write some more specific test-cases, such as:

- If 4 or less anomalies exist for the past 10 minutes, no alert should be triggered.
- If 5 or more anomalies exist for the past 10 minutes, an alert should be triggered.

### Let's write some code

Ok, time to write our 1st test! Now, where to put it, and what shall we test? Given that I know I want to use a convention of putting classes that solve business logic problems into our `Domain`, and that these classes will be called `XService`, where X relates to the object or domain, we can create our 1st test class called `TemperatureServiceTests`.

A common starting point is to test that we "Can Construct" the object we intend to test, so lets start there. Our 1st test then is `Test_CanConstruct`. The test will be as follows:

```c#
using Xunit;

namespace TemperatureAlert.Domain.Tests
{
    public class TemperatureServiceTests
    {
        public TemperatureServiceTests() { }

        [Fact]
        public void Test_CanConstruct()
        {
            //arrange

            //act
            var service = new TemperatureService();

            //assert
            Assert.NotNull(service);
        }
    }
}
```

Our tests folder now looks like this:

![TemperatureServiceTests.cs](/images/tdd/temperature-service-tests.png)

A few things to note here: we are using a pattern called "arrange/act/assert" to give structure to our tests. Not all test will have an arrange step necessarily and so we may clean that comment up later on. I've created a ctor `TemperatureServiceTests` because it's a habit and I know I'll need it later on.

Of course our code now doesn't compile, because we haven't created the `TemperatureService` class, so let's do that next. As mentioned earlier, we want to put our `Service` classes in our `Domain` & so this class goes in the project `TemperatureAlert.Domain`. Here it is, and now our code compiles and our test passes.

```c#
namespace TemperatureAlert.Domain
{
    public class TemperatureService
    {
        public TemperatureService()
        {
        }
    }
}
```

This is the pattern we are going to follow throughout our development here. 1. Test case, 2. Write test, 3. Write the code to pass the test.

## Pros, Cons & Considerations of TDD

It must be remembered that a unit-test does not constitute an integration or end-to-end test, so of course we cannot assume our applcation is working as intended without testing these things as well. However, if our business logic or algorithm can be 100% encoded without external dependencies, then we can say this component of the application is working as intended. We should be mindful that, depending on the feature, it can be difficult to define a set of test-cases that completely cover everything we actually want out of that feature.

TDD can aid us in not writing code that we don't need, e.g. adhering to the principle of [YAGNI](https://martinfowler.com/bliki/Yagni.html)

## Conclusion

TDD is a powerful technique for enabling development to align with testing. As with all things programming, one should always remember to take the context into account, however, it is a tool we can use frequently for many coding tasks. The completed sample we created here can be viewed on [Github](https://github.com/JPThorne/demos-temperature-alert-tdd/tree/feat/completed-tdd-exercise)

## References

- [Wikipedia on TDD](https://en.wikipedia.org/wiki/Test-driven_development)

[Home]({{ site.url }})
