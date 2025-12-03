# Workflow: analyze-plugin

Objetivo: Analisar código de plugins e Azure Functions e sugerir estrutura de testes sem gerar código.

Passos:
1. Detectar projetos em `src/` e localizar:
	- `Plugins/` (`IPlugin`), `Functions/`, `Services/`, `Repositories/`.
2. Parse de nomes e assinaturas para inferir:
	- Mensagens (Create/Update/Delete), estágios (pré/pós), dependências.
	- Pontos de validação e possíveis exceções.
3. Sugerir cenários de teste:
	- Caminho feliz, entradas inválidas, exceções, performance/tempo, idempotência.
4. Mapear templates de `knowledge/test-templates.md` aos alvos.
5. Emitir plano de testes em `.bmad/qa-reports/analyze-plugin-plan.md`.

Saída:
- Lista de classes alvo e seus cenários sugeridos.
- Recomendações de pacotes e estrutura de pastas `src/<Projeto>.Tests/`.
