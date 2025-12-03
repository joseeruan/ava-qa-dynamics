# Knowledge Base Integration - Dynamics QA Expert

## 📋 Resumo da Integração

O agente **Dynamics QA Expert** agora tem acesso a **padrões de teste universais** do BMM TestArch, adaptados para uso com C#/Dynamics 365/NUnit/FakeXrmEasy.

**Data da integração**: December 3, 2025

---

## 🗂️ Estrutura da Knowledge Base

```
dynamics-qa-expert-sidecar/
├── instructions.md                    ✅ ATUALIZADO
├── memories.md                        (mantido)
├── knowledge/
│   ├── best-practices.md              ✅ (já existia)
│   ├── project-patterns.md            ✅ (já existia)
│   ├── test-templates.md              ✅ (já existia)
│   └── testarch-patterns/             ➕ NOVO
│       ├── README.md                  ➕ Guia de uso dos padrões TestArch
│       ├── data-factories.md          ⭐⭐⭐ (do BMM TestArch)
│       ├── error-handling.md          ⭐⭐⭐ (do BMM TestArch)
│       ├── fixture-architecture.md    ⭐⭐⭐ (do BMM TestArch)
│       ├── test-quality.md            ⭐⭐  (do BMM TestArch)
│       ├── test-levels-framework.md   ⭐⭐  (do BMM TestArch)
│       ├── contract-testing.md        ⭐   (do BMM TestArch)
│       ├── test-priorities-matrix.md  ⭐   (do BMM TestArch)
│       └── component-tdd.md           ⭐   (do BMM TestArch)
└── workflows/
    └── (mantidos sem alteração)
```

---

## 📚 Arquivos Integrados do TestArch (8 total)

### ⭐⭐⭐ Core Patterns (Alta Prioridade)

1. **`data-factories.md`**
   - **Conceito**: Factory functions com overrides, nested factories, parallel-safe data
   - **Alinha com**: `TestDataFactory.cs` já existente no projeto
   - **Uso**: Criando entities de teste (Occurrence, Contact, Account)

2. **`error-handling.md`**
   - **Conceito**: Scoped exception handling, retry patterns, telemetry logging
   - **Alinha com**: Testes de `InvalidPluginExecutionException` em plugins
   - **Uso**: Testando tratamento de exceções, fallback behavior

3. **`fixture-architecture.md`**
   - **Conceito**: Pure functions + fixture wrappers, composition over inheritance, cleanup
   - **Alinha com**: `FakeXrmEasyTestBase.cs` já existente
   - **Uso**: Estruturando base classes, helpers, auto-cleanup

### ⭐⭐ Quality Patterns (Média Prioridade)

4. **`test-quality.md`**
   - **Conceito**: Test independence, determinism, parallel safety, flakiness prevention
   - **Alinha com**: Princípios universais de qualidade
   - **Uso**: Validando e melhorando qualidade de testes

5. **`test-levels-framework.md`**
   - **Conceito**: Unit vs Integration vs E2E classification
   - **Alinha com**: Plugin tests = Unit, Workflow tests = Integration
   - **Uso**: Classificando tipos de teste corretamente

### ⭐ Strategy Patterns (Contextual)

6. **`contract-testing.md`**
   - **Conceito**: Schema validation, breaking change detection
   - **Alinha com**: Validação de Entity schemas, AttributeMetadata
   - **Uso**: Garantir compatibilidade com schema do CRM

7. **`test-priorities-matrix.md`**
   - **Conceito**: Risk-based prioritization (impact × probability)
   - **Alinha com**: Focar em cenários críticos de negócio primeiro
   - **Uso**: Decidir quais testes criar primeiro

8. **`component-tdd.md`**
   - **Conceito**: Test-Driven Development workflow (Red-Green-Refactor)
   - **Alinha com**: ATDD para desenvolvimento de plugins
   - **Uso**: Adotar TDD no desenvolvimento de plugins

---

## 🔄 Mudanças nos Arquivos do Agente

### 1. `dynamics-qa-expert.md` (Agent Definition)

**Mudanças**:
- ✅ Step 6: Agora carrega knowledge base incluindo `testarch-patterns/`
- ✅ Step 8: Referencia TestArch patterns quando aplicável
- ✅ Prompt `recall-patterns`: Agora inclui TestArch patterns

### 2. `instructions.md` (Private Instructions)

**Adições**:
- ✅ **Knowledge Base Architecture**: Nova seção documentando estrutura
- ✅ **Learning Triggers**: Lista quando referenciar cada TestArch pattern
- ✅ **Session Protocol**: Atualizado para carregar TestArch patterns
- ✅ **During Session**: Aplica princípios TestArch adaptados para C#

### 3. `testarch-patterns/README.md` (Novo)

**Conteúdo**:
- 📖 Guia completo de uso dos 8 fragmentos TestArch
- 💡 Exemplos de adaptação TypeScript → C#
- ❌ Lista de 19 fragmentos NÃO incluídos (web-specific)
- 🔄 Sincronização com BMM framework

---

## 🎯 Como o Agente Usa os Padrões

### No Workflow `*generate-tests`

1. Carrega `project-patterns.md` (padrões aprendidos)
2. Carrega `test-templates.md` (templates customizados)
3. **Referencia TestArch patterns** conforme necessário:
   - Criando factories → `data-factories.md`
   - Testando exceções → `error-handling.md`
   - Estruturando fixtures → `fixture-architecture.md`

