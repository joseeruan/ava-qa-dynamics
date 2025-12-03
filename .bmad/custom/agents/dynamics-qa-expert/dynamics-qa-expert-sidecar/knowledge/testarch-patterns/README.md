# TestArch Patterns for Dynamics 365 QA

## About This Directory

Este diretório contém padrões de teste adaptados do **BMM TestArch** (`.bmad/bmm/testarch/knowledge/`) para uso em testes unitários de **Dynamics 365 plugins** com **C#/NUnit/FakeXrmEasy**.

> **Nota**: Estes padrões são **referências teóricas** - os exemplos em TypeScript/Playwright devem ser **adaptados para C#** ao aplicar em projetos Dynamics 365.

## 📚 Fragmentos Disponíveis

### ⭐⭐⭐ Core Patterns (Alta Prioridade)

#### `data-factories.md`
**Quando usar**: Criando factories de dados de teste (Entity, Contact, Account, etc.)

**Conceitos aplicáveis**:
- Factory functions com overrides (`createUser(overrides)` → `TestDataFactory.CreateOccurrence(overrides)`)
- Nested factories para relacionamentos (Order → User + Products)
- Parallel-safe data com IDs únicos (Guid.NewGuid(), Faker)
- API-first seeding (usar helpers, não UI)

**Alinha com seu projeto**: ✅ Você já usa `TestDataFactory.cs` - este fragmento valida e expande esse padrão

**Adaptação C#**:
```csharp
// Padrão do fragmento (TypeScript):
// const user = createUser({ email: 'test@example.com', role: 'admin' });

// Adaptação Dynamics 365 (C#):
var occurrence = TestDataFactory.CreateOccurrence(
    cpf: "12345678901",
    status: TestDataFactory.CreateStatusOptionSet(LogicalNames.STATUSABERTO)
);
```

---

#### `error-handling.md`
**Quando usar**: Testando tratamento de exceções em plugins (InvalidPluginExecutionException)

**Conceitos aplicáveis**:
- Scoped exception handling (ignore expected errors, catch regressions)
- Retry validation patterns (network resilience)
- Telemetry logging with context (error details + redaction)
- Graceful degradation tests (fallback behavior)

**Alinha com seu projeto**: ✅ Testes de exceção críticos para Dynamics plugins

**Adaptação C#**:
```csharp
// Padrão do fragmento (Playwright):
// page.on('pageerror', (error) => { if (error.includes('NetworkError')) return; throw error; });

// Adaptação Dynamics 365 (NUnit):
[Test]
public void Execute_Should_ThrowInvalidPluginExecutionException_When_RequiredFieldMissing()
{
    // Arrange
    var entity = TestDataFactory.CreateOccurrence(cpf: null); // Missing required field
    
    // Act & Assert
    var ex = Assert.Throws<InvalidPluginExecutionException>(() => 
        _service.Execute(context, Service)
    );
    
    Assert.That(ex.Message, Does.Contain("CPF is required"));
}
```

---

#### `fixture-architecture.md`
**Quando usar**: Estruturando base classes de teste, helpers, fixtures, cleanup

**Conceitos aplicáveis**:
- Pure functions first, fixture wrappers second (testability)
- Composition over inheritance (mergeTests → helper composition)
- Cleanup patterns (track created entities, auto-delete in teardown)
- Reusability via package exports (shared helpers)

**Alinha com seu projeto**: ✅ `FakeXrmEasyTestBase.cs` é uma fixture architecture

**Adaptação C#**:
```csharp
// Padrão do fragmento (Playwright mergeTests):
// export const test = mergeTests(apiFixture, authFixture, logFixture);

// Adaptação Dynamics 365 (NUnit base class + helpers):
public abstract class FakeXrmEasyTestBase
{
    protected XrmFakedContext Context;
    protected IOrganizationService Service;
    protected List<Guid> CreatedEntities = new List<Guid>();

    [SetUp]
    public void Setup()
    {
        Context = new XrmFakedContext();
        Service = Context.GetOrganizationService();
    }

    [TearDown]
    public void Cleanup()
    {
        // Auto-cleanup tracked entities
        foreach (var id in CreatedEntities)
        {
            Service.Delete("entity_name", id);
        }
        CreatedEntities.Clear();
    }
}
```

---

### ⭐⭐ Quality Patterns (Média Prioridade)

#### `test-quality.md`
**Quando usar**: Validando qualidade de testes (independência, determinismo, parallel safety)

**Conceitos aplicáveis**:
- Test independence (cada teste roda isoladamente)
- Deterministic data (não usar DateTime.Now direto, usar factories)
- Parallel safety (evitar IDs hardcoded, usar UUIDs)
- Flakiness prevention (mocks consistentes, sem sleeps)

**Alinha com seu projeto**: ✅ Universal - aplica a qualquer teste

**Adaptação C#**: Princípios diretos, sem código específico de framework

---

#### `test-levels-framework.md`
**Quando usar**: Classificando testes (Unit vs Integration vs E2E)

**Conceitos aplicáveis**:
- **Unit**: Plugin logic isolado (FakeXrmEasy, mocks)
- **Integration**: Plugin + Service + Repository (in-memory CRM)
- **E2E**: Deploy to sandbox, test via UI Automation

**Alinha com seu projeto**: ✅ Seus testes são Unit (plugins isolados)

**Classificação Dynamics 365**:
- Plugin tests com FakeXrmEasy = **Unit**
- Workflow/Custom API tests = **Integration**
- Power Automate flows = **E2E**

---

### ⭐ Strategy Patterns (Contextual)

#### `contract-testing.md`
**Quando usar**: Validando schemas de entidades Dynamics (AttributeMetadata, OptionSets)

**Conceitos aplicáveis**:
- Schema validation (ensure Entity has expected fields)
- Contract testing between plugin and CRM schema
- Breaking change detection (field removed/renamed)

**Adaptação C#**:
```csharp
[Test]
public void OccurrenceEntity_Should_HaveRequiredFields()
{
    var entity = new Entity(LogicalNames.ENTITY);
    
    // Assert: Required attributes exist
    Assert.That(entity.LogicalName, Is.EqualTo("new_occurrence"));
    Assert.That(LogicalNames.FIELDCPF, Is.Not.Null);
    Assert.That(LogicalNames.FIELDSTATUS, Is.Not.Null);
}
```

---

#### `test-priorities-matrix.md`
**Quando usar**: Priorizando cenários de teste (criticidade × complexidade)

**Conceitos aplicáveis**:
- Risk matrix (High Impact × High Probability = Test First)
- Business-critical paths (payment, auth) > Edge cases
- Coverage strategy (critical scenarios → common paths → edge cases)

**Aplicação**: Priorize testes de Create/Update/Delete antes de casos extremos

---

#### `component-tdd.md`
**Quando usar**: Adotando TDD (Test-Driven Development) para plugins

**Conceitos aplicáveis**:
- Red-Green-Refactor cycle
- Write failing test → Implement → Refactor
- ATDD (Acceptance Test-Driven Development)

**Workflow TDD para Dynamics**:
1. Red: Escrever teste que falha (`[Test] Create_Should_SetDefaultDate()`)
2. Green: Implementar plugin até teste passar
3. Refactor: Melhorar código mantendo testes verdes

---

## 🚫 Fragmentos NÃO Incluídos

Os seguintes fragmentos do TestArch **não** foram incluídos por serem específicos de Playwright/Cypress/Web:

- `playwright-config.md`, `selector-resilience.md`, `visual-debugging.md` - UI web testing
- `api-request.md`, `network-recorder.md`, `intercept-network-call.md` - HTTP/REST patterns
- `auth-session.md`, `email-auth.md` - Web authentication flows
- `burn-in.md`, `ci-burn-in.md` - Playwright CI-specific
- `file-utils.md`, `log.md`, `recurse.md` - Playwright utils
- `feature-flags.md` - Web feature toggles
- `network-error-monitor.md`, `network-first.md` - Network testing
- `timing-debugging.md` - Async timing web
- `fixtures-composition.md` - Playwright `mergeTests` API
- `overview.md` - Playwright utils package overview

Estes conceitos **não aplicam** a testes unitários de plugins Dynamics 365.

---

## 📖 Como Usar

### No Workflow de Geração de Testes

Quando o agente executa `*generate-tests`:
1. Carrega `project-patterns.md` (padrões do projeto atual)
2. Carrega `test-templates.md` (templates customizados)
3. **Referencia TestArch patterns** quando aplicável:
   - Gerando factories → `data-factories.md`
   - Testando exceções → `error-handling.md`
   - Estruturando base classes → `fixture-architecture.md`

### No Workflow de Revisão

Quando o agente executa `*review-tests`:
1. Valida contra `best-practices.md`
2. Verifica compliance com `test-quality.md`
3. Sugere melhorias baseadas em TestArch patterns

### Aprendizado Contínuo

O agente pode:
- Aprender novos padrões do projeto → salvar em `project-patterns.md`
- Adaptar templates baseado em feedback → atualizar `test-templates.md`
- **Não modificar** TestArch patterns (são referências do BMM framework)

---

## 🔄 Sincronização com BMM TestArch

- **Source**: `.bmad/bmm/testarch/knowledge/`
- **Adapted for**: Dynamics 365 C#/NUnit/FakeXrmEasy
- **Maintained by**: BMM framework (não editar diretamente)
- **Last sync**: December 3, 2025

Para atualizar estes padrões, sincronize com a versão mais recente do BMM framework.

---

**Uso pelo Agente**: Estes fragmentos são **referências teóricas** que o agente usa para:
1. Validar patterns em testes existentes
2. Sugerir melhorias baseadas em princípios universais
3. Adaptar conceitos TypeScript/Playwright para C#/Dynamics 365

Exemplos em TypeScript devem ser **adaptados para C#** antes de aplicar em código real.
