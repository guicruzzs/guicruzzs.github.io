---
layout: post
title:  "Conexões SQL em Go"
date:   2016-12-11
description: Utilizando conexões SQL em Go por meio da interface de database/sql? Tenho alguns pontos pra compartilhar contigo.
---

<p class="intro"><span class="dropcap">U</span>tilizando conexões SQL em Go por meio da interface de <a href="https://golang.org/pkg/database/sql/">database/sql</a>? Tenho alguns pontos pra compartilhar contigo.</p>

Você está lá empolgadão com a sua nova API, que vai performar lindamente em Go, aí chega na parte da integração com o seu banco e fica confuso. Não que tenha acontecido comigo, mas conheço um colega meu que passou por isso.

### Driver

Vamos tirar esse rancor do seu coração e vamos olhar para a integração de banco  de dados em Go por meio da biblioteca [database/sql](https://golang.org/pkg/database/sql/). Alguém já se preocupou em padronizar essa comunicação SQL e criou essa interface padrão que está definida nesse projeto *database/sql*. A partir daqui, a única coisa que você precisa é se apropriar de um driver da sua aplicação de banco que foi implementado em cima dessa interface padrão. Tem uma [lista oficial aqui](https://golang.org/pkg/database/sql/) das principais implementações de drivers, escolha aquela que mais rolar um clima. 

Eu escolhi um driver de [MySQL](https://github.com/go-sql-driver/mysql/) pra exemplificar o uso, mas como a interface é a mesma pra todos, o conhecimento aqui compartilhado pode servir para outras integrações.

### Conexão

O primeiro passo, é abrir a conexão com o banco de dados:
{% highlight go %}
package db
import(
    "fmt"
    "database/sql"
/* importei com '_' pois é preciso inicializar a biblioteca, e evitar
   efeitos colaterais (como o fato de não chamá-la diretamente no seu
   código) */
    _ "github.com/go-sql-driver/mysql"
)

// a ideia dessa função aqui é retornar uma forma de conexão com o banco
func Connect() *sql.DB {
//  aqui você substitui as variáveis pelas suas configs
    driverConfig := fmt.Sprintf("%s:%s@tcp(%s:%s)/%s", "user", "pass", "host", "port", "database")
    connection, err := sql.Open("mysql", driverConfig)
    if err != nil {
        fmt.Println("database.Connect ERROR: %s", err)
    }
    return connection
}
{% endhighlight %}

Algo muito importante de se entender é que o método *sql.Open* não entrega uma única conexão com o banco, mas sim um pool "auto gerenciado" de conexões. Entretanto, você vai tratar esse **sql.DB* como se fosse uma conexão mesmo. Dá pra limitar a quantidade de conexões do pool pela [quantidade de conexões que estão ociosas](https://golang.org/pkg/database/sql/#DB.SetMaxIdleConns), ou [pela quantidade conexões abertas](https://golang.org/pkg/database/sql/#DB.SetMaxOpenConns) ou ainda [pelo tempo conexão](https://golang.org/pkg/database/sql/#DB.SetConnMaxLifetime).

Sendo assim, o mais sensato é iniciar sua aplicação criando essa "conexão" com o banco e fechá-la ao encerrar sua aplicação. Dessa maneira, o pool de conexões estará disponível ao longo de toda a execução de sua aplicação, as conexões surgirão de acordo com a necessidade e se eliminarão com as suas configurações de quantidade de conexões no pool(caso hajam).

Pra encerrar o pool de conexões com o banco, basta chamar a função *Close*:
{% highlight go %}
connection := Connect()
connection.Close()

// Você pode também criar uma função pra isso, caso desejar
func Disconnect(connection *sql.DB) {
    connection.Close()
}
{% endhighlight %}

### Pra fechar

Obviamente que existem alguns frameworks que já gerenciam isso de alguma forma pra você, mas caso seja da sua vontade utilizar e integrar com essas soluções diretamente, esse aqui é o caminho das pedras.

Conhece alguma forma melhor de fazer isso, ou tem alguma sugestão? Deixe nos comentários uma contribuição ;)

Bom dia e boa sorte!