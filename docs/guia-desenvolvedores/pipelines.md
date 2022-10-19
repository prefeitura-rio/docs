Como você deve se recordar, no capítulo sobre [Visão geral da infraestrutura](/guia-desenvolvedores/visao-geral-infra), comentamos sobre o Prefect, que é a ferramenta que utilizamos para automatizar os processos de ETL (Extração, Transformação e Carga de Dados). Neste capítulo, vamos aprofundar um pouco sobre o funcionamento do Prefect e desenvolver uma pipeline de exemplo.

## Como o Prefect funciona?

O Prefect trabalha em um **modelo híbrido**: a **orquestração** de tarefas é feita em um servidor, os **códigos** das tarefas são armazenados em outro e a **execução** é feita em um terceiro. Essa arquitetura permite a descentralização completa das execuções em diferentes centros de custo, além de permitir uma maior privacidade e segurança de todos os servidores envolvidos. Para entender melhor como isso funciona, traduzimos e adaptamos [um texto extraído do site oficial do Prefect](https://www.prefect.io/why-prefect/hybrid-model/), que mostra os passos de desenvolvimento com Prefect:

![Passo 1: Construir seu Flow](../static/img/tutoriais/pipelines/step1.png)

**Passo 1: Construir seu _Flow_**: uma pipeline, no Prefect, recebe o nome de _Flow_. Você deve utilizar o pacote Python `prefect` (em nosso caso, na versão 0.15.9) para desenvolver seu fluxo de trabalho. Depois de desenvolver, você pode executá-lo e testá-lo em sua máquina.

![Passo 2: Registrar seu Flow](../static/img/tutoriais/pipelines/step2.png)

**Passo 2: Registrar seu _Flow_**: esse passo consiste em armazenar seu código em um repositório de objetos, em nosso caso o Google Cloud Storage, de forma que ele possa ser acessado posteriormente. Depois de armazená-lo, sabendo sua localização, você envia esse metadado para o servidor do Prefect (registra o _Flow_). Isso permite que você possa localizá-lo na interface web do Prefect e inspecioná-lo.

_Obs.:_ no caso das pipelines desenvolvidas em nosso repositório no GitHub, esse processo de registrar _Flows_ é automatizado.

![Passo 3: Executar um Agent](../static/img/tutoriais/pipelines/step3.png)

**Passo 3: Executar um agente**: agentes do Prefect são processos que se comunicam com o servidor do Prefect, procurando por cargas de trabalho agendada. Quando ele encontra, lança sua execução. Agentes podem ser executados em qualquer lugar, o que permite que as cargas de trabalho sejam lançadas em diferentes centros de custo.

_Obs.:_ para a Prefeitura do Rio, já existem diversos agentes registrados, esse passo não será necessário.

![Passo 4: Agendar cargas de trabalho](../static/img/tutoriais/pipelines/step4.png)

**Passo 4: Agendar cargas de trabalho**: você pode agendar cargas de trabalho para serem executadas em um determinado horário, ou com certa recorrência. Você pode fazer isso através do próprio código, em Python (recomendado para manter consistência), ou através da interface web do Prefect.

![Passo 5: Executar o _Flow_](../static/img/tutoriais/pipelines/step5.png)

**Passo 5: Executar o _Flow_**: o agente do Prefect encontrará a carga de trabalho agendada e a executará. Dessa forma, a carga de trabalho será executada onde o agente estiver rodando. Ao mesmo tempo, todo o processo de execução e logs poderão ser acompanhados através da interface web do Prefect, já que o agente os comunica ao servidor.

![Passo 6: Monitorar e gerenciar](../static/img/tutoriais/pipelines/step6.png)

**Passo 6: Monitorar e gerenciar**: você pode monitorar e gerenciar as cargas de trabalho através da interface web do Prefect. Você pode ver o status de cada carga de trabalho, visualizar os logs, inspecionar o código, etc. de quaisquer cargas de trabalho, seja em execução ou já finalizadas.

## Desenvolvendo seu primeiro Flow

Aqui, vamos tratar do desenvolvimento de uma pipeline de exemplo, que consiste em:

1. Baixar dados de uma API;
2. Converter esses dados em um formato tabular;
3. Armazenar esses dados em um arquivo CSV.

**Obs:** esse exemplo tratará exclusivamente de uma execução local, em sua máquina, sem interações com o servidor do Prefect ou mecanismos de armazenamento, como o Google Cloud Storage. Abaixo, nessa mesma página, haverão exemplos mais complexos.

### Preparando o ambiente

Requisitos:

- Python 3.9, pois a versão deve coincidir com a que usamos
- `pip`, para instalar dependências

A primeira boa prática é criar um arquivo `requirements.txt` para armazenar as dependências do seu projeto. Nesse arquivo, você deve colocar todas as bibliotecas que serão utilizadas no desenvolvimento do seu _Flow_. O que vamos usar, por exemplo, é o seguinte:

```
prefect==0.15.9
requests
pandas
```

Note que o único pacote que tem sua versão especificada é o Prefect, pois a versão dele deve corresponder à que usamos em nossos servidores.

Em seguida, para instalar as dependências declaradas nesse arquivo, você deve executar o seguinte comando:

```bash
pip install -r requirements.txt
```

E pronto, você já tem tudo o que precisa para começar a desenvolver seu _Flow_. Caso, no futuro, você deseje adicionar novas dependências, basta adicionar no arquivo `requirements.txt` e executar o comando acima novamente.

Para manter um padrão de desenvolvimento, devemos criar os seguintes arquivos:

- `tasks.py`: arquivo que conterá as funções que serão utilizadas no _Flow_.
- `flows.py`: arquivo que conterá o _Flow_ propriamente dito.
- `utils.py`: arquivo que conterá funções auxiliares, como por exemplo, funções para imprimir logs.

### Criando funções auxiliares

Começando então no arquivo `utils.py`, vamos criar uma função que imprime logs no console. Essa função será útil para acompanhar o andamento do _Flow_. Como é uma função que envolve conhecer o Prefect um pouquinho mais a fundo, vamos explicar o que ela faz.

```python
import prefect

def log(message) -> None:
    """Logs a message"""
    prefect.context.logger.info(f"\n{message}")
```

A função `log` recebe uma mensagem e a imprime no console. Ela faz isso através do objeto `prefect.context.logger`, que é um objeto que contém o logger do Prefect. Esse logger é um objeto que está dentro do `prefect.context`, esse que contém informações sobre o _Flow_ e a execução atual, como por exemplo, o nome do _Flow_ e o ID da execução atual. Por isso, é importante que essa função seja chamada dentro de um _Flow_, pois ela depende do objeto `prefect.context` para funcionar.

### Criando _tasks_

Aqui é onde vamos criar as funções que implementam os passos da nossa pipeline. Essas funções serão chamadas de _tasks_. Então, abrindo o arquivo `tasks.py`, vamos criar as seguintes funções:

```python
from io import StringIO

import pandas as pd
from prefect import task
import requests

from utils import log

@task
def download_data(n_users: int) -> str:
    """
    Baixa dados da API https://randomuser.me e retorna um texto em formato CSV.

    Args:
        n_users (int): número de usuários a serem baixados.

    Returns:
        str: texto em formato CSV.
    """
    response = requests.get(
        "https://randomuser.me/api/?results={}&format=csv".format(n_users)
    )
    log("Dados baixados com sucesso!")
    return response.text

@task
def parse_data(data: str) -> pd.DataFrame:
    """
    Transforma os dados em formato CSV em um DataFrame do Pandas, para facilitar sua manipulação.

    Args:
        data (str): texto em formato CSV.

    Returns:
        pd.DataFrame: DataFrame do Pandas.
    """
    df = pd.read_csv(StringIO(data))
    log("Dados convertidos em DataFrame com sucesso!")
    return df

@task
def save_report(dataframe: pd.DataFrame) -> None:
    """
    Salva o DataFrame em um arquivo CSV.

    Args:
        dataframe (pd.DataFrame): DataFrame do Pandas.
    """
    dataframe.to_csv("report.csv", index=False)
    log("Dados salvos em report.csv com sucesso!")
```

Como as funções estão comentadas, não há muito o que explicar. Mas vale ressaltar que elas são decoradas com o `@task`, que é um decorator do Prefect. Esse decorator é necessário para que o Prefect reconheça as funções como _tasks_.

### Definindo o _Flow_

Agora, vamos criar o _Flow_ propriamente dito. Aqui vamos definir como as _tasks_ interagem entre si e os parâmetros que elas recebem. Para isso, abra o arquivo `flows.py` e crie a seguinte função:

```python
from prefect import Flow, Parameter

from tasks import (
    download_data,
    parse_data,
    save_report,
)

with Flow("Users report") as flow:

    # Parâmetros
    n_users = Parameter("n_users", default=10)

    # Tasks
    data = download_data(n_users)
    dataframe = parse_data(data)
    save_report(dataframe)
```

Aqui, criamos um _Flow_ chamado `Users report` com o seguinte parâmetro:

- `n_users`: número de usuários a serem baixados.

Observe que foi configurado um valor padrão para esse parâmetro, que é 10. Isso significa que, caso o parâmetro não seja passado na hora de executar o _Flow_, ele será executado com o valor padrão.

Além disso, inserimos as três _tasks_ que criamos anteriormente. Observe que a _task_ `download_data` recebe o parâmetro `n_users` como argumento. Isso significa que, quando a _task_ for executada, ela receberá o valor do parâmetro `n_users` como argumento. Depois disso, a _task_ `parse_data` recebe o retorno da _task_ `download_data` como argumento, fazendo com que o dado baixado seja passado a ela, para converter em um DataFrame. Por fim, a _task_ `save_report` recebe o retorno da _task_ `parse_data` como argumento, fazendo com que o DataFrame seja salvo em um arquivo CSV.

### Executando

Ufa! Com tudo pronto, vamos executar o _Flow_! Para facilitar a execução, vamos criar um arquivo `run.py` e colocar o seguinte código nele:

```python
from flows import flow

flow.run()
```

Depois, podemos executar o _Flow_ com o seguinte comando:

```bash
python3 run.py
```

A saída deve ser semelhante ao seguinte:

```
[2022-10-05 14:40:40-0300] INFO - prefect.FlowRunner | Beginning Flow run for 'Users report'
[2022-10-05 14:40:40-0300] INFO - prefect.TaskRunner | Task 'n_users': Starting task run...
[2022-10-05 14:40:40-0300] INFO - prefect.TaskRunner | Task 'n_users': Finished task run for task with final state: 'Success'
[2022-10-05 14:40:40-0300] INFO - prefect.TaskRunner | Task 'download_data': Starting task run...
[2022-10-05 14:40:40-0300] INFO - prefect.download_data |
Dados baixados com sucesso!
[2022-10-05 14:40:40-0300] INFO - prefect.TaskRunner | Task 'download_data': Finished task run for task with final state: 'Success'
[2022-10-05 14:40:40-0300] INFO - prefect.TaskRunner | Task 'parse_data': Starting task run...
[2022-10-05 14:40:40-0300] INFO - prefect.parse_data |
Dados convertidos em DataFrame com sucesso!
[2022-10-05 14:40:40-0300] INFO - prefect.TaskRunner | Task 'parse_data': Finished task run for task with final state: 'Success'
[2022-10-05 14:40:40-0300] INFO - prefect.TaskRunner | Task 'save_report': Starting task run...
[2022-10-05 14:40:40-0300] INFO - prefect.save_report |
Dados salvos em report.csv com sucesso!
[2022-10-05 14:40:40-0300] INFO - prefect.TaskRunner | Task 'save_report': Finished task run for task with final state: 'Success'
[2022-10-05 14:40:40-0300] INFO - prefect.FlowRunner | Flow run SUCCESS: all reference tasks succeeded
```

Observe que um arquivo chamado `report.csv` foi criado. Esse arquivo contém os dados baixados e convertidos em um DataFrame.

Observe também que as mensagens de log foram impressas na saída. Isso acontece porque, como vimos anteriormente, as funções `log` são decoradas com o `@task`, que é um decorator do Prefect. Isso faz com que o Prefect reconheça as funções como _tasks_ e, por isso, as mensagens de log são impressas na saída.

### Executando com parâmetros

Agora, vamos executar o _Flow_ passando um valor para o parâmetro `n_users`. Para isso, vamos modificar o arquivo `run.py` para o seguinte:

```python
from flows import flow

flow.run(n_users=20)
```

Novamente, podemos executar o _Flow_ com o seguinte comando:

```bash
python3 run.py
```

E pronto! O _Flow_ foi executado com sucesso e um arquivo `report.csv` foi criado. Observe que, dessa vez, o arquivo contém 20 linhas, pois o parâmetro `n_users` foi passado com o valor 20.
