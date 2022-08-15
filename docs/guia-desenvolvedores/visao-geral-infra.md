E aí, quer entender melhor nossa infraestrutura de dados, os componentes selecionados, como interagem
entre si e onde estão hospedados? Então vamos lá! 👨‍💻

## Introdução ao fluxo de dados

Não há nada extravagante em nosso fluxo de dados, na realidade, ele é tão intuitivo quanto pode ser.
No entanto, é muito importante entender cada etapa para que seja possível compreender o funcionamento
da infraestrutura como um todo.

Então, vamos colocar aqui um diagrama que ajudará a entender por onde o dado transita e no que consistem
as etapas:

![Diagrama de etapas de processamento dos dados](../static/img/tutoriais/visao-geral-infra/visao-geral-sem-componentes.jpg)

Como estamos falando sobre a infraestrutura de dados, vamos focar única e exclusivamente na parte
central desse diagrama, ou seja, as três grandes etapas - Ingestão, Transformação e Aplicação - e o
bloco central, o Armazém de Dados.

> Sem mencionar componentes, colocar um diagrama mostrando o fluxo de extração, loading, transformação,
> loading de novo e disponibilização dos dados. Explicitar fontes de dados e usos possíveis para os dados
> tratados.

## Apresentação dos componentes

> Mostrar cada componente selecionado, os motivos de escolher cada um (ver matéria de infra pra levantar
> alguns pontos mencionados lá, mencionar tbem importância do open-source e da facilidade de migração)

## Como estão hospedados

> Explicar como as coisas são hospedadas, falar da descentralização, escalabilidade, Kubernetes,
> repositórios, GitHub Actions, etc.

## Comunicação e interações

> Os componentes precisam se comunicar pra fazer com que tudo funcione. Mencionar como isso é feito,
> falar do basedosdados que é um pacote importante que a gente usa, etc. Falar do BigQuery, GCS,
> tabelas externas, etc.
