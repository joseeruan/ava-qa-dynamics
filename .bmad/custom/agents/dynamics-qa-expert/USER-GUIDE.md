# 🧪 Dynamics QA Expert - Guia do Usuário

## 📋 Visão Geral

O **Dynamics QA Expert** é um agente especializado em gerar e revisar testes unitários para plugins do Microsoft Dynamics 365, usando NUnit e FakeXrmEasy.

## 🚀 Como Começar

### 1. Ativar o Agente

```
@dynamics-qa-expert
```

### 2. Comandos Disponíveis

| Comando | Descrição | Quando Usar |
|---------|-----------|-------------|
| `*generate-tests` | Gera testes unitários completos | Quando você tem plugins sem testes |
| `*analyze-plugin` | Analisa e sugere estrutura de testes | Antes de gerar para planejar |
| `*review-tests` | Revisa testes existentes | Para melhorar qualidade de testes |
| `*quick-setup` | Setup completo de ambiente | Primeiro uso no projeto |
| `*learn` | Salva padrões do projeto | Após fazer testes manualmente |
| `*recall-patterns` | Mostra padrões aprendidos | Para manter consistência |
| `*save-session` | Salva contexto da sessão | Ao final de cada sessão |
| `*link-project` | Detecta projeto .sln | Para configurar paths |
| `*nunit-setup` | Configura NUnit | Se ainda não tem projeto de testes |

## 📖 Fluxos de Trabalho Comuns

### Cenário 1: Primeiro Uso (Projeto Novo)

```
1. @dynamics-qa-expert
2. *quick-setup
   → Analisa projeto
   → Cria estrutura de testes
   → Gera todos os testes
3. *save-session
```

### Cenário 2: Adicionar Testes para Plugin Específico

```
1. @dynamics-qa-expert
2. *generate-tests
   → Escolher "Arquivos específicos"
   → Informar caminho do plugin
3. *save-session
```

### Cenário 3: Revisar e Melhorar Testes Existentes

```
1. @dynamics-qa-expert
2. *review-tests
   → Escolher escopo
   → Receber relatório
   → Aplicar sugestões
3. *save-session
```

### Cenário 4: Análise Antes de Implementar

```
1. @dynamics-qa-expert
2. *analyze-plugin
   → Ver estrutura sugerida
   → Identificar cenários críticos
3. Implementar manualmente ou usar *generate-tests
```

## 💡 Melhores Práticas

### ✅ FAÇA

- **Sempre execute `*save-session`** ao final para não perder contexto
- **Use `*quick-setup`** no primeiro uso para configurar tudo
- **Execute `*analyze-plugin`** antes de gerar para entender o plano
- **Revise testes gerados** e ajuste conforme necessário
- **Use `*learn`** após criar testes manualmente para ensinar o agente

### ❌ NÃO FAÇA

- Não feche sem salvar (`*save-session`)
- Não gere testes sem analisar primeiro em projetos complexos
- Não aceite testes sem compilar e validar
- Não ignore sugestões do `*review-tests`

## 🔧 Solução de Problemas

### Problema: "Agente não encontra meu projeto"

**Solução:**
```
*link-project
```
Isso detectará automaticamente `.sln` e `.csproj` em `src/`

### Problema: "Testes não compilam"

**Causa Comum:** Versões de pacotes incompatíveis

**Solução:** Verifique que usa as versões obrigatórias:
- `Microsoft.CrmSdk.CoreAssemblies 9.0.2.*`
- `NUnit 3.13.3`
- `FakeXrmEasy.365 1.58.1`

### Problema: "Testes falham aleatoriamente no CI/CD"

**Causa Comum:** Testes não são independentes

**Solução:** Execute `*review-tests` e procure por:
- Variáveis estáticas compartilhadas
- Estado compartilhado entre testes
- Dependências de ordem de execução

## 🎓 Recursos de Aprendizado

### Padrões do TestArch

O agente tem acesso aos seguintes padrões universais:

- **data-factories.md**: Factory functions para dados de teste
- **error-handling.md**: Testes de exceções
- **fixture-architecture.md**: Estrutura de base classes
- **test-quality.md**: Princípios de qualidade
- **contract-testing.md**: Validação de schemas

### Memórias e Aprendizado

O agente aprende com seu projeto:
- Armazena padrões de nomenclatura
- Lembra de plugins testados anteriormente
- Adapta-se ao estilo do seu código

Localização: `.bmad/custom/agents/dynamics-qa-expert/dynamics-qa-expert-sidecar/`

## 📞 Dúvidas Frequentes

**P: O agente lembra de sessões anteriores?**  
R: Sim, se você executar `*save-session` ao final. O agente salva em `memories.md`.

**P: Posso ensinar o agente os padrões do meu projeto?**  
R: Sim! Use `*learn` após criar testes manualmente.

**P: Os testes gerados são prontos para produção?**  
R: São um excelente ponto de partida, mas sempre revise e ajuste conforme necessário.

**P: Funciona com xUnit ou MSTest?**  
R: O agente prioriza NUnit, mas pode adaptar-se. Informe sua preferência.

**P: E para Azure Functions integradas ao Dynamics?**  
R: Use `*azure-functions-tests` para ver diretrizes específicas.

## 🆘 Suporte

Se encontrar problemas:
1. Verifique este guia primeiro
2. Execute `*recall-patterns` para ver configuração atual
3. Execute `*setup-sidecar` para recriar estrutura se corrompida
4. Consulte a documentação em `docs/`

---

**Versão:** 1.0  
**Última Atualização:** 3 de Dezembro de 2025  
**Framework Principal:** NUnit 3.13.3
