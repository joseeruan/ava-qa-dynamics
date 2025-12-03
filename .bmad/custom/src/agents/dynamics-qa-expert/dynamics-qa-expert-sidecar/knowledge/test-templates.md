# Templates de Teste (NUnit)

Este arquivo fornece templates prontos para criação de testes unitários em projetos Dynamics 365 (plugins) e Azure Functions, utilizando NUnit, Moq e FakeXrmEasy quando aplicável.

## DEPENDÊNCIAS OBRIGATÓRIAS (MANDATÓRIO)

**SEMPRE use estas versões exatas dos pacotes:**

```xml
<PackageReference Include="Microsoft.CrmSdk.CoreAssemblies" Version="9.0.2.*" PrivateAssets="All" />
<PackageReference Include="Microsoft.PowerApps.MSBuild.Plugin" Version="1.*" PrivateAssets="All" />
<PackageReference Include="Microsoft.NETFramework.ReferenceAssemblies" Version="1.0.*" PrivateAssets="All" />
<PackageReference Include="NUnit" Version="3.13.3" />
<PackageReference Include="NUnit3TestAdapter" Version="4.5.0" PrivateAssets="All" />
<PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.8.0" />
<PackageReference Include="FakeXrmEasy.365" Version="1.58.1" />
```

**CRÍTICO:** Nunca sugira ou use versões diferentes. Estas são testadas e validadas.

## 1) Plugins Dynamics 365 (NUnit + FakeXrmEasy + Moq)

Pré-requisitos:
- Pacotes: Ver seção "DEPENDÊNCIAS OBRIGATÓRIAS" acima para versões exatas
- Estrutura de projeto: `src/<Projeto>.Tests/`

Exemplo de template:

```csharp
using NUnit.Framework;
using FakeXrmEasy;
using Microsoft.Xrm.Sdk;
using Moq;

namespace <Projeto>.Tests.Plugins
{
    [TestFixture]
    public class <PluginName>Tests
    {
        private XrmFakedContext _ctx;
        private IOrganizationService _service;
        private IPluginExecutionContext _pluginContext;

        [SetUp]
        public void SetUp()
        {
            _ctx = new XrmFakedContext();
            _service = _ctx.GetOrganizationService();
            _pluginContext = _ctx.GetDefaultPluginContext();
        }

        [Test]
        public void Deve_Executar_Caminho_Feliz()
        {
            // Arrange: entradas válidas
            // _pluginContext.MessageName = "Create"; // exemplo
            // _pluginContext.Stage = 40; // pós-operação

            // Act: executar plugin
            // _ctx.ExecutePluginWithTarget(new <PluginName>(), new Entity("account"));

            // Assert: verificar efeitos
            // Assert.That(condicao, Is.True);
        }

        [Test]
        public void Deve_Lancar_Excecao_Para_Entrada_Invalida()
        {
            // Arrange: entradas inválidas

            // Act + Assert: plugin lança exceção
            // Assert.Throws<InvalidPluginExecutionException>(() =>
            //     _ctx.ExecutePluginWithTarget(new <PluginName>(), new Entity("account"))
            // );
        }

        [Test]
        public void Deve_Validar_PreValidacao()
        {
            // Arrange
            // _pluginContext.Stage = 10; // pré-validação

            // Act & Assert
        }
    }
}
```

Critérios de validação:
- Cobrir mensagens: `Create`, `Update`, `Delete` conforme aplicável
- Cobrir estágios: pré-validação, pós-operação
- Verificar tratamento de exceções (`InvalidPluginExecutionException`)
- Testar integrações com `IOrganizationService` via mocks/fakes

## 2) Azure Functions (NUnit + Moq)

Pré-requisitos:
- Pacotes: `NUnit`, `Moq`
- Estrutura de projeto: `src/<Projeto>.Tests/`

Template HTTP Trigger:

```csharp
using NUnit.Framework;
using Moq;
using System.Threading.Tasks;

namespace <Projeto>.Tests.Functions
{
    [TestFixture]
    public class <FunctionName>Tests
    {
        [Test]
        public async Task Deve_Retornar_200_Para_Entrada_Valida()
        {
            // Arrange: mock de dependências e request válido

            // Act: chamar função
            // var response = await <FunctionName>.Run(request, logger, dependenciaMock.Object);

            // Assert
            // Assert.AreEqual(200, response.StatusCode);
        }

        [Test]
        public void Deve_Validar_Entrada_e_Lancar_Excecao()
        {
            // Arrange: request inválido

            // Act + Assert
            // Assert.ThrowsAsync<ArgumentException>(async () =>
            //     await <FunctionName>.Run(requestInvalido, logger, dependencia.Object));
        }

        [Test]
        public async Task Deve_Ser_Idempotente()
        {
            // Arrange: mesmo payload executado duas vezes

            // Act: duas execuções

            // Assert: resultados consistentes sem efeitos colaterais duplicados
        }
    }
}
```

Critérios de validação:
- Validar códigos de retorno e payloads
- Cobrir entradas inválidas e exceções
- Garantir idempotência quando aplicável
- Mockar dependências externas com Moq

## 3) Naming Convention

- `PluginName_Metodo_Scenario`
- `FunctionName_Cenario` (ex.: `ContactWebhook_Deve_Validar_Entrada_Invalida`)

## 4) Estrutura de Pastas (Sugerida)

```
src/
  AvaEdu/
    Plugins/
    Services/
    Repositories/
  AvaEdu.Tests/
    Plugins/
    Functions/
    Services/
    Repositories/
```

## 5) Checklist de Qualidade

- Cobertura de cenários principais (sucesso, falha, extremos)
- Verificações de segurança e validações
- Mocks/fakes apropriados
- Testes independentes e rápidos
- Documentação mínima em cada teste
