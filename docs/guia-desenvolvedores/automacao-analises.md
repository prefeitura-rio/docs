
## Organização de Datasets e Tabelas

A organização eficiente dos datasets e tabelas no BigQuery é fundamental para manter um ambiente de dados limpo e gerenciável. Durante o desenvolvimento de uma nova análise ou dashboard, é recomendável seguir um conjunto de práticas específicas para garantir a clareza e a organização dos dados.

### Projetos de Desenvolvimento

Durante o desenvolvimento de análises e dashboards, recomenda-se a utilização de um projeto de desenvolvimento específico. Por exemplo, para o projeto de produção `rj-smfp` utilize o projeto de desenvolvimento `rj-smfp-dev`. Isso ajuda a separar claramente os ambientes de produção e desenvolvimento.

### Nomenclatura de Datasets e Tabelas

A estrutura de nomeação de datasets e tabelas deve seguir boas práticas para facilitar a identificação e a compreensão dos dados. A seguir, estão algumas diretrizes para nomear datasets e tabelas:

- Use todas as letras minúsculas (inclusive siglas), sem acentos, conectadas por underscores (_).
- Evite abreviações e inclusão de conectores como "de", "da", "dos", "e", "a", "em", etc.
- Ao criar datasets destinados a análises ou dashboards, mantenha o nome original do dataset e acrescente o sufixo dashboard ou analise, juntamente com a identificação do órgão responsável. Por exemplo para um dashboard relacionado ao sistema ERGON feito pela SUBGCC; as tabelas se encontram no dataset_id: `recursos_humanos_ergon`, os dados relevantes para esse dashboard devem ser armazenados no dataset_id `recursos_humanos_ergon_dashboard_subgcc`.
- Utilize nomes de datasets e tabelas descritivos para refletir claramente o conteúdo ou finalidade dos dados.


### Tipos de Tabelas no BigQuery

O BigQuery oferece três tipos diferentes de tabelas, cada uma com suas características:

1. **Tabelas Externas**: Tabelas externas funcionam de forma semelhante a tabelas normais do BigQuery. Os detalhes sobre a estrutura da tabela, como as colunas e os tipos de dados, são armazenados no BigQuery. No entanto, os dados em si permanecem na fonte externa. Isso pode incluir fontes como **Google Cloud Storage (GCS)**, **Google Sheets**, **Google Drive** e outras. Para saber mais, consulte [como consultar fontes de dados externas](https://cloud.google.com/bigquery/docs/external-data-sources).

2. **Tabelas Nativas**: Tabelas nativas contêm os dados diretamente dentro do BigQuery. Essas tabelas oferecem alta velocidade de consulta, mas possuem um limite de 4 mil partições, o que pode ser um fator limitante para conjuntos de dados muito grandes.

3. **Views (Visualizações)**: Uma view é uma tabela virtual criada a partir de uma consulta SQL. Ela não armazena dados fisicamente, mas, em vez disso, fornece uma maneira conveniente de organizar e acessar dados de outras tabelas. Para obter mais informações, consulte [como criar visualizações](https://cloud.google.com/bigquery/docs/views).

Esses três tipos de tabelas oferecem flexibilidade para lidar com uma variedade de cenários de armazenamento e consulta de dados no BigQuery. A escolha do tipo de tabela depende das necessidades específicas do seu projeto e da fonte dos dados que você está gerenciando.


### Importação de Dados do Google Sheets

Uma maneira eficaz de trazer dados de fontes externas para o BigQuery é por meio da importação de dados do Google Sheets. Essa integração permite que você trabalhe com dados de planilhas do Google diretamente no ambiente do BigQuery.


1. Nas opções do dataset clique em criar nova tabela

2. Na página de configuração da nova tabela, escolha a opção Drive como origem dos dados, indique a URL da tabela no google sheets e selecione o formato Google Sheet em seguida defina um nome para sua tabela. No final você deve ter algo como na imagem abaixo: ![Criar Tabela](../static/img/automacao-analises/create_table_google_sheets_config.png)

3. Voce pode definir o Schema manualmente ou utilizar a função de auto deteção. Na seção de opções avançadas indique a linha que represente o header da tabela para que ele não seja interpretado como um dado: ![Schema](../static/img/automacao-analises/create_table_google_sheets_config_schema.png)

4. É importante que a tabela no google sheets seja pública ou que os usuarios tenham permissão de visualização para executar queries na tabela criada.![Query](../static/img/automacao-analises/create_table_google_sheets_query.png)



## Automação com BigQuery Dataform

### Criando Tabelas no BigQuery Dataform

O BigQuery Dataform permite automatizar transformações de dados. Suponha que você deseje criar uma tabela de vendas agregadas. No Dataform:

1. **Definindo um Projeto Dataform**:
   - Crie um projeto chamado `projeto_vendas` com as configurações necessárias.

2. **Criando Modelos**:
   - Crie um modelo chamado `vendas_agregadas.sql` com a seguinte consulta:
   
   ```sql
   SELECT EXTRACT(MONTH FROM data) AS mes, produto, SUM(receita) AS receita
   FROM `projeto_desenvolvimento.raw_data.vendas`
   GROUP BY mes, produto
   ```

3. **Definindo Dependências**:
   - Configure a dependência deste modelo em relação aos dados brutos.

4. **Compilando e Executando**:
   - Compile o modelo e execute-o para criar a tabela no BigQuery.

### Criando um Fluxo de Trabalho (Workflow)

Com o Dataform, você pode automatizar a execução de modelos em uma sequência específica. Por exemplo, crie um fluxo de trabalho para consolidar vendas diárias e, em seguida, as vendas mensais:

1. **Criando Tarefas**:
   - Crie tarefas "vendas_diarias" e "vendas_mensais" que representam a compilação e execução dos modelos correspondentes.

2. **Configurando Dependências**:
   - Configure a dependência "vendas_mensais" em relação à tarefa "vendas_diarias" para garantir a ordem correta de execução.

3. **Executando o Fluxo de Trabalho**:
   - Inicie o fluxo de trabalho para automatizar o processo de ETL.

## Conectando com o Produto Final

### Integração com Looker Studio

Integrar o BigQuery com o Looker Studio permite criar painéis de controle interativos. Exemplo:

1. No Looker, conecte-se ao BigQuery como fonte de dados.
2. Crie um modelo no Looker com as métricas desejadas.
3. Crie um painel de controle com base no modelo para visualizar os dados em tempo real.

### Integração com Google Sheets

Para compartilhar dados do BigQuery no Google Sheets, siga as etapas:

1. No Google Sheets, use a função `GOOGLECLOCK` para acessar os dados do BigQuery.
2. Configure as credenciais e permissões necessárias para acessar os dados.
3. Atualize automaticamente os dados no Google Sheets conforme os dados do BigQuery mudam.

### Integração com Python Notebook

Conectar o BigQuery a um notebook Python é útil para análises avançadas. Exemplo de configuração:

```python
import pandas as pd
from google.cloud import bigquery

# Configurar credenciais
client = bigquery.Client.from_service_account_json('seu-arquivo-de-credenciais.json')

# Consulta ao BigQuery
query = """
SELECT produto, AVG(receita) AS receita_media
FROM `projeto_desenvolvimento.processed_data.vendas_consolidadas`
GROUP BY produto
"""
df = client.query(query).to_dataframe()

# Análise e visualização com Pandas e Matplotlib
```

Isso permite executar consultas e análises mais avançadas usando Python. Certifique-se de ter as bibliotecas necessárias instaladas e configuradas para autenticação no BigQuery.