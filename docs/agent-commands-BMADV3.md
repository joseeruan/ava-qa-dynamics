## Comandos e Capacidades do Agente

### Capacidades Centrais Identificadas

1. **Gerar testes unitários completos** para plugins Dynamics 365 e Azure Functions
2. **Analisar código** (plugins e funções) e criar/atualizar projeto de testes em `src/[nome-projeto]`
3. **Revisar testes existentes** e sugerir melhorias
4. **Gerar relatórios de cobertura** de testes
5. **Ensinar boas práticas** de testes para Dynamics 365 e Azure Functions
6. **Aprender padrões** do projeto ao longo do tempo (Expert feature)
7. **Recordar padrões** aprendidos de projetos anteriores

### Estrutura de Comandos

```yaml
menu:
  - trigger: generate-tests
    workflow: '{agent-folder}/dynamics-qa-expert-sidecar/workflows/generate-tests.md'
    description: 'Gera testes unitários completos para plugins Dynamics 365 e Azure Functions'
    
  - trigger: analyze-plugin
    workflow: '{agent-folder}/dynamics-qa-expert-sidecar/workflows/analyze-plugin.md'
    description: 'Analisa plugin/funcão e sugere estrutura de testes sem gerar código'
    
  - trigger: review-tests
    workflow: '{agent-folder}/dynamics-qa-expert-sidecar/workflows/review-tests.md'
    description: 'Revisa testes existentes e sugere melhorias'
    
  - trigger: coverage-report
    workflow: '{agent-folder}/dynamics-qa-expert-sidecar/workflows/coverage-report.md'
    description: 'Gera relatório de cobertura de testes com análise de qualidade'
    
  - trigger: teach
    workflow: '{agent-folder}/dynamics-qa-expert-sidecar/workflows/teach-practices.md'
    description: 'Ensina boas práticas de testes para Dynamics 365 e Azure Functions'
    
  - trigger: learn
    action: 'Atualiza {agent-folder}/dynamics-qa-expert-sidecar/knowledge/project-patterns.md com padrões específicos do projeto'
    description: 'Salva padrões do projeto atual na knowledge base'
    
  - trigger: recall-patterns
    action: '#recall-patterns-prompt'
    description: 'Mostra padrões aprendidos de projetos anteriores'

validation:
  chat:
    checklist:
      - 'Triggers resolvem para workflows/ações existentes'
      - 'Workflows respondem com plano ou geração conforme esperado'
      - 'Templates NUnit disponíveis em knowledge/test-templates.md'
    examples:
      - 'generate-tests src/AvaEdu/Plugins/CreatePlugin.cs'
      - 'analyze-plugin src/AvaEdu/Repositories/Implementations/OcorrenciaRepository.cs'
```

### Plano de Integração de Workflows

**Workflows no Sidecar** (Recurso de Expert Agent):
- `generate-tests.md` - Workflow completo para geração de testes
- `analyze-plugin.md` - Análise detalhada de plugins
- `review-tests.md` - Revisão estruturada de testes
- `coverage-report.md` - Geração de relatórios
- `teach-practices.md` - Modo educacional interativo

**Direct Actions**:
- `learn` - Atualiza knowledge base diretamente
- `recall-patterns` - Usa prompt para acessar memórias

### Estrutura do Sidecar

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

### Recursos Avançados

**Integração com Memória:**
- Lembra de plugins já testados e suas peculiaridades
- Aprende naming conventions do projeto
- Reconhece padrões de teste preferidos pela equipe

**Capacidade de Aprendizado:**
- Salva estruturas de teste bem-sucedidas
- Adapta templates baseado em feedback
- Evolui com o uso contínuo

**Restrição de Domínio:**
- Opera principalmente no sidecar para segurança
- Gera testes em `{project-root}/src/` conforme especificado
- Mantém knowledge base privada no sidecar

### Notas de Implementação

**Considerações para Expert Agent:**
- critical_actions deve carregar memories.md e instructions.md
- Workflows no sidecar permitem evolução independente
- Knowledge base cresce com uso do agente
- Restrição de domínio garante segurança

**Estratégia de Saída:**
- Testes gerados: `{project-root}/src/[nome-projeto]/`
- Análises e relatórios: `{output_folder}/qa-reports/`
- Aprendizados: `{agent-folder}/dynamics-qa-expert-sidecar/knowledge/`

**Padrão de Workflow:**
- Todos workflows serão criados com abordagem intent-based
- Conversacionais e adaptativos ao contexto do usuário
- Integram com memories para contexto histórico

### Detecção de Projeto e Contexto (src/)

Ao executar `generate-tests` ou `analyze-plugin`, o agente deve:
- Detectar o(s) projeto(s) dentro de `src/` (ex.: `src/AvaEdu/`)
- Mapear contexto aplicável: `Plugins/`, `Services/`, `Repositories/`
- Identificar classes de plugin (que herdam de `IPlugin`), serviços e repositórios
- Inferir métodos, objetivos e cenários de erro a partir de nomes, comentários e uso
- Criar/atualizar projeto de testes com NUnit em `src/[Projeto].Tests/`
- Gerar validações de integridade e múltiplos casos de erro (exceções, entradas inválidas, estados inesperados)
