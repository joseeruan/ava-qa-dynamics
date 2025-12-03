## Agent Commands and Capabilities

### Core Capabilities Identified

1. **Gerar testes unitários completos** para plugins Dynamics 365
2. **Analisar plugins** e criar estrutura de projeto de testes em `src/[nome-projeto]`
3. **Revisar testes existentes** e sugerir melhorias
4. **Gerar relatórios de cobertura** de testes
5. **Ensinar boas práticas** de testes para Dynamics 365
6. **Aprender padrões** do projeto ao longo do tempo (Expert feature)
7. **Recordar padrões** aprendidos de projetos anteriores

### Command Structure

```yaml
menu:
  - trigger: generate-tests
    workflow: '{agent-folder}/dynamics-qa-expert-sidecar/workflows/generate-tests.md'
    description: 'Gera testes unitários completos para um plugin Dynamics 365'
    
  - trigger: analyze-plugin
    workflow: '{agent-folder}/dynamics-qa-expert-sidecar/workflows/analyze-plugin.md'
    description: 'Analisa plugin e sugere estrutura de testes sem gerar código'
    
  - trigger: review-tests
    workflow: '{agent-folder}/dynamics-qa-expert-sidecar/workflows/review-tests.md'
    description: 'Revisa testes existentes e sugere melhorias'
    
  - trigger: coverage-report
    workflow: '{agent-folder}/dynamics-qa-expert-sidecar/workflows/coverage-report.md'
    description: 'Gera relatório de cobertura de testes com análise de qualidade'
    
  - trigger: teach
    workflow: '{agent-folder}/dynamics-qa-expert-sidecar/workflows/teach-practices.md'
    description: 'Ensina boas práticas de testes para Dynamics 365'
    
  - trigger: learn
    action: 'Atualiza {agent-folder}/dynamics-qa-expert-sidecar/knowledge/project-patterns.md com padrões específicos do projeto'
    description: 'Salva padrões do projeto atual na knowledge base'
    
  - trigger: recall-patterns
    action: '#recall-patterns-prompt'
    description: 'Mostra padrões aprendidos de projetos anteriores'
```

### Workflow Integration Plan

**Personal Sidecar Workflows** (Expert Agent Feature):
- `generate-tests.md` - Workflow completo para geração de testes
- `analyze-plugin.md` - Análise detalhada de plugins
- `review-tests.md` - Revisão estruturada de testes
- `coverage-report.md` - Geração de relatórios
- `teach-practices.md` - Modo educacional interativo

**Direct Actions**:
- `learn` - Atualiza knowledge base diretamente
- `recall-patterns` - Usa prompt para acessar memórias

### Sidecar Structure

```
dynamics-qa-expert-sidecar/
├── memories.md                    # Histórico de plugins testados
├── instructions.md                # Protocolos internos do agente
├── knowledge/
│   ├── project-patterns.md        # Padrões aprendidos do projeto
│   ├── test-templates.md          # Templates de teste customizados
│   └── best-practices.md          # Boas práticas consolidadas
└── workflows/
    ├── generate-tests.md          # Workflow de geração
    ├── analyze-plugin.md          # Workflow de análise
    ├── review-tests.md            # Workflow de revisão
    ├── coverage-report.md         # Workflow de relatório
    └── teach-practices.md         # Workflow educacional
```

### Advanced Features

**Memory Integration:**
- Lembra de plugins já testados e suas peculiaridades
- Aprende naming conventions do projeto
- Reconhece padrões de teste preferidos pela equipe

**Learning Capability:**
- Salva estruturas de teste bem-sucedidas
- Adapta templates baseado em feedback
- Evolui com o uso contínuo

**Domain Restriction:**
- Opera principalmente no sidecar para segurança
- Gera testes em `{project-root}/src/` conforme especificado
- Mantém knowledge base privada no sidecar

### Implementation Notes

**Expert Agent Considerations:**
- critical_actions deve carregar memories.md e instructions.md
- Workflows no sidecar permitem evolução independente
- Knowledge base cresce com uso do agente
- Restrição de domínio garante segurança

**Output Strategy:**
- Testes gerados: `{project-root}/src/[nome-projeto]/`
- Análises e relatórios: `{output_folder}/qa-reports/`
- Aprendizados: `{agent-folder}/dynamics-qa-expert-sidecar/knowledge/`

**Workflow Pattern:**
- Todos workflows serão criados com abordagem intent-based
- Conversacionais e adaptativos ao contexto do usuário
- Integram com memories para contexto histórico
