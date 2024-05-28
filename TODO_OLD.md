# DESCRIÇÃO DO TRABALHO PRÁTICO - PROJETO DE INGESTÃO DE DADOS

## Índice
- [Instruções](#instruções)
  - [Requisitos obrigatórios](#requisitos-obrigatórios)
  - [Requisitos opcionais](#requisitos-opcionais)
- [Resumo sobre o dataset](#resumo-sobre-o-dataset)
  - [Dados externos enviados pelos fornecedores](#dados-externos-enviados-pelos-fornecedores)
  - [Dados internos da empresa](#dados-internos-da-empresa)
  - [Premissas do dataset](#premissas-do-dataset)
- [Tarefas a serem realizadas](#tarefas-a-serem-realizadas)

## Instruções
### Requisitos obrigatórios:
- Usar Apache Spark v3.5+
- Ingestão dos dados no datalake precisa ser em formato parquet
- Necessário que o código seja capaz de rodar em Apache Spark v3.5+, podendo ser feito em qualquer linguagem de programação compatível com o Spark
- É aceito como entrega desde um simples arquivo texto com uma lista de comandos quanto um projeto de software contendo diversos arquivos. Dica: o mesmo pode ser realizado no próprio Spark shell e salvo como visto em aula, não necessitando grande elaboração
- Arquivo README com uma breve descrição das ferramentas usadas, como foi entregue o código e como usar;
- Como datalake do projeto, vamos considerar um diretório local com a seguinte estrutura (a mesma está no drive da disciplina):

```sh
<path_base_de_cada_um>/simulatedDataLake
  /refined
    /batch

      (Local onde deve ser armazenados os dados que são recebidos externamente, ou seja, autores, livros e reviews de filmes baseados em livros, por exemplo, um diretório com arquivos parquet para cada tipo, se de autores podendo ser .../batch/authors.parquet/<arquivos de autores>) 
    
    /stream
    
      (Local onde deve ser armazenados os dados que são recebidos internamente, ou seja, filmes, usuários e atividades de streaming, por exemplo, um diretório com arquivos parquet para cada tipo, se de livros podendo ser .../stream/books.parquet/<arquivos de livros>)

  /raw
    /internal
      movies.csv
      streams.csv
      users.csv
    /external
      authors.json
      books.json
      reviews.json
```

### Requisitos opcionais:
- Usar Apache Kafka qualquer versão a partir de 2.x. 

> IMPORTANTE: O uso de Kafka, desde que sem erros de execução do código, irá acrescentar 10 PONTOS ao trabalho, ou seja, podendo elevar o total de pontos dele a 80. Se a parte que usar Kafka não funcionar, o trabalho seguirá sendo avaliado em 70 pontos.

- Para 1 ou mais dos dados internos da empresa fictícia (users, movies, streams) a ingestão ser feita usando "streaming" (Spark Structured Streaming), com ou sem Kafka.

> IMPORTANTE: O uso de Streaming, desde que sem erros de execução do código, irá acrescentar 10 PONTOS ao trabalho, ou seja, podendo elevar o total de pontos dele a 80. Se a parte que usar Streaming não funcionar, o trabalho seguirá sendo avaliado em 70 pontos.

> IMPORTANTE: Usar Kafka + Streaming, ou qualquer deles individualmente é TOTALMENTE OPCIONAL. O limite de pontuação do trabalho NÃO PODE ULTRAPASSAR 80 pontos.


## Resumo Sobre o Dataset

O dataset disponibilizado contém informações sobre um aplicativo de streaming de uma pequena startup no mês de Dezembro de 2021. A empresa mantém dados sobre usuários, filmes e a atividade de streaming (quem assistiu qual filme). Também para enriquecer o produto e a parte analítica da empresa, dados de fontes externas são capturados com informações sobre livros, autores e reviews de filmes baseados em livros que podem ser relacionados e ligados aos dados de filmes internos.

### Dados externos enviados pelos fornecedores

- authors.json: Informação sobre autores famosos.
- books.json: Informação agregada sobre livros famosos.
- reviews.json: Reviews sobre filmes baseados em livros.

### Dados internos da empresa

- users.csv: Usuários ativos do aplicativo.
- movies.csv: Filmes disponíveis no aplicativo.
- streams.csv: Dados históricos de Dezembro de 2021 sobre atividade de streaming de filmes.

### Premissas do dataset

- 2 autores diferentes não usam o mesmo nome ao publicar livros;
- 2 livros diferentes não podem ser publicados usando exatamente o mesmo título;
- Usuários podem assistir mais de um filme ao mesmo tempo;
- Emails de usuário são verificados e únicos no banco de dados.


## Tarefas a serem realizadas

1) Trecho de código Spark para ingestão dos 6 tipos de dados, colocando no datalake os mesmos em formato "parquet", ao qual é apropriado para consultas analíticas, com as seguintes regras:
  - a) Dados externos são recebidos como conjuntos completos, ou seja, a cada execução substitui a base toda.
  
> ATENÇÃO: Se o código de vocês for executado 2 ou mais vezes o mesmo NÃO pode duplicar os dados externos, isso vai ser avaliado. Neste conjunto será simulado o recebimento de dados de fornecedores via "batch";

  - b) Dados internos são recebidos em conjuntos incrementais, ou seja, a cada execução os mesmos são "apendados" ou "incrementados".
  
> ATENÇÃO: Se o código de vocês for executado 2 ou mais vezes com os mesmos dados ele DEVE duplicar os dados internos, e isso será avaliado. Neste conjunto será simulado o recebimento de dados do app via "streaming";

  - c) Independente do tipo do dado original recebido nos conjuntos do item a) e b), o parquet gerado deverá "transformar" os campos da seguinte forma:
    - Campos de data e hora, deverão ser do tipo "timestamp". Exemplo: no conjunto de autores, o campo "birth_date" deverá ser corretamente convertido para timestamp no parquet que conter autores;
    - Campos contendo números inteiros, deverão ser convertidos para tipo "int". Exemplo: no conjunto de filmes, o campo "duration_mins" deverá ser corretamente convertido para timestamp no parquet que conter filmes; 
    - Campos que possuem números reais (possuem ponto como separador decimal), deverão ser convertidos para tipo "decimal". Exemplo: no conjunto de filmes assistidos (streamed), o campo "size_mb" deverá ser corretamente convertido para decimal no parquet que conter estes dados após ingestão;

> OBS: é opcional fazer de fato "streaming" e / ou usar Kafka, como já explicado nos requisitos, pode simular usando formulação "batch" no Spark.

2) No datalake "refined" (<path_base_de_cada_um>/simulatedDataLake/refined), o código deve resultar em conjuntos de dados em parquet que sejam capazes de resolver consultas com Spark SQL. Para que vcs possam validar se os dados resultantes na tarefa 1 estão corretos com as regras a, b e c do item 1), abaixo estão o exemplo sintético de 2 queries e resultado esperado:

> IMPORTANTE: A avaliação da corretude dos dados em parquet após ingestão será feita aplicando um conjunto de 6 queries em Spark SQL, dentre elas, as 2 que constam abaixo para validação de todos. A minha dica é que vocês usem estas queries(SQL1 e SQL2) e confiram se o resultado esperado é idêntico, caso negativo, revalidar se cumpriram as regras do item 1).

> OBSERVAÇÃO: Trecho de código em Spark usando linguagem Scala, mas, o importante é mostrar a query SQL.

SQL1 - Qual o percentual de filmes assistidos (streamed) são baseados em livros?
      
```
val query1 = spark.sql(
      """
        |SELECT Count_if(r.movie IS NOT NULL)            streams_based_books_tot,
        |       Count(*)                                 streams_tot,
        |       Count_if(r.movie IS NOT NULL) / Count(*) books_based_perc
        |FROM   internal_streams s
        |       LEFT JOIN (SELECT movie
        |                  FROM   external_reviews
        |                  GROUP  BY movie) AS r
        |              ON ( s.movie_title = r.movie )
        |""".stripMargin)

    var initTime:Long = System.currentTimeMillis
    query1.show(false)
    log.error("Query1 time: " + (System.currentTimeMillis - initTime) + " ms.")
    /*
    Resultado esperado:

    +-----------------------+-----------+------------------+
    |streams_based_books_tot|streams_tot|books_based_perc  |
    +-----------------------+-----------+------------------+
    |9023                   |9652       |0.9348321591380024|
    +-----------------------+-----------+------------------+

    Resposta: 0.9348321591380024 ou 93,48321591380024%
    */
```

SQL2 - Quantos filmes baseados em livros escritos por autores de Singapura foram assistidos no mês disponível nos dados (Dezembro de 2021)?

```
val query3 = spark.sql(
  """
    |SELECT Count(DISTINCT s.movie_title, s.user_email, s.size_mb, s.start_at, s.end_at)
    |       movie_streamed_tot
    |FROM   external_authors a
    |       INNER JOIN external_books b
    |               ON ( a.name = b.author )
    |       INNER JOIN external_reviews r
    |               ON ( b.name = r.book )
    |       INNER JOIN internal_streams s
    |               ON ( s.movie_title = r.movie )
    |WHERE  a.nationality = 'Singaporeans'
    |""".stripMargin)
initTime = System.currentTimeMillis
query3.show(false)
log.error("Query3 time: " + (System.currentTimeMillis - initTime) + " ms.")

/*
Resultado esperado:

+------------------+
|movie_streamed_tot|
+------------------+
|               326|
+------------------+

Resposta: 326 filmes assistidos (streamed)
*/
```