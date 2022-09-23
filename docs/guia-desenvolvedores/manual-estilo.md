 
Nesta seção listamos todos os padrões do nosso manual de estilo e diretrizes de dados que usamos no Escritório de Dados. Eles nos ajudam a manter os dados e metadados que publicamos com qualidade alta.
 
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
- `terceiro_nivel`: refere-se à secretaria ou sistema (opcional)
 
Exemplo: educacao_basica_aluno; educacao_basica_escola; meio_ambiente_clima_satelite; meio_ambiente_clima_radar
 
### Tabelas (table_id)
 
Nomear tabelas é algo menos estruturado e, por isso, requer bom senso. Mas temos algumas regras:
 
- Não incluir a unidade temporal no nome. Exemplo: educacao_basica.aluno, e não educacao_basica.aluno_ano
- Deixar nomes no singular. Exemplo: educacao_basica_escola.escola, e não educacao_basica_escola.escolas
- Uso de sinônimos para facilitar o entendimento do usuário. Exemplo: educacao_basica_avaliacao.prova_rio, e não educacao_basica_avaliacao.prova
 
### Restrições de nomeação
 
Para mantermos uma consistência nas nomeações são necessárias algumas restrições:
 
 
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
 
 
## Nome e ordem das colunas
 
A ordem das colunas em tabelas é padronizada para manter uma consistência no repositório. Nossas regras são:
 
- Chaves primárias à esquerda, em ordem descendente de abrangência. Exemplo de ordem: `ano`, `sigla_uf`, `id_municipio`, `id_escola`, `nota_ideb`.   
- Agrupar e ordenar colunas por importância ou temas.
 
### Tipos de colunas
 
Nós utilizamos algumas das opções de [tipos do BigQuery](https://cloud.google.com/bigquery/docs/reference/standard-sql/data-types): `STRING`, `INT64`, `FLOAT64`, `DATE`, `TIME`, `GEOGRAPHY`.
 
Quando escolher:
 
- `STRING`:
   - Colunas de texto
   - Chaves de colunas categóricas com dicionário
- `INT64`:
   - Colunas de números inteiros com as quais é possível fazer contas (adição, subtração).
- `FLOAT64`:
   - Colunas de números com casas decimais com as quais é possível fazer contas (adição, subtração).
- `DATE`:
   - Colunas de data no formato `YYYY-MM-DD`.
- `TIME`:
   - Colunas de tempo no formato `HH:MM:SS`.
- `GEOGRAPHY`:
   - Colunas de geografia.
 
 
### Quais colunas manter, quais adicionar e quais remover
 
Mantemos nossas tabelas parcialmente [normalizadas](https://www.guru99.com/database-normalization.html), e temos regras para quais colunas incluirmos em produção. Elas são:
 
- Remover colunas de nomes de entidades que já estão nos `dados_mestres`. Exemplo: retirar `bairro` da tabela que já inclui `id_bairro`.
- Remover colunas servindo de partição. Exemplo: remover `ano` e `id_bairro` se a tabela é particionada nessas duas dimensões.
- Adicionar chaves primárias principais para cada entidade já existente. Exemplo: adicionar `id_bairro` a tabelas que só incluem `bairro`.
- Manter todas as chaves primárias que já vem com a tabela, mas (1) adicionar chaves relevantes (e.g. `id_bairro`) e (2) retirar chaves irrelevantes (e.g. `regiao`).
 
 
### Limpando STRINGs
 
- Colunas categóricas: inicial maiúscula e resto minúsculo, com acentos.
- STRINGs não-estruturadas: manter igual aos dados originais.
 
### Unidades de medida
 
A regra é manter colunas com suas unidades de medida originais, com a exceção de colunas financeiras onde convertermos moedas antigas para as atuais (e.g. Cruzeiro para Real).
 
Colunas devem ter sempre unidades de medida com base 1. Ou seja, ter `BRL` ao invés de `1000 BRL`, ou `pessoa` ao invés de `1000 pessoas`. Essa informação, como outros metadados de colunas, são registradas na tabela de arquitetura da tabela.
 
### Formatos de valores
 
- Decimal: formato americano, i.e. sempre `.` (ponto) ao invés de `,` (vírgula).
- Data: `YYYY-MM-DD`
- Horário (24h): `HH:MM:SS`
- Datetime ([ISO-8601](https://en.wikipedia.org/wiki/ISO_8601)): `YYYY-MM-DDTHH:MM:SS.sssZ`
- Valor nulo: `""` (csv), `NULL` (Python), `NA` (R), `.` ou `""` (Stata)
- Proporção/porcentagem: entre 0-100
 
### Dicionários
 
 A tabela template para preenchimento do dicionário pode ser [baixada aqui](https://docs.google.com/spreadsheets/d/1ZKsa1-DMTyfF6F3KtdKX2Zii4inCX_Hk/edit?usp=sharing&ouid=102857770245123086140&rtpof=true&sd=true)

- Cada base inclui somente um dicionário (que cobre uma ou mais tabelas).
- Para cada tabela, coluna, e cada chave mapeia unicamente um valor.
- Chaves não podem ter valores nulos.
- Dicionários devem cobrir todas as chaves disponíveis nas tabelas originais.
- Chaves só podem possuir zeros à esquerda quando o número de dígitos da coluna tiver significado. Quando a coluna for `enum` padrão, nós excluimos os zeros à esquerda.
 
- Valores são padronizados: sem espaços extras, inicial maiúscula e resto minúsculo, etc.
 
### Particionamento
 
 
Uma tabela particionada é uma tabela especial dividida em segmentos, chamados de partições, que facilitam o gerenciamento e a consulta de seus dados. Ao dividir uma grande tabela em partições menores, você pode melhorar o desempenho da consulta e pode controlar os custos reduzindo o número de bytes lidos por uma consulta. Por isso, sempre recomendamos que tabelas grandes sejam particionadas. Leia mais a respeito na [documentação da Google Cloud](https://cloud.google.com/bigquery/docs/partitioned-tables).
 
Em geral, o particionamento é feito sobre o valors de uma data criando-se três colunas novas cujos nomes são dados por: `ano_particao`, `mes_particao`, `data_particao` e que possuem os respectivos formatos 'YYYY', 'MM', 'YYYY-MM-DD'.
 
**Datas nulas em coluna de particionamento** - caso a coluna utilizada para particionamento seja nula, preencher esses valores pela Unix epoch (1 de janeiro de 1970).
 
## Dados Mestres
 
Dados Mestres são as pedras fundamentais da estrutura do nosso _datalake_. Nossas regras para gerenciar esses dados são:
 
- Representam _entidades_ do repositório que tenham chaves primárias (e.g. `bairro`, `logradouro`)
- Cada tabela tem ao menos uma chave primária com valores únicos e sem nulos. Exemplos: `bairro:id_bairro`, `logradouro:id_trecho`.
- Nomes de colunas com prefixo `id_` são reservadas para chaves
 primárias de entidades.
 
 
Veja todas as [tabelas públicas já disponíveis aqui.](https://www.data.rio/search?q=datalake)
 
## **Pensou em melhorias para os padrões definidos?**
 
Abra um [issue no nosso Github](https://github.com/prefeitura-rio/docs/issues/new/choose) ou mande uma mensagem no [Discord](https://discord.gg/WcvNkTbYdA) para conversarmos :)
 


