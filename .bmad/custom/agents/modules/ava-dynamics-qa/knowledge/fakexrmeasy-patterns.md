# FakeXrmEasy Testing Patterns

## Overview

FakeXrmEasy é a biblioteca mais popular para mockar o Dynamics 365 SDK em testes unitários. Permite simular `IOrganizationService`, `IPluginExecutionContext`, e operações do Dataverse sem ambiente real.

---

## Setup Básico

### 1. Instalação
```xml
<PackageReference Include="FakeXrmEasy.Core" Version="4.x" />
<PackageReference Include="FakeXrmEasy.Plugins" Version="4.x" />
```

### 2. Estrutura Base de Teste
```csharp
using FakeXrmEasy;
using FakeXrmEasy.Plugins;
using Microsoft.Xrm.Sdk;
using Xunit;

public class PluginTests
{
    private readonly XrmFakedContext _context;
    private readonly IOrganizationService _service;

    public PluginTests()
    {
        _context = new XrmFakedContext();
        _service = _context.GetOrganizationService();
    }

    [Fact]
    public void TestPlugin()
    {
        // Arrange
        var account = new Entity("account")
        {
            Id = Guid.NewGuid(),
            ["name"] = "Test Account"
        };
        _context.Initialize(new[] { account });

        // Act
        var plugin = new MyPlugin();
        var pluginContext = _context.GetDefaultPluginContext();
        pluginContext.MessageName = "Update";
        pluginContext.Stage = 20; // PreOperation
        
        _context.ExecutePluginWith(pluginContext, plugin);

        // Assert
        var updated = _service.Retrieve("account", account.Id, new ColumnSet(true));
        Assert.Equal("Expected Value", updated["fieldname"]);
    }
}
```

---

## Common Patterns

### Pattern 1: Mocking IOrganizationService
```csharp
// Initialize with test data
var contact = new Entity("contact")
{
    Id = Guid.NewGuid(),
    ["firstname"] = "John",
    ["lastname"] = "Doe"
};

_context.Initialize(new[] { contact });

// Use service in test
var retrieved = _service.Retrieve("contact", contact.Id, new ColumnSet("firstname"));
Assert.Equal("John", retrieved["firstname"]);
```

---

### Pattern 2: Mocking Plugin Context
```csharp
var pluginContext = _context.GetDefaultPluginContext();
pluginContext.MessageName = "Create";
pluginContext.Stage = 20; // PreOperation
pluginContext.PrimaryEntityName = "account";

var target = new Entity("account")
{
    ["name"] = "New Account"
};
pluginContext.InputParameters["Target"] = target;

_context.ExecutePluginWith(pluginContext, new MyPlugin());
```

---

### Pattern 3: Mocking PreImage
```csharp
var preImage = new Entity("account")
{
    Id = accountId,
    ["name"] = "Old Name",
    ["revenue"] = new Money(10000)
};

var pluginContext = _context.GetDefaultPluginContext();
pluginContext.MessageName = "Update";
pluginContext.PreEntityImages.Add("PreImage", preImage);

var target = new Entity("account")
{
    Id = accountId,
    ["name"] = "New Name" // Only changed field
};
pluginContext.InputParameters["Target"] = target;

_context.ExecutePluginWith(pluginContext, new MyPlugin());
```

---

### Pattern 4: Mocking PostImage
```csharp
var postImage = new Entity("account")
{
    Id = accountId,
    ["name"] = "Updated Name",
    ["revenue"] = new Money(15000)
};

var pluginContext = _context.GetDefaultPluginContext();
pluginContext.MessageName = "Update";
pluginContext.Stage = 40; // PostOperation
pluginContext.PostEntityImages.Add("PostImage", postImage);

_context.ExecutePluginWith(pluginContext, new MyPlugin());
```

---

### Pattern 5: Testing Create Operation
```csharp
[Fact]
public void Plugin_OnAccountCreate_SetsDefaultValues()
{
    // Arrange
    var plugin = new AccountPlugin();
    var pluginContext = _context.GetDefaultPluginContext();
    pluginContext.MessageName = "Create";
    pluginContext.Stage = 20;
    
    var target = new Entity("account")
    {
        ["name"] = "Test Account"
    };
    pluginContext.InputParameters["Target"] = target;

    // Act
    _context.ExecutePluginWith(pluginContext, plugin);

    // Assert
    Assert.True(target.Contains("customfield"));
    Assert.Equal("Default Value", target["customfield"]);
}
```

