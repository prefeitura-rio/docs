Nessa seção listamos todos os padrões do nosso manual de estilo e diretrizes de dados que usamos no Escritório de Dados. Eles nos ajudam a manter os dados e metadados que publicamos com qualidade alta.

## Como estão estruturados os dados no ED?

A estrutura de dados utilizada segue a mesma hierarquia do BigQuery com project no nível mais alto, seguido de dataset e table.

!['Hierarquia de objetos'](https://cloud.google.com/static/bigquery/images/table-details.png?hl=pt-br)

## Quais as melhores práticas na criação das tabelas?

### Conjuntos de dados (dataset_id)

A estrutura utilizada para nomeação de tabelas seguirá boas práticas de nomeação. Entretanto, o objetivo principal é nomear de modo que seja possível identificar a essência da base disponível.
É importante que os datasets possuam alguma granularidade, pois não queremos um unico dataset com dezenas de tabelas.

Nomeamos datasets no formato `<primeiro_nivel>_<segundo_nivel>_<terceiro_nivel>`
Portanto a composição dos nomes serão em dois níveis.

- `primeiro_nivel`: refere-se ao tema pertinente à área.
- `segundo_nivel`: refere-se ao objeto do conjunto
- `terceiro_nivel`: refere-se a secretaria ou sistema (opcional)

 Exemplo: educacao_basica_aluno; educacao_basica_escola; meio_ambiente_clima_satelite; meio_ambiente_clima_radar

### Tabelas (table_id)

Nomear tabelas é algo menos estruturado e, por isso, requer bom senso. Mas temos algumas regras:

- Não incluir a unidade temporal no nome. Exemplo: educacao_basica.aluno, e não educacao_basica.aluno_ano
- Deixar nomes no singular. Exemplo: educacao_basica_escola.escola, e não educacao_basica_escola.escolas
- Uso de sinônimos para facilitar o entendimento do usuário. Exemplo: educacao_basica_avaliacao.prova_rio, e não educacao_basica_avaliacao.prova

### Restrições de nomeação

Para mantermos uma consistencia nas nomeações são necessarias algumas restrições


- Ter sempre 2 níveis de identificação do dado: dataset_id (nome do conjunto) + table_id (nome da tabela)
- Ter todas letras minúsculas (inclusive siglas), sem acentos, conectados por \_
- Não incluir conectores como de, da, dos, e, a, em, etc.
- Não deve ser utilizado como segundo nível “Outros…”. Caso a categoria do seu conjunto não se encaixe em nenhuma das opções, é permitida a sugestão de novos termos desde que aprovado na reunião de Gestão do Datalake. (ex: administracao_processo).
- Nenhuma palavra deve ser repetida no dataset*id ou tabel_id (ex: educacao_basica.turmas deve ser usado, e não educacao_educacao*…)
- O dataset_id pode ser ser nomeado como "dados_mestres", caso a tabela em questão seja uma referência para outras tabelas de outros sistemas incluídos no datalake.


**Referências:**

- [Tabela de temas do Escritório de Dados](https://docs.google.com/spreadsheets/d/12A6NWEAPtYnFkeIMbBNSeq04PLSQJhPa4HIwaDXAo3I/edit#gid=296761183)
- [Manual de estilo e diretrizes de dados - Documentação (Base dos Dados)](https://basedosdados.github.io/mais/style_data/)
- [Ontologia SMTR](https://github.com/RJ-SMTR/maestro/wiki/Ontologia-de-nomea%C3%A7%C3%A3o-Datasets-e-Tabelas)


### Nome e ordem das colunas

O nome e a ordem das colunas tem que respeitar o mesmo padrão descrito nas partes "Nomeação de variáveis" e "Ordenamento de variáveis" do [Manual de estilo e diretrizes de dados - Documentação (Base dos Dados)](https://basedosdados.github.io/mais/style_data/)

### Conteúdo das variáveis

Sobre o conteúdo das colunas deve-se respeitar o mesmo padrão descrito nas partes "Tipos de Variáveis", "Limpando STRINGs", "Formatos de valores" e "Quais variáveis manter, quais adicionar e quais remover" do [Manual de estilo e diretrizes de dados - Documentação (Base dos Dados)](https://basedosdados.github.io/mais/style_data/)






### Particionamento

Tabelas grandes ou com grande potencial de crescimento devem ser particionadas a fim de melhorarmos o desempenho de consulta e reduzir os custos.

Em geral, o particionamento é feito sobre o valors de uma data criando-se três colunas novas cujos nomes são dados por: `ano_particao`, `mes_particao`, `data_particao` e que possuem os respectivos formatos 'YYYY', 'MM', 'YYYY-MM-DD'.

**Datas nulas em coluna de particionamento** - caso a coluna utilizada para particionamento seja nula, preencher esses valores pela Unix epoch (1 de janeiro de 1970).

Em caso de necessidade de particionamento por outra coluna consultar o tópico "Particionamento de tabelas" no [Manual de estilo e diretrizes de dados - Documentação (Base dos Dados)](https://basedosdados.github.io/mais/style_data/).