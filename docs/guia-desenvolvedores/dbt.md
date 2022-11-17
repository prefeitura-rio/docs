## O que é o DBT e para que serve?

Em um _pipeline_ ELT, os dados brutos são extraídos (_extract_) de uma fonte e carregados (_load_) em uma _data warehouse_ (no nosso caso: o _BigQuery Storage_). Em seguida, os dados brutos são transformados em tabelas utilizáveis, usando consultas **SQL** executadas sobre os dados estocados nessa _data warehouse_.

O **dbt** (_Data Build Tool_) é uma ferramenta de linha de comando que oferece uma maneira fácil para que analistas e engenheiros de dados criem, transformem e validem os dados em um _data warehouse_ de maneira mais eficiente. O **dbt** faz o T nos processos ELT (Extrair, Carregar, Transformar) ilustrados na figura abaixo.

![Fluxo de Dados com DBT](../static/img/tutoriais/dbt/fluxo_dbt.png)

No **dbt**, trabalhamos com modelos, que nada mais é que um arquivo **SQL** com uma instrução `select`. Esses modelos podem depender de outros modelos, ter testes definidos neles e podem ser criados como tabelas ou visualizações.

Fazendo um paralelo com o processo de ELT do Escritório de Dados: utilizamos _pipelines_ como `dump_db` ou `dump_url` para extrair os dados de uma base **SQL** ou uma planilha Google e subir esses dados no _Google Cloud Storage_. Após esse passo, os dados já estão disponíveis para consulta, visualização, construção de dashboards e todas as funções e facilitadores do _BigQuery_ podem ser aplicadas nos dados. Porém, na grande maioria das vezes, os dados brutos, diretos da fonte de dados original, precisam ser tratados antes de serem efetivamente utilizados.

Alguns exemplos de tratamento comumente feitos são:

- Tipagem de colunas (`FLOAT64`, `STRING`, `GEOMETRY`, `INT64`, etc.).
- (Re)nomeação de colunas.
- Mapeamento de _seed values_ (troca de `id_bairro` por `nome_bairro` ou vice-versa).
- Criação de novas colunas a partir dos próprios dados brutos.
- Complementação de dados a partir de outras bases que estão no mesmo _data warehouse_ (_Google Cloud Storage_).

## Escrevendo seu primeiro modelo DBT

O primeiro passo para escrever um modelo **dbt** é identificar qual o _project_id_ em que as tabelas tratadas serão materializadas. Na maioria dos casos, será o mesmo _project_id_ em que os dados brutos foram armazenados em _staging_. Uma vez identificado o _project_id_, deve-se clonar o repositório de nome `queries-` + `'project_id'`, onde o modelo será construído. (Exemplos de repositórios: `queries-rj-smfp`, `queries-datario`, `queries-rj-sme`, etc.).

Uma vez que o repositório foi clonado e se está em uma nova _branch_, deve-se criar uma nova pasta dentro do diretório `queries-(...).models` com um nome breve que identifique as tabelas que serão materializadas. Exemplos: `dashboard_metas`, `educacao_basica_avaliacao`, ou o próprio _dataset_id_ das tabelas que serão materializadas. Aqui, utilizarei `test_formacao`.

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

No modelo `elementos`, foi realizada uma tipagem nas colunas de data e número, além de uma renomeação das colunas de inglês para português. No modelo `paises_americanos`, foi utilizada a função `LPAD` para que todos os _id_pais_ ficassem com tamanho 2, além da criação de uma nova coluna de _id_continente_.

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

🚨🚨 **Atenção** 🚨🚨: Essas descrições têm que ser exatamente as mesmas que foram preenchidas no arquivo de arquitetura e no [meta.dados.rio](meta.dados.rio/).