---

### Pattern 6: Testing Update Operation
```csharp
[Fact]
public void Plugin_OnAccountUpdate_CalculatesRevenue()
{
    // Arrange
    var accountId = Guid.NewGuid();
    var preImage = new Entity("account")
    {
        Id = accountId,
        ["revenue"] = new Money(10000)
    };

    var pluginContext = _context.GetDefaultPluginContext();
    pluginContext.MessageName = "Update";
    pluginContext.Stage = 20;
    pluginContext.PreEntityImages.Add("PreImage", preImage);

    var target = new Entity("account")
    {
        Id = accountId,
        ["revenue"] = new Money(15000)
    };
    pluginContext.InputParameters["Target"] = target;

    // Act
    _context.ExecutePluginWith(pluginContext, new AccountPlugin());

    // Assert
    Assert.Equal(15000m, ((Money)target["revenue"]).Value);
}
```

---

### Pattern 7: Testing Delete Operation
```csharp
[Fact]
public void Plugin_OnAccountDelete_PreventsDeletion()
{
    // Arrange
    var accountId = Guid.NewGuid();
    var account = new Entity("account")
    {
        Id = accountId,
        ["name"] = "Protected Account",
        ["customflag"] = true
    };
    _context.Initialize(new[] { account });

    var pluginContext = _context.GetDefaultPluginContext();
    pluginContext.MessageName = "Delete";
    pluginContext.Stage = 10; // PreValidation
    pluginContext.InputParameters["Target"] = new EntityReference("account", accountId);
    pluginContext.PreEntityImages.Add("PreImage", account);

    // Act & Assert
    var ex = Assert.Throws<InvalidPluginExecutionException>(() =>
    {
        _context.ExecutePluginWith(pluginContext, new AccountPlugin());
    });
    
    Assert.Contains("Cannot delete protected account", ex.Message);
}
```

---

### Pattern 8: Testing Related Entities
```csharp
[Fact]
public void Plugin_CreatesRelatedContact()
{
    // Arrange
    var accountId = Guid.NewGuid();
    var account = new Entity("account")
    {
        Id = accountId,
        ["name"] = "Test Account"
    };
    _context.Initialize(new[] { account });

    var pluginContext = _context.GetDefaultPluginContext();
    pluginContext.MessageName = "Create";
    pluginContext.Stage = 40; // PostOperation
    pluginContext.InputParameters["Target"] = account;

    // Act
    _context.ExecutePluginWith(pluginContext, new CreateContactPlugin());

    // Assert
    var contacts = _service.RetrieveMultiple(
        new QueryExpression("contact")
        {
            ColumnSet = new ColumnSet("parentcustomerid"),
            Criteria = new FilterExpression
            {
                Conditions =
                {
                    new ConditionExpression("parentcustomerid", ConditionOperator.Equal, accountId)
                }
            }
        }
    );
    
    Assert.Single(contacts.Entities);
}
```

---

### Pattern 9: Testing ITracingService
```csharp
[Fact]
public void Plugin_LogsExecutionSteps()
{
    // Arrange
    var tracingService = _context.GetTracingService();
    var pluginContext = _context.GetDefaultPluginContext();
    pluginContext.MessageName = "Create";

    var target = new Entity("account") { ["name"] = "Test" };
    pluginContext.InputParameters["Target"] = target;

    // Act
    _context.ExecutePluginWith(pluginContext, new MyPlugin());

    // Assert
    var traceLog = tracingService.DumpTrace();
    Assert.Contains("Plugin execution started", traceLog);
    Assert.Contains("Validation completed", traceLog);
}
```

---

### Pattern 10: Testing Exception Handling
```csharp
[Fact]
public void Plugin_OnInvalidData_ThrowsException()
{
    // Arrange
    var pluginContext = _context.GetDefaultPluginContext();
    pluginContext.MessageName = "Create";

    var target = new Entity("account"); // Missing required field
    pluginContext.InputParameters["Target"] = target;

    // Act & Assert
    var ex = Assert.Throws<InvalidPluginExecutionException>(() =>
    {
        _context.ExecutePluginWith(pluginContext, new AccountPlugin());
    });
    
    Assert.Equal("Account name is required", ex.Message);
}
```

---

## Integration Test Pattern

