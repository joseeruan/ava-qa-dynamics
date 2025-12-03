# Dynamics 365 Testing Best Practices

## Core Principles

### 1. Test Pyramid for Dynamics
```
           /\
          /E2E\        <- Few (Slow, Fragile, Expensive)
         /------\
        / Integ  \     <- Some (Medium Speed, Realistic)
       /----------\
      /   Unit     \   <- Many (Fast, Isolated, Reliable)
     /--------------\
```

**Distribution Recommendation:**
- **70% Unit Tests** - Fast, isolated, test business logic
- **20% Integration Tests** - Test multi-artifact flows
- **10% E2E Tests** - Critical user journeys only

---

## Unit Testing Best Practices

### ✅ DO: Test Business Logic in Isolation
```csharp
// Good: Tests pure business logic
[Fact]
public void CalculateDiscount_HighValueAccount_Returns20Percent()
{
    var calculator = new DiscountCalculator();
    var discount = calculator.Calculate(revenue: 100000);
    
    Assert.Equal(0.20m, discount);
}
```

### ❌ DON'T: Test Dataverse Operations Directly
```csharp
// Bad: Requires real Dataverse connection
[Fact]
public void SaveAccount_ToRealDataverse_Works()
{
    var service = GetRealOrganizationService(); // ❌
    var account = new Entity("account");
    service.Create(account); // ❌ Slow, environment-dependent
}
```

---

### ✅ DO: Mock IOrganizationService
```csharp
[Fact]
public void Plugin_CreatesContact_WhenAccountCreated()
{
    var context = new XrmFakedContext();
    var service = context.GetOrganizationService();
    
    // Test with mocked service
    var plugin = new CreateContactPlugin();
    plugin.Execute(context);
    
    var contacts = service.RetrieveMultiple(new QueryExpression("contact"));
    Assert.Single(contacts.Entities);
}
```

---

### ✅ DO: Use Descriptive Test Names
```csharp
// Good: Clear what's being tested
[Fact]
public void AccountPlugin_OnCreate_WithMissingName_ThrowsException()

// Bad: Unclear intent
[Fact]
public void TestPlugin()
```

**Naming Pattern:** `MethodUnderTest_Scenario_ExpectedBehavior`

---

### ✅ DO: Test Edge Cases
```csharp
[Theory]
[InlineData(null)]           // Null input
[InlineData("")]             // Empty string
[InlineData("   ")]          // Whitespace
[InlineData("VeryLongStringThatExceeds100Characters...")] // Boundary
public void ValidateName_InvalidInput_ThrowsException(string name)
{
    var validator = new NameValidator();
    Assert.Throws<ArgumentException>(() => validator.Validate(name));
}
```

---

### ✅ DO: Keep Tests Independent
```csharp
// Good: Each test has its own data
[Fact]
public void Test1()
{
    var context = new XrmFakedContext(); // Fresh context
    // Test logic
}

[Fact]
public void Test2()
{
    var context = new XrmFakedContext(); // Independent context
    // Test logic
}
```

### ❌ DON'T: Share State Between Tests
```csharp
// Bad: Shared static state
private static XrmFakedContext _sharedContext; // ❌

[Fact]
public void Test1()
{
    _sharedContext.Initialize(...); // ❌ Affects Test2
}

[Fact]
public void Test2()
{
    // Test might fail if Test1 runs first
}
```

---

## Plugin Testing Best Practices

### ✅ DO: Test Each Message Type Separately
```csharp
[Fact]
public void Plugin_OnCreate_SetsDefaults() { }

[Fact]
public void Plugin_OnUpdate_RecalculatesFields() { }

[Fact]
public void Plugin_OnDelete_ValidatesReferences() { }
```

---

### ✅ DO: Test Each Stage Separately
```csharp
[Fact]
public void Plugin_PreValidation_ValidatesInput() { }

[Fact]
public void Plugin_PreOperation_EnrichesData() { }

[Fact]
public void Plugin_PostOperation_CreatesRelatedRecords() { }
```

---

### ✅ DO: Always Mock PreImage/PostImage Explicitly
```csharp
[Fact]
public void Plugin_OnUpdate_UsesPreImage()
{
    var pluginContext = context.GetDefaultPluginContext();
    
    // Explicit PreImage mock
    var preImage = new Entity("account")
    {
        Id = accountId,
        ["revenue"] = new Money(10000)
    };
    pluginContext.PreEntityImages.Add("PreImage", preImage);
    
    // Now test can safely use PreImage
}
```

