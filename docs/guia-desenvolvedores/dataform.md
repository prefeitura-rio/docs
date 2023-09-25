# Desenvolvimento e Produção de Analises com BigQuery


## Conseitos basicos do BigQuery

### Organização de Datasets

Nesta seção, exploramos as melhores práticas para organizar datasets em projetos de desenvolvimento e produção.

### Importação de Dados do Google Sheets

Saiba como importar dados de suporte diretamente do Google Sheets para seus projetos de dados.

### Desenvolvimento de Pipelines no BigQuery (BQ)

Aqui, abordamos o processo de desenvolver pipelines de dados no BigQuery, incluindo boas práticas e exemplos.

### Uso de Views

Descubra como criar e usar views no BigQuery para simplificar consultas e melhorar a reutilização de código.

## Automatização com BigQuery Dataform

### Criando Tabelas no BigQuery Dataform

O Dataform é uma poderosa ferramenta de automação e gerenciamento de transformações de dados que se integra perfeitamente com o BigQuery,. Com o Dataform, você pode criar, testar e agendar transformações de dados de forma eficiente e confiável.

Esta ferramenta permite que você defina modelos de dados e fluxos de trabalho, simplificando o processo de transformação de dados em projetos de desenvolvimento e produção. Para obter informações detalhadas sobre como usar o Dataform em conjunto com o BigQuery, consulte a [documentação oficial do Google Cloud](https://cloud.google.com/dataform/docs/overview). 

1. **Definindo um Projeto Dataform**:
   - Na interface gráfica do Dataform, crie um novo projeto Dataform para o seu projeto de dados.
   - Configure as opções do projeto, como o armazém (BigQuery), o esquema padrão e o banco de dados padrão.

2. **Criando Modelos**:
   - Use a interface gráfica para criar modelos, que representam as transformações desejadas para seus dados.
   - Escreva consultas SQL na interface gráfica para definir o modelo, incluindo operações como seleção, agregação e filtragem.

3. **Definindo Dependências**:
   - Na interface gráfica, você pode definir dependências entre modelos para garantir a ordem correta de execução.

4. **Compilando e Executando**:
   - Use a interface gráfica para compilar seus modelos em consultas SQL.
   - Execute os modelos na interface gráfica para carregá-los no BigQuery.

### Criando um Fluxo de Trabalho (Workflow)

Você também pode criar fluxos de trabalho no Dataform para automatizar a execução de modelos em uma sequência específica. Aqui está um exemplo de como fazer isso usando a interface gráfica:

1. **Criando Tarefas**:
   - Na interface gráfica do Dataform, crie tarefas que representem as ações que você deseja executar, como compilar, testar e publicar.

2. **Configurando Dependências**:
   - Configure as dependências entre as tarefas para garantir a ordem correta de execução.

3. **Executando o Fluxo de Trabalho**:
   - Inicie o fluxo de trabalho na interface gráfica, que executará as tarefas na ordem especificada.

## Conectando com o Produto Final

### Integração com Looker Studio

Aprenda como integrar seus dados do BigQuery com Looker Studio para criar visualizações e painéis de controle poderosos.

### Integração com Google Sheets

Descubra como compartilhar e atualizar dados do BigQuery em tempo real no Google Sheets.

### Integração com Python Notebook

Saiba como conectar seus dados do BigQuery a um notebook Python para análises avançadas e visualizações personalizadas.