### Simulating Multi-Plugin Flow
```csharp
[Fact]
public void IntegrationTest_CreateAccount_ExecutesMultiplePlugins()
{
    // Arrange: Initialize context with related data
    var parentAccount = new Entity("account")
    {
        Id = Guid.NewGuid(),
        ["name"] = "Parent Account"
    };
    _context.Initialize(new[] { parentAccount });

    // Plugin 1: PreOperation - Set defaults
    var plugin1Context = _context.GetDefaultPluginContext();
    plugin1Context.MessageName = "Create";
    plugin1Context.Stage = 20;
    
    var target = new Entity("account")
    {
        ["name"] = "Child Account",
        ["parentaccountid"] = new EntityReference("account", parentAccount.Id)
    };
    plugin1Context.InputParameters["Target"] = target;

    // Act: Execute PreOperation plugin
    _context.ExecutePluginWith(plugin1Context, new PreOperationPlugin());
    
    // Simulate MainOperation: Save to context
    target.Id = Guid.NewGuid();
    _context.Initialize(new[] { parentAccount, target });

    // Plugin 2: PostOperation - Create related records
    var plugin2Context = _context.GetDefaultPluginContext();
    plugin2Context.MessageName = "Create";
    plugin2Context.Stage = 40;
    plugin2Context.InputParameters["Target"] = target;

    _context.ExecutePluginWith(plugin2Context, new PostOperationPlugin());

    // Assert: Verify entire flow result
    var createdAccount = _service.Retrieve("account", target.Id, new ColumnSet(true));
    Assert.NotNull(createdAccount["defaultfield"]); // Set by PreOperation
    
    var relatedContacts = _service.RetrieveMultiple(
        new QueryExpression("contact")
        {
            Criteria = new FilterExpression
            {
                Conditions =
                {
                    new ConditionExpression("parentcustomerid", ConditionOperator.Equal, target.Id)
                }
            }
        }
    );
    Assert.Single(relatedContacts.Entities); // Created by PostOperation
}
```

---

## Advanced Patterns

### Pattern 11: Mocking Custom APIs
```csharp
[Fact]
public void TestCustomAPI()
{
    var request = new OrganizationRequest("new_CustomAction");
    request["InputParameter"] = "TestValue";

    var response = _service.Execute(request);
    
    Assert.NotNull(response["OutputParameter"]);
}
```

---

### Pattern 12: Testing Query Performance
```csharp
[Fact]
public void Plugin_UsesEfficientQuery()
{
    // Arrange: Initialize large dataset
    var accounts = Enumerable.Range(1, 10000)
        .Select(i => new Entity("account")
        {
            Id = Guid.NewGuid(),
            ["name"] = $"Account {i}"
        });
    _context.Initialize(accounts);

    var pluginContext = _context.GetDefaultPluginContext();
    
    // Act & Assert: Validate query uses paging
    var stopwatch = Stopwatch.StartNew();
    _context.ExecutePluginWith(pluginContext, new MyPlugin());
    stopwatch.Stop();
    
    Assert.True(stopwatch.ElapsedMilliseconds < 1000, "Query took too long, may not be using paging");
}
```

---

## Best Practices

1. ✅ **Initialize context with minimal data** (only what's needed)
2. ✅ **Use ColumnSet to specify fields** (realistic queries)
3. ✅ **Test each message type separately** (Create/Update/Delete)
4. ✅ **Mock PreImage/PostImage explicitly** (don't assume)
5. ✅ **Test exception scenarios** (validation, null checks)
6. ✅ **Verify trace logs** (ensure debugging info present)
7. ✅ **Test related entity operations** (integration scenarios)
8. ✅ **Validate depth handling** (prevent infinite loops)

---

## Common Pitfalls

- ❌ Not initializing context with required data
- ❌ Forgetting to set MessageName/Stage
- ❌ Assuming PreImage exists without mocking
- ❌ Testing only happy path (no error scenarios)
- ❌ Not testing related entity operations
- ❌ Ignoring ITracingService output
- ❌ Not validating depth in recursive scenarios

---

## Test Structure Template

```csharp
public class PluginNameTests
{
    private readonly XrmFakedContext _context;
    private readonly IOrganizationService _service;

    public PluginNameTests()
    {
        _context = new XrmFakedContext();
        _service = _context.GetOrganizationService();
    }

    [Fact]
    public void MethodUnderTest_Scenario_ExpectedBehavior()
    {
        // Arrange: Setup context, data, plugin context
        
        // Act: Execute plugin
        
        // Assert: Verify results
    }
}
```
