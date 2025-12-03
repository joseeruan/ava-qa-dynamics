# Templates de Teste - Base de Conhecimento

## About This File

Este arquivo contém templates customizados de teste que o agente aprende e adapta baseado em feedback e uso contínuo.

## Template de Teste de Plugin (NUnit)

```csharp
// Template base para testes de plugin
// Será customizado baseado em padrões aprendidos

using NUnit.Framework;
using Microsoft.Xrm.Sdk;
using FakeXrmEasy;
using System;

namespace [Namespace]
{
    public class [PluginName]Tests
    {
        private XrmFakedContext _context;
        private IOrganizationService _service;
        
        [SetUp]
        public void SetUp()
        {
            _context = new XrmFakedContext();
            _service = _context.GetOrganizationService();
        }
        
        [Test]
        public void [MethodName]_[Scenario]_[ExpectedResult]()
        {
            // Arrange
            
            // Act
            
            // Assert
        }
    }
}
```

## Templates de Setup de Mocks

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

## Templates de Cenários

### Testes de Operação Create
```csharp
// Template para testar operações Create
// Será customizado por projeto
```

### Testes de Operação Update
```csharp
// Template para testar operações Update
// Será customizado por projeto
```

### Testes de Operação Delete
```csharp
// Template para testar operações Delete
// Será customizado por projeto
```

### Testes de Pré-Validação
```csharp
// Template para testar pré-validações
// Será customizado por projeto
```

### Testes de Tratamento de Exceções

## Template de Teste para Azure Functions (NUnit)

```csharp
using NUnit.Framework;
using System.Net;
using System.Net.Http;
using System.Threading;
using System.Threading.Tasks;

public class [FunctionName]Tests
{
    private HttpClient _httpClient;

    [SetUp]
    public void SetUp()
    {
        var handler = new FakeHttpMessageHandler(); // implemente retornos previsíveis
        _httpClient = new HttpClient(handler);
    }

    [Test]
    public async Task Execute_WithValidPayload_ShouldReturnOk()
    {
        var request = new HttpRequestMessage(HttpMethod.Post, "http://localhost/api/[FunctionName]")
        {
            Content = new StringContent("{ 'id': '123' }")
        };

        var response = await _httpClient.SendAsync(request);
        Assert.That(response.StatusCode, Is.EqualTo(HttpStatusCode.OK));
    }
}

public class FakeHttpMessageHandler : HttpMessageHandler
{
    protected override Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, CancellationToken cancellationToken)
    {
        // retornar respostas previsíveis para cenários de teste
        return Task.FromResult(new HttpResponseMessage(HttpStatusCode.OK));
    }
}
```
```csharp
// Template para testar tratamento de exceções
// Será customizado por projeto
```

---

**Customization Note:** Estes templates são pontos de partida. O agente  os adapta baseado em feedback e padrões específicos de cada projeto.
