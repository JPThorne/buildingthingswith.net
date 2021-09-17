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

- understand the requirement we need to fulfill, its test-cases etc.
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

```sh
NOTE: Of course there would normally be a large amount of test-cases, user-stories, acceptance-criteria etc. for something like this,
and we would naturally fit a component like this into a larger system, 
however, here we are going to just focus on the core of the program.
```

- For a deviceId, if the temperature reading lies outside a range of acceptable values then store this anomaly in a record.
- For a deviceId, if the temperature is not abnormal, no record should be created and no alert should be triggered.
- For a deviceId, if `x` number of anomalous records exist, for a time range `y`, create an alert.

Some business rules:

- We define "number of anomalous records `x` that should trigger an alert" as say, 5.
- Time range `y` as 10 minutes.

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

A few things to note here: we are using a pattern called "arrange/act/assert" to give structure to our tests. Not all test will have an arrange step necessarily and so we may clean that comment up later on. I've created a constructor `TemperatureServiceTests` because it's a habit and I know I'll need it later on.

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

This is the pattern we are going to follow throughout our development here:

1. Test-case
2. Write unit-test
3. Write the code to pass the test.

Okay, next up, our 1st "real" test. Given our initial test criteria, we can call this test something along the lines of `Test_AnomalyIsRecordedIfTemperatureIsAbnormal`. We want to assert that some record was created if the given value matches a certain criteria.

Here's my 1st attempt at this test:

```c#
[Fact]
public async Task Test_AnomalyIsRecordedIfTemperatureIsAbnormal()
{
    //arrange
    var deviceId = "1";
    var abnormalTemperature = 45.534234m;

    var normalMinTemperature = 10m;
    var normalMaxTemperature = 35m;

    var repository = Substitute.For<ITemperatureRepository>();

    repository.GetNormalTemperatureRange(deviceId).Returns(new TemperatureRule
    {
        MinTemperature = normalMinTemperature,
        MaxTemperature = normalMaxTemperature
    });

    var service = new TemperatureService(repository);

    //act
    var result = await service.AnalyzeTemperature(deviceId, abnormalTemperature);

    //assert
    Assert.NotNull(result);
    Assert.Equal("Abnormal", result.Status);
    Assert.Equal($"{abnormalTemperature} was higher than allowed maximum: {normalMaxTemperature}", result.Message);
    await repository.Received(1).GetNormalTemperatureRange(deviceId);
    await repository.Received(1).RecordTemperatureAnomaly(deviceId, abnormalTemperature);
}
```

My thinking is as follows

- We need some datastore to load business rules (mins and maxes) & to store values. For now that can be the same store, i.e. some `IRepository`.
- I want to work with decimals for temperatures.
- We want to scope the analysis to the `deviceId`, as different id's may have different rules, as specified in the test-case.
- It would be nice if the analysis could return some information to us about how it went, hence the `result`.

Of course, now our code doesn't compile. The implementation from this point is almost "automatic" using VS - we can `alt-enter` on our missing codes and VS will be able to create a lot of it for us. Of course, it's not perfect & doesn't know exactly what we want, so we need to specify locations of files, some return types etc. The resulting output to get this to compile looks like the following:

```c#
namespace TemperatureAlert.Domain
{
    public class AnalysisResult
    {
        public string Status { get; set; }
        public string Message { get; set; }
    }
}
```

```c#
namespace TemperatureAlert.Domain
{
    public class TemperatureRule
    {
        public decimal MinTemperature { get; set; }
        public decimal MaxTemperature { get; set; }
    }
}
```

```c#
using System.Threading.Tasks;

namespace TemperatureAlert.Domain
{
    public interface ITemperatureRepository
    {
        Task<TemperatureRule> GetNormalTemperatureRange(string deviceId);
        Task RecordTemperatureAnomaly(string deviceId, decimal abnormalTemperature);
    }
}
```

We also added `NSubstitute` to our test project for easy mocking as well as defined a default `RootNamespace` for our `Abstractions` project.

```xml
<PackageReference Include="NSubstitute" Version="4.2.2" />
```

```xml
<RootNamespace>TemperatureAlert.Domain</RootNamespace>
```

Finally, our `TemperatureService` itself needed updating. We now expect a constructor which takes an `ITemperatureRepository` and an initial implementation of `AnalyzeTemperature`:

```c#
public TemperatureService(ITemperatureRepository repository)
{
}
```

