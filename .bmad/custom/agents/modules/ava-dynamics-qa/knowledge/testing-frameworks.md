# Testing Frameworks for .NET / Dynamics 365

## Framework Comparison

### XUnit (⭐ Recommended)
**Pros:**
- ✅ Modern, actively maintained
- ✅ Best .NET Core/.NET 5+ support
- ✅ Parallel test execution by default
- ✅ Clean, attribute-based syntax
- ✅ Strong community and tooling
- ✅ Used by Microsoft .NET team

**Cons:**
- ⚠️ Less mature than NUnit (but catching up)

**Best For:** New projects, .NET 5+, modern Dynamics development

**Example:**
```csharp
using Xunit;

public class AccountPluginTests
{
    [Fact]
    public void Create_SetsDefaultValues()
    {
        // Arrange
        var plugin = new AccountPlugin();
        
        // Act
        plugin.Execute(serviceProvider);
        
        // Assert
        Assert.Equal("Expected", actual);
    }

    [Theory]
    [InlineData("Test1")]
    [InlineData("Test2")]
    public void Create_WithDifferentNames_Works(string name)
    {
        // Test with multiple inputs
    }
}
```

---

### NUnit (Classic Choice)
**Pros:**
- ✅ Very mature, battle-tested
- ✅ Rich assertion library
- ✅ Great IDE support
- ✅ Extensive documentation
- ✅ Large community

**Cons:**
- ⚠️ Slightly older paradigm
- ⚠️ Not parallel by default (requires config)

**Best For:** Legacy projects, teams familiar with NUnit, enterprise environments

**Example:**
```csharp
using NUnit.Framework;

[TestFixture]
public class AccountPluginTests
{
    [Test]
    public void Create_SetsDefaultValues()
    {
        // Arrange
        var plugin = new AccountPlugin();
        
        // Act
        plugin.Execute(serviceProvider);
        
        // Assert
        Assert.That(actual, Is.EqualTo("Expected"));
    }

    [TestCase("Test1")]
    [TestCase("Test2")]
    public void Create_WithDifferentNames_Works(string name)
    {
        // Test with multiple inputs
    }
}
```

---

### MSTest (Microsoft Native)
**Pros:**
- ✅ Built-in Visual Studio support
- ✅ Microsoft-backed
- ✅ Simple setup
- ✅ Good for beginners

**Cons:**
- ⚠️ Less flexible than XUnit/NUnit
- ⚠️ Fewer advanced features
- ⚠️ Smaller community

**Best For:** Teams already using Microsoft stack exclusively, simple test scenarios

**Example:**
```csharp
using Microsoft.VisualStudio.TestTools.UnitTesting;

[TestClass]
public class AccountPluginTests
{
    [TestMethod]
    public void Create_SetsDefaultValues()
    {
        // Arrange
        var plugin = new AccountPlugin();
        
        // Act
        plugin.Execute(serviceProvider);
        
        // Assert
        Assert.AreEqual("Expected", actual);
    }

    [DataTestMethod]
    [DataRow("Test1")]
    [DataRow("Test2")]
    public void Create_WithDifferentNames_Works(string name)
    {
        // Test with multiple inputs
    }
}
```

---

## Feature Comparison Table

| Feature | XUnit | NUnit | MSTest |
|---------|-------|-------|--------|
| **Parallel Execution** | ✅ Default | ⚠️ Requires config | ⚠️ Requires config |
| **.NET Core Support** | ✅ Excellent | ✅ Good | ✅ Good |
| **IDE Integration** | ✅ VS, Rider | ✅ VS, Rider | ✅ VS (native) |
| **Assertion Library** | ✅ Simple | ✅ Rich (Assert.That) | ⚠️ Basic |
| **Parameterized Tests** | `[Theory]` | `[TestCase]` | `[DataTestMethod]` |
| **Setup/Teardown** | Constructor/Dispose | `[SetUp]`/`[TearDown]` | `[TestInitialize]`/`[TestCleanup]` |
| **Community** | ✅ Large | ✅ Very Large | ⚠️ Smaller |
| **Learning Curve** | ⚠️ Modern paradigm | ✅ Easy | ✅ Very Easy |

---

## Recommendation Decision Tree

```
Are you starting a NEW project?
├─ YES → Use XUnit ⭐
└─ NO → Do you have existing tests?
    ├─ NUnit → Keep NUnit (consistency)
    ├─ MSTest → Consider migrating to XUnit
    └─ None → Use XUnit ⭐

Does your team prefer Microsoft-native tools?
├─ YES → MSTest is acceptable
└─ NO → XUnit or NUnit

Do you need rich assertion syntax?
├─ YES → NUnit (Assert.That is powerful)
└─ NO → XUnit (simple is better)

Is parallel execution critical?
├─ YES → XUnit (default parallel)
└─ NO → Any framework works
```

---

## Common Test Patterns (Framework-Agnostic)

### Pattern 1: Arrange-Act-Assert (AAA)
```csharp
[Fact] // or [Test] or [TestMethod]
public void Plugin_ValidInput_SuccessfulExecution()
{
    // Arrange: Setup test data
    var context = CreateContext();
    var plugin = new MyPlugin();

    // Act: Execute code under test
    plugin.Execute(context);

    // Assert: Verify results
    Assert.Equal(expected, actual);
}
```