### No Workflow `*review-tests`

1. Valida contra `best-practices.md`
2. Verifica `test-quality.md` (independência, determinismo)
3. Sugere melhorias baseadas em TestArch patterns

### No Workflow `*analyze-plugin`

1. Classifica testes usando `test-levels-framework.md`
2. Prioriza cenários usando `test-priorities-matrix.md`
3. Sugere TDD approach usando `component-tdd.md`

---

## 📖 Exemplos de Adaptação

### Example 1: Data Factories

**TestArch (TypeScript)**:
```typescript
const user = createUser({ 
  email: 'test@example.com', 
  role: 'admin' 
});
```

**Adaptado (C# Dynamics)**:
```csharp
var occurrence = TestDataFactory.CreateOccurrence(
    cpf: "12345678901",
    status: TestDataFactory.CreateStatusOptionSet(LogicalNames.STATUSABERTO)
);
```

### Example 2: Error Handling

**TestArch (Playwright)**:
```typescript
page.on('pageerror', (error) => {
  if (error.includes('NetworkError')) return;
  throw error;
});
```

**Adaptado (C# NUnit)**:
```csharp
[Test]
public void Execute_Should_ThrowInvalidPluginExecutionException_When_RequiredFieldMissing()
{
    var ex = Assert.Throws<InvalidPluginExecutionException>(() => 
        _service.Execute(context, Service)
    );
    Assert.That(ex.Message, Does.Contain("CPF is required"));
}
```

### Example 3: Fixture Architecture

**TestArch (Playwright mergeTests)**:
```typescript
export const test = mergeTests(
  apiFixture, 
  authFixture, 
  logFixture
);
```

**Adaptado (C# NUnit)**:
```csharp
public abstract class FakeXrmEasyTestBase
{
    protected XrmFakedContext Context;
    protected IOrganizationService Service;
    
    [SetUp]
    public void Setup()
    {
        Context = new XrmFakedContext();
        Service = Context.GetOrganizationService();
    }
    
    [TearDown]
    public void Cleanup()
    {
        // Auto-cleanup
    }
}
```

---

## ❌ Fragmentos NÃO Incluídos (19 de 32)

Os seguintes fragmentos do BMM TestArch **não foram incluídos** por serem específicos de Playwright/Cypress/Web:

- Playwright configuration, selectors, visual debugging
- Network interception, API mocking, HAR recording
- Web authentication flows, session management
- CI burn-in scripts (Playwright-specific)
- File utils, logging utilities (Playwright utils)
- Feature flags (web toggles)
- Network monitoring, timing debugging
- Fixture composition (Playwright `mergeTests` API)

**Motivo**: Não aplicam a testes unitários de plugins Dynamics 365 C#.

---

## 🔍 Fonte dos Padrões

- **Source**: `.bmad/bmm/testarch/knowledge/`
- **Total fragments**: 32 (21 core + 11 playwright-utils)
- **Included**: 8 fragments (universais, adaptáveis para C#)
- **Excluded**: 24 fragments (web-specific, não aplicável)

---

## 📝 Notas Importantes

### 1. **Referências Teóricas**
- Fragmentos TestArch são **guias teóricos**, não código pronto
- Exemplos em TypeScript/Playwright devem ser **adaptados para C#**
- Agente usa conceitos, não copia código literalmente

### 2. **Não Editar TestArch Patterns**
- Arquivos em `testarch-patterns/` são **cópias do BMM framework**
- Edite apenas: `best-practices.md`, `project-patterns.md`, `test-templates.md`
- Para atualizar TestArch patterns, sincronize com BMM framework

### 3. **Aprendizado Contínuo**
- Agente aprende padrões do projeto → salva em `project-patterns.md`
- Agente adapta templates → atualiza `test-templates.md`
- Agente **não modifica** TestArch patterns (são referências fixas)

---

## 🚀 Próximos Passos

### Para o Usuário

1. **Testar integração**: Execute `*recall-patterns` para ver padrões disponíveis
2. **Gerar testes**: Use `*generate-tests` e observe referências a TestArch
3. **Revisar testes**: Execute `*review-tests` e veja validações contra TestArch

### Para o Agente

1. **Carregar knowledge**: Sempre carregar `testarch-patterns/README.md` primeiro
2. **Adaptar exemplos**: Converter TypeScript → C# ao sugerir código
3. **Referenciar naturalmente**: Mencionar padrões sem "dumping" verbatim
4. **Aprender do projeto**: Atualizar `project-patterns.md` com padrões observados

---

## 📊 Impacto da Integração

### Benefícios

✅ **Padrões universais**: Acesso a best practices de testing (BMM framework)
✅ **Validação robusta**: Agente valida contra princípios consolidados
✅ **Consistência**: Testes seguem padrões da indústria
✅ **Aprendizado**: Agente evolui com projeto + padrões universais

### Sem Breaking Changes

✅ **Retrocompatível**: Não quebra workflows existentes
✅ **Opcional**: TestArch patterns são referências, não obrigatórios
✅ **Incremental**: Agente usa padrões conforme aplicável

---

**Documentação criada**: December 3, 2025
**Última atualização**: December 3, 2025
**Versão**: 1.0
