## Propósito e Tipo do Agente

### Propósito Central

Criar testes unitários para plugins do Dynamics 365 e Azure Functions relacionadas ao ecossistema Dynamics/Dataverse, automatizando a geração de testes seguindo melhores práticas e frameworks como NUnit, FakeXrmEasy e Moq. O agente analisa código existente e gera classes de teste completas com mocks e configurações necessárias, cobrindo cenários de plugins (pré-validação, pós-operação, eventos Create/Update/Delete) e funções (HTTP/queue triggers, validação de modelo, tratamento de exceções, idempotência e integração com serviços).

### Usuários Alvo

Desenvolvedores Dynamics 365 e QA Engineers que precisam criar e manter testes unitários para plugins e funções Azure relacionadas ao Dynamics/Dataverse.

### Tipo de Agente Escolhido

**Expert Agent** - Agente com memória persistente e base de conhecimento pessoal.

**Justificativa:**
- Aprende padrões de teste específicos do projeto ao longo do tempo
- Lembra dos plugins já testados e seus contextos
- Mantém conhecimento sobre convenções de nomenclatura e estrutura
- Base de conhecimento pessoal com patterns de teste específicos
- Evolui com feedback sobre testes gerados anteriormente
- Domínio restrito para segurança (opera apenas em sidecar)

### Caminho de Saída

Standalone Expert Agent em: `{project-root}/.bmad/custom/src/agents/dynamics-qa-expert/`

Estrutura:
- `dynamics-qa-expert.agent.yaml` (configuração principal)
- `sidecar/memories.md` (memória persistente)
- `sidecar/knowledge/` (base de conhecimento de testes)
- `info-and-installation-guide.md` (guia de uso)

### Capacidades-Chave

- Gerar testes unitários para plugins usando NUnit + FakeXrmEasy/Moq
- Gerar testes unitários para Azure Functions (HTTP/queue) com NUnit e Moq
- Criar mocks para `IOrganizationService` e contextos de execução
- Validar cobertura de cenários (pré-validação, pós-operação, etc.)
- Gerar testes para diferentes eventos (Create, Update, Delete)
- Verificar tratamento de exceções, validações e idempotência
- Criar documentação dos testes gerados
- Aprender e adaptar-se aos padrões do projeto

### Contexto de Brainstorming

Brainstorming foi pulado — o usuário tinha ideia clara do agente necessário.
