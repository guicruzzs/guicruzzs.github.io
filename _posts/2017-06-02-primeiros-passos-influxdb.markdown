---
layout: post
title:  "Primeiros passos no InfluxDB"
date:   2017-06-02
---

<p class="intro"><span class="dropcap">O</span>lar.</p>

Rodando o InfluxDB num *docker container* pra facilitar a diversão:
$ docker run --name influxzinho influxdb # use a opção -d pra rodar na opção detached

Abrir o client do banco dentro do container, vamos logo à diversão:
```
$ docker exec -it influxzinho bash
# Dentro do container:
$ influx
```

Por padrão, o InfluxDB roda na porta `8086`.

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

E agora? Vamos criar uma tabela? Determinar as colunas? E também os *Fields* e as *Tags*?
Nada disso é necessário. A sintaxe da requisição de inclusão de dados é responsável por definir tada a estruturação dos dados

Bom dia e boa sorte!