## Lib prefeitura-rio


Pacote Python que implementa utilidades para nossas Pipelines.

## State handlers 


Definição de credenciais na definição dos flows

```py
from functools import partial

dump_sme_flow.state_handlers = [
    partial(handler_inject_bd_credentials, path="/bd-rj-chatbot-dev"),
    handler_initialize_sentry,
]
```

----



## Flow dump_db

### Paramêtro break_query

O parametro break_query mitiga o problema de perda de conexão com banco. Ele quebra a query em multiplas queries usando os parametros `break_query_start, break_query_end, break_query_frequency` e vai subindo query por query sequencialmente. Caso o banco morra pelo menos ja subiu uma parte dos dados e é só continuar de onde parou. 

`break_query_frequency` aceita `year`, `month`, `day`, `week`, `bimester`, `trimester`, `quadrimester`, `semester` e `None`

`break_query_start` e `break_query_end` aceitam `current_day`, `current_month`, `current_year`, alguma data no mesmo formato do `partition_date_format` e `None`


### Paramêtro lower_bound_date

Define uma data de onde a query particionada ira iniciar. Aceta os valores `current_day`, `current_month`, `current_year`, alguma data no mesmo formato do `partition_date_format` e `None`



## Flow run_dbt_model

### Paramêtro `infisical_secret_dict` e `dbt_project_materialization`

Em conjunto permitem trocar o projeto e as credencias do projeto de materializacao durante a execução

```py
"infisical_credential_dict" =  {
    "secret_path": "secret_path",
    "secret_name": "secret_name",
},
"dbt_project_materialization" = "project-name"
```