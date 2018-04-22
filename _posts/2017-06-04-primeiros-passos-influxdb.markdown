---
layout: post
title:  "Primeiros passos no InfluxDB"
date:   2017-06-04
description: Manipular o InfluxDB pode parecer uma missão muito difícil inicialmente, mas quem está acostumado com SQL verá que não é tão diferente disso.
---

<p class="intro"><span class="dropcap">M</span>anipular o InfluxDB pode parecer uma missão muito difícil inicialmente, mas quem está acostumado com SQL verá que não é tão diferente disso.</p>

Eu já falei sobre a estrutura do InfluxDB e sobre o que é uma série temporal em [outro post aqui no blog](/blog/series-temporais-e-influxdb). Desta vez, quero mostrar a manipulação dos dados no banco, então bora colocar a mão na massa!

### Setup rápido
Eu sugiro rodar o InfluxDB num [docker container](https://www.docker.com/what-container) pra facilitar a diversão:

{% highlight shell %}
$ docker run --name influxzinho influxdb # use a opção -d pra rodar na opção detached
{% endhighlight %}

Mas você pode instalar o InfluxDB normalmente seguindo o [manual de instalação oficial](https://portal.influxdata.com/downloads#influxdb) :)

Então, é só abrir o client do banco dentro do container, vamos logo começar a festa:

{% highlight shell %}
$ docker exec -it influxzinho bash
# Dentro do container:
$ influx
{% endhighlight %}

É importante ressaltar que, por padrão, o InfluxDB roda na porta `8086`.

O InfluxDB possui uma sintaxe própria para manipular suas estruturas de dados: o InfluxQL (InfluxDB Query Language). A partir daqui, é somente ele que utilizo para manipular o banco.

### Databases

Para listar as *databases* existentes:

{% highlight sql %}
> SHOW DATABASES
name: databases
name
----
_internal

>
{% endhighlight %}

O nome da *database* padrão do InfluxDB é a `_internal`.

Para criar uma *database* nova basta usar o comando

{% highlight sql %}
> CREATE DATABASE test
>
{% endhighlight %}

Uma vez criada, basta então alternar entre as *databases*:

{% highlight sql %}
> USE test
Using database test
>
{% endhighlight %}

### Inserção de dados
E agora? Vamos criar uma tabela? Determinar as colunas? E também os *Fields*, as *Tags* e os seus tipos?
Nada disso é necessário, jovem Padawan. A sintaxe da inserção de dados é responsável por definir toda a estrutura dos dados.

O comando de criação de um ponto (registro) no banco é o `INSERT` (use ele com cuidado e carinho) que é descrito da seguinte maneira:

{% highlight xml %}
<measurement>[,<tag_key>=<tag_value>[,<tag_key>=<tag_value>]] <field_key>=<field_value>[,<field_key>=<field_value>] [<timestamp>]
{% endhighlight %}

O `measurement` é obrigatório, por isso é que está envolvido pelos sinais `<>`. O que está entre colchetes simboliza que o seu conteúdo é opcional.

Na prática, os comandos têm baixa complexidade. Vamos tomar o exemplo de medições de temperaturas no meu banco:

{% highlight sql %}
> INSERT temperature celcius=25.4
>
{% endhighlight %}

`temperature` será a minha **measurement** e `celcius` com seu valor(`25.4`) será um **field**. Caso eu queira adicionar mais um **field**, basta utilizar uma vírgula(`,`) para separar os elementos:

{% highlight sql %}
> INSERT temperature celcius=25.4,fahrenheit=77.54
>
{% endhighlight %}

Além disso, é fundamental utilizar as **tags** para indexar a busca e filtrar os registros inseridos na base, [como já visto no episódio anterior](/blog/series-temporais-e-influxdb/#tags). Para fazer isso, é só incluir a **tag** após o **measurement** separados por uma vírgula(`,`) e deixar um espaço (` `) no final pra separá-la dos **fields**:

{% highlight sql %}
> INSERT temperature,city=Marília celcius=25.4,fahrenheit=77.54
>
{% endhighlight %}

Caso queira ter mais de uma **tag**, basta repetir a regra dos **fields**: separe eles com uma vírgula

{% highlight sql %}
> INSERT temperature,city=Marília,state=SP celcius=25.4,fahrenheit=77.54
>
{% endhighlight %}

Em todos os cenários anteriores o `timestamp` é definido automaticamente com o valor do instante em que o registro é inserido. Entretanto, é possível determinar o valor do `timestamp`:

{% highlight sql %}
> INSERT temperature,city=Marília,state=SP celcius=25.4,fahrenheit=77.54 1496510681952374020
>
{% endhighlight %}

### Consulta de dados
Para exibir todos os registros de uma *measurement*, o comando em InfluxQL é muito parecido com SQL:

{% highlight sql %}
> SELECT * FROM temperature
name: temperature
time                celcius city      fahrenheit state
----                ------- ----      ---------- -----
1496510104709742573 25.4
1496510114111767822 25.4              77.54
1496510533023926338 25.4    Marília   77.54
1496510681952374018 25.4    Marília   77.54      SP
1496510681952374020 25.4    Marília   77.54      SP

>
{% endhighlight %}

Você pode olhar os fields definidos pra essa *measurement*:

{% highlight sql %}
> SHOW FIELD KEYS FROM temperature
name: temperature
fieldKey   fieldType
--------   ---------
celcius    float
fahrenheit float

>
{% endhighlight %}

E também as tags:

{% highlight sql %}
> SHOW TAG KEYS FROM temperature
name: temperature
tagKey
------
city
state

>
{% endhighlight %}

O comando `SELECT` aceita também os condicionais de `WHERE` para filtrar os resultados:

{% highlight sql %}
> SELECT * FROM temperature WHERE city='Marília'
name: temperature
time                celcius city    fahrenheit state
----                ------- ----    ---------- -----
1496510533023926338 25.4    Marília 77.54
1496510681952374018 25.4    Marília 77.54      SP
1496510681952374020 25.4    Marília 77.54      SP

>
{% endhighlight %}

### Considerações finais
Caso seja de seu interesse se aprofundar nos conceitos do InfluxQL, eu sugiro uma boa lida no [manual de referência da linguagem](https://docs.influxdata.com/influxdb/latest/query_language/spec/).

Fora essa integração via client `influx`, o InfluxDB também já possui uma integração HTTP. Existem bibliotecas em várias linguagens que já integram com esse banco de dados, então o caminho já está trilhado.

Eu não falei sobre autenticação para enfatizar a manipulação dos dados. Entretanto, o usuário padrão do InfluxDB é o `admin`, cuja senha também é `admin`. Você deve criar os seus usuários de acordo com a necessidade de suas aplicações (seja cuidadoso com segurança **sempre**). Utilize o comando `CREATE USER` seguindo as [indicações do manual](https://docs.influxdata.com/influxdb/v1.2/query_language/spec/).

Essa ferramenta é muito poderosa, use ela com cuidado! Use as tags pra indexação e fields pros dados que realmente são a sua série temporal.

Espero que esse post ajude em sua jornada, jovem. Contribuições são bem vindas :)

Bom dia e boa sorte!