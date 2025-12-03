# QA Revisão de Código - Instruções de Análise Automática de Qualidade

<critical>O mecanismo de execução de workflows é regido por: {project-root}/.bmad/core/tasks/workflow.xml</critical>
<critical>Este é um workflow de DOCUMENTO - gera relatório de revisão em markdown em {default_output_file}</critical>
<critical>Comunique-se em {communication_language}</critical>

<workflow>

<step n="1" goal="Definir escopo da revisão">
  <ask>O que você gostaria de revisar?
    1. Projeto completo (todo o código)
    2. Mudanças recentes (git diff)
    3. Arquivos ou componentes específicos
    4. Pull request / comparação de branch
  </ask>
  <action>Armazenar como {{review_scope}}</action>
  
  <action if="scope=full project">
    <ask>Caminho do código-fonte? (Padrão: {default_source_path})</ask>
    <action>Armazenar como {{review_path}}</action>
  </action>
  
  <action if="scope=recent changes">
    <ask>Comparar com qual branch? (Padrão: main)</ask>
    <action>Armazenar como {{base_branch}}</action>
    <action>Usar git diff para identificar arquivos alterados</action>
    <action>Armazenar arquivos alterados como {{review_files}}</action>
  </action>
  
  <action if="scope=specific files">
    <ask>Digite caminhos de arquivos (separados por vírgula) ou nomes de classes:</ask>
    <action>Armazenar como {{review_files}}</action>
  </action>
  
  <action if="scope=pull request">
    <ask>Número do PR ou nome do branch:</ask>
    <action>Buscar diff do PR e extrair arquivos alterados</action>
    <action>Armazenar como {{review_files}} e {{pr_context}}</action>
  </action>
  
  <ask>Rigor da revisão: {antipattern_strictness} - Alterar? [s/n]</ask>
  <action if="yes">Perguntar: relaxado, balanceado ou estrito</action>
  <action>Armazenar como {{review_strictness}}</action>
</step>

<step n="2" goal="Carregar base de anti-padrões">
  <action>Carregar definições de anti-padrões de: {data_path}/dynamics-antipatterns.json</action>
  <action>Carregar boas práticas de: {data_path}/best-practices.json</action>
  
  <action>Filter patterns based on {{review_strictness}}:
    - Relaxed: Only critical severity
    - Balanced: Critical + high severity
    - Strict: All severity levels
  </action>
  
  <action>Preparar regras de detecção para:
    - Validação de profundidade ausente
    - Exceções não tratadas
    - Consultas sem paginação
    - GUIDs hardcoded
    - Verificações de null ausentes
    - Chamadas HTTP síncronas
    - Tracing ausente
    - Acesso a imagens sem validação
    - E mais de 20 padrões...
  </action>
</step>

