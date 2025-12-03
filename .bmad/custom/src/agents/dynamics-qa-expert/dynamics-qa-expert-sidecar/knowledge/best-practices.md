# Best Practices - Knowledge Base

## About This File

Este arquivo consolida boas práticas de testes para Dynamics 365 que o Marcos referencia e ensina aos usuários.

## Dependências Obrigatórias (MANDATÓRIO)

**SEMPRE use estas versões exatas dos pacotes para testes de plugins:**

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
- `Microsoft.CrmSdk.CoreAssemblies 9.0.2.*`: Assemblies principais do Dynamics 365 SDK
- `Microsoft.PowerApps.MSBuild.Plugin 1.*`: Ferramentas de build do PowerApps para plugins
- `Microsoft.NETFramework.ReferenceAssemblies 1.0.*`: Assemblies de referência do .NET Framework
- `NUnit 3.13.3`: Framework de teste (versão estável)
- `NUnit3TestAdapter 4.5.0`: Adaptador de teste do VS para NUnit
- `Microsoft.NET.Test.Sdk 17.8.0`: SDK de teste .NET
- `FakeXrmEasy.365 1.58.1`: Framework de mocking para Dynamics 365

**Regras:**
- Sempre valide estas versões nos arquivos de projeto
- Recomende estas versões ao criar novos projetos de teste
- Nunca sugira versões alternativas, a menos que explicitamente solicitado

## Core Testing Principles

### 1. Arrange-Act-Assert Pattern
- **Arrange**: Setup do contexto e dados de teste
- **Act**: Executar a operação sendo testada
- **Assert**: Verificar os resultados esperados

### 2. Test Independence
- Cada teste deve ser independente e poder rodar isoladamente
- Não compartilhar estado entre testes
- Use `[TestInitialize]` e `[TestCleanup]` apropriadamente

### 3. Clear Test Names
- Nomes devem descrever o cenário e resultado esperado
- Pattern: `MethodName_Scenario_ExpectedBehavior`
- Exemplo: `Create_WithValidData_ShouldSetDefaultValues`

### 4. One Assertion Per Test
- Preferencialmente um conceito por teste
- Múltiplas assertions OK se validam o mesmo resultado
- Facilita identificar falhas específicas

## Dynamics 365 Specific Practices

### Plugin Testing

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

### Test Coverage Goals

1. **Happy Path**: Cenário ideal sem erros
2. **Validation Failures**: Dados inválidos ou incompletos
3. **Business Logic Branches**: Todas as condições importantes
4. **Exception Handling**: Comportamento em caso de erros
5. **Edge Cases**: Limites e situações extremas

### Common Pitfalls to Avoid

❌ **Não fazer**: Testar funcionalidade do CRM
✅ **Fazer**: Testar apenas lógica do seu plugin

❌ **Não fazer**: Testes dependentes de ordem de execução
✅ **Fazer**: Testes completamente independentes

❌ **Não fazer**: Mocks complexos demais
✅ **Fazer**: Mocks simples e focados

❌ **Não fazer**: Testes que levam muito tempo
✅ **Fazer**: Testes rápidos (< 1 segundo cada)

## Performance Best Practices

### Test Execution Speed
- Minimize operações de I/O
- Use dados de teste mínimos necessários
- Evite sleeps ou waits desnecessários

### Mock Efficiency
- Reuse mock configurations quando possível
- Setup apenas o necessário para o teste
- Cleanup após testes se necessário

## Documentation Practices

### Self-Documenting Tests
```csharp
[TestMethod]
[TestCategory("Plugin")]
[TestCategory("Create")]
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
    Assert.ThrowsException<InvalidPluginExecutionException>(() => 
    {
        ExecutePlugin(contact);
    });
}
```

### Test Categories
Use `[TestCategory]` para organizar:
- Por entidade: `[TestCategory("Contact")]`
- Por operação: `[TestCategory("Create")]`
- Por fase: `[TestCategory("PreValidation")]`
- Por criticidade: `[TestCategory("Critical")]`

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
