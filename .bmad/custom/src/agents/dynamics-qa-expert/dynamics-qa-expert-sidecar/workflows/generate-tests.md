# Workflow: generate-tests

Objetivo: Gerar testes unitários com NUnit para plugins Dynamics 365 e Azure Functions, detectando o projeto em `src/` e criando/atualizando `src/<Projeto>.Tests/`.

Passos:
1. Detectar projetos em `src/` (ex.: `src/AvaEdu/`).
   - Identificar o(s) projeto(s) principal(is) dentro de `src/` e selecionar o alvo padrão quando houver apenas um.
   - Validar existência de `*.csproj` e inferir o tipo do projeto (Plugins Dynamics, Azure Functions) a partir da estrutura.
2. Mapear contexto aplicável:
	- `Plugins/` (classes que implementam `IPlugin`)
	- `Services/` e `Repositories/` (dependências utilizadas pelos plugins/funções)
	- `Functions/` (Azure Functions — se existirem)
3. Analisar todo o projeto, estrutura e contexto:
   - Ler classes em `Plugins/`, `Services/`, `Repositories/`, `Functions/` e `Constants/` para entender fluxos e dependências.
   - Identificar pontos de entrada, validações, efeitos colaterais e integrações.
   - Classificar cenários por tipo (pré-validação, pós-operação, Create/Update/Delete, HTTP/Queue triggers, etc.).
4. Para cada plugin/função identificado:
	- Inferir métodos, objetivos e cenários com base em nomes e uso.
	- Selecionar template adequado de `knowledge/test-templates.md`.
	- Gerar classe de teste NUnit em `src/<Projeto>.Tests/` seguindo convenções.
5. Garantir referências de pacotes:
	- NUnit, Moq, FakeXrmEasy (plugins), pacotes necessários das functions.
6. Criar casos:
	- Caminho feliz
	- Entradas inválidas e exceções
	- Pré-validação/pós-operação (plugins)
	- Idempotência (funções, quando aplicável)
7. Adicionar documentação mínima e checklist de qualidade.
8. Executar testes e emitir relatório de resultado:
   - Rodar `dotnet test` no projeto de testes gerado ou existente.
   - Coletar `TestResult.trx` e extrair contagem: total, aprovados, falhos.
   - Exibir resumo: quantos foram feitos, quantos passaram e quantos falharam.
9. Emitir relatório de arquivos gerados e passos pendentes.

Convenções:
- Naming: `PluginName_Metodo_Scenario` e `FunctionName_Cenario`.
- Pastas: `src/<Projeto>.Tests/Plugins` e `src/<Projeto>.Tests/Functions`.
- Mantém testes independentes e rápidos.

Saída:
- Arquivos `.cs` de teste criados/atualizados em `src/<Projeto>.Tests/`.
- Relatório em `.bmad/qa-reports/generate-tests-report.md`.
- Execução dos testes concluída com resumo de contagens (total, pass, fail) e cópia de `TestResult.trx` em `src/<Projeto>/reports/` quando aplicável.
