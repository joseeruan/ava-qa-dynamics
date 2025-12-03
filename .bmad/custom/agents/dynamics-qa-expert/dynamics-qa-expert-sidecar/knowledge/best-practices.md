# Boas Práticas - Base de Conhecimento

## About This File

Este arquivo consolida boas práticas de testes para Dynamics 365 que o Marcos referencia e ensina aos usuários.

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
