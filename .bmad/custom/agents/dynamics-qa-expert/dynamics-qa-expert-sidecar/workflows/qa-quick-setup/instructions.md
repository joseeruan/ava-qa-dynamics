# QA Quick Setup – Geração Completa de Suite de Testes (pt-BR)

<critical>O mecanismo de execução de workflows é regido por: {project-root}/.bmad/core/tasks/workflow.xml</critical>
<critical>Comunique-se em {communication_language} durante todo o processo</critical>

<workflow>

<step n="1" goal="Boas-vindas e coleta de contexto">
  <action>Dar boas-vindas ao usuário ao QA Quick Setup</action>
  <action>Explicar que este fluxo irá:
    1. Analisar a estrutura do projeto Dynamics
    2. Criar projeto de testes com arquitetura adequada
    3. Gerar testes de unidade para todos os componentes
    4. Gerar testes de integração para fluxos críticos
    5. Fornecer relatório de qualidade e próximos passos
  </action>
  
  <ask>Qual é o caminho do código-fonte do Dynamics 365? (Padrão: {default_source_path})</ask>
  <action>Armazenar caminho como {{source_path}}</action>
  <action>Se o usuário aceitar o padrão, usar {default_source_path}</action>
  
  <ask>Confirmar local de saída dos testes: {test_output_location} - Está correto? [s/n]</ask>
  <action if="no">Solicitar caminho alternativo e armazenar como {{test_output_location}}</action>
</step>

<step n="2" goal="Analisar estrutura do projeto">
  <action>Examinar {{source_path}} por artefatos do Dynamics:</action>
  <action>• Procurar arquivos .cs com implementações de IPlugin</action>
  <action>• Procurar classes que herdam CodeActivity (workflows customizados)</action>
  <action>• Procurar definições de Custom API</action>
  <action>• Procurar projetos de controle PCF</action>
  <action>• Identificar namespaces e estrutura de projetos</action>
  
  <action>Criar inventário de artefatos:
    - Contar plugins encontrados
    - Contar atividades de workflow encontradas
    - Contar custom APIs encontradas
    - Contar controles PCF encontrados
    - Mapear dependências entre artefatos
  </action>
  
  <action>Detectar framework de teste existente se {auto_detect_framework} = "yes":
    - Verificar arquivos .csproj em diretórios de teste
    - Procurar referências de pacotes XUnit, NUnit ou MSTest
    - Se encontrado, sugerir uso do framework existente
  </action>
  
  <ask if="framework detected">Framework de teste detectado: {{detected_framework}}. Usar este no lugar do configurado {test_framework}? [s/n]</ask>
  <action if="yes">Definir {{active_framework}} = {{detected_framework}}</action>
  <action if="no">Definir {{active_framework}} = {test_framework}</action>
  <action if="no framework detected">Definir {{active_framework}} = {test_framework}</action>
  
  <template-output>analysis_summary</template-output>
</step>

<step n="3" goal="Executar detecção de antipadrões">
  <action>Analisar código para antipadrões comuns do Dynamics com base em {antipattern_strictness}:</action>
  
  <action>Para cada plugin encontrado:
    - Verificar validação de profundidade (context.Depth > 1)
    - Verificar uso de ITracingService
    - Verificar GUIDs hardcoded
    - Verificar tratamento correto de exceções
    - Verificar validação de imagens antes do acesso
    - Verificar null checks em atributos de entidade
  </action>
  
  <action>Para cada consulta encontrada:
    - Verificar paginação (TopCount ou PagingInfo)
    - Verificar especificação de ColumnSet (evitar AllColumns)
  </action>
  
  <action>Gerar relatório de antipadrões com:
    - Problemas críticos (corrigir obrigatoriamente)
    - Avisos (deveria corrigir)
    - Sugestões (bom ter)
    - Para cada item: localização, descrição, recomendação de correção
  </action>
  
  <template-output>antipattern_report</template-output>
</step>

<step n="4" goal="Criar estrutura do projeto de testes">
  <action>Criar diretório do projeto de testes: {{test_output_location}}</action>
  <action>Criar subdiretórios:
    - /UnitTests
    - /IntegrationTests
    - /Helpers
    - /Mocks
    - /TestData
  </action>
  
  <action>Criar arquivo de projeto de testes (.csproj) com:
    - Target framework: .NET 6.0 ou superior
    - Referências de pacote para {{active_framework}}
    - Referência de pacote para FakeXrmEasy (versão mais recente)
    - Referência de pacote para FluentAssertions
    - Referência de projeto para o projeto de origem
  </action>
  
  <action>Carregar templates de classes base de {templates_path}:
    - TestBase.cs - Classe base para todos os testes
    - FakeContextFactory.cs - Fábrica para contexto FakeXrmEasy
    - MockServiceProvider.cs - Provedor de serviços mock
    - TestEntityFactory.cs - Helper para criar entidades de teste
  </action>
  
  <action>Gerar classes auxiliares base em /Helpers:
    - Usar templates carregados acima
    - Customizar namespaces para combinar com o projeto
    - Adicionar comentários XML conforme {comment_level}
  </action>
  
  <template-output>test_project_structure</template-output>
</step>