---

### Pattern 2: Setup and Teardown

**XUnit:**
```csharp
public class PluginTests : IDisposable
{
    private readonly XrmFakedContext _context;

    public PluginTests() // Setup
    {
        _context = new XrmFakedContext();
    }

    public void Dispose() // Teardown
    {
        // Cleanup
    }
}
```

**NUnit:**
```csharp
[TestFixture]
public class PluginTests
{
    private XrmFakedContext _context;

    [SetUp]
    public void Setup()
    {
        _context = new XrmFakedContext();
    }

    [TearDown]
    public void Teardown()
    {
        // Cleanup
    }
}
```

**MSTest:**
```csharp
[TestClass]
public class PluginTests
{
    private XrmFakedContext _context;

    [TestInitialize]
    public void Setup()
    {
        _context = new XrmFakedContext();
    }

    [TestCleanup]
    public void Teardown()
    {
        // Cleanup
    }
}
```

---

### Pattern 3: Parameterized Tests

**XUnit:**
```csharp
[Theory]
[InlineData(10, 5, 15)]
[InlineData(20, 10, 30)]
public void Add_TwoNumbers_ReturnsSum(int a, int b, int expected)
{
    var result = Calculator.Add(a, b);
    Assert.Equal(expected, result);
}
```

**NUnit:**
```csharp
[TestCase(10, 5, 15)]
[TestCase(20, 10, 30)]
public void Add_TwoNumbers_ReturnsSum(int a, int b, int expected)
{
    var result = Calculator.Add(a, b);
    Assert.That(result, Is.EqualTo(expected));
}
```

**MSTest:**
```csharp
[DataTestMethod]
[DataRow(10, 5, 15)]
[DataRow(20, 10, 30)]
public void Add_TwoNumbers_ReturnsSum(int a, int b, int expected)
{
    var result = Calculator.Add(a, b);
    Assert.AreEqual(expected, result);
}
```

---

## Assertion Syntax Comparison

### Equality Check
```csharp
// XUnit
Assert.Equal(expected, actual);

// NUnit
Assert.That(actual, Is.EqualTo(expected));

// MSTest
Assert.AreEqual(expected, actual);
```

### Null Check
```csharp
// XUnit
Assert.NotNull(obj);

// NUnit
Assert.That(obj, Is.Not.Null);

// MSTest
Assert.IsNotNull(obj);
```

### Exception Check
```csharp
// XUnit
Assert.Throws<InvalidPluginExecutionException>(() => plugin.Execute(context));

// NUnit
Assert.Throws<InvalidPluginExecutionException>(() => plugin.Execute(context));

// MSTest
Assert.ThrowsException<InvalidPluginExecutionException>(() => plugin.Execute(context));
```

### Collection Check
```csharp
// XUnit
Assert.Contains(item, collection);
Assert.Empty(collection);

// NUnit
Assert.That(collection, Contains.Item(item));
Assert.That(collection, Is.Empty);

// MSTest
CollectionAssert.Contains(collection, item);
Assert.AreEqual(0, collection.Count);
```

---

## Integration with CI/CD

### All Three Support:
- ✅ Azure DevOps Pipelines
- ✅ GitHub Actions
- ✅ Jenkins
- ✅ TeamCity

### Example: Azure Pipeline
```yaml
- task: DotNetCoreCLI@2
  inputs:
    command: 'test'
    projects: '**/*Tests.csproj'
    testRunTitle: 'Unit Tests'
```

---

## Best Practices (All Frameworks)

1. ✅ **One assertion per test** (easier to identify failures)
2. ✅ **Use descriptive test names** (`MethodName_Scenario_ExpectedResult`)
3. ✅ **Keep tests independent** (no shared state)
4. ✅ **Mock external dependencies** (use FakeXrmEasy for Dynamics)
5. ✅ **Test edge cases** (null, empty, boundary values)
6. ✅ **Use parameterized tests** (reduce duplication)
7. ✅ **Keep tests fast** (unit tests < 100ms ideal)
8. ✅ **Organize tests logically** (mirror source structure)

---

## AVA Module Recommendation

**Default Choice: XUnit**

**Reasons:**
- Modern, future-proof
- Best alignment with .NET 5+ and Dynamics cloud development
- Parallel execution = faster test runs
- Simple, clean syntax
- Microsoft .NET team uses it

**Alternative Support:**
Allow users to choose during installation (config prompt) for teams with existing preferences.

**Installation Packages:**
```xml
<!-- XUnit -->
<PackageReference Include="xunit" Version="2.5.0" />
<PackageReference Include="xunit.runner.visualstudio" Version="2.5.0" />

<!-- NUnit -->
<PackageReference Include="NUnit" Version="3.14.0" />
<PackageReference Include="NUnit3TestAdapter" Version="4.5.0" />

<!-- MSTest -->
<PackageReference Include="MSTest.TestFramework" Version="3.1.1" />
<PackageReference Include="MSTest.TestAdapter" Version="3.1.1" />
```
