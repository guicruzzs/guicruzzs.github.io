---
layout: post
title:  "Primeiras dicas em Go"
date:   2016-10-30
---

<p class="intro"><span class="dropcap">R</span>olou um workshop de Go essa semana, gostaria de compartilhar alguns conhecimentos e apresentar uma oportunidade pra conhecer mais sobre o assunto.</p>

Eu ando brincando com a linguagem Go há um mês. Desde então, uso as minhas horas vagas pra aprender e escrever algumas linhas de código em uma API bem simples de autenticação e banco de dados relacional.

No dia 29/10, tive o privilégio de me encontrar com uma parte da comunidade de Go no escritório da Red Hat em São Paulo. Rolou um workshop com o caríssimo [Cesar Gimenes](https://twitter.com/crgimenes), ele trouxe o [Go Hands On](https://github.com/crgimenes/Go-Hands-On), que é basicamente um repositório que ele mesmo criou com exemplos e explicações sobre alguns pontos importantes para a introdução à linguagem Go. Caso queira ficar por dentro dos eventos da comunidade de Go na *terra da garoa*, basta se unir a essa turma no [Meetup](http://www.meetup.com/pt-BR/golangbr/).

Caso você queira ver como foi o encontro, o carismático [Everaldo Canuto](https://twitter.com/ecanuto) fez um [post em seu blog](https://www.tocadocanuto.com.br/2016/workshop-basico-de-go-lang/) mostrando a galera ;)

Pra mim, foi uma excelente oportunidade de aplicar minha experiência de Ruby em outra linguagem. Tivemos presente programadores com uma bagagem legal em Go, o que nos permitiu explorar vários paralelos com outros cenários e linguagens. O próprio César confessou que poderíamos fazer um bingo com todas as linguagens que ele citava ao longo das explicações. Como a linguagem Go não é orientada a objetos, o conceito de pacotes é a forma que se tem de organizar a casa. Isso tem sido um ponto muito interessante pra minha evolução enquanto programador, pois essa mudança de paradigma me faz pensar fora da caixa (tanto em Go quanto em Ruby).

Você pode aprender sobre Go no próprio [site da linguagem](https://golang.org/), que além de ter um [Getting Started](https://golang.org/doc/install), tem também a [documentação](https://golang.org/doc/) e um [tour](https://tour.golang.org/welcome/1) que é *repetaculê*! Decidi trazer aqui algumas coisas que eu vi no workshop e achei curiosas. Lembrando que você pode encontrar o material completo no repositório [Go Hands On](https://github.com/crgimenes/Go-Hands-On).

### Environment
Existe um comando pra formatar seu código fonte no padrão da linguagem Go. É esse meninão aqui:
{% highlight shell %}
$ go fmt meu_arquivo.go
{% endhighlight %}

Duas coisas que chamaram minha atenção aqui:

* O uso de *tabs* ao invés de espaços
* As chaves abrem na mesma linha em que o método é declarado

E pronto! Fim de discussão! *Habemus padrão*.

### Variáveis:
Funções anônimas existem e podem ser armazenadas em variáveis:

{% highlight go %}
ola := func() {
  fmt.Printf("Olá da função anônima!\r\n")
}
ola()
{% endhighlight %}

Sempre admirei javascript pela forma com que trabalha com esse tipo de função. Gostei do que Go fez com essa possibilidade.

### Funções:
Mande a sua função pra dançar como parâmetro de uma função:

{% highlight go %}
// função que recebe uma função como parâmetro
func printFunc(f func(string) string, valor string) {
  aux := f(valor)
  fmt.Printf(aux)
}
{% endhighlight %}

Finalmente entendi por que os métodos começam com letra maiúscula. Na verdade, a "maiúsculês" dessa primeira letra determina que esse método do pacote é público. Caso deseje criar um método privado, basta criar o método com a primeira letra minúscula.

* público _=>_ maiúsculo
* privado _=>_ minúsculo

Um conceito minimalista da máxima *Menos é mais*.

### For
Quando você vê essa estrutura de iteração em Go você percebe que ela resume todas as outras formas de iteração conhecidas de outras linguagens. Isso significa que o *while*, *loop*, *do while* e outras tantas formas de iteração nativas que você conhece é o *for* de Go. Algo que me chamou a atenção foi o uso do *range*:

{% highlight go %}
potato := "Batata"
for k, v := range potato {
	fmt.Printf("potato[%v] = %q\r\n", k, v)
}
{% endhighlight %}

Ele resolve iteração de string, array e map de uma forma simples: Entrega sempre chave(índice) e valor.

### Concorrência
O ponto que chama muito a atenção das pessoas em Go é o tratamento de concorrência. Além do conceito de *go routines*, algo que me chamou a atenção foi o condicional *select* que é um consumidor de canais, ele executa uma única vez o canal que comunicar algo primeiro, ou seja: ouve múltiplos canais e responde aquele que falar primeiro.

{% highlight go %}
select {
case msg1 := <-c1:
	fmt.Println("canal 1 retornou :", msg1)
case msg2 := <-c2:
	fmt.Println("canal 2 retornou :", msg2)
}
{% endhighlight %}

Outro ponto interessante sobre concorrência é o comportamento dos processos chamados pela *go routine*: caso a aplicação seja finalizada, os canais e as rotinas são exterminados. Portanto, caso esse cenário não seja desejado, devemos criar uma condição de tratamento. Pra isso, é preciso que a aplicação receba sinais e os trate corretamente:

{% highlight go %}
go func() {
	sc := make(chan os.Signal, 1)
	signal.Notify(sc, os.Interrupt)
	// espera pelo sinal
	<-sc

	fmt.Printf("Bye bye Human!\r\n")
	os.Exit(0)
}()
{% endhighlight %}

Descobri um site *repetaculê* pra ilustrar paralelismo em Go, que é o [Visualizing Concurrency in Go](http://divan.github.io/posts/go_concurrency_visualize/). Tente não ficar impressionado com o carinho que houve pra construir esse conteúdo :)

### Pacotes
Tem uma documentação bacana sobre pacotes disponíveis em [gopkg.in](http://labix.org/gopkg.in). Lá estão centralizadas várias bibliotecas que podem ajudar no dia a dia de trabalho, encontrar algumas ferramentas pra escavar e sair usando.

### Pra fechar
Fora esses pontos que eu trouxe, vimos muitas outras coisas no workshop, mas aí eu sugiro que você abra o [repositório](https://github.com/crgimenes/Go-Hands-On) e confira o material na íntegra, além, é claro, de contribuir com ele na construção do conhecimento!

Gostaria de aproveitar e agradecer ao [Cesar](https://twitter.com/crgimenes) pelo material e pela disponibilidade, além da organização do [Jeff Prestes](https://twitter.com/jeffprestes/) e deixar o meu muitíssimo obrigado ao pessoal da Red Hat por apoiar o encontro.

Agora me sinto mais preparado pra continuar a desenvolver o trabalho com a minha humilde API. Espero em breve poder fazer coisas cada vez maiores com essa linguagem.

Bom dia e boa sorte!