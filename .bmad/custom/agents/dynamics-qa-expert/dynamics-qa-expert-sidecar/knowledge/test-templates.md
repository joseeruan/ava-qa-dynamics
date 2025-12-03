# Test Templates - Knowledge Base

## About This File

Este arquivo contém templates customizados de teste que o agente aprende e adapta baseado em feedback e uso contínuo.

## Plugin Test Template

```csharp
// Template base para testes de plugin
// Será customizado baseado em padrões aprendidos

using Microsoft.VisualStudio.TestTools.UnitTesting;
using Microsoft.Xrm.Sdk;
using FakeXrmEasy;
using System;

namespace [Namespace]
{
    [TestClass]
    public class [PluginName]Tests
    {
        private XrmFakedContext _context;
        private IOrganizationService _service;
        
        [TestInitialize]
        public void Setup()
        {
            _context = new XrmFakedContext();
            _service = _context.GetOrganizationService();
        }
        
        [TestMethod]
        public void [MethodName]_[Scenario]_[ExpectedResult]()
        {
            // Arrange
            
            // Act
            
            // Assert
        }
    }
}
```

## Mock Setup Templates

```csharp
// Templates para setup de mocks comuns
// Serão expandidos baseado em padrões observados

// IOrganizationService Mock
var serviceMock = new Mock<IOrganizationService>();

// IPluginExecutionContext Mock
var contextMock = new Mock<IPluginExecutionContext>();

// IServiceProvider Mock
var serviceProviderMock = new Mock<IServiceProvider>();
```

## Scenario Templates

### Create Operation Tests
```csharp
// Template para testar operações Create
// Será customizado por projeto
```

### Update Operation Tests
```csharp
// Template para testar operações Update
// Será customizado por projeto
```

### Delete Operation Tests
```csharp
// Template para testar operações Delete
// Será customizado por projeto
```

### Pre-Validation Tests
```csharp
// Template para testar pré-validações
// Será customizado por projeto
```

### Exception Handling Tests
```csharp
// Template para testar tratamento de exceções
// Será customizado por projeto
```

---

**Customization Note:** Estes templates são pontos de partida. O agente  os adapta baseado em feedback e padrões específicos de cada projeto.
