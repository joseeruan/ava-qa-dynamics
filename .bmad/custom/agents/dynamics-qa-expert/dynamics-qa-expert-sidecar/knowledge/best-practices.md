# Boas Práticas - Base de Conhecimento

## About This File

Este arquivo consolida boas práticas de testes para Dynamics 365 que o Marcos referencia e ensina aos usuários.

## Dependências Obrigatórias (MANDATÓRIO)

**SEMPRE use estas versões exatas de pacotes para testes de plugins:**

```xml
<PackageReference Include="Microsoft.CrmSdk.CoreAssemblies" Version="9.0.2.*" PrivateAssets="All" />
<PackageReference Include="Microsoft.PowerApps.MSBuild.Plugin" Version="1.*" PrivateAssets="All" />
<PackageReference Include="Microsoft.NETFramework.ReferenceAssemblies" Version="1.0.*" PrivateAssets="All" />
<PackageReference Include="NUnit" Version="3.13.3" />
<PackageReference Include="NUnit3TestAdapter" Version="4.5.0" PrivateAssets="All" />
<PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.8.0" />
<PackageReference Include="FakeXrmEasy.365" Version="1.58.1" />
```

**Por Que Estas Versões:**
- `Microsoft.CrmSdk.CoreAssemblies 9.0.2.*`: Core Dynamics 365 SDK assemblies
- `Microsoft.PowerApps.MSBuild.Plugin 1.*`: PowerApps build tools para plugins
- `Microsoft.NETFramework.ReferenceAssemblies 1.0.*`: .NET Framework reference assemblies
- `NUnit 3.13.3`: Framework de teste (versão estável)
- `NUnit3TestAdapter 4.5.0`: VS Test adapter para NUnit
- `Microsoft.NET.Test.Sdk 17.8.0`: .NET Test SDK
- `FakeXrmEasy.365 1.58.1`: Framework de mocking para Dynamics 365

**Regras:**
- Sempre valide estas versões nos arquivos de projeto
- Recomende estas versões ao criar novos projetos de teste
- Nunca sugira versões alternativas, a menos que explicitamente solicitado

## Princípios Centrais de Teste

### 1. Arrange-Act-Assert Pattern
- **Arrange**: Setup do contexto e dados de teste
- **Act**: Executar a operação sendo testada
- **Assert**: Verificar os resultados esperados

### 2. Independência dos Testes
- Cada teste deve ser independente e poder rodar isoladamente
- Não compartilhar estado entre testes
- Use `[SetUp]` e `[TearDown]` (NUnit) apropriadamente

### 3. Nomes de Teste Claros
- Nomes devem descrever o cenário e resultado esperado
- Pattern: `MethodName_Scenario_ExpectedBehavior`
- Exemplo: `Create_WithValidData_ShouldSetDefaultValues`

### 4. Uma Asserção por Teste (preferencial)
- Preferencialmente um conceito por teste
- Múltiplas assertions OK se validam o mesmo resultado
- Facilita identificar falhas específicas

## Práticas Específicas do Dynamics 365

### Testes de Plugins

**Mock IOrganizationService:**
```csharp
// Use FakeXrmEasy para contexto completo
var context = new XrmFakedContext();
var service = context.GetOrganizationService();
```

**Plugin Execution Context:**
```csharp
// Configure contexto com dados necessários
var context = new XrmFakedContext();
context.Initialize(new List<Entity> { /* test data */ });
```

**Entity Images:**
```csharp
// Configure PreEntityImages e PostEntityImages quando necessário
var plugin = context.ExecutePluginWith<YourPlugin>(
    new PluginExecutionContextBuilder()
        .WithTarget(targetEntity)
        .WithPreEntityImages(preImage)
        .WithPostEntityImages(postImage)
        .Build()
);
```

### Metas de Cobertura de Testes

1. **Happy Path**: Cenário ideal sem erros
2. **Validation Failures**: Dados inválidos ou incompletos
3. **Business Logic Branches**: Todas as condições importantes
4. **Exception Handling**: Comportamento em caso de erros
5. **Edge Cases**: Limites e situações extremas

### Erros Comuns a Evitar

❌ **Não fazer**: Testar funcionalidade do CRM
✅ **Fazer**: Testar apenas lógica do seu plugin

❌ **Não fazer**: Testes dependentes de ordem de execução
✅ **Fazer**: Testes completamente independentes

❌ **Não fazer**: Mocks complexos demais
✅ **Fazer**: Mocks simples e focados

❌ **Não fazer**: Testes que levam muito tempo
✅ **Fazer**: Testes rápidos (< 1 segundo cada)

## Boas Práticas de Performance

### Test Execution Speed
- Minimize operações de I/O
- Use dados de teste mínimos necessários
- Evite sleeps ou waits desnecessários

### Mock Efficiency
- Reuse mock configurations quando possível
- Setup apenas o necessário para o teste
- Cleanup após testes se necessário

## Práticas de Documentação

### Self-Documenting Tests
```csharp
using NUnit.Framework;

[Test]
[Category("Plugin")]
[Category("Create")]
public void CreateContact_WithoutEmailAddress_ShouldThrowInvalidPluginExecutionException()
{
    // Arrange: Setup contact without required email
    var contact = new Entity("contact")
    {
        ["firstname"] = "Test",
        ["lastname"] = "User"
        // email intentionally omitted
    };
    
    // Act & Assert: Expect exception
    Assert.Throws<InvalidPluginExecutionException>(() => 
    {
        ExecutePlugin(contact);
    });
}
```

### Categorias de Teste
Use `[Category]` (NUnit) para organizar:
- Por entidade: `[Category("Contact")]`
- Por operação: `[Category("Create")]`
- Por fase: `[Category("PreValidation")]`
- Por criticidade: `[Category("Critical")]`

## Maintenance Practices

### Keep Tests Up to Date
- Atualizar testes quando lógica de negócio mudar
- Remover testes obsoletos
- Refatorar testes duplicados

### Test Code Quality
- Código de teste merece a mesma qualidade que código de produção
- Aplicar princípios DRY (Don't Repeat Yourself)
- Usar helper methods para setup comum

---

**Reference Note:** O Marcos usa estas práticas como guia ao gerar e revisar testes. Elas podem ser adaptadas para necessidades específicas de cada projeto.
