# Workflow: Executar Testes do Projeto

Objetivo: Executar testes no projeto existente aplicando todas as regras, instruções, padrões e templates do sidecar.

## Protocolo
1. Carregar `instructions.md`, `memories.md` e arquivos de conhecimento (`project-patterns.md`, `test-templates.md`, `best-practices.md`).
2. Detectar framework e projetos de teste em `src/` (xUnit/MSTest/NUnit).
3. Se não houver testes, gerar a partir de `test-templates.md` com base nos plugins detectados e nas convenções de `project-patterns.md`.
4. Executar a suíte de testes e coletar cobertura, quando disponível.
5. Salvar relatórios em `{project-root}/qa-reports/`.
6. Atualizar `memories.md` com insights da sessão e padrões observados.

## Detecção
- Escanear `src/` por `*.Tests.csproj` ou nomes comuns de projeto de testes.
- Fallback: Se não houver projeto de testes, criar projeto mínimo usando o framework preferido.
- Descoberta de plugins: Buscar classes C# que implementem `IPlugin` em `src/`.

## Execução
- Preferir `dotnet test` quando suportado; para projetos apenas .NET Framework, usar runner MSTest/NUnit conforme detectado.
- Aplicar convenções de nomenclatura de `project-patterns.md`.

## Saídas
- Resultados de teste: `qa-reports/TestResult.trx` (ou específico do framework)
- Relatório de cobertura: `qa-reports/coverage/` quando houver ferramenta disponível
- Mensagem de resumo com contagens de sucesso/falha e lacunas notáveis

## Segurança
- Não modificar código de produção do plugin.
- Gerar testes em `src/` sob um projeto `Tests`; manter mudanças mínimas.