---

### ✅ DO: Test Depth Validation
```csharp
[Fact]
public void Plugin_WithDepthGreaterThan1_ExitsEarly()
{
    var pluginContext = context.GetDefaultPluginContext();
    pluginContext.Depth = 2; // Simulate recursive call
    
    context.ExecutePluginWith(pluginContext, new MyPlugin());
    
    // Assert plugin exited without executing logic
    var tracingService = context.GetTracingService();
    Assert.Contains("Depth exceeded", tracingService.DumpTrace());
}
```

---

### ✅ DO: Test Exception Messages
```csharp
[Fact]
public void Plugin_InvalidInput_ThrowsDescriptiveException()
{
    var ex = Assert.Throws<InvalidPluginExecutionException>(() =>
    {
        plugin.Execute(context);
    });
    
    Assert.Equal("Account name is required and cannot be empty", ex.Message);
    // ✅ User gets helpful error message
}
```

---

## Integration Testing Best Practices

### ✅ DO: Test Multi-Artifact Flows
```csharp
[Fact]
public void IntegrationTest_CreateAccount_ExecutesPluginAndWorkflow()
{
    // Arrange: Setup multiple plugins
    var preOpPlugin = new PreOperationPlugin();
    var postOpPlugin = new PostOperationPlugin();
    
    // Act: Simulate full pipeline
    var target = new Entity("account") { ["name"] = "Test" };
    
    // 1. PreOperation
    var preContext = CreatePluginContext(stage: 20, message: "Create");
    preContext.InputParameters["Target"] = target;
    context.ExecutePluginWith(preContext, preOpPlugin);
    
    // 2. Simulate save
    target.Id = Guid.NewGuid();
    context.Initialize(new[] { target });
    
    // 3. PostOperation
    var postContext = CreatePluginContext(stage: 40, message: "Create");
    postContext.InputParameters["Target"] = target;
    context.ExecutePluginWith(postContext, postOpPlugin);
    
    // Assert: Verify end-to-end result
    Assert.True(target.Contains("enrichedfield")); // From PreOp
    var relatedRecords = service.RetrieveMultiple(...);
    Assert.NotEmpty(relatedRecords.Entities); // From PostOp
}
```

---

### ✅ DO: Test Pipeline Conflicts
```csharp
[Fact]
public void IntegrationTest_TwoPluginsModifySameField_LastOneWins()
{
    var plugin1 = new SetDefaultsPlugin();
    var plugin2 = new OverridePlugin();
    
    // Both run in PreOperation, test execution order
    // Plugin1 sets field = "Default"
    // Plugin2 sets field = "Override"
    
    // Assert final value matches expected order
    Assert.Equal("Override", target["field"]);
}
```

---

### ✅ DO: Test PreImage/PostImage Dependencies
```csharp
[Fact]
public void IntegrationTest_UpdateFlow_ValidatesPreImageAccess()
{
    // PreOperation: Uses PreImage
    var preOpPlugin = new PreOperationPlugin();
    var preContext = CreatePluginContext(stage: 20);
    var preImage = new Entity("account") { ["revenue"] = new Money(10000) };
    preContext.PreEntityImages.Add("PreImage", preImage);
    
    context.ExecutePluginWith(preContext, preOpPlugin);
    
    // PostOperation: Uses PostImage
    var postOpPlugin = new PostOperationPlugin();
    var postContext = CreatePluginContext(stage: 40);
    var postImage = new Entity("account") { ["revenue"] = new Money(15000) };
    postContext.PostEntityImages.Add("PostImage", postImage);
    
    context.ExecutePluginWith(postContext, postOpPlugin);
    
    // Verify both stages worked correctly
}
```

---

## Code Review Best Practices

### ✅ DO: Review Test Coverage
- Every plugin should have unit tests
- Critical paths must have integration tests
- Edge cases must be covered

### ✅ DO: Review Test Quality
- Tests should be readable (like documentation)
- No magic numbers (use constants)
- Clear arrange-act-assert structure

### ❌ DON'T: Accept Tests That Always Pass
```csharp
// Bad: Test that doesn't validate anything
[Fact]
public void Test_Something()
{
    var result = DoSomething();
    // ❌ No assertion! Test always passes
}
```

---

## Performance Best Practices

