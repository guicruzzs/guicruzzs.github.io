---
layout: post
title:  "Primeiros passos no InfluxDB"
date:   2017-06-02
---

<p class="intro"><span class="dropcap">O</span>lar.</p>

### Setup rápido
Rodando o InfluxDB num *docker container* pra facilitar a diversão:
$ docker run --name influxzinho influxdb # use a opção -d pra rodar na opção detached

Abrir o client do banco dentro do container, vamos logo à diversão:
```
$ docker exec -it influxzinho bash
# Dentro do container:
$ influx
```

Por padrão, o InfluxDB roda na porta `8086`.


### Databases
A partir daqui, utilizo o InfluxQL (Influx Query Language) para manipular o banco.
Para listar as *databases* existentes:
```
> SHOW DATABASES
name: databases
name
----
_internal
test

>
```

O nome da *database* padrão do InfluxDB é a `_internal`.

Criar uma *database* nova basta usar o comando
```
> CREATE DATABASE test
>
```

Para alternar entre as *databases*:
```
> USE test
Using database test
>

```

### Inserção de dados
E agora? Vamos criar uma tabela? Determinar as colunas? E também os *Fields*, as *Tags* e os seus tipos?
Nada disso é necessário. A sintaxe da requisição de inclusão de dados é responsável por definir tada a estruturação dos dados.

O comando de criação de um ponto (registro) no banco é o `INSERT` que é descrito da seguinte maneira:
```
<measurement>[,<tag_key>=<tag_value>[,<tag_key>=<tag_value>]] <field_key>=<field_value>[,<field_key>=<field_value>] [<timestamp>]
```
O `measurement` é obrigatório, por isso é que está envolvido pelos sinais `< >`. O que está entre colchetes simboliza que o seu conteúdo é opcional.
Na prática, os comandos tem baixa complexidade. Supondo que o eu esteja armazenando temperaturas no meu banco:
```
> INSERT temperature celcius=25.4
>
```

`temperature` será a minha **measurement** e `celcius` será um **field**. Caso eu queira adicionar mais um **field**, basta utilizar uma vírgula(`,`) para separar os elementos:
```
> INSERT temperature celcius=25.4,fahrenheit=77.54
>
```

Além disso, é fundamental utilizar as **tags** para indexar a busca e filtrar os registros inseridos na base. Para fazer isso, é só incluir a **tag** após o **measurement** separados por uma vírgula(`,`) e deixar um espaço (` `) no final pra separá-la dos **fields**:
```
> INSERT temperature,city=Marília celcius=25.4,fahrenheit=77.54
>
```

Caso queira ter mais de uma **tag**, basta repetir a regra dos **fields**: separe eles com uma vírgula
```
> INSERT temperature,city=Marília,state=SP celcius=25.4,fahrenheit=77.54
>
```

Em todos os cenários anteriores o `timestamp` é definido automaticamente com o valor do instante em que o registro é inserido. Entretanto, é possível determinar o valor do `timestamp`:
```
> INSERT temperature,city=Marília,state=SP celcius=25.4,fahrenheit=77.54 1496510681952374020
>
```

### Consulta de dados
Para exibir todos os registros de uma *measurement*, o comando em InfluxQL é muito parecido com SQL:
```
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
```

Você pode olhar os fields definidos pra essa *measurement*:
```
> SHOW FIELD KEYS FROM temperature
name: temperature
fieldKey   fieldType
--------   ---------
celcius    float
fahrenheit float

>
```

E também as tags:
```
> SHOW TAG KEYS FROM temperature
name: temperature
tagKey
------
city
state

>
```

O comando `SELECT` aceita também os condicionais de `WHERE` para filtrar os resultados:
```
> SELECT * FROM temperature WHERE city='Marília'
name: temperature
time                celcius city    fahrenheit state
----                ------- ----    ---------- -----
1496510533023926338 25.4    Marília 77.54
1496510681952374018 25.4    Marília 77.54      SP
1496510681952374020 25.4    Marília 77.54      SP

>
```

### Considerações finais

- HTTP

Bom dia e boa sorte!