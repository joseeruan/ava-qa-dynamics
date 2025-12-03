## Agent Purpose and Type

### Core Purpose

Criar testes unitários para plugins do Dynamics 365, automatizando a geração de testes seguindo melhores práticas e frameworks como FakeXrmEasy e Moq. O agente analisa código de plugins existentes e gera classes de teste completas com mocks, configurações necessárias, e cobertura de diferentes cenários (pré-validação, pós-operação, eventos Create/Update/Delete).

### Target Users

Desenvolvedores Dynamics 365 e QA Engineers que precisam criar e manter testes unitários para plugins customizados.

### Chosen Agent Type

**Expert Agent** - Agente com memória persistente e base de conhecimento pessoal.

**Rationale:**
- Aprende padrões de teste específicos do projeto ao longo do tempo
- Lembra dos plugins já testados e seus contextos
- Mantém conhecimento sobre convenções de nomenclatura e estrutura
- Base de conhecimento pessoal com patterns de teste específicos
- Evolui com feedback sobre testes gerados anteriormente
- Domínio restrito para segurança (opera apenas em sidecar)

### Output Path

Standalone Expert Agent em: `{project-root}/.bmad/custom/src/agents/dynamics-qa-expert/`

Estrutura:
- `dynamics-qa-expert.agent.yaml` (configuração principal)
- `sidecar/memories.md` (memória persistente)
- `sidecar/knowledge/` (base de conhecimento de testes)
- `info-and-installation-guide.md` (guia de uso)

### Key Capabilities

- Gerar testes unitários para plugins usando FakeXrmEasy/Moq
- Criar mocks para IOrganizationService e contextos de execução
- Validar cobertura de cenários (pré-validação, pós-operação, etc.)
- Gerar testes para diferentes eventos (Create, Update, Delete)
- Verificar tratamento de exceções e validações
- Criar documentação dos testes gerados
- Aprender e adaptar-se aos padrões do projeto

### Context from Brainstorming

Brainstorming foi pulado - usuário tinha ideia clara do agente necessário.
