# QA Quick Setup - Template de Relatório de Configuração

---
setup_date: {{date}}
source_path: {{source_path}}
test_output_location: {{test_output_location}}
test_framework: {{active_framework}}
---

# Relatório de Quick Setup - Suite de Testes Completa

**Data de Configuração:** {{date}}  
**Caminho do Código-Fonte:** `{{source_path}}`  
**Local dos Testes:** `{{test_output_location}}`  
**Framework de Teste:** {{active_framework}}

---

## Resumo Executivo

{{executive_summary}}

### Estatísticas do Projeto

| Métrica | Valor |
|---------|-------|
| Plugins encontrados | {{plugins_count}} |
| Workflows customizados | {{workflows_count}} |
| Custom APIs | {{custom_apis_count}} |
| Controles PCF | {{pcf_controls_count}} |
| Testes unitários gerados | {{unit_tests_count}} |
| Testes de integração gerados | {{integration_tests_count}} |

---

## Análise do Projeto

{{analysis_summary}}

---

## Estrutura de Testes Criada

```
{{test_project_structure}}
```

---

## Anti-padrões Detectados

{{antipattern_report}}

---

## Testes Gerados

### Testes Unitários
{{unit_tests_summary}}

### Testes de Integração
{{integration_tests_summary}}

---

## Classes Helper Criadas

{{helper_classes}}

---

## Configuração de Dependências

As seguintes dependências foram configuradas no projeto de testes:

```xml
{{test_dependencies}}
```

---

## Métricas de Qualidade Inicial

| Métrica | Valor | Status |
|---------|-------|--------|
| Cobertura de código | {{code_coverage}}% | {{coverage_status}} |
| Complexidade ciclomática média | {{avg_complexity}} | {{complexity_status}} |
| Anti-padrões críticos | {{critical_antipatterns}} | {{antipatterns_status}} |
| Testes passando | {{passing_tests}}/{{total_tests}} | {{tests_status}} |

---

## Próximos Passos

1. ✅ **Executar Suite de Testes:** `dotnet test {{test_output_location}}`
2. ✅ **Revisar Testes Gerados:** Ajustar conforme necessidades específicas
3. ✅ **Corrigir Anti-padrões Críticos:** Priorizar issues de alta severidade
4. ✅ **Adicionar Testes Customizados:** Para lógica de negócio específica
5. ✅ **Configurar CI/CD:** Integrar testes ao pipeline de build
6. ✅ **Configurar Cobertura:** Adicionar análise de cobertura de código

---

## Comandos Úteis

### Executar Todos os Testes
```bash
dotnet test {{test_output_location}}
```

### Executar com Cobertura
```bash
dotnet test {{test_output_location}} /p:CollectCoverage=true
```

### Executar Testes Específicos
```bash
dotnet test {{test_output_location}} --filter "FullyQualifiedName~{{plugin_name}}"
```

---

## Observações e Recomendações

{{additional_notes}}

{{recommendations}}

---

**Configurado por:** Dynamics QA Expert  
**Versão do Agente:** {{agent_version}}  
**Duração do Setup:** {{setup_duration}}
