# QA Gerar Testes - Instruções Granulares de Geração de Testes

<critical>O mecanismo de execução de workflows é regido por: {project-root}/.bmad/core/tasks/workflow.xml</critical>
<critical>Comunique-se em {communication_language}</critical>

<workflow>

<step n="1" goal="Definir escopo de geração">
  <ask>Qual é o caminho do seu código-fonte do Dynamics 365? (Padrão: {default_source_path})</ask>
  <action>Armazenar caminho como {{source_path}}</action>
  
  <ask>Qual tipo de testes você deseja gerar?
    1. Apenas testes unitários
    2. Apenas testes de integração
    3. Ambos (unitário e integração)
  </ask>
  <action>Armazenar como {{test_type}}</action>
  
  <ask>Escopo de geração:
    1. Todos os componentes (projeto completo)
    2. Tipos específicos (plugins, workflows, APIs)
    3. Arquivos ou classes específicas
  </ask>
  <action>Armazenar como {{generation_scope}}</action>
  
  <action if="scope=specific types">Perguntar quais tipos: Plugins, Workflows, Custom APIs, PCF</action>
  <action if="scope=specific files">Solicitar caminhos de arquivos ou nomes de classes (separados por vírgula)</action>
</step>

<step n="2" goal="Verificar se o projeto de testes existe e validar dependências">
  <action>Checar se o projeto de testes existe em: {test_output_location}</action>
  
  <check if="test project exists">
    <action>Confirmar: "Projeto de testes encontrado em {test_output_location}"</action>
    <action>VALIDAÇÃO CRÍTICA: Verificar versões de pacotes no .csproj:
      - Microsoft.CrmSdk.CoreAssemblies deve ser 9.0.2.*
      - NUnit deve ser 3.13.3
      - FakeXrmEasy.365 deve ser 1.58.1
      Se versões diferentes encontradas, alertar usuário e oferecer atualização
    </action>
    <ask>Usar este local? [s/n]</ask>
    <action if="no">Solicitar caminho alternativo para projeto de testes</action>
  </check>
  
  <check if="test project does NOT exist">
    <ask>Nenhum projeto de testes encontrado. Você gostaria de:
      1. Criar projeto de testes agora (recomendado)
      2. Especificar local diferente
      3. Cancelar e executar [QS] Quick Setup primeiro
    </ask>
    
    <action if="option 1">
      <action>Criar estrutura do projeto de testes:
        - Criar diretório: {test_output_location}
        - Criar .csproj com referências de {test_framework}
        - Criar classes base (TestBase, FakeContextFactory)
        - Criar diretórios de helpers
      </action>
    </action>
    
    <action if="option 2">Solicitar caminho do projeto de testes e validar</action>
    <action if="option 3">Sair do workflow com mensagem para executar Quick Setup</action>
  </check>
  
  <template-output>test_project_ready</template-output>
</step>

<step n="3" goal="Escanear e analisar componentes alvo">
  <action>Escanear {{source_path}} por componentes que correspondam a {{generation_scope}}:</action>
  
  <action>For each target component:
    - Extrair metadados (nome, namespace, entidade, mensagem, estágio)
    - Parsear estrutura e lógica do código
    - Identificar ramos de decisão
    - Identificar dependências e mocks necessários
    - Analisar complexidade (complexidade ciclomática)
    - Identificar requisitos de imagens (PreImage/PostImage)
    - Identificar uso de InputParameters
    - Enumerar todos os métodos public/protected e responsabilidades
    - Mapear invariantes e regras de integridade (segurança, depth, validação, efeitos colaterais)
  </action>
  
  <action>Build generation plan:
    - Lista de componentes para gerar testes
    - Para cada componente: quantidade estimada de testes, nível de complexidade
    - Para cada método: testes unitários requeridos para validar invariantes e integridade
    - Total estimado de testes
    - Tempo estimado de geração
  </action>
  
  <action>Exibir plano de geração ao usuário</action>
  <ask>Prosseguir com a geração? [s/n/ajustar]</ask>
  
  <action if="adjust">Permitir excluir componentes específicos ou ajustar escopo</action>
  <action if="no">Sair do workflow</action>
  
  <template-output>generation_plan</template-output>
</step>

