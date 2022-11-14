## O que é o DBT e para que serve?
Em um _pipeline_ ELT, os dados brutos são extraídos (_extract_) de uma fonte e carregados (_load_) em uma _data warehouse_ (no nosso caso: o _BigQuery Storage_).   Em seguida, os dados brutos são transformados em tabelas utilizáveis, usando consultas **SQL** executadas sobre os dados estocados nessa _data warehouse_.

O **dbt** (_Data Build Tool_) é uma ferramenta de linha de comando que oferece uma maneira fácil para que analistas e engenheiros de dados criem, transformem e validem os dados em um _data warehouse_ de maneira mais eficiente. O **dbt** faz o T nos processos ELT (Extrair, Carregar, Transformar) ilustrados na figura abaixo.

![Fluxo de Dados com DBT](../static/img/tutoriais/dbt/fluxo_dbt.png)

Em **dbt**, trabalhamos com modelos, que é um arquivo **SQL** com uma instrução `select`. Esses modelos podem depender de outros modelos, ter testes definidos neles e podem ser criados como tabelas ou visualizações. 

Fazendo um paralelo com o processo de ELT do Escritório de Dados: utilizamos _pipelines_ como `dump_db` ou `dump_url` para extrair os dados de uma base **SQL** ou uma planilha Google e subir esses dados no _Google Cloud Storage_. Após esse passo, os dados já estão disponíveis para consulta, visualização, construção de dashboards e todas as funções e facilitadores do _BigQuery_ podem ser aplicadas nos dados. Porém, na grande maioria das vezes, os dados brutos, diretos da fonte de dados original, precisam ser tratados antes de serem efetivamente utilizados.

Alguns exemplos de tratamento comumente feitos são:

- Tipagem de colunas (`FLOAT64`, `STRING`, `GEOMETRY`, `INT64`, etc.).
- (Re)nomeação de colunas.
- Mapeamento de _seed values_ (troca de `id_bairro` por `nome_bairro` ou vice-versa).
- Criação de novas colunas a partir dos próprios dados brutos.
- Complementação de dados a partir de outras bases que estão no mesmo _data warehouse_ (_Google Cloud Storage_). 

## Escrevendo seu primeiro modelo DBT

O primeiro passo para escrever um modelo **dbt** é identificar qual o _project_id_ em que as tabelas tratadas serão materializadas. Na maioria dos casos, será o mesmo _project_id_ em que os dados brutos foram armazenados em _staging_. Uma vez identificado o _project_id_, deve-se clonar o repositório de nome `queries-` + `'project_id'`, onde o modelo será construído. (Exemplos de repositórios: `queries-rj-smfp`, `queries-datario`, `queries-rj-sme`, etc.).

Uma vez que o repositório foi clonado e se está em uma nova _branch_, deve-se criar uma nova pasta dentro do diretório `queries-(...).models` com um nome breve que identifique as tabelas que serão materializadas. Exemplos: `dashboard_metas`, `educacao_basica_avaliacao`, ou o próprio *dataset_id* das tabelas que serão materializadas. Aqui, utilizarei `exemplo_formacao_infra`.

Dentro dessa pasta, ficam os modelos de dados, que nada mais são que arquivos **SQL** com uma instrução `select`. Além disso, os nomes dos modelos criados pelo **dbt** são os próprios nomes dos arquivos, então nomeie os arquivos cuidadosamente. Seguem abaixo 2 arquivos, `elementos.sql` e `paises_americanos.sql`, como exemplos de modelos.

=== "elementos.sql"

```sql
SELECT 
  SAFE_CAST(data as DATE) as data,
  SAFE_CAST(number as INT64) as numero,
  element as elemento
FROM `rj-escritorio-dev.test_formacao_staging.test_table_2`
```

=== "paises_americanos.sql"

```sql
SELECT
  LPAD(id_pais, 2, '0') as id_pais,
  CASE 
    WHEN continente = 'América do Sul' THEN '01'
    WHEN continente = 'América Central' THEN '02'
    WHEN continente = 'América do Norte' THEN '03'
    ELSE '00'
  END as id_continente,
  pais,
  capital
FROM `rj-escritorio-dev.test_formacao_staging.test_table`
```

No modelo `elementos`, foi realizada uma tipagem nas colunas de data e numero, além de uma renomeação das colunas de inglês para português. No modelo `paises_americanos`, foi utilizada a função `LPAD` para que todos os *id_pais* ficassem com tamanho 2, além da criação de uma nova coluna de *id_continente*.

Note que as _queries_ acima rodam no próprio console do _BigQuery_, inclusive, foram desenvolvidas e testadas lá. Lembre-se de testar seu código **SQL** antes de subir um _pull request_ com um novo modelo.

Uma vez criados os modelos, é necessário criar o arquivo `schema.yml`, que contém seus metadados, como a descrição do modelo (tabela), o nome das colunas e suas descrições. Segue abaixo o `schema.yml` para os modelos criados acima:

=== "schema.yml"

```sql
version: 2
models :
  - name: elementos
    description: "**Descrição**: Tabelas com os 4 elementos da natureza, seus números e datas aleatórias.\ 
    Utilizado como exemplo para a Formação Infra 2022 do Escritório de Dados do Rio de Janeiro\n\ 
    **Frequência de atualização**: Não possui"
    columns:
      - name: data
        description: Data em que os dados da linha foram inseridos.
      - name: numero
        description: Código do elemento.
      - name: elemento
        description: Nome do elemento, em inglês.
  - name: paises_americanos
    description: "**Descrição**: Tabelas com todos os países da América e informações sobre eles. \
    Utilizado como exemplo para a Formação Infra 2022 do Escritório de Dados do Rio de Janeiro\n\
    **Frequência de atualização**: Não possui"
    columns:
      - name: id_pais
        description: Código do país.
      - name: id_continente
        description: Código do continente a qual o país pertence.
      - name: pais
        description: Nome do país.
      - name: capital
        description: Capital do país.
```

Após criar os modelos e o arquivo _schema_, o último passo é alterar o arquivo `dbt_project.yml`, que fica na base do repositório. No final do arquivo há um trecho com `models:` e o `project_id` do seu projeto, é ali que deve-se adicionar o nome da pasta que os novos modelos foram criados (nesse caso, `exemplo_formacao_infra`) e o tipo de materialização para os modelos daquela pasta (_view_, _table_ or _incremental_). Nesse caso, utilizaremos _table_. (Aqui tem um [exemplo](https://github.com/prefeitura-rio/queries-rj-smfp/blob/master/dbt_project.yml) desse arquivo totalmente preenchido).

=== "dbt_project.yml"

```sql
(...)
models:
  +persist_docs:
    relation: true
    columns: true
  rj_escritorio:
    exemplo_formacao_infra:
      +materialized: table
      +schema: exemplo_formacao_infra
```

E é isso! Se você chegou até aqui você já criou seu primeiro modelo **dbt**.

## Parametrizando queries

## Integrando com as pipelines do Prefect