<step n="5" goal="Gerar testes de unidade para plugins">
  <action>Para cada plugin descoberto no passo 2:</action>
  
  <action>Analisar código do plugin:
    - Identificar estágio registrado (PreValidation, PreOperation, PostOperation)
    - Identificar mensagem registrada (Create, Update, Delete, etc.)
    - Identificar entidade alvo
    - Interpretar a lógica do método Execute
    - Identificar ramos de decisão e caminhos
    - Identificar requisitos de imagens (PreImage/PostImage)
  </action>
  
  <action>Gerar classe de teste:
    - Nome da classe: {{PluginName}}Tests
    - Herdar de TestBase
    - Adicionar [TestFixture] ou [TestClass] conforme {{active_framework}}
  </action>
  
  <action>Gerar métodos de teste para cada caminho de execução:
    - Setup: Criar FakeContext, mocks de serviços, preparar entidade alvo
    - Act: Executar plugin
    - Assert: Verificar comportamento esperado
    - Adicionar nomes descritivos (ex.: "Execute_WhenAccountCreated_ShouldSetDefaultValues")
  </action>
  
  <action>Adicionar testes para cenários de erro:
    - Campos obrigatórios ausentes
    - Valores nulos
    - Dados inválidos
  </action>
  
  <action>Adicionar comentários conforme {comment_level}:
    - Detalhado: Explicar cada linha (Arrange, Act, Assert e por quê)
    - Padrão: Explicar propósito e asserções principais
    - Mínimo: Apenas nome e propósito do teste
  </action>
  
  <template-output>unit_tests_generated</template-output>
</step>

<step n="6" goal="Gerar testes de integração para fluxos-chave">
  <action>Identificar cenários de teste de integração:
    - Procurar plugins que disparam outros plugins (profundidade > 1)
    - Procurar cadeias Create→Plugin→Update
    - Procurar workflows disparados por atualizações de plugin
  </action>
  
  <action>Para cada cenário de integração:</action>
  
  <action>Gerar teste de integração:
    - Setup com contexto FakeXrmEasy completo
    - Registrar todos os plugins envolvidos com estágios corretos
    - Criar entidade gatilho inicial
    - Executar simulação completa do pipeline
    - Asserções sobre estado final das entidades afetadas
  </action>
  
  <action>Adicionar comentários explicando:
    - Qual sequência de pipeline está sendo testada
    - Por que essa integração importa
    - O que está sendo verificado
  </action>
  
  <template-output>integration_tests_generated</template-output>
</step>

<step n="7" goal="Gerar relatório abrangente de qualidade">
  <action>Compilar relatório final com:</action>
  
  <section name="Resumo Executivo">
    - Total de artefatos analisados: {{plugin_count + workflow_count + api_count}}
    - Projeto de testes criado: {{test_output_location}}
    - Framework de testes usado: {{active_framework}}
    - Testes de unidade gerados: {{unit_test_count}}
    - Testes de integração gerados: {{integration_test_count}}
    - Antipadrões detectados: {{antipattern_count}}
  </section>
  
  <section name="Análise do Projeto">
    - Plugins: {{plugin_count}}
    - Atividades de Workflow: {{workflow_count}}
    - Custom APIs: {{api_count}}
    - Controles PCF: {{pcf_count}}
    - Dependências mapeadas: {{dependency_count}}
  </section>
  
  <section name="Cobertura de Testes">
    - Componentes com testes de unidade: {{covered_components}}/{{total_components}}
    - Fluxos de integração testados: {{integration_flow_count}}
    - Cobertura de código estimada: {{estimated_coverage}}%
  </section>
  
  <section name="Problemas de Qualidade">
    - Antipadrões críticos: {{critical_count}}
    - Avisos: {{warning_count}}
    - Sugestões: {{suggestion_count}}
    - Lista detalhada com localizações e correções
  </section>
  
  <section name="Próximos Passos">
    1. Revisar e corrigir antipadrões críticos
    2. Executar testes gerados: `dotnet test {{test_output_location}}`
    3. Revisar resultados e ajustar conforme necessário
    4. Integrar testes ao pipeline de CI/CD
    5. Usar fluxo [CR] Code Review para checagens contínuas
    6. Usar [GT] Generate Tests para novos componentes durante o desenvolvimento
  </section>
  
  <section name="Como Executar Testes">
    ```bash
    cd {{test_output_location}}
    dotnet restore
    dotnet build
    dotnet test
    ```
  </section>
  
  <template-output>final_quality_report</template-output>
</step>

<step n="8" goal="Conclusão e orientações">
  <action>Parabenizar o usuário por concluir o QA Quick Setup! 🎉</action>
  
  <action>Fornecer próximos passos acionáveis:
    1. Navegar até o projeto de testes: {{test_output_location}}
    2. Abrir no Visual Studio ou VS Code
    3. Revisar os testes gerados
    4. Corrigir os antipadrões críticos identificados
    5. Executar testes para validar funcionamento
    6. Ajustar e personalizar testes conforme necessário
  </action>
  
  <ask>Você gostaria de:
    - [V] Ver o relatório de qualidade
    - [R] Reexecutar com configurações diferentes
    - [E] Sair e revisar por conta própria
  </ask>
  
  <action if="V">Exibir relatório de qualidade completo</action>
  <action if="R">Reiniciar fluxo a partir do passo 1</action>
  <action if="E">Agradecer o usuário e sair</action>
</step>

</workflow>
