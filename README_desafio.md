# Desafio NTConsult

# Arquitetura de Dados para o Pipeline de ETL de COVID-19 na Azure

# Parte 1
1. Camada de Ingestão
- Fonte de Dados:
    - O arquivo CSV com dados de COVID-19 está disponível em um URL público.
- Ferramentas Utilizadas:
    - Azure Data Factory (ADF): Será utilizado para orquestrar o processo de ingestão, agendando a captura dos dados diariamente.
    - Linked Services no ADF: Configuração de um serviço vinculado para acessar o arquivo CSV na web.
- Processo:
    - Pipeline no ADF: Um pipeline será criado no ADF para:
        - Conectar-se à URL pública.
        - Baixar o arquivo CSV.
        - Armazenar o CSV bruto em uma Azure Data Lake Storage Gen2 (ADLS Gen2), em uma camada de dados "raw" (bruta).
2. Camada de Processamento/Transformação
    - Transformação de Dados:
        - Limpeza, filtragem, agregação e enriquecimento dos dados.
    - Ferramentas Utilizadas:
        - Azure Databricks: O Databricks será usado para processar e transformar os dados brutos armazenados no ADLS Gen2.
    - Notebooks Databricks: Serão criados notebooks em PySpark para realizar as etapas de transformação.
    - Processo:
        - Ingestão de Dados no Databricks: Leitura do arquivo CSV armazenado na camada "raw" do ADLS Gen2.
        - Transformações no Databricks: Aplicação de transformações, como:
            - Remoção de valores nulos.
            - Filtragem de colunas relevantes.
            - Agregação por região e data.
            - Enriquecimento com dados adicionais se necessário.
            - Armazenamento Transformado: Os dados transformados serão armazenados de volta no ADLS Gen2 em uma camada de dados "processed" (processados).
3. Camada de Armazenamento
    - Formato de Armazenamento:
        - Delta Lake no ADLS Gen2:
            - O Delta Lake será utilizado para armazenar os dados transformados, fornecendo transações ACID, versionamento, e otimização para consultas rápidas.
        - Estrutura de Dados:
            - Dados particionados por região e data, facilitando a recuperação e análise.
        - Ferramentas:
            - Azure Databricks: Será configurado para salvar os dados transformados no formato Delta diretamente no ADLS Gen2.
            - Azure Data Lake Storage Gen2: Oferece escalabilidade e segurança para armazenar grandes volumes de dados.
4. Camada de Análise e Consulta
    - Ferramentas Utilizadas:
        - Azure Databricks SQL Analytics: Para consultas SQL interativas e criação de dashboards.
        - Power BI: Para visualizações avançadas e criação de relatórios interativos, conectando-se diretamente ao Delta Lake no ADLS Gen2.
    - Análises:
        - Descritivas e exploratórias, como total de casos, mortes, recuperações por região e evolução temporal.
    - Processo:
        - Consultas e Dashboards: Databricks SQL ou Power BI serão usados para criar relatórios e dashboards interativos, consumindo diretamente os dados armazenados no Delta Lake.
        - Visualizações: Gráficos de linha, barras, e mapas de calor para análise de tendências e padrões.
5. Camada de Segurança e Governança
    - Controle de Acesso:
        - Azure Role-Based Access Control (RBAC): Será configurado para controlar o acesso aos recursos do Azure (ADLS, Databricks, ADF).
    - Criptografia:
        - Criptografia em Repouso: O ADLS Gen2 oferece criptografia automática dos dados armazenados.
    - Auditoria:
        - Azure Monitor e Log Analytics: Configurados para monitorar e auditar o acesso aos dados e mudanças no pipeline, garantindo a conformidade e a segurança.
6. Camada de Monitoramento
    - Métricas Monitoradas:
        - Desempenho do pipeline no ADF (tempo de execução, erros).
        - Uso de recursos no Databricks (CPU, memória).
        - Integridade dos dados (percentual de valores nulos, consistência de registros).
    - Ferramentas Utilizadas:
        - Azure Monitor: Para monitorar o pipeline de ETL e configurar alertas automáticos.
        - Azure Log Analytics: Para centralizar os logs de execução e erros do pipeline.
    - Notificações:
        - Configuração de alertas via e-mail ou webhook para falhas ou desempenho abaixo do esperado no pipeline.

# Parte 2
A implementação será feita em um notebook Databricks utilizando Python e PySpark.
Cada etapa (extração, transformação, carregamento) será documentada com comentários explicando as operações realizadas.
# Extração dos dados
df = spark.read.csv("https://storage.googleapis.com/covid19-open-data/v3/latest/aggregated.csv", header=True, inferSchema=True)

# Transformação dos dados
df_cleaned = df.na.drop().filter("location_key IS NOT NULL")

# Carregamento dos dados
df_cleaned.write.format("delta").mode("overwrite").save("/mnt/delta/covid19_data")

# Parte 3
Formato escolhido: Delta Lake

Justificativa: O Delta Lake oferece recursos como gerenciamento de transações ACID, otimização automática e suporte a versionamento, que são cruciais para a confiabilidade e performance em ambientes de análise de grandes volumes de dados

# Parte 4
# Análises Descritivas

Realizar análises como: 
    - Total de casos, mortes e recuperações por região.
    - Evolução temporal de casos e mortes.
    - Visualizações em gráficos de linha, barras e mapas.

# Exemplo de agregação
df_summary = df_cleaned.groupBy("location_key").agg(sum("new_confirmed").alias("total_cases"))

# Exemplo de visualização
display(df_summary) 

A análise revelará, por exemplo, as regiões mais afetadas, padrões de crescimento, e períodos de pico. Insights serão documentados no próprio notebook

# Parte 5
# Segurança dos Dados durante o ETL

Garantia de Segurança:
    - Uso de SSL para transferências de dados.
    - Controle de acesso baseado em funções (RBAC) para acesso aos notebooks e dados no Databricks.
    - Auditoria de logs para rastreamento de acesso e modificações.

Medidas de Segurança Implementadas:
    - Dados sensíveis, como informações pessoais, serão anonimizados.
    - Armazenamento criptografado no Delta Lake.

# Parte 6
# Monitoramento do Pipeline de ETL

Estratégia de Monitoramento:
    - Monitoramento de tarefas do Spark usando o Spark UI e métricas de performance.
    - Monitoramento contínuo dos dados carregados usando Delta Lake para detecção de anomalias.
    - Alertas configurados no Databricks para falhas ou atrasos no pipeline.

Métricas a serem Monitoradas:
    - Tempo de execução das tarefas de ETL.
    - Integridade dos dados (percentual de valores nulos, consistência de registros).
    - Performance de consultas (tempo de resposta, número de registros retornados).