### ✅ DO: Keep Unit Tests Fast (< 100ms)
```csharp
[Fact]
public void FastTest_ComplexLogic_ValidatesQuickly()
{
    var stopwatch = Stopwatch.StartNew();
    
    // Test logic
    
    stopwatch.Stop();
    Assert.True(stopwatch.ElapsedMilliseconds < 100, "Test too slow");
}
```

---

### ✅ DO: Use Minimal Test Data
```csharp
// Good: Only necessary fields
var account = new Entity("account")
{
    ["name"] = "Test"
};

// Bad: Unnecessary fields slow down test
var account = new Entity("account")
{
    ["name"] = "Test",
    ["field1"] = "...",
    ["field2"] = "...",
    // ... 50 more fields
};
```

---

### ✅ DO: Parallelize Independent Tests
```csharp
// XUnit: Parallel by default
// NUnit: Use [Parallelizable] attribute
[Parallelizable(ParallelScope.All)]
public class PluginTests { }
```

---

## Maintenance Best Practices

### ✅ DO: Update Tests When Code Changes
- Test is part of the feature, not an afterthought
- Broken tests = broken code or broken assumptions

### ✅ DO: Refactor Tests Like Production Code
- Extract common setup to helper methods
- Use fixtures for shared initialization
- Keep test code DRY (Don't Repeat Yourself)

---

### ✅ DO: Use Test Helpers
```csharp
public class TestHelpers
{
    public static Entity CreateAccount(string name, decimal revenue)
    {
        return new Entity("account")
        {
            Id = Guid.NewGuid(),
            ["name"] = name,
            ["revenue"] = new Money(revenue)
        };
    }
    
    public static XrmFakedPluginExecutionContext CreatePluginContext(
        int stage, 
        string message, 
        string entityName = "account")
    {
        var context = new XrmFakedPluginExecutionContext();
        context.Stage = stage;
        context.MessageName = message;
        context.PrimaryEntityName = entityName;
        return context;
    }
}
```

---

## Anti-Patterns to Avoid

### ❌ DON'T: Test Implementation Details
```csharp
// Bad: Tests how something is done, not what it does
[Fact]
public void Plugin_CallsServiceTwice()
{
    // Testing number of calls = fragile test
    // Refactor breaks test even if behavior is correct
}
```

### ❌ DON'T: Write Flaky Tests
```csharp
// Bad: Test sometimes passes, sometimes fails
[Fact]
public void FlakyTest()
{
    Thread.Sleep(1000); // ❌ Timing-dependent
    var result = GetAsyncResult(); // ❌ Race condition
    Assert.NotNull(result);
}
```

### ❌ DON'T: Test External Dependencies
```csharp
// Bad: Tests external API
[Fact]
public void Plugin_CallsExternalAPI_Works()
{
    var response = HttpClient.Get("https://external.com"); // ❌
    // Test will fail if API is down
}
```

**Fix:** Mock external dependencies

---

## CI/CD Integration

### ✅ DO: Run Tests on Every Commit
```yaml
# Azure Pipeline
- task: DotNetCoreCLI@2
  displayName: 'Run Unit Tests'
  inputs:
    command: 'test'
    projects: '**/*Tests.csproj'
```

### ✅ DO: Fail Build on Test Failures
- Broken tests = broken code
- Don't allow merge if tests fail

### ✅ DO: Track Test Coverage
```yaml
- task: DotNetCoreCLI@2
  inputs:
    command: 'test'
    arguments: '--collect:"XPlat Code Coverage"'
```

**Target:** Aim for 80%+ code coverage on critical business logic

---

## Summary Checklist

**Unit Tests:**
- ✅ Test business logic in isolation
- ✅ Mock IOrganizationService with FakeXrmEasy
- ✅ Descriptive test names
- ✅ Test edge cases (null, empty, boundary)
- ✅ Keep tests independent (no shared state)
- ✅ Fast execution (< 100ms)

**Plugin Tests:**
- ✅ Test each message type separately
- ✅ Test each stage separately
- ✅ Mock PreImage/PostImage explicitly
- ✅ Test depth validation
- ✅ Test exception messages

**Integration Tests:**
- ✅ Test multi-artifact flows
- ✅ Test pipeline conflicts
- ✅ Test PreImage/PostImage dependencies
- ✅ Realistic scenarios

**General:**
- ✅ 70% unit, 20% integration, 10% E2E
- ✅ Run tests on every commit
- ✅ Aim for 80%+ coverage on business logic
- ✅ Refactor tests like production code
- ✅ Update tests when code changes
