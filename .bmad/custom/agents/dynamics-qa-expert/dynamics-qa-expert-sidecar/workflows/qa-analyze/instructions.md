# QA Analisar - Instruções de Análise Profunda do Projeto

<critical>The workflow execution engine is governed by: {project-root}/.bmad/core/tasks/workflow.xml</critical>
<critical>Este é um workflow de DOCUMENTO - gera relatório de análise em markdown em {default_output_file}</critical>
<critical>Comunique-se em {communication_language}</critical>

<workflow>

<step n="1" goal="Inicializar escopo de análise">
  <ask>Qual é o caminho do seu código-fonte do Dynamics 365? (Padrão: {default_source_path})</ask>
  <action>Armazenar caminho como {{source_path}}</action>
  
  <ask>Profundidade da análise: {pipeline_analysis_depth} - Alterar? [s/n]</ask>
  <action if="yes">Perguntar: rápida, padrão ou profunda</action>
  <action>Armazenar como {{analysis_depth}}</action>
  
  <action>Explicar o que será analisado:
    - Rápida: Inventário básico de artefatos e relacionamentos
    - Padrão: Mapeamento completo de dependências, ordem de execução, anti-padrões
    - Profunda: Tudo + análise de complexidade, pontuação de risco, detecção de edge cases
  </action>
</step>

<step n="2" goal="Escanear e inventariar artefatos">
  <action>Escanear recursivamente {{source_path}} por todas customizações do Dynamics:</action>
  
  <action>Encontrar Plugins:
    - Search for classes implementing IPlugin interface
    - Extract: Name, namespace, target entity, message, stage, execution mode
    - Parse registered steps if plugin registration code exists
  </action>
  
  <action>Encontrar Custom Workflow Activities:
    - Search for classes inheriting CodeActivity
    - Extract: Name, input/output parameters, business logic summary
  </action>
  
  <action>Encontrar Custom APIs:
    - Search for Custom API definitions (JSON, code, or comments)
    - Extract: Name, bound entity, parameters, permissions
  </action>
  
  <action>Encontrar PCF Controls:
    - Look for PCF project structures
    - Extract: Control name, properties, events
  </action>
  
  <action>Encontrar JavaScript Web Resources:
    - Look for .js files with Dataverse context usage
    - Extract: Form events, ribbon commands, custom logic
  </action>
  
  <template-output>artifact_inventory</template-output>
</step>

<step n="3" goal="Mapear dependências e relacionamentos">
  <action>Para cada artefato, identificar dependências:</action>
  
  <action>Dependências diretas:
    - Entity references (lookup fields)
    - Shared utility classes
    - External service calls
    - Configuration entities
  </action>
  
  <action>Dependências de execução:
    - Plugin A updates field that triggers Plugin B
    - Plugin creates record that starts Power Automate
    - Form script validates data before plugin runs
  </action>
  
  <action>Construir grafo de dependências:
    - Nodes: Artifacts
    - Edges: Dependencies (with type: data, execution, utility)
    - Identify circular dependencies
    - Identify single points of failure
  </action>
  
  <template-output>dependency_map</template-output>
</step>

<step n="4" goal="Analisar ordem de execução do pipeline">
  <action>Para cada entidade que possui customizações:</action>
  
  <action>Build execution timeline for each message (Create, Update, Delete):
    1. PreValidation sync plugins
    2. PreOperation sync plugins
    3. Core platform operation
    4. PostOperation sync plugins
    5. PostOperation async plugins
    6. Power Automate/workflows
  </action>
  
  <action>Identificar conflitos de execução:
    - Multiple plugins on same stage/message for same entity
    - Order dependencies not explicitly set
    - Sync plugin that should be async (long-running)
    - Async plugin that should be sync (immediate validation)
  </action>
  
  <action>Identificar riscos de loop infinito:
    - Plugin A updates entity X, triggering Plugin B
    - Plugin B updates entity X, triggering Plugin A again
    - No depth check or depth > reasonable threshold
  </action>
  
  <action>Mapear requisitos de imagens:
    - Which plugins need PreImage
    - Which plugins need PostImage
    - Missing image registrations
  </action>
  
  <template-output>pipeline_analysis</template-output>
</step>