<step n="4" goal="Gerar testes unitários" if="test_type includes unit">
  <action>Para cada componente no plano de geração:</action>
  
  <substep n="4a" title="Load appropriate template">
    <action>Carregar template em {templates_path} conforme tipo do componente:
      - Plugin → plugin-test-template.cs
      - Workflow Activity → workflow-test-template.cs
      - Custom API → api-test-template.cs
      - PCF Control → pcf-test-template.cs
    </action>
  </substep>
  
  <substep n="4b" title="Generate test class">
    <action>Criar arquivo de classe de teste: {{ComponentName}}Tests.cs</action>
    <action>Definir namespace para corresponder à estrutura do projeto</action>
    <action>Adicionar documentação XML em nível de classe de acordo com {comment_level}</action>
    <action>Herdar de TestBase</action>
    <action>Adicionar atributos do framework de teste ([TestFixture] para NUnit)</action>
  </substep>
  
  <substep n="4c" title="Generate setup method">
    <action>Criar método de Setup/Initialize:
      - Inicializar contexto FakeXrmEasy
      - Registrar plugin com configuração correta
      - Criar dados de teste comuns
      - Configurar serviços mock
    </action>
    <action>Adicionar comentários explicando o setup conforme {comment_level}</action>
  </substep>
  
  <substep n="4d" title="Generate test methods for each execution path">
    <action>Para cada caminho de código identificado:</action>
    
    <action>Create test method:
      - Nome: Execute_When{{Condition}}_Should{{ExpectedBehavior}}
      - Arrange: Preparar dados de teste, configurar mocks, preparar entidade
      - Act: Executar plugin/componente
      - Assert: Verificar resultados esperados
    </action>
    
    <action>Add detailed comments based on {comment_level}:
      - Detailed: Explicar cada linha (por que organizamos assim, o que está sendo testado, por que a asserção importa)
      - Standard: Explicar propósito do teste e asserções principais
      - Minimal: Apenas nome do teste (auto-documentado)
    </action>
    
    <action>Example tests to generate:
      - Happy path (execução normal)
      - Edge cases (valores de limite, dados vazios)
      - Error scenarios (valores nulos, campos ausentes, dados inválidos)
      - Permission scenarios (diferentes perfis de usuário)
      - Image scenarios (com/sem PreImage/PostImage)
      - Depth safety (validar context.Depth para evitar loops de reentrada)
      - Checagens de integridade por método (invariantes, pré/pós-condições, validação de efeitos colaterais)
      - Correção de InputParameters e OutputParameters
    </action>

    <action>For plugins with multiple methods (helpers/services inside the class):
      - Gerar testes unitários para TODOS os métodos public/protected
      - Validar invariantes e regras de integridade para cada método
      - Usar mocks/fakes para isolar dependências externas
      - Garantir que os testes permaneçam independentes e determinísticos
    </action>
  </substep>
  
  <substep n="4e" title="Generate helper methods if needed">
    <action>Se o componente tiver setup complexo, criar métodos auxiliares:
      - CreateTestEntity{{EntityName}}()
      - SetupMock{{ServiceName}}()
      - AssertExpected{{Behavior}}()
    </action>
  </substep>
  
  <substep n="4f" title="Save test file">
    <action>Salvar em: {test_output_location}/UnitTests/{{ComponentName}}Tests.cs</action>
    <action>Formatar o código apropriadamente (indentação, espaçamento)</action>
    <action>Adicionar cabeçalho com timestamp de geração e atribuição DQA</action>
  </substep>
  
  <action>Reportar progresso: "Testes unitários gerados para {{component_name}} ({{test_count}} testes)"</action>
  
  <template-output>unit_tests_progress</template-output>
</step>

<step n="5" goal="Gerar testes de integração" if="test_type includes integration">
  <action>Identificar cenários de testes de integração:</action>
  
  <action>Escanear interações multi-componentes:
    - Cadeias de plugins (Plugin A → triggers → Plugin B)
    - Cascatas de Create/Update
    - Workflows acionados por plugins
    - Dependências cross-entity
  </action>
  
  <action>Para cada cenário de integração:</action>
  
  <substep n="5a" title="Generate integration test class">
    <action>Criar: {{ScenarioName}}IntegrationTests.cs</action>
    <action>Configurar ambiente completo FakeXrmEasy:
      - Registrar todos os plugins envolvidos com estágios corretos
      - Configurar metadados de entidades
      - Configurar relacionamentos
      - Preparar dados iniciais
    </action>
  </substep>
  
  <substep n="5b" title="Generate integration test method">
    <action>Criar método de teste:
      - Nome: IntegrationTest_{{ScenarioDescription}}
      - Arrange: Configurar contexto completo com todos artefatos
      - Act: Acionar ação inicial (ex.: Create de entidade)
      - Assert: Verificar execução completa do pipeline e estado final
    </action>
    
    <action>Adicionar comentários explicando:
      - Qual sequência de pipeline está sendo testada
      - Quais artefatos executam em qual ordem
      - Qual é o estado final esperado
      - Por que esta integração importa
    </action>
  </substep>
  
  <substep n="5c" title="Save integration test file">
    <action>Salvar em: {test_output_location}/IntegrationTests/{{ScenarioName}}IntegrationTests.cs</action>
  </substep>
  
  <action>Reportar progresso: "Testes de integração gerados para {{scenario_name}}"</action>
  
  <template-output>integration_tests_progress</template-output>
