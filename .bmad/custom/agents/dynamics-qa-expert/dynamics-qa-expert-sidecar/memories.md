# Marcos - Memory Bank

## About This File

Este arquivo mantém a memória persistente do Marcos entre sessões. Aqui são armazenados insights sobre plugins testados, preferências do usuário, e contextos importantes de projetos.

## User Preferences

<!-- O Marcos aprenderá e documentará suas preferências ao longo do tempo -->

### Testing Framework Preferences

- Framework padrão: NUnit
- Uso de FakeXrmEasy: (será aprendido na primeira sessão)
- Convenções de nomenclatura: (será aprendido na primeira sessão)

## Session History

<!-- Histórico de sessões importantes e plugins testados -->

### Format
```
## [Data] - [Nome do Plugin]
- Tipo de plugin: (Create/Update/Delete/etc)
- Complexidade: (Baixa/Média/Alta)
- Testes gerados: (quantidade e tipos)
- Observações: (particularidades ou aprendizados)
```

## [2025-12-03] - ava_edu Occurrence Plugins
- **Tipo de plugin:** Create/Update/Delete (CRUD completo)
- **Entidade:** ava_ocorrencia
- **Complexidade:** Média-Alta
- **Testes gerados:** 40 testes unitários
  - CreatePlugin: 12 testes
  - UpdatePlugin: 18 testes
  - DeletePlugin: 10 testes
- **Arquitetura:** Repository + Service pattern com separação de camadas
- **Frameworks utilizados:** xUnit, FakeXrmEasy 3.4.3, Moq, FluentAssertions
- **Observações:**
  - Projeto bem estruturado com separação clara de responsabilidades
  - UpdatePlugin tem lógica complexa de validação de campos protegidos em ocorrências fechadas
  - Issue identificado: ValidateModificationInClosedOccurrence sempre lança exceção mesmo sem mudanças
  - Falta validação de depth nos plugins (risco de loop infinito)
  - Duplicate validation robusta baseada em CPF + Tipo + Assunto
  - Cálculo de data de expiração baseado em prazo configurável no tipo de ocorrência

## Project Patterns

<!-- Padrões específicos observados em diferentes projetos -->

### Format
```
## [Nome do Projeto]
- Estrutura de testes: (organização de pastas e namespaces)
- Frameworks utilizados: (xUnit, FakeXrmEasy, Moq, etc)
- Convenções especiais: (regras específicas do projeto)
```

## ava_edu (plugins_avaEdu)
- **Estrutura de testes:** 
  - Namespace: `plugins_avaEdu.Tests`
  - Organização: `Plugins/{EntityName}/{PluginName}Tests.cs`
  - Helpers separados: `Helpers/TestBase.cs`, `Helpers/TestDataFactory.cs`
- **Frameworks utilizados:**
  - xUnit 2.6.2
  - FakeXrmEasy.Core 3.4.3
  - FakeXrmEasy.Plugins 3.4.3
  - Moq 4.20.70
  - FluentAssertions 6.12.0
- **Convenções especiais:**
  - Nomenclatura de testes: `Execute_When{Condition}_Should{ExpectedBehavior}`
  - Pattern AAA (Arrange-Act-Assert) rigorosamente seguido
  - FluentAssertions com mensagens descritivas
  - TestDataFactory com métodos específicos para cada tipo de entidade
  - Target Framework: .NET Framework 4.6.2
  - Uso de LogicalNames class para constantes de campos
  - Plugin base customizado (PluginBase) com LocalPluginContext

## Important Insights

<!-- Insights e observações importantes que o Marcos deve lembrar -->

---

**Nota:** Este arquivo é atualizado automaticamente pelo Marcos após cada sessão significativa. Ele usa essas memórias para fornecer assistência cada vez mais personalizada e consistente com seus padrões de trabalho.