<step n="5" goal="Detectar anti-padrões e issues de qualidade">
  <action>Executar detecção abrangente de anti-padrões baseada em {antipattern_strictness}:</action>
  
  <action>Escanear cada plugin por issues:</action>
  
  <check category="Crítico">
    - Missing depth validation (infinite loop risk)
    - Unhandled exceptions (no try-catch)
    - Synchronous plugin with HTTP calls (timeout risk)
    - Missing null checks on entity attributes
    - Accessing images without validation
  </check>
  
  <check category="Alto">
    - Queries without pagination
    - Queries with AllColumns (performance)
    - Hardcoded GUIDs
    - Missing ITracingService (no diagnostics)
    - Complex business logic in PreValidation (should be PreOperation)
  </check>
  
  <check category="Médio">
    - Inconsistent error messages
    - Missing XML documentation
    - Long methods (>100 lines)
    - High cyclomatic complexity (>10)
    - Repeated code blocks
  </check>
  
  <check category="Baixo" if="strictness=strict">
    - Variable naming conventions
    - Code formatting inconsistencies
    - Missing unit tests
    - No logging strategy
  </check>
  
  <action>Para cada issue encontrada:
    - Record location (file, line number, method)
    - Describe the problem
    - Explain why it's an issue
    - Provide fix recommendation with code example
    - Assign severity score (1-10)
  </action>
  
  <template-output>quality_issues</template-output>
</step>

<step n="6" goal="Calcular métricas de qualidade" if="analysis_depth=deep">
  <action>Calcular métricas abrangentes de qualidade:</action>
  
  <metric name="Estimativa de Cobertura de Testes">
    - Count artifacts with corresponding test files
    - Estimate: (artifacts_with_tests / total_artifacts) * 100
  </metric>
  
  <metric name="Densidade de Anti-Padrões">
    - Count total issues per category
    - Calculate: issues_per_1000_lines_of_code
  </metric>
  
  <metric name="Pontuação de Complexidade">
    - Average cyclomatic complexity across all methods
    - Identify top 10 most complex methods
  </metric>
  
  <metric name="Pontuação de Risco do Pipeline">
    - Factor in: circular dependencies, missing depth checks, execution conflicts
    - Scale: 0-100 (100 = highest risk)
  </metric>
  
  <metric name="Índice de Manutenibilidade">
    - Based on: complexity, documentation, code duplication
    - Scale: 0-100 (100 = most maintainable)
  </metric>
  
  <template-output>quality_metrics</template-output>
</step>

<step n="7" goal="Gerar recomendações">
  <action>Com base na análise, gerar recomendações priorizadas:</action>
  
  <priority level="P0 - Crítico">
    - Fix infinite loop risks immediately
    - Add missing depth validation
    - Handle unhandled exceptions
    - Fix missing null checks
  </priority>
  
  <priority level="P1 - Alto">
    - Add pagination to queries
    - Move sync plugins to async where appropriate
    - Remove hardcoded GUIDs
    - Add ITracingService for monitoring
  </priority>
  
  <priority level="P2 - Médio">
    - Improve code organization
    - Add missing documentation
    - Refactor complex methods
    - Add unit tests for critical paths
  </priority>
  
  <priority level="P3 - Baixo">
    - Standardize naming conventions
    - Improve code formatting
    - Add more comprehensive logging
  </priority>
  
  <action>For each recommendation:
    - Estimated effort (hours)
    - Expected impact (high/medium/low)
    - Dependencies or prerequisites
  </action>
  
  <template-output>recommendations</template-output>
</step>

<step n="8" goal="Gerar sugestões de visualização" optional="true">
  <ask>Gostaria de sugestões para visualizar esta análise? [s/n]</ask>
  
  <action if="yes">
    Provide suggestions for:
    - Dependency graph visualization (Graphviz, Mermaid)
    - Pipeline execution diagrams
    - Quality metrics dashboard
    - Trend tracking over time
  </action>
  
  <template-output>visualization_guide</template-output>
</step>

<step n="9" goal="Finalizar relatório de análise">
  <action>Revisar documento de análise completo</action>
  <action>Garantir que todas as seções estejam completas</action>
  <action>Adicionar sumário</action>
  <action>Adicionar resumo executivo</action>
  <action>Salvar relatório final em: {default_output_file}</action>
  
  <action>Informar usuário:
    - Análise concluída! 🎉
    - Relatório salvo em: {default_output_file}
    - Revisar recomendações e priorizar correções
    - Utilize [CR] Revisão de Código para análise focada de anti-padrões
    - Utilize [GT] Gerar Testes para melhorar cobertura
  </action>
</step>

</workflow>