</step>

<step n="6" goal="Gerar helpers de dados de teste" optional="true">
  <ask>Gerar classes helper de dados de teste? (Recomendado para consistência) [s/n]</ask>
  
  <action if="yes">
    <action>Criar TestDataFactory.cs:
      - Métodos para criar entidades comuns de teste
      - Métodos para configurar relacionamentos comuns
      - Métodos para criar usuários e equipes mock
      - Constantes para valores de teste frequentemente usados
    </action>
    
    <action>Adicionar documentação XML extensa</action>
    <action>Salvar em: {test_output_location}/Helpers/TestDataFactory.cs</action>
  </action>
  
  <template-output>test_data_helpers</template-output>
</step>

<step n="7" goal="Validar qualidade e compilar">
  <action>VALIDAÇÃO DE QUALIDADE (CI/CD Ready):
    - ✅ Independência: Cada teste pode rodar isoladamente sem ordem específica
    - ✅ Determinismo: Testes produzem sempre o mesmo resultado
    - ✅ Sem estado compartilhado: Cada teste cria seus próprios dados
    - ✅ Cleanup automático: Usar [TearDown] ou equivalente para limpeza
    - ✅ Sem dependências externas: Todos os serviços mockados
    - ✅ Sem sleeps ou delays: Testes síncronos e rápidos
  </action>
  
  <action>Validar arquivos de teste gerados:
    - Checar sintaxe (sem erros de compilação)
    - Verificar todas as referências resolvidas
    - Garantir namespaces corretos
    - Checar atributos do framework de teste corretos
    - Validar cobertura dos invariantes e regras de integridade mapeadas
    - CRÍTICO: Verificar que nenhum teste usa variáveis estáticas compartilhadas
    - CRÍTICO: Verificar que cada teste tem seu próprio FakeXrmContext
  </action>
  
  <ask>Tentar compilar o projeto de testes agora? [s/n]</ask>
  
  <action if="yes">
    <action>Executar: dotnet build {test_output_location}</action>
    <action>Exibir saída da compilação</action>
    
    <check if="build successful">
      <action>✅ Build bem-sucedido! Testes prontos para rodar.</action>
      <action>Recomendar: "Execute 'dotnet test' para validar que todos os testes passam"</action>
    </check>
    
    <check if="build failed">
      <action>⚠️ Build falhou. Revise erros e corrija antes de rodar os testes.</action>
      <action>Exibir erros de compilação com arquivo/linha</action>
    </check>
  </action>
  
  <template-output>build_validation</template-output>
</step>

<step n="8" goal="Gerar relatório resumido">
  <action>Compilar resumo da geração:</action>
  
  <section name="Generation Summary">
    - Caminho de origem: {{source_path}}
    - Projeto de testes: {test_output_location}
    - Framework de testes: {test_framework}
    - Escopo de geração: {{generation_scope}}
    - Tipos de testes: {{test_type}}
  </section>
  
  <section name="Tests Generated">
    - Classes de teste unitário: {{unit_test_classes_count}}
    - Métodos de teste unitário: {{unit_test_methods_count}}
    - Classes de teste de integração: {{integration_test_classes_count}}
    - Métodos de teste de integração: {{integration_test_methods_count}}
    - Total de testes: {{total_test_count}}
  </section>
  
  <section name="Files Created">
    {{list_of_generated_files}}
  </section>
  
  <section name="Next Steps">
    1. Revisar testes gerados em: {test_output_location}
    2. Customizar testes conforme seus cenários específicos
    3. Rodar testes: `dotnet test {test_output_location}`
    4. Adicionar testes ao controle de versão
    5. Integrar ao pipeline de CI/CD
  </section>
  
  <section name="Running Tests">
    ```cmd
    cd {test_output_location}
    dotnet test
    
    rem Rodar classe de teste específica
    dotnet test --filter "FullyQualifiedName~{{TestClassName}}"
    
    rem Rodar com saída detalhada
    dotnet test --logger "console;verbosity=detailed"
    ```
  </section>
  
  <template-output>generation_summary</template-output>
</step>

<step n="9" goal="Conclusão">
  <action>✅ Geração de testes concluída! 🎉</action>
  
  <action>Provide quick actions:
    - [V] Ver resumo da geração
    - [R] Rodar testes agora
    - [G] Gerar mais testes (escopo diferente)
    - [E] Sair
  </action>
  
  <action if="V">Exibir relatório completo</action>
  <action if="R">Executar: dotnet test {test_output_location}</action>
  <action if="G">Reiniciar workflow a partir do passo 1</action>
  <action if="E">Agradecer ao usuário e sair</action>
</step>

</workflow>
