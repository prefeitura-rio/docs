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

> - Criar repositório de exemplo
> - Criar tasks simples
> - Criar Flow
> - Executar Flow localmente

<!-- ## O repositório de pipelines


## Concatenando Flows

## Adicionando dependências

## Como testar pipelines em diversos ambientes

## Colocando uma pipeline em produção

## UI do Prefect -->