Após criar os modelos e o arquivo _schema_, o último passo é alterar o arquivo `dbt_project.yml`, que fica na base do repositório. No final do arquivo há um trecho com as keys `models:` e `project_id:` do seu projeto no qual são definidos os modelos (queries) que o dbt deverá executar. É nessa parte que deve-se adicionar o nome da pasta que você criou com os novos modelos (nesse caso, `exemplo_formacao_infra`) e o tipo de materialização para os modelos daquela pasta (_view_, _table_ or _incremental_). Nesse caso, utilizaremos _table_. (Aqui tem um [exemplo](https://github.com/prefeitura-rio/queries-rj-smfp/blob/master/dbt_project.yml) desse arquivo totalmente preenchido).

=== "dbt_project.yml"

```sql
(...)
models:
  +persist_docs:
    relation: true
    columns: true
  rj_escritorio:
    test_formacao:
      +materialized: table
      +schema: test_formacao
```

🚨🚨 **Atenção** 🚨🚨: A identação de cada uma das partes desses arquivos é fundamental para o correto funcionamento do DBT. Mantenha sempre o padrão de identação.

E é isso! Se você chegou até aqui você já criou seu primeiro modelo **dbt**.

Para se aprofundar mais sobre esse tema, acesse a documentação oficial do [DBT](https://docs.getdbt.com/reference/model-configs.).

## Parametrizando queries

Em muitos casos, existem parâmetros relevantes para o funcionamento das queries cuja mudança vai alterar drasticamente o resultado, como:

- Você coloca a taxa de imposto de algo e o governo muda esse valor, a única forma de mudar isso é mudar o código, achar todas as ocorrências desse valor na sua query e alterar um por um;

- Uma função trabalha com 10 itens em algo que ela recebe, mas em algum momento há uma mudança e vem 12 itens que você deveria avaliar, então é necessário repensar todas as lógicas realizadas e fazer as substituições;

- Uma query está filtrada para pegar registros dos últimos 3 anos, mas agora houve uma mudança e é necessário pegar dados dos últimos 5 anos, então você tem que substituir esse valor todas as vezes em que ele foi referenciado.

Os casos acima são exemplos de _hard code_, que significa que uma informação relevante para o funcionamento do sistema foi colocado no código. Isso resulta em muito trabalho pra fazer alterações, dificultando a reprodução do código e até a própria (re)utilização do mesmo por outras pessoas.

Felizmente, o **dbt** tem uma solução pra esse problema! É possível declarar variáveis no arquivo `dbt_project.yml` e se referenciar à elas dentro das queries, de modo que quando algum valor dessas variáveis mudar, tudo que precisa ser feito é alterar esse valor uma única vez.

Continuando o exemplo que foi feito acima, suponhamos que seja necessário selecionar elementos da natureza, no modelo `elementos.sql` que foram declarados antes de uma certa data, e além disso, que queiramos criar uma nova coluna que indica se um país americano tem a capital que inicia com determinada letra, no modelo `paises_americanos.sql`. O primeiro passo é declarar os valores dessas variáveis dentro do arquivo `dbt_project.yml`:

=== "dbt_project.yml"

```sql
(...)
vars:
  ELEMENTOS_DATA_INICIAL: "2022-01-03"
  PAISES_AMERICANOS_LETRA_INICIAL_CAPITAL: "B"

models:
  +persist_docs:
    relation: true
    columns: true
  rj_escritorio:
    test_formacao:
      +materialized: table
      +schema: test_formacao
```

Depois, incluímos essas variáveis dentro da query, referenciando-as com `{{var('NOME_DA_VARIAVEL')}}`:

=== "elementos.sql"

```sql
SELECT
  SAFE_CAST(data as DATE) as data,
  SAFE_CAST(number as INT64) as numero,
  element as elemento,
FROM `rj-escritorio-dev.test_formacao_staging.test_table_2`
WHERE data < "{{var('ELEMENTOS_DATA_INICIAL')}}"
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
  capital,
  CASE
    WHEN STARTS_WITH(capital, "{{var('PAISES_AMERICANOS_LETRA_INICIAL_CAPITAL')}}")
      THEN CONCAT("Começa com ", "{{var('PAISES_AMERICANOS_LETRA_INICIAL_CAPITAL')}}")
    ELSE CONCAT("Não começa com ", "{{var('PAISES_AMERICANOS_LETRA_INICIAL_CAPITAL')}}")
    END indicador_capital_com_{{var('PAISES_AMERICANOS_LETRA_INICIAL_CAPITAL')}}
FROM `rj-escritorio-dev.test_formacao_staging.test_table`
```

Ainda sobre parametrização de queries, uma outra possibilidade é a de referenciar outros modelos(queries) dentro de um modelo. Por exemplo, caso dentro do modelo `elementos.sql`, eu deseje fazer um `SELECT` no modelo `paises_americanos.sql`, isso é possível, basta referenciar esse modelo na forma `{{ ref('NOME_DO_MODELO') }}`.

=== "elementos.sql"

```sql
WITH select_paises AS (
  SELECT
    *
  FROM {{ ref('paises_americanos') }}
)

SELECT
  SAFE_CAST(data as DATE) as data,
  SAFE_CAST(number as INT64) as numero,
  element as elemento,
FROM `rj-escritorio-dev.test_formacao_staging.test_table_2`
WHERE data < "{{var('ELEMENTOS_DATA_INICIAL')}}"
```

O caso acima é especialmente útil quando um modelo depende de outro pra ser executado corretamente. Nesse caso, é necessário que a query do modelo dependente seja executada depois da query do modelo indepentende. Isso é possível de configurar em uma pipeline do prefect através dos parâmetros `upstream` e `downstream`, que veremos mais abaixo.

## Integrando com as pipelines do Prefect

Para adicionarmos essa parte de materialização dos dados via DBT precisamos adicionar essa etapa no nosso `flows.py`. Isso vai depender se nossa pipeline foi criada através de um flow pré-definido (como no caso de flows que acessam os bancos de dados da prefeitura ou planilhas do google sheets) ou se a iniciamos do zero.

### DBT para Flows pré-definidos

Vamos reutilizar os códigos da aula anterior que extrai informações e um google sheets:

=== "flows.py"

```python
# -*- coding: utf-8 -*-
"""
Database dumping flows for formation project
"""

from copy import deepcopy

from prefect.run_configs import KubernetesRun
from prefect.storage import GCS

from pipelines.constants import constants

from pipelines.rj_escritorio.dump_db_formacao.schedules import gsheets_one_minute_update_schedule
from pipelines.utils.dump_url.flows import dump_url_flow  # alterado
from pipelines.utils.utils import set_default_parameters

formacao_gsheets_flow = deepcopy(dump_url_flow)
formacao_gsheets_flow.name = "EMD: Formação GSheets - Ingerir tabelas de URL"
formacao_gsheets_flow.storage = GCS(constants.GCS_FLOWS_BUCKET.value)
formacao_gsheets_flow.run_config = KubernetesRun(
    image=constants.DOCKER_IMAGE.value,
    labels=[
        constants.RJ_ESCRITORIO_DEV_AGENT_LABEL.value,
    ],
)

formacao_gsheets_flow_parameters = {
    "dataset_id": "test_formacao",
    "dump_mode": "overwrite",
    "url": "https://docs.google.com/spreadsheets/d/1uF-Gt5AyZmxCQQEaebvWF4ddRHeVuL6ANuoaY_-uAXE\
        /edit#gid=0",
    "url_type": "google_sheet",
    "gsheets_sheet_name": "sheet_1",
    "table_id": "test_table",
}

formacao_gsheets_flow = set_default_parameters(
    formacao_gsheets_flow, default_parameters=formacao_gsheets_flow_parameters
)

formacao_gsheets_flow.schedule = gsheets_one_minute_update_schedule

```

Dentro do `flows.py` precisamos apenas alterar a origem do flow que estamos copiando.

=== "schedule.py"

```python
# -*- coding: utf-8 -*-
"""
Schedules for the database dump pipeline
"""

from datetime import datetime, timedelta

import pytz
from prefect.schedules import Schedule
from pipelines.constants import constants
from pipelines.utils.dump_url.utils import generate_dump_url_schedules
from pipelines.utils.utils import untuple_clocks as untuple

#####################################
#
# EGPWeb Schedules
#
#####################################

gsheets_urls = {
    "test_table": {
        "url": "https://docs.google.com/spreadsheets/d/1uF-Gt5AyZmxCQQEaebvWF4ddRHeVuL6ANuoaY_-uAXE\
            /edit#gid=0",
        "url_type": "google_sheet",
        "gsheets_sheet_name": "sheet_1",
        "dump_mode": "overwrite", # alterado
        "materialize_after_dump": True, # alterado
        "materialization_mode": "dev", # alterado
        "materialize_to_datario": True, # alterado
        "dump_to_gcs": True, # alterado
    },
}


gsheets_clocks = generate_dump_url_schedules(
    interval=timedelta(minutes=1),
    start_date=datetime(2022, 10, 21, 15, 0, tzinfo=pytz.timezone("America/Sao_Paulo")),
    labels=[
        constants.RJ_ESCRITORIO_DEV_AGENT_LABEL.value,
    ],
    dataset_id="test_dataset_formacao",
    table_parameters=gsheets_urls,
)

gsheets_one_minute_update_schedule = Schedule(clocks=untuple(gsheets_clocks))

```

Agora temos 5 novas chaves dentro do nosso dicionário de parâmetros no arquivo `schedule.py`:

- **dump_mode**: ["overwrite", "append"] Define se os novos dados substituirão os dados anteriores no GCP (overwrite) ou se serão adicionados (append) aos que já existem no GCP;

- **materialize_after_dump**: [True, False] Define se os dados serão materializados no BQ (BigQuery) do próprio projeto utilizando o DBT;

- **materialization_mode**: ["dev", "prod"] Define se a materialização acontecerá em ambiente de desenvolvimento ou produção;

- **materialize_to_datario**: [True, False] Define se os dados serão materializados no datario utilizando o DBT.

- **dump_to_gcs**: [True, False] Define se os dados serão salvos em um csv no Google Cloud Storage;

### DBT para Flows iniciados do zero

=== "flows.py"

```python
# -*- coding: utf-8 -*-
"""
Example flow
"""
from prefect import case, Parameter # adicionado
from prefect.run_configs import KubernetesRun
from prefect.storage import GCS
from prefect.tasks.prefect import create_flow_run, wait_for_flow_run # adicionado

from pipelines.formacao.exemplo.constants import (
    constants as formacao_constants,
)

from pipelines.constants import constants
from pipelines.formacao.exemplo.tasks import download_data, parse_data, save_report
from pipelines.utils.constants import constants as utils_constants # adicionado
from pipelines.utils.decorators import Flow
from pipelines.utils.dump_db.constants import constants as dump_db_constants # adicionado
from pipelines.utils.dump_to_gcs.constants import constants as dump_to_gcs_constants # adicionado
from pipelines.utils.tasks import ( # adicionado
    create_table_and_upload_to_gcs,
    get_current_flow_labels,
)

with Flow("EMD: formacao - Exemplo de flow do Prefect") as formacao_exemplo_flow:
    # Parâmetros
    n_users = Parameter("n_users", default=10)

    # Parâmetros para a Materialização
    materialize_after_dump = Parameter(
        "materialize_after_dump", default=False, required=False
    )
    materialize_to_datario = Parameter(
        "materialize_to_datario", default=False, required=False
    )
    materialization_mode = Parameter("mode", default="dev", required=False)

    # Parâmetros para salvar dados no GCS
    dataset_id = formacao_constants.DATASET_ID.value
    table_id = formacao_constants.TABLE_ID.value
    dump_mode = "append"

    # Dump to GCS after? Should only dump to GCS if materializing to datario
    dump_to_gcs = Parameter("dump_to_gcs", default=False, required=False)

    maximum_bytes_processed = Parameter(
        "maximum_bytes_processed",
        required=False,
        default=dump_to_gcs_constants.MAX_BYTES_PROCESSED_PER_TABLE.value,
    )

    # Cria fluxo das Tasks
    data = download_data(n_users)
    dataframe = parse_data(data)
    save_report(dataframe)


    # Create table in BigQuery
    upload_table = create_table_and_upload_to_gcs(
        data_path=PATH,
        dataset_id=dataset_id,
        table_id=table_id,
        dump_mode=DUMP_MODE,
        wait=PATH,
    )

    # Trigger DBT flow run
    with case(materialize_after_dump, True):
        current_flow_labels = get_current_flow_labels()
        materialization_flow = create_flow_run(
            flow_name=utils_constants.FLOW_EXECUTE_DBT_MODEL_NAME.value,
            project_name=constants.PREFECT_DEFAULT_PROJECT.value,
            parameters={
                "dataset_id": dataset_id,
                "table_id": table_id,
                "mode": materialization_mode,
                "materialize_to_datario": materialize_to_datario,
            },
            labels=current_flow_labels,
            run_name=f"Materialize {dataset_id}.{table_id}",
        )

        materialization_flow.set_upstream(upload_table)

        wait_for_materialization = wait_for_flow_run(
            materialization_flow,
            stream_states=True,
            stream_logs=True,
            raise_final_state=True,
        )

        wait_for_materialization.max_retries = (
            dump_db_constants.WAIT_FOR_MATERIALIZATION_RETRY_ATTEMPTS.value
        )
        wait_for_materialization.retry_delay = timedelta(
            seconds=dump_db_constants.WAIT_FOR_MATERIALIZATION_RETRY_INTERVAL.value
        )

        with case(DUMP_TO_GCS, True):
            # Trigger Dump to GCS flow run with project id as datario
            dump_to_gcs_flow = create_flow_run(
                flow_name=utils_constants.FLOW_DUMP_TO_GCS_NAME.value,
                project_name=constants.PREFECT_DEFAULT_PROJECT.value,
                parameters={
                    "project_id": "datario",
                    "dataset_id": dataset_id,
                    "table_id": table_id,
                    "maximum_bytes_processed": MAXIMUM_BYTES_PROCESSED,
                },
                labels=[
                    "datario",
                ],
                run_name=f"Dump to GCS {dataset_id}.{table_id}",
            )
            dump_to_gcs_flow.set_upstream(wait_for_materialization)

            wait_for_dump_to_gcs = wait_for_flow_run(
                dump_to_gcs_flow,
                stream_states=True,
                stream_logs=True,
                raise_final_state=True,
            )


formacao_exemplo_flow.storage = GCS(constants.GCS_FLOWS_BUCKET.value)
formacao_exemplo_flow.run_config = KubernetesRun(
    image=constants.DOCKER_IMAGE.value,
    labels=[constants.RJ_COR_AGENT_LABEL.value],
)
formacao_exemplo_flow.schedule = None
```

=== "constants.py"

```python
# -*- coding: utf-8 -*-
"""
Constant values for the satelite tables
"""

from enum import Enum


class constants(Enum):
    """
    Constant values for the satelite project
    """

    DATASET_ID = "test_formacao"
    TABLE_ID = "test_table"
```

Modificações que precisamos fazer no arquivo `flows.py`:

- especificar um caminho no qual o dado será salvo localmente antes de ser enviado ao GCP

- fazer o upload os dados no GCP utilizando o método `create_table_and_upload_to_gcs` e o caminho especificado anteriormente

- adicionar a condicional pronta (`with case`) que verificará se os dados serão materializados
