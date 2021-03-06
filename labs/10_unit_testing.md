# Lab 10 -  Unit testing

## Goal

The goal of this lab is to have our orchestrator function unit tested to gain confidence our business logic is implemented as expected. The logic in this case are statements in the orchestrator that controls when the `StoreProcessedNeoEventActivity` and the `SendNotificationActivity` are being called.

## Steps

### 1. Create a new unit test project

First add a new unit testing project (based on .NET Core 3.1) to the solution (e.g. `Demo.NEO.EventProcessing.UnitTests`). I'm an xUnit enthusiast, but we can use any C# unit testing framework.

### 2. Mocking library

We want to test the functionality of the orchestrator function. This function has a dependency on the `IDurableOrchestrationContext` interface. We need to mock the behavior of this context so the actual activity functions are not executed and interfering with the unit tests.

We can use any mocking framework we like, some nice ones I can recommend are Moq, NSubstitute or FakteItEasy.

### 3. Create the unit tests

I suggest to write unit tests where we verify if the `CallActivityWithRetryAsync` method for the `StoreProcessedNeoEventActivity` and the `SendNotificationActivity` activities have been called.

In order to do this we need to create the `IDurableOrchestrationContext` mock and setup the method calls which are performed in the orchestrator function. I suggest to configure a so-called strict mock behavior which means that all methods on the `IDurableOrchestrationContext` which are used should be specified.

Note that method calls to the orchestrator and activities are async. This means our setup code and unit test methods should be able to handle this.

The unit tests should look something like the following: 

```csharp
[Fact]
public async Task WhenTorinoImpactIsGreaterThan0_ThenStoreProcessedNeoEventActivityShouldBeCalled()
{
    // Arrange
    var contextMock = DurableOrchestrationContextBaseBuilder.BuildContextWithSpecificTorinoImpactGreaterThan0();
    var logger = new Mock<ILogger>();
    var orchestrator = new NeoEventProcessingOrchestrator();
    
    // Act
    var orchestratorResult = await orchestrator.Run(contextMock.Object, logger.Object);
    
    // Assert
    contextMock.VerifyAll();
}
```

Note that a new `DurableOrchestrationContextBaseBuilder` class is used which sets up the mock of the `IDurableOrchestrationContext`. In case we get stuck have a look at [my unit test code](../src/Demo.NEO.EventProcessing.UnitTests/TestBuilders/DurableOrchestrationContextBaseBuilder.cs) so see how to setup the mock.

Continue to the [next lab](11_fan-out-in.md) to call Activity Functions in parallel.
