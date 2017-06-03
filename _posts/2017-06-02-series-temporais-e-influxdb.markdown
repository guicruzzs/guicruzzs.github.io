---
layout: post
title:  "Séries temporais e o InfluxDB"
date:   2017-06-02
---

<p class="intro"><span class="dropcap">O</span>lar.</p>

Série temporal (*Time series*)


InfluxDB
 - Todo registro no banco é obrigatório que se registre a coluna `time` com o conteúdo de uma timestamp.
 - *Fields* representam colunas que você nomea e armazena dados que são importantes para a sua respresentação dos dados, funciona no modelo chave-valor.
  - *Field Key* é uma string e representa o nome da coluna.
  - *Field Value* são os dados armazenado do campo
  - Não se pode ter um registro sem um field, são obrigatórios.
  - *Fields* não são indexados, isto é, a busca por um valor no banco implica na leitura de todos os valores e após isso a sua filtragem. O que implica em consulta mais lentas.
  - *Fields* não devem (diferente de podem) possuir dados que precisam de critérios de busca (indexação).

  O *Field* `time` e qualquer outro *Field* são obrigatórios por serem a base de um banco de dados orientado a séries temporais, afinal o principal objetivo é possuir um

- *Tags* representam colunas que você nome e armazena dados também, no mesmo formato que os *Field Keys*: em chave-valor. Entretanto, o formato da chave e do valor são *strings*, ou seja, quando armazenar alguma informação, mesmo que de carater numérico, será convertida em *string*.
  - *Tags* são indexadas, ou seja, as preferencialmente adotadas como critério de busca.
  - *Tags* são opcionais na estrutura de dados, mas se não utiizá-las implicará em busca de informações mais lentas.

- *Measurement* é um agrupador de *Tags* e *Fields*, em paralelo ao modelo de Entidade relacionamento, o conceito de *Measurement* é equivalente ao da tabela.
  - O nome do *Measurement* representa o valor dos valores que são armazenados nele.

- Uma série (*series*) é um conjunto de dados (pontos) resultantes da parametrização das *Tags*.
- *Database* é o mesmo conceito do que o já aplicado e conhecido por bancos SQL tradicionais.


Bom dia e boa sorte!