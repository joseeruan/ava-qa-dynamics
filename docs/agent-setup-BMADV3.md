## Configuração do Workspace do Agente

### Tipo de Agente

Expert Agent

### Configuração do Workspace

Estrutura completa de sidecar criada para memória persistente, base de conhecimento e workflows personalizados.

### Elementos de Setup

**Memória e Gestão de Sessão:**
- `memories.md` - Banco de memória persistente para rastrear plugins/funções testados, preferências, histórico de sessão e padrões do projeto

**Estrutura da Base de Conhecimento:**
- `knowledge/project-patterns.md` - Convenções de nomenclatura, estruturas de projeto e preferências de framework aprendidas
- `knowledge/test-templates.md` - Templates de teste personalizáveis que evoluem com o uso
- `knowledge/best-practices.md` - Boas práticas consolidadas para testes em Dynamics 365 e Azure Functions

**Instruções Privadas:**
- `instructions.md` - Diretrizes centrais, princípios operacionais, limites de domínio e protocolos de sessão

**Capacidades de Workflow Pessoal:**
- Pasta `workflows/` pronta para 5 workflows especializados:
  - generate-tests.md
  - analyze-plugin.md
  - review-tests.md
  - coverage-report.md
  - teach-practices.md

**Aprendizado e Adaptação:**
- Integração de memória para lembrar sessões de teste anteriores
- Crescimento da base de conhecimento com padrões específicos do projeto
- Evolução de templates baseada em feedback
- Continuidade entre sessões

### Localização

**Localização Principal do Agente:**
`{project-root}/.bmad/custom/src/agents/dynamics-qa-expert/`

**Localização do Sidecar:**
`{project-root}/.bmad/custom/src/agents/dynamics-qa-expert/dynamics-qa-expert-sidecar/`

**Estrutura Completa:**
```
dynamics-qa-expert/
├── dynamics-qa-expert.agent.yaml (to be created)
└── dynamics-qa-expert-sidecar/
    ├── memories.md ✅
    ├── instructions.md ✅
    ├── knowledge/
    │   ├── project-patterns.md ✅
    │   ├── test-templates.md ✅
    │   └── best-practices.md ✅
    └── workflows/
        └── (workflows to be created in next steps)
```

### Recursos do Workspace

**Persistência de Memória:** Marcos lembrará conversas, plugins/funções testados e preferências do usuário entre sessões

**Crescimento de Conhecimento:** A base se expandirá conforme Marcos aprende padrões e convenções específicas do projeto

**Aprendizado Adaptativo:** Templates e padrões evoluem com gerações de teste bem-sucedidas e feedback

**Privacidade e Segurança:** Restrições de domínio garantem que arquivos do sidecar permaneçam privados enquanto a geração de testes ocorre nas pastas apropriadas do projeto

---

**Status:** Workspace configurado com sucesso e pronto para finalização do agente! 🎉

### Uso no Chat

Para interagir com o agente no chat e acionar seus comandos:
- Mensagens livres (conversa): o agente usa memória e conhecimento para orientar.
- Comandos estruturados (menu): use os triggers abaixo.

Comandos disponíveis (triggers):
- `generate-tests` — Gera testes unitários para plugins e Azure Functions.
- `analyze-plugin` — Analisa código e sugere estrutura de testes, sem gerar.
- `review-tests` — Revisa testes existentes e recomenda melhorias.
- `coverage-report` — Gera relatório de cobertura.
- `teach` — Ensina boas práticas.
- `learn` — Atualiza padrões na base de conhecimento.
- `recall-patterns` — Mostra padrões aprendidos.

Exemplos de prompts:
- "generate-tests para o plugin Create de Account"
- "analyze-plugin em `src/AvaEdu/Plugins/CreatePlugin.cs`"
- "teach práticas de teste para Azure Functions HTTP"
- "recall-patterns"

Checklist de integração:
- Triggers definidos em `agent-yaml-BMADV3.md` mapeiam para workflows/ações.
- Workflows `generate-tests.md` e `analyze-plugin.md` existem no sidecar.
- Base de conhecimento `knowledge/test-templates.md` está atualizada (NUnit).
