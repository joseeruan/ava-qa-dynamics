# QA Gerar Testes - Template de Relatório de Geração

---
generation_date: {{date}}
source_path: {{source_path}}
test_framework: {{active_framework}}
tests_generated: {{tests_count}}
---

# Relatório de Geração de Testes

**Data de Geração:** {{date}}  
**Caminho do Código-Fonte:** `{{source_path}}`  
**Framework de Teste:** {{active_framework}}  
**Testes Gerados:** {{tests_count}}

---

## Resumo Executivo

{{executive_summary}}

### Estatísticas de Geração

| Métrica | Valor |
|---------|-------|
| Plugins analisados | {{plugins_count}} |
| Testes unitários gerados | {{unit_tests_count}} |
| Testes de integração gerados | {{integration_tests_count}} |
| Cobertura de cenários | {{scenario_coverage}}% |
| Tempo de geração | {{generation_time}} |

---

## Componentes Testados

{{components_tested}}

---

## Estrutura de Testes Criada

```
{{test_project_structure}}
```

---

## Testes Gerados por Componente

{{tests_by_component}}

---

## Cenários Cobertos

### Cenários de Sucesso (Happy Path)
{{success_scenarios}}

### Cenários de Validação
{{validation_scenarios}}

### Cenários de Exceção
{{exception_scenarios}}

### Casos Extremos (Edge Cases)
{{edge_case_scenarios}}

---

## Dependências de Teste

As seguintes dependências foram configuradas:

```xml
{{test_dependencies}}
```

---

## Próximos Passos

1. ✅ Revisar testes gerados e ajustar conforme necessário
2. ✅ Executar testes para validar funcionamento
3. ✅ Adicionar testes adicionais para cenários específicos do negócio
4. ✅ Integrar execução de testes ao pipeline CI/CD
5. ✅ Configurar relatórios de cobertura de código

---

## Observações

{{additional_notes}}

---

**Gerado por:** Dynamics QA Expert  
**Versão do Agente:** {{agent_version}}