<step n="3" goal="Analisar código por anti-padrões">
  <action>Para cada arquivo em {{review_files}}:</action>
  
  <action>Analisar estrutura do código:
    - Identificar classes e métodos
    - Extrair fluxo lógico
    - Mapear dependências
    - Calcular complexidade ciclomática
  </action>
  
  <action>Executar detecção de anti-padrões:</action>
  
  <check category="Critical" severity="10">
    <pattern id="missing-depth-check">
      <detection>Método Execute de plugin sem validação de context.Depth</detection>
      <location>{{file}}:{{line}}</location>
      <description>Sem verificação de depth há risco de loop infinito</description>
      <fix>Adicionar no início do Execute: if (context.Depth > 1) return;</fix>
      <example>
        ```csharp
        // Before (DANGEROUS)
        public void Execute(IServiceProvider serviceProvider) {
            var context = (IPluginExecutionContext)serviceProvider.GetService(...);
            // No depth check!
        }
        
        // After (SAFE)
        public void Execute(IServiceProvider serviceProvider) {
            var context = (IPluginExecutionContext)serviceProvider.GetService(...);
            if (context.Depth > 1) return; // Prevent infinite loops
        # QA Review – Instruções (pt-BR)

        Este fluxo orienta a revisão técnica e funcional de Plugins do Dynamics 365/Dataverse e Funções Azure relacionadas, priorizando legibilidade, corretude, integridade e manutenção. Utilize Português-Brasil.

        ## Objetivos
        - Identificar problemas de arquitetura, legibilidade e manutenção.
        - Detectar antipadrões e riscos funcionais.
        - Validar integridade das regras de negócio e contratos (entrada/saída).
        - Sugerir melhorias objetivas com exemplos práticos.

        ## Escopo
        - Plugins Dynamics 365/Dataverse (C#).
        - Funções Azure que integram com Dataverse.
        - Testes automatizados com prioridade para NUnit; uso de FakeXrmEasy e Moq.

        ## Checklist de Revisão
        1. Arquitetura e Organização
          - Separação de responsabilidades (SRP), baixo acoplamento, nomeação clara.
          - Interfaces e injeção de dependências quando aplicável.
          - Evitar lógica extensa em `Execute`; preferir serviços/fábricas auxiliares.
        2. Legibilidade e Manutenção
          - Métodos curtos, nomes descritivos, early return para reduzir complexidade.
          - Comentários apenas quando agregam contexto; evitar comentários redundantes.
          - Padronização de estilos e convenções do projeto.
        3. Correção Funcional
          - Pré-condições: nulos, ranges, tipos, formatos bem validados.
          - Tratamento de exceções com mensagens úteis sem vazar detalhes sensíveis.
          - Validação de parâmetros de entrada e consistência da saída.
        4. Integridade e Regras de Negócio
          - Enumerar invariantes e regras que devem se manter sempre.
          - Verificar efeitos colaterais: criação/atualização/exclusão em entidades corretas.
          - Garantir idempotência quando necessário; evitar inconsistências transacionais.
        5. Desempenho e Resiliência
          - Minimizar chamadas desnecessárias ao Dataverse; usar batching quando aplicável.
          - Retentativas com backoff para operações externas (quando apropriado).
          - Timeouts e limites adequados; circuit breakers onde cabível.
        6. Segurança e Conformidade
          - Sem credenciais em código; usar configuração segura.
          - Controle de acesso e escopo corretamente aplicados.
          - Sanitização de entradas e logs sem dados sensíveis.
        7. Testes Automatizados (NUnit prioritário)
          - Cobrir todos os métodos públicos relevantes, inclusive caminhos de erro.
          - Usar FakeXrmEasy e Moq para isolar dependências.
          - Incluir asserts de integridade cobrindo invariantes e contratos.

        ## Itens Específicos – Plugins
        - Validar estágio `PreOperation`/`PostOperation` e contexto de mensagem (Create/Update/Delete).
        - Checar uso de `IOrganizationService` com `CallerId` correto quando necessário.
        - Mapear entidades e atributos obrigatórios; prevenir `InvalidPluginExecutionException` inconsistente.
        - Evitar `Service Fault` genérico; usar mensagens claras e categorizadas.

        ## Itens Específicos – Azure Functions
        - Triggers e Bindings corretos (HTTP/Queue/ServiceBus); modelos de request/response.
        - `HttpClient`: `HttpMessageHandler` mockável, timeouts, reutilização e DI.
        - Validação de headers, autenticação e payloads; códigos de status previsíveis.
        - Integração com Dataverse: tratamento de falhas, retentativas e consistência.

        ## Saída Esperada
        - Lista de problemas priorizada (Alta/Média/Baixa) com exemplos.
        - Recomendações específicas com snippets (NUnit quando aplicável).
        - Itens de ação: correções rápidas e plano de refatoração.

        ## Exemplo de Recomendação (NUnit)
        ```csharp
        [Test]
        public void Deve_Lancar_Excecao_Quando_Entrada_Invalida()
        {
           // Arrange
           var sut = new MeuPlugin(servico, contextoValido);
           object entrada = null;

           // Act & Assert
           Assert.Throws<InvalidPluginExecutionException>(() => sut.Execute(entrada));
        }
        ```

        ## Checklist Complementar de Integridade (NUnit)
        - Cobertura de invariantes: cada regra de negócio possui ao menos um teste de violação e um de conformidade.
        - Contratos de I/O: validar tipos, formatos e estados resultantes após operação.
        - Efeitos colaterais: asserts em entidades/atributos alterados e eventos gerados.
        - Exceções: usar `Assert.Throws`/`Assert.DoesNotThrow` para caminhos negativos/positivos.

        ## Próximos Passos
        - Se faltarem testes: utilize o fluxo “QA Gerar Testes” (NUnit) para cobrir métodos e invariantes.
        - Problemas de arquitetura: planejar refatorações graduais, criar serviços auxiliares e reduzir complexidade de `Execute`.
        - Para Funções Azure: adicionar testes com `HttpMessageHandler` mockado e validar códigos de status e headers.
      <detection>Code blocks repeated > 2 times</detection>
      <location>Multiple locations</location>
      <description>Duplication makes maintenance difficult</description>
      <fix>Extract to shared method or utility class</fix>
    </pattern>
    
    <pattern id="missing-documentation">
      <detection>Public methods without XML documentation</detection>
      <location>{{file}}:{{line}}</location>
      <description>Lack of documentation hinders maintainability</description>
      <fix>Add /// XML comments explaining purpose and parameters</fix>
    </pattern>
  </check>
  
  <check category="Low" severity="1-3" if="strictness=strict">
    <pattern id="naming-convention">
      <detection>Variables not following camelCase/PascalCase</detection>
      <location>{{file}}:{{line}}</location>
      <description>Inconsistent naming reduces readability</description>
      <fix>Follow C# naming conventions</fix>
    </pattern>
    
    <pattern id="unused-variable">
      <detection>Declared variables never used</detection>
      <location>{{file}}:{{line}}</location>
      <description>Dead code clutters codebase</description>
      <fix>Remove unused declarations</fix>
    </pattern>
    
    <pattern id="commented-code">
      <detection>Large blocks of commented-out code</detection>
      <location>{{file}}:{{line}}</location>
      <description>Commented code should be removed (use source control)</description>
      <fix>Delete commented blocks, trust git history</fix>
    </pattern>
  </check>
  
  <action>Registrar cada issue detectada com:
    - Pattern ID
    - File and line number
    - Code snippet (3 lines context)
    - Severity score (1-10)
    - Description of problem
    - Fix recommendation
    - Code example (before/after)
  </action>
  
  <template-output>antipattern_detection_results</template-output>
</step>

<step n="4" goal="Calcular pontuação de qualidade">
  <action>Calcular pontuação geral de qualidade (0-100):</action>
  
  <calculation>
    Base score: 100
    
    For each issue detected:
      - Critical (severity 10): -10 points
      - High (severity 7-9): -5 points
      - Medium (severity 4-6): -2 points
      - Low (severity 1-3): -0.5 points
    
    Minimum score: 0
    Maximum score: 100
  </calculation>
  
  <action>Calcular distribuição por categoria:</action>
  <metric>Code Safety Score: Based on critical issues (depth checks, exception handling, null checks)</metric>
  <metric>Performance Score: Based on query optimization, pagination, column sets</metric>
  <metric>Maintainability Score: Based on complexity, documentation, code organization</metric>
  <metric>Best Practices Score: Based on tracing, naming, patterns adherence</metric>
  
  <action>Determinar nota de qualidade:
    - A (90-100): Excellent
    - B (80-89): Good
    - C (70-79): Acceptable
    - D (60-69): Needs Improvement
    - F (0-59): Critical Issues
  </action>
  
  <template-output>quality_score</template-output>
</step>

<step n="5" goal="Analisar cobertura de testes" optional="true">
  <ask>Checar cobertura de testes para componentes revisados? [s/n]</ask>
  
  <action if="yes">
    <action>For each reviewed class:
      - Look for corresponding test file ({{ClassName}}Tests.cs)
      - If found: Analyze test completeness
      - If not found: Flag as "No tests"
    </action>
    
    <action>Calcular métricas de cobertura:
      - Classes with tests: {{tested_classes}}/{{total_classes}}
      - Methods with tests: {{tested_methods}}/{{total_methods}}
      - Estimated line coverage: {{estimated_coverage}}%
    </action>
  </action>
  
  <template-output>test_coverage_analysis</template-output>
</step>

<step n="6" goal="Gerar recomendações priorizadas">
  <action>Ordenar todas as issues detectadas por severidade</action>
  <action>Agrupar por categoria (Segurança, Performance, Manutenibilidade, Boas Práticas)</action>
  
  <action>Generate action plan:</action>
  
  <priority level="URGENT" color="red">
    - All critical issues (severity 10)
    - Must fix before deployment
    - Estimated effort per fix
  </priority>
  
  <priority level="HIGH" color="orange">
    - High severity issues (7-9)
    - Should fix this sprint
    - Estimated effort per fix
  </priority>
  
  <priority level="MEDIUM" color="yellow">
    - Medium severity issues (4-6)
    - Plan for next sprint
    - Can be batched together
  </priority>
  
  <priority level="LOW" color="green">
    - Low severity issues (1-3)
    - Add to backlog
    - Address during refactoring
  </priority>
  
  <action>Calculate total estimated effort (hours) for each priority level</action>
  
  <template-output>prioritized_recommendations</template-output>
</step>

<step n="7" goal="Gerar métricas de comparação" if="review_scope=recent changes OR pull request">
  <action>Comparar estado atual com anterior:</action>
  
  <metric>Issues introduced in this change: {{new_issues_count}}</metric>
  <metric>Issues fixed in this change: {{fixed_issues_count}}</metric>
  <metric>Net quality change: {{quality_delta}}</metric>
  
  <action>Determine if quality improved, degraded, or stayed same</action>
  
  <action if="PR context">
    <action>Gerar resumo de comentários da revisão de PR:
      - Avaliação geral (Aprovar / Solicitar mudanças / Comentar)
      - Bloqueadores críticos (se houver)
      - Melhorias sugeridas
      - Feedback positivo sobre boas práticas
    </action>
  </action>
  
  <template-output>comparison_metrics</template-output>
</step>

<step n="8" goal="Finalizar relatório de revisão de código">
  <action>Compilar relatório de revisão completo</action>
  <action>Adicionar resumo executivo</action>
  <action>Incluir todas as seções: pontuação, issues, recomendações, métricas</action>
  <action>Adicionar visualizações (gráficos) se aplicável</action>
  <action>Salvar em: {default_output_file}</action>
  
  <action>Inform user:
    - Code review complete! 🧪
    - Quality Score: {{overall_score}}/100 (Grade {{grade}})
    - Critical Issues: {{critical_count}}
    - Report saved to: {default_output_file}
  </action>
</step>

<step n="9" goal="Fornecer próximos passos acionáveis">
  <action>Com base nos resultados da revisão, sugerir:</action>
  
  <action if="critical issues found">
    ⚠️ URGENTE: Resolver {{critical_count}} issues críticas imediatamente
    Use o relatório para localizar e corrigir cada issue
    Re-execute a revisão após as correções
  </action>
  
  <action if="no tests found">
    💡 Gerar testes usando o workflow [GT] Gerar Testes
    Recomendado para {{untested_classes_count}} classes sem teste
  </action>
  
  <action if="score < 70">
    📈 Considerar sessão de refatoração
    Focar nas {{top_issues_count}} issues de maior impacto
    Acompanhar melhoria com revisões mensais
  </action>
  
  <action if="score >= 90">
    ✨ Qualidade de código excelente!
    Manter padrões com revisões automatizadas em CI/CD
    Compartilhar boas práticas com a equipe
  </action>
  
  <ask>Gostaria de:
    - [V] Ver relatório completo
    - [F] Corrigir issues agora (abrir arquivos no editor)
    - [R] Reexecutar revisão (após correções)
    - [E] Sair
  </ask>
  
  <action if="V">Display complete report</action>
  <action if="F">Open flagged files with issue annotations</action>
  <action if="R">Restart workflow from step 1</action>
  <action if="E">Thank user and exit</action>
</step>

</workflow>