```c#
public Task<AnalysisResult> AnalyzeTemperature(string deviceId, decimal abnormalTemperature)
{
    throw new NotImplementedException();
}
```

Naturally, our test output is now 1 pass and 1 fail, with the fail reading something along the lines of `System.NotImplementedException : The method or operation is not implemented.`. So let's fix that.

If we now want to make the test pass, this code will achieve that.

```c#
using System.Threading.Tasks;

namespace TemperatureAlert.Domain
{
    public class TemperatureService
    {
        private ITemperatureRepository Repository { get; init;}

        public TemperatureService(ITemperatureRepository repository)
        {
            Repository = repository;
        }

        public async Task<AnalysisResult> AnalyzeTemperature(string deviceId, decimal temperature)
        {
            var temperatureRule = await Repository.GetNormalTemperatureRange(deviceId);

            await Repository.RecordTemperatureAnomaly(deviceId, temperature);

            return new AnalysisResult
            {
                Status = "Abnormal",
                Message = "45,534234 was higher than allowed maximum: 35"
            };
        }
    }
}
```

Running `dotnet test` outputs

```sh
Starting test execution, please wait...
A total of 1 test files matched the specified pattern.

Passed!  - Failed:     0, Passed:     2, Skipped:     0, Total:     2, Duration: 3 ms - TemperatureAlert.Domain.Tests.dll (net5.0)
```

The tests now pass! I see nothing wrong here :-)

Of course, I am joking, clearly we have just fudged the data into the implementation to "make it pass" and this will not work in a real world scenario. But it does serve to highlight a useful point - the quality of our assertions in our tests matter a lot. Also, more tests are needed and we are not done yet.

Next up we aim to test for `Test_IfTemperatureIsNormal_NoAnomalyIsRecorded`. The test for that can look something like this:

```c#
[Fact]
public async Task Test_IfTemperatureIsNormal_NoAnomalyIsRecorded()
{
    //arrange
    var deviceId = "1";
    var normalTemperature = 25.123m;
    var dateTime = DateTime.UtcNow;

    var normalMinTemperature = 10m;
    var normalMaxTemperature = 35m;

    var repository = Substitute.For<ITemperatureRepository>();

    repository.GetNormalTemperatureRange(deviceId).Returns(new TemperatureRule
    {
        MinTemperature = normalMinTemperature,
        MaxTemperature = normalMaxTemperature
    });

    var service = new TemperatureService(repository);

    //act
    var result = await service.AnalyzeTemperature(deviceId, normalTemperature, dateTime);

    //assert
    Assert.NotNull(result);
    Assert.Equal("Normal", result.Status);
    Assert.Equal($"{normalTemperature} was OK.", result.Message);
    await repository.Received(1).GetNormalTemperatureRange(deviceId);
    await repository.Received(0).RecordTemperatureAnomaly(Arg.Any<string>(), Arg.Any<decimal>());
}
```

And of course, this test does not pass. The reason is clear - our implementation is always just returning "Abnormal" results regardless of the input. Let's fix that.

```c#
public async Task<AnalysisResult> AnalyzeTemperature(string deviceId, decimal temperature, DateTime dateTime)
{
    var temperatureRule = await Repository.GetNormalTemperatureRange(deviceId);

    if (temperature < temperatureRule.MinTemperature || temperature > temperatureRule.MaxTemperature)
    {
        await Repository.RecordTemperatureAnomaly(deviceId, temperature);

        return new AnalysisResult
        {
            Status = "Abnormal",
            Message = "45,534234 was higher than allowed maximum: 35"
        };
    }
    else
    {
        return new AnalysisResult
        {
            Status = "Normal",
            Message = $"25,123 was OK."
        };
    }
}
```

We're still carrying around the hardcoded values, because we aren't done and we didn't have to remove them yet - the tests pass.

Next up, we can address the scenario I am calling `Test_XOrMoreAnomaliesForTimeRangeY_ShouldTriggerAlert`. This is for test-case "For a deviceId, if `x` number of anomalous records exist, for a time range `y`, create an alert". The test for this looks like the following:

```c#
[Theory]
[InlineData(5, 10)]
[InlineData(6, 10)]
public async Task Test_XOrMoreAnomaliesForTimeRangeY_ShouldTriggerAlert(int numberOfAnomalies, int numberOfMinutes)
{
    //arrange
    var deviceId = "1";
    var abnormalTemperature = 42m;
    var now = DateTime.UtcNow;

    var normalMinTemperature = 10m;
    var normalMaxTemperature = 35m;

    var repository = Substitute.For<ITemperatureRepository>();

    repository.GetNormalTemperatureRange(deviceId).Returns(new TemperatureRule
    {
        MinTemperature = normalMinTemperature,
        MaxTemperature = normalMaxTemperature,
        MaximumMinutes = 10,
        MaximumNumberOfAnomalies = 5
    });

    var nowMinusXMinutes = now.AddMinutes(-numberOfMinutes);

    repository.GetTemperatureAnomalyCount(deviceId, nowMinusXMinutes).Returns(numberOfAnomalies);

    var service = new TemperatureService(repository, AlertService);

    //act
    var result = await service.AnalyzeTemperature(deviceId, abnormalTemperature, now);

    //assert
    Assert.NotNull(result);
    Assert.Equal("Abnormal", result.Status);
    Assert.Equal($"{abnormalTemperature} was higher than allowed maximum: {normalMaxTemperature}", result.Message);
    await repository.Received(1).GetNormalTemperatureRange(deviceId);
    await repository.Received(1).RecordTemperatureAnomaly(deviceId, abnormalTemperature);
    await repository.Received(1).GetTemperatureAnomalyCount(deviceId, nowMinusXMinutes);
    await AlertService.Received(1).SendTemperatureAlert(deviceId, abnormalTemperature, now);
}
```

There are a few things to note. We've defined a new `IAlertService` which, as the name suggests, would be used to send alerts as needed. Naturally this service needs to be injected into the constructor here and to fix the other tests. This could be approached in another way - the result from the `TemperatureService` could be used to trigger and alert or not and this could be actioned in some higher-up object, but for this exercise I'm just leaving it here.

Of course, the test fails currently. We now need to address some hard-coding and lacking logic. To do that we can add the following:

```c#
var numberOfAnomalies = await Repository.GetTemperatureAnomalyCount(deviceId, dateTime.AddMinutes(-temperatureRule.MaximumMinutes));

if (numberOfAnomalies >= temperatureRule.MaximumNumberOfAnomalies)
{
    await AlertService.SendTemperatureAlert(deviceId, temperature, dateTime);
}
```

We also need to change some hard-coding to satisfy the test.

```c#
"45,534234 was higher than allowed maximum: 35"
```

to this

```c#
$"{temperature} was higher than allowed maximum: 35"
```

You can also see that we've added some additional props to `TemperatureRule` and a new method to the repo.

Ok, so we've dealt with 5 or more, now lets deal with 4 or less, as per our current business rules. The implementation for that is surprisingly simple, in fact, we happen to have already coded for this in our previous implementation. We can add another test, `Test_XOrLessAnomaliesForTimeRangeY_ShouldNotTriggerAlert` and the only difference between it and the previous test is of course the test data, and the assertion for the `AlertService`. So we have: 

```c#
[Theory]
[InlineData(4, 10)]
[InlineData(3, 10)]
public async Task Test_XOrLessAnomaliesForTimeRangeY_ShouldNotTriggerAlert(int numberOfAnomalies, int numberOfMinutes)
{
    //arrange
    var deviceId = "1";
    var abnormalTemperature = 42m;
    var now = DateTime.UtcNow;

    var normalMinTemperature = 10m;
    var normalMaxTemperature = 35m;

    var repository = Substitute.For<ITemperatureRepository>();

    repository.GetNormalTemperatureRange(deviceId).Returns(new TemperatureRule
    {
        MinTemperature = normalMinTemperature,
        MaxTemperature = normalMaxTemperature,
        MaximumMinutes = 10,
        MaximumNumberOfAnomalies = 5
    });

    var nowMinusXMinutes = now.AddMinutes(-numberOfMinutes);

    repository.GetTemperatureAnomalyCount(deviceId, nowMinusXMinutes).Returns(numberOfAnomalies);

    var service = new TemperatureService(repository, AlertService);

    //act
    var result = await service.AnalyzeTemperature(deviceId, abnormalTemperature, now);

    //assert
    Assert.NotNull(result);
    Assert.Equal("Abnormal", result.Status);
    Assert.Equal($"{abnormalTemperature} was higher than allowed maximum: {normalMaxTemperature}", result.Message);
    await repository.Received(1).GetNormalTemperatureRange(deviceId);
    await repository.Received(1).RecordTemperatureAnomaly(deviceId, abnormalTemperature);
    await repository.Received(1).GetTemperatureAnomalyCount(deviceId, nowMinusXMinutes);
    await AlertService.Received(0).SendTemperatureAlert(deviceId, abnormalTemperature, now);
}
```

The only differences are the `[InlineData]`, method name of course, and the assertion that the alert service was *not* called.

```c#
await AlertService.Received(0).SendTemperatureAlert(deviceId, abnormalTemperature, now);
```

It looks like we've addressed all our test-cases, so are we done? Kind of, but, if we read our implementation we can see that one of our tests could use some more specific data. Our `TemperatureService` is still hard-coding the normal result:

```c#
return new AnalysisResult
{
    Status = "Normal",
    Message = $"25,123 was OK."
};
```

We can fix that by updating `Test_IfTemperatureIsNormal_NoAnomalyIsRecorded` to use `[InlineData]`. Now our test becomes:

```c#
[Theory]
[InlineData(10.1)]
[InlineData(25.123)]
[InlineData(34.99999)]
public async Task Test_IfTemperatureIsNormal_NoAnomalyIsRecorded(decimal normalTemperature)
{
    //arrange
    var deviceId = "1";
    var dateTime = DateTime.UtcNow;

    var normalMinTemperature = 10m;
    var normalMaxTemperature = 35m;

    var repository = Substitute.For<ITemperatureRepository>();

    repository.GetNormalTemperatureRange(deviceId).Returns(new TemperatureRule
    {
        MinTemperature = normalMinTemperature,
        MaxTemperature = normalMaxTemperature
    });

    var service = new TemperatureService(repository, AlertService);

    //act
    var result = await service.AnalyzeTemperature(deviceId, normalTemperature, dateTime);

    //assert
    Assert.NotNull(result);
    Assert.Equal("Normal", result.Status);
    Assert.Equal($"{normalTemperature} was OK.", result.Message);
    await repository.Received(1).GetNormalTemperatureRange(deviceId);
    await repository.Received(0).RecordTemperatureAnomaly(Arg.Any<string>(), Arg.Any<decimal>());
}
```

And the fix is to change the message to use the incoming value: `Message = $"{temperature} was OK."`

Ok, one last thing! We are still hard-coding `35` here: `$"{temperature} was higher than allowed maximum: 35"`. Let's fix that, we an update any of the tests that assert that result to have a different max and then we will prove a failure. Let's update `Test_AnomalyIsRecordedIfTemperatureIsAbnormal` to have `var normalMaxTemperature = 40m;`.

We can now fix our implementation by changing:

```c#
Message = $"{temperature} was higher than allowed maximum: 35"
```

to

```c#
Message = $"{temperature} was higher than allowed maximum: {temperatureRule.MaxTemperature}"
```

### Possible next steps for the solution

Now that we have somewhat of a decent implementation, covered by some unit-tests, we can do a few things.

- Refactor to increase the separation of concerns. Arguably the `TemperatureService` is doing too many things, thereby violating `SRP`.
- Refactor to reduce the number of `Repository` reads. Arguably the solution is inefficient in how it reads from the datastore.
- The tests are using a pattern of `[Fact]`'s or `[Theory]` with `[InlineData]`, there are other ways to structure our test of course, we could use something like `AutoFixture`, we could also look at shifting our data out of the test using `InlineData` members etc.
- I have also not actually implemented a set of tests to check that `deviceId` variations are considered. E.g., if records exist for a `deviceId` `A`, but none for `deviceId` `B` then readings for `B` should not be impacted by existing values for `A`. The implementation does use device ids, but it would be good to test that.

## Pros, Cons & Considerations of TDD

It must be remembered that a unit-test does not constitute an integration or end-to-end test, so of course we cannot assume our application is working as intended without testing these things as well. However, if our business logic or algorithm can be 100% encoded without external dependencies, then we can say this component of the application is working as intended. We should be mindful that, depending on the feature, it can be difficult to define a set of test-cases that completely cover everything we actually want out of that feature.

TDD can aid us in not writing code that we don't need, e.g. adhering to the principle of [YAGNI](https://martinfowler.com/bliki/Yagni.html)

## Conclusion

TDD is a powerful technique for enabling development to align with testing. As with all things programming, one should always remember to take the context into account, however, it is a tool we can use frequently for many coding tasks. The completed sample we created here can be viewed on [Github](https://github.com/JPThorne/demos-temperature-alert-tdd/tree/feat/completed-tdd-exercise)

## References

- [Wikipedia on TDD](https://en.wikipedia.org/wiki/Test-driven_development)

[Home]({{ site.url }})
