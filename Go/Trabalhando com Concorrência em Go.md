---
id: 9be8ddca-ad3d-4cff-b7c8-3313348a3755
title: "Trabalhando com Concorrência em Go"
summary: "Abordagem dos principais pontos de concorrência no Go: Goroutines, Canais, Mutex e Wait Groups."
---

## Goroutines

Na programação simultânea, quando uma tarefa é iniciada, todas as outras tarefas também iniciam, mas em vez de concluí-las uma a uma, a máquina executa um pouco de cada tarefa ao mesmo tempo. Embora Go permita a programação simultânea, as tarefas também podem ser executadas em paralelo quando a máquina tem vários núcleos. Do ponto de vista do programador, entretanto, essa distinção não é tão importante, pois as tarefas são criadas com a ideia de que serão realizadas em paralelo, e de qualquer forma que a máquina as realizará. Vamos descobrir mais neste tópico.

*Programação concorrente* é uma técnica que permite realizar várias tarefas ao mesmo tempo. *Programação concorrente* geralmente requer o uso de construções com as threads e locks/bloqueios para realizar a sincronização e evitar deadlocks.

Uma goroutine é um contexto de execução gerenciado pelo runtime do Go (em oposição a uma thread que é gerenciado pelo sistema operacional). São funções executadas simultaneamente com outras funções. Uma goroutine geralmente tem uma sobrecarga de inicialização muito menor do que um thread do sistema operacional. Uma goroutine começa com uma pequena pilha que cresce conforme necessário. Criar novas goroutines é mais rápido e barato do que criar threads do sistema operacional. O agendador do Go atribui threads do sistema operacional para executar goroutines.

Quando você usa *goroutines*, seu programa será mais responsivo. Você pode usar goroutines ao realizar tarefas que lidam com diferentes fontes de entrada. Por exemplo, seu programa pode precisar interagir com os usuários e, ao mesmo tempo, comunicar-se com servidores back-end por meio da rede. Como os acessos à rede normalmente incorrem em latência de rede significativa, é comum executar a função que acessa a rede simultaneamente como uma goroutine.

*Goroutines* são criadas usando a palavra-chave `go` seguida por uma chamada de função:

```go
go f()

go g(i, j)

go func() {
...
}()

go func(i,j int) {
...
}(1,2)
```

A palavra-chave `go` inicia a função fornecida em uma nova goroutine. A goroutine existente / principal (`main`) continua funcionando simultaneamente com a goroutine recém-criada. *A função executada na goroutine pode receber parâmetros, mas não pode retornar um valor.* Os parâmetros da função da goroutine são avaliados antes do início da goroutine e passados ​​​​para a função assim que a goroutine começa a ser executada.

O runtime do Go inicia várias goroutines quando o programa é iniciado. Porém, existe pelo menos um para o garbage collector e outra para a goroutine principal. A goroutine principal simplesmente chama a função principal - `main` e encerra o programa quando ele retorna.

*Obs.: Quando a função `main` retorna e o programa termina, todas as goroutines em execução terminam abruptamente, no meio da função, sem chance de realizar qualquer limpeza.*

A melhor maneira de entender goroutines é usar um exemplo. Considere o seguinte programa:

```go
package main

import (
	"fmt"
	"time"
)

func say(s string, times int) {
	for i := 0; i < times; i++ {
		// inject a 100 ms delay
		time.Sleep(100 * time.Millisecond)
		fmt.Println(i, s)
	}
}

func main() {
	say("Hello", 3)
	say("World", 2)
}
```

Neste programa, você tem uma função chamada `say()`, que recebe dois argumentos: uma string a ser impressa no console e o número de vezes que a string deve ser impressa. Na função `main()`, você chama a função `say()` duas vezes, primeiro com a string `"Hello"` e novamente com a string `"World"`.

Ao executar o programa, você vê a seguinte saída:

```
0 Hello
1 Hello
2 Hello
0 World
1 World
```

Isso é o que você esperava porque a primeira chamada para a função `say()` deve terminar antes que a segunda chamada possa prosseguir. Mas e se você quiser que as duas chamadas sejam executadas simultaneamente? Veja abaixo a mudança do mesmo código acima mais agora de forma concorrente.

```go
func main() {
	go say("Hello", 3)
	go say("World", 2)

	fmt.Scanln()
}
```

A primeira instrução chama a função `say()` como uma goroutine. Essencialmente, significa "Execute a função `say()` independentemente e retorne imediatamente o controle para a instrução de chamada - `main`". A segunda declaração faz o mesmo. Agora você tem duas instâncias separadas da função `say()` executando simultaneamente. O resultado pode aparecer assim (você pode obter um resultado diferente):

```
0 World
0 Hello
1 World
1 Hello
2 Hello
```

Cada vez que você executa este programa, pode obter uma seqüência ligeiramente diferente das palavras impressas. Isso ocorre porque o runtime do Go gerencia como essa função é executada e você não tem controle sobre qual é impressa primeiro. Observe que a função `main()` tem a seguinte instrução:

```
fmt.Scanln()
```

Sem essa declaração, você provavelmente não conseguiria ver nenhuma saída. Isso ocorre porque cada vez que uma goroutine é chamada, o controle retorna imediatamente para a instrução de chamada - `main`. Sem a função `Scanln()` para esperar pela entrada do usuário, o programa termina automaticamente após a segunda goroutine ser chamada. Depois que o programa é encerrado, todas as goroutines também são encerradas e nenhuma saída será impressa.

*Obs.: Se a função `main()` for encerrada / finalizada, todas as goroutines atualmente em execução também serão encerradas.*

### Função `main` x Goroutines

Vejamos o que acontece quando criamos a goroutine abaixo:

```go
func f() {
	fmt.Println("Hello from goroutine")
}

func main() {
	go f()

	fmt.Println("Hello from main")

	time.Sleep(100)
}
```

Este programa começa com a goroutine principal - `main`. Quando a instrução `go f()` é executada, uma nova goroutine é criada. Lembre-se, uma goroutine é um contexto de execução, o que significa que a palavra-chave `go` faz com que o runtime aloque uma nova pilha e a configure para executar a função `f()`. Então esta goroutine é marcada como pronta para execução. A goroutine principal continua em execução sem esperar que `f()` seja chamada e imprime `Hello from main` para o console. Então ele espera 100 milissegundos. Durante esse período, a nova goroutine pode começar a ser executada, chamar `f()` e imprimir `Hello from goroutine`. `fmt.Println` possui exclusão mútua integrada para garantir que as duas goroutines não corrompam as saídas uma da outra.

‌Este programa pode gerar uma das seguintes opções:

- `Hello from main` e `Hello from goroutine`: Este é o caso quando a goroutine principal imprime primeiro a saída e depois a goroutine a imprime.

- `Hello from goroutine` e `Hello from main`: Este é o caso quando a goroutine criada em `main()` é executada primeiro e depois a goroutine principal imprime a saída.

- `Hello from main`: Este é o caso quando a goroutine principal continua em execução, mas a nova goroutine nunca encontra uma chance de ser executada nos 100 milissegundos fornecidos de espera, fazendo com que `main` retorne. Assim que a `main` retornar, o programa termina sem que a goroutine encontre uma chance de ser executada. É improvável que este caso seja observável, mas é possível.

### Condição de corrida de dados com closures

Veja o seguinte código abaixo:

```go
func main() {
	for _, s := range []string{"a", "b", "c"} {
		go func() {
			fmt.Printf("Goroutine %s\n", s)
		}()
	}

	time.Sleep(1000)
}
```

Aqui está o resultado:

```
Goroutine c
Goroutine c
Goroutine c
```

Então, o que está acontecendo?

Primeiro, esta é uma condição de corrida de dados, porque existe uma variável compartilhada que é escrita por uma goroutine e lida por outras três sem qualquer sincronização. Isso se torna mais evidente se usarmos como da seguinte forma:

```go
func main() {
	var s string

	s = "a"

	go func() {
		fmt.Printf("Goroutine %s\n", s)
	}()

	s = "b"
	go func() {
		fmt.Printf("Goroutine %s\n", s)
	}()

	s = "c"
	go func() {
		fmt.Printf("Goroutine %s\n", s)
	}()

	time.Sleep(1000)
}
```

Neste exemplo, cada função anônima é um closure. Estamos executando três goroutines, cada uma com um closure que captura a variável `s` do escopo de `main`. Por causa disso, temos três goroutines que leem a variável compartilhada e uma goroutine (a goroutine principal) escrevendo nela simultaneamente. Esta é uma condição de corrida de dados. Na execução anterior, todas as três goroutines foram executadas após a última atribuição a `s`. Existem outras execuções possíveis. Na verdade, este programa pode até funcionar corretamente e imprimir a saída esperada.

Esse é o perigo da condição de corrida de dados. Um programa como esse raramente é executado corretamente, por isso é fácil diagnosticar e corrigir antes que o código seja implantado em um ambiente de produção. Eles são a causa de muitos mal-entendidos no desenvolvimento de Go porque simplesmente refatorar uma função declarada como uma função anônima pode ter consequências inesperadas.

Uma closure é uma função com um contexto que inclui algumas variáveis ​​incluídas em seu escopo envolvente. No exemplo anterior, existem três closures, e cada closure captura a variável `s` do seu escopo. O escopo define todos os nomes de símbolos acessíveis em um determinado ponto de um programa. Em Go, o escopo é determinado sintaticamente, portanto, onde declaramos a função anônima, o escopo inclui todas as funções exportadas, variáveis, nomes de tipos, a função principal e a variável `s`. O compilador Go analisa o código-fonte para determinar se uma variável definida em uma função pode ser referenciada após o retorno da função. Este é o caso quando, por exemplo, você passa um ponteiro para uma variável definida em uma função para outra função. Ou quando você atribui uma variável de ponteiro global a uma variável definida em uma função. Assim que a função que declara essa variável retornar, a variável global apontará para um local de memória obsoleto. Os locais das pilhas vêm e vão conforme as funções entram e retornam. Quando tal situação é detectada (ou mesmo um potencial para tal situação é detectado, como a criação de uma goroutine ou a chamada de outra função), a variável escapa para o *heap*. Ou seja, em vez de alocar aquela variável na pilha, o compilador aloca a variável dinamicamente no *heap*, de forma que mesmo que a variável saia do escopo, seu conteúdo permanece acessível. Isso é exatamente o que está acontecendo em nosso exemplo. A variável `s` escapa para o *heap* porque existem goroutines que podem continuar rodando e acessando essa variável mesmo após o retorno principal. Esta situação é ilustrada na imagem abaixo:

[Variable concurrency in closures](Variable concurrency in closures.png)

Closures como goroutines podem ser uma ferramenta muito poderosa, mas devem ser usados ​​com cuidado. A maioria das closures executadas como goroutines compartilham memória, portanto, são propensos a condição de corridas. Podemos consertar nosso programa criando uma cópia da variável `s` em cada iteração. A primeira iteração define `s` como `a`. Criamos uma cópia dele e capturamos essa cópia no encerramento. Então a próxima iteração define `s` como `b`. Isso é bom porque o encerramento criado durante a primeira iteração ainda usa `a`. Criamos uma nova cópia de `s`, desta vez com valor `b`, e assim continua. Isso é mostrado no código a seguir:

```go
for _, s := range []string{"a", "b", "c"} {
	s := s // Redeclare s, create a copy
	// Here, the redeclared s shadows the loop variable s
	go func() {…}
}
```

Outra forma é passá-lo como parâmetro:

```go
for _, s := range []string{"a", "b", "c"} {
	go func(s string) {
		fmt.Printf("Goroutine %s\n", s)
	}(s) // This will pass a copy of s to the function
}
```

Em qualquer solução, a variável de dentro do loop `s` não escapa mais para o *heap*, porque uma cópia dela é capturada. Na primeira solução usando uma variável redeclarada, a cópia escapa para o *heap*, mas a variável de dentro do loop `s` não.

Uma das perguntas mais frequentes sobre goroutines é: como interrompemos uma goroutine em execução? Não existe uma função mágica que encerre ou pause uma goroutine. Se você quiser parar uma goroutine, você deve enviar alguma mensagem ou definir um sinalizador compartilhado com a goroutine, e a goroutine deve responder à mensagem ou ler a variável compartilhada e retornar. Se quiser pausá-lo, você terá que usar um dos mecanismos de sincronização para bloqueá-lo. Este fato causa certa ansiedade entre os desenvolvedores que não conseguem encontrar uma forma eficaz de encerrar suas goroutines. No entanto, esta é uma das realidades da programação simultânea. A capacidade de criar blocos de execução simultâneos é apenas uma parte do problema. Uma vez criados, você deve estar ciente de como encerrá-los de forma responsável.

Um `panic` pode encerrar uma goroutine. Se ocorrer um pânico em uma goroutine, ele será propagado pela pilha de chamadas até que um `recover` seja encontrada ou até que a goroutine retorne. Se o pânico não for tratado, uma mensagem de pânico será impressa e o programa irá travar.

Antes de encerrar este tópico, pode ser útil falar sobre como o runtime do Go gerencia goroutines. Go usa um agendador *M:N* que executa *M* goroutines em *N* threads do sistema operacional. Internamente, o runtime do Go monitora as threads do sistema operacional e as goroutines. Quando uma thread do sistema operacional está pronta para executar uma goroutine, o agendador seleciona aquele que está pronto para ser executado e o atribui a thread. A thread do sistema operacional executa essa goroutine até que ela bloqueie, ceda ou seja interrompida. Existem várias maneiras de bloquear uma goroutine. Bloqueio por operações de canal ou mutexes é gerenciado pelo runtime do Go. Se a goroutine for bloqueada devido a uma operação de E/S síncrona, a thread que executa essa goroutine também será bloqueado (isso é gerenciado pelo sistema operacional). Nesse caso, o runtime do Go inicia uma nova thread ou usa uma já disponível e continua a operação. Quando o encadeamento do sistema operacional é desbloqueado (ou seja, a operação de E/S termina), o encadeamento é colocado novamente em uso ou retornado ao conjunto de encadeamentos. O runtime do Go limita o número de threads ativas do sistema operacional executando goroutines do usuário com a variável `GOMAXPROCS`. No entanto não há limite para o número de threads do sistema operacional aguardando operações de E/S. Portanto, a contagem real de threads do sistema operacional que um programa Go usa pode ser muito maior que `GOMAXPROCS`. No entanto, apenas `GOMAXPROCS` desses threads estariam executando goroutines do usuário.

A imagem abaixo ilustra isso. Suponha que `GOMAXPROCS=2`. **Thread 1** e **Thread 2** são threads do sistema operacional que executam goroutines. Goroutine **G1**, que está sendo executado na **Thread 1**, executa uma operação de E/S síncrona, bloqueando a **Thread 1**. Como a **Thread 1** não está mais operacional, o runtime do Go aloca a **Thread 3** e continua executando goroutines. Observe que, embora existam três threads no sistema operacional, há duas threads ativas e uma thread bloqueada. Quando a chamada do sistema em execução na **Thread 1** for concluída, a goroutine **G1** se tornará executável novamente, mas agora há uma thread extra. O runtime do Go continua em execução com o **Thread 3** e para de usar o **Thread 1**.

[System calls block OS threads](System calls block OS threads.png)

## Comunicação Entre Goroutines Usando Canais - Channels

Os canais permitem que goroutines compartilhem memória por meio de comunicação, em vez de se comunicarem por meio de compartilhamento de memória. Ao trabalhar com canais, você deve ter em mente que canais são duas coisas combinadas: são ferramentas de sincronização e são canais para dados. Pense nos canais como um armazenamento temporário para passar valores entre as goroutines.

Um canal é na verdade um ponteiro (**os canais são tipos de referência**) para uma estrutura de dados que contém seu estado interno, portanto, o *zero value* de uma variável de canal é `nil`. Por causa disso, os canais devem ser inicializados usando a palavra-chave `make`. Se você esquecer de inicializar um canal, ele nunca estará pronto para aceitar um valor ou fornecer um valor, portanto, a leitura ou gravação em um canal `nil` será bloqueada indefinidamente. Quando você passa um canal para uma função, você estão realmente passando um ponteiro do canal.

O garbage collector do Go coletará canais que não estão mais em uso. Se não houver goroutines que façam referência direta ou indireta a um canal, o canal será coletado no garbage collector, mesmo que seu buffer contenha elementos. Você não precisa fechar canais para torná-los elegíveis para o garbage collector. Na verdade, fechar um canal tem mais significado do que apenas limpar recursos.

Receber dados de um canal fechado é uma operação válida. Na verdade, um recebimento de um canal fechado sempre será bem-sucedido com o zero value do tipo de canal. Escrever em um canal fechado é um bug: escrever / enviar dados em um canal fechado sempre entrará em pânico.

### Leitura, escrita e buffering

Um channel é o que o nome sugere essencialmente - é algo onde as mensagens podem ser canalizadas e qualquer goroutine pode enviar ou receber mensagens através de um channel. Use o operador `<-` para interagir com um canal. Você lê de um canal colocando o operador `<-` à esquerda da variável de canal, e você escreve para um canal por lugar colocando-o à direita:

```go
a := <-ch // reads a value from ch and assigns it to a
ch <- b // write the value in b to ch
```

*Cada valor gravado em um canal só pode ser lido uma vez. Se vários goroutines estão lendo do mesmo canal, um valor escrito no canal só será lido por um deles.*

É possível instanciar o channel com o seguinte:

```go
ch := make(chan int)
// or
var ch chan int
ch = make(chan int)
```

*Um canal pode ser de qualquer tipo, como `integer`, `boolean`, `float` e qualquer `struct` que possa ser definida, e até mesmo slices e ponteiros, embora os dois últimos sejam geralmente usados ​​com menos frequência.*

Canais podem ser passados ​​como parâmetros para funções, e é assim que diferentes Goroutines podem compartilhar dados. Vamos ver como enviar uma mensagem a um canal:

```go
ch <- 2
```

Nesse caso, enviamos o valor `2` para o canal `ch`, que é um canal de inteiros. Claro, tentar enviar algo diferente de um inteiro para um canal de inteiros causará um erro.

Depois de enviar uma mensagem, precisamos ser capazes de receber uma mensagem de um canal. Para fazer isso, podemos apenas fazer o seguinte:

```go
<- ch
```

Isso garante que a mensagem seja recebida; no entanto, a mensagem não é armazenada. Pode parecer inútil perder a mensagem, mas veremos que pode realmente fazer sentido. No entanto, podemos querer manter o valor recebido do canal e podemos fazer isso armazenando o valor em uma nova variável:

```go
i := <- ch
```

Ao atribuir um canal para uma variável ou campo, ou passando para uma função, use uma seta antes da palavra-chave `chan` (`ch <-chan int`) para indicar que o goroutine só lê do canal. Use uma seta após a palavra-chave `chan` (`ch chan<- int`) para indicar que o goroutine só escreve para o canal. Isso permite que o compilador Go garanta que um canal só é lido ou escrito por uma função.

Vamos ver um programa simples que nos mostra como usar o que aprendemos até agora:

```go
package main

import "log"

func main() {
	ch := make(chan int, 1)
	ch <- 1
	i := <-ch
	log.Println(i)
}
```

Este programa essencialmente cria um novo canal, seta o inteiro `1`, então o lê e, finalmente, imprime o valor de `i`, que deve ser `1`. Este código não é tão útil na prática, mas com uma pequena mudança podemos ver algo interessante. Vamos tornar o *canal sem buffer* alterando a definição do canal para o seguinte:

```go
ch := make(chan int)
```

Se você executar o código, obterá a seguinte saída:

```
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:
main.main()
	/tmp/sandbox3412235348/prog.go:7 +0x37
```

Nesse caso específico, o problema é simples: se não sabemos o tamanho do canal, as goroutines aguardam indefinidamente, e isso é chamado de deadlock. Isso não significa que não podemos lidar com canais sem buffer. Veremos depois como lidar com eles, pois exige a execução de mais de uma goroutines. Com apenas uma goroutine, após enviarmos a mensagem, bloqueamos a execução e não há outra goroutine capaz de receber a mensagem; portanto, temos um impasse.

Assim, uma operação de envio será bloqueada até que outra goroutine receba dela. Uma operação de recebimento será bloqueada até que outra goroutine envie para ela. Em outras palavras, **um canal sem buffer é uma forma de transferir dados entre goroutines atomicamente.**

Por padrão, os canais não têm buffer. **Cada gravação em um canal aberto e sem buffer faz com que a escrita na goroutine pare até que outra goroutine leia a partir do mesmo canal.** Da mesma forma, **uma leitura de um canal aberto e sem buffer faz com que a leitura goroutine pare até que outro goroutine grave no mesmo canal**. Isso significa você não pode escrever ou ler de um *canal sem buffer* sem pelo menos um dos dois concorrentes atualmente executando goroutines pare a sua execução.

Em um *unbuffered channel*, o emissor/sender bloqueia até que o valor seja recebido por um receptor/receiver.

*Dica: Quando você tenta recuperar (receiver) um valor de um canal unbuffered e não há nenhum valor disponível, seu código será bloqueado. Quando você envia (sender) um valor para um canal sem buffer, seu código será bloqueado até que o valor seja recuperado do canal.*

Vejamos mais um exemplo de como canais sem buffer são usados ​​para enviar mensagens e sincronizar goroutines usando o seguinte trecho:

```go
package main

import (
	"fmt"
)

func main() {
	chn := make(chan bool)

	go func() {
		chn <- true
	}()

	go func() {
		var y bool
		y = <-chn
		fmt.Println(y)
	}()

	fmt.Scanln()
}
```

Existem duas execuções possíveis neste ponto: `G1` tenta enviar (linha 11) antes de `G2` estar pronto para receber (linha 18), ou `G2` tenta receber (linha 18) antes de `G1` estar pronto para enviar (linha 11). O primeiro diagrama da imagem ilustra o caso em que `G1` é executado primeiro. Na linha 11, `G1` tenta enviar para o canal. Porém, neste momento, o `G2` ainda não está pronto para receber. Como o canal não tem buffer e não há receptores disponíveis, `G1` fica bloqueado então.

[Two possible runs using an unbuffered channel](Two possible runs using an unbuffered channel)

Depois de um tempo, `G2` executa a linha 18. Esta é uma operação de recebimento de canal e há uma goroutine (`G1`) esperando para enviar para ele. Por conta disso, o primeiro `G1` é desbloqueado e envia o valor para o canal, e `G2` o recebe sem bloqueio. Cabe agora ao escalonador decidir quando o `G1` pode ser executado.

O segundo cenário possível, onde `G2` é executado primeiro, é mostrado na imagem acima do lado direito. Como `G1` ainda não foi enviado ao canal, `G2` bloqueia. Quando `G1` está pronto para enviar, `G2` já está esperando para receber, então `G1` não bloqueia e envia o valor, e `G2` desbloqueia e recebe o valor. O escalonador decide quando o `G2` pode ser executado novamente.

Observe que um canal sem buffer atua como um ponto de sincronização entre duas goroutines. Ambas as goroutines devem estar alinhadas para que a transferência da mensagem aconteça.

*Go também tem canais buffered. Esses canais armazenam em buffer um número limitado de gravações sem bloquear.* Se o buffer for preenchido antes que haja qualquer leitura do canal, uma escrita subsequente no canal pausa a rotina de gravação até que o canal seja lido. Assim como escrever em um canal com blocos de buffer, ler em um canal com um buffer vazio também bloqueia.

Um canal de buffer é criado especificando a capacidade do buffer ao criar o canal:

```go
ch := make(chan int, 2)
```

A declaração anterior cria e inicializa um canal que pode transportar valores inteiros com capacidade de 2. Um canal buffered é estruturado em FIFO - primeiro ao entrar é o primeiro ao sair. Ou seja, se você enviar alguns valores para um canal, o receptor receberá esses valores na ordem em que foram escritos. Use a seguinte sintaxe para enviar ou receber canais:

```go
ch <- 1    // Send 1 to the channel
<- ch      // Receive a value from the channel
x = <- ch  // Receive value from the channel and assign it to x
x := <- ch // Receive value from the channel, declare variable x
		   // using the same type as the value read (which is int), and assign the value to x.
```

As funções integradas `len` e `cap` retornam informações sobre um canal buffered. Usar `len` para descobrir quantos valores estão atualmente no buffer e use `cap` para descobrir o tamanho máximo do buffer. A capacidade do buffer não pode ser alterada.

*Passar um canal unbuffered para `len` e `cap` retorna 0. Isso faz sentido porque, por definição, um canal sem buffer não tem um buffer para armazenar valores.*

A disponibilidade dessas funções não significa que o código a seguir esteja correto:

```go
// Don't do this!
if len(ch) > 0 {
	x := <-ch
}
```

Este código verifica se o canal contém alguns dados e, vendo que sim, os lê. Este código tem uma condição de corrida. Mesmo que o canal possa conter dados quando seu comprimento for verificado, outra goroutine poderá recebê-los no momento em que esta goroutine tentar fazê-lo. Em outras palavras, se `len(ch)` retornar um valor diferente de zero, significa que o canal tinha alguns valores quando seu comprimento foi verificado, mas não significa que ele tinha alguns valores após o retorno da função `len`.

A imagem abaixo ilustra uma possível sequência de operações com este canal utilizando duas goroutines. A primeira goroutine envia os valores `1` e `2` para o canal, que são armazenados no buffer do canal (`len(ch)=2` , `cap(ch)=2`). Então a outra goroutine recebe `1`. Neste ponto, um valor `2` é o próximo a ser lido no canal, e o buffer do canal contém apenas um valor. A primeira goroutine envia `3`. O canal está cheio, então a operação para enviar `4` para o canal é bloqueada. Quando a segunda goroutine recebe o valor `2` do canal, o envio da primeira goroutine é bem-sucedido e a primeira goroutine é re-ativada.

[Possible sequence of operations with a buffered channel of capacity 2](Possible sequence of operations with a buffered channel of capacity 2.png)

Este exemplo mostra que uma operação de envio para um canal será bloqueada até que o canal esteja pronto para aceitar um valor. Se o canal não estiver pronto para aceitar o valor, a operação de envio será bloqueada.

Da mesma forma, a figura abaixo mostra uma operação de bloqueio de recebimento. A primeira goroutine envia `1` e a segunda goroutine o recebe. Agora `len(ch)=0`, então a próxima operação de recebimento da segunda goroutine é bloqueado. Quando a primeira goroutine envia um valor `2` para o canal, a segunda goroutine recebe esse valor e processa.

[Blocking receive operation](Blocking receive operation.png)

**Portanto, o recebimento de um canal será bloqueado até que o canal esteja pronto para fornecer um valor.**

*Na maioria das vezes, você deve usar canais sem buffer (unbuffered channels).*

### Fechando o canal

Vamos ver mais uma característica dos canais, eles podem ser fechados. Os canais precisam ser fechados quando a tarefa para a qual foram criados for concluída. Para fechar um canal, digite o seguinte:

```go
close(ch)
```

Como alternativa, você pode adiar o fechamento, conforme mostrado no seguinte snippet de código:

```go
defer close(ch)
for i := 0; i < 100; i++ {
	ch <- i
}

return
```

Neste caso, após a instrução de `return`, o canal é fechado, pois o fechamento é adiado para ser executado após a instrução de `return`.

**Uma vez que um canal é fechado, qualquer tentativa de escrever para o canal ou fechá-lo novamente entrará em pânico.** Curiosamente, tentar ler a partir de um canal fechado sempre é bem-sucedido. Se o canal estiver em buffer e houver valores que ainda não foram lidos, eles será devolvido em ordem. Se o canal estiver sem buffer ou se o canal em buffer não tiver mais valores, o valor zero para o tipo do canal é retornado. Isso leva a uma pergunta que pode soar familiar em nossa experiência com mapas: quando lemos de um canal, como podemos dizer a diferença entre um valor zero que foi escrito e um valor zero que foi retornado porque o canal está fechado? Desde a Go tenta ser uma linguagem consistente, temos uma resposta familiar: usamos o *comma ok idiom* para detectar se um canal foi fechado ou não:

```go
v, ok := <-ch
```

Se `ok=true` estiver definido como verdadeiro, o canal está aberto - valor recebido. Se for definido como falso - `ok=false`, o canal está fechado e o valor é simplesmente o zero value. Não existe uma sintaxe semelhante para envio porque o envio para um canal fechado causará pânico.

*Sempre que estiver lendo de um canal que pode estar fechado, use comma ok idiom para garantir que o canal ainda está aberto.*

*A responsabilidade do fechamento é da goroutine que escreve para o canal.* Esteja ciente de que fechar um canal só é necessário se houver uma espera da goroutine para o canal fechar (como um que usa um loop `for-range` para ler a partir do canal). Uma vez que um canal é apenas outra variável, o runtime do Go pode detectar canais que não são mais usados ​​e usar o garbage collector para coletar.

### for-range

Você também pode ler de um canal usando um loop `for-range`:

```go
for v := range ch {
	fmt.Println(v)
}
```

Ao contrário de outros loops `for-range`, há apenas uma única variável declarada para o canal, que é o valor. O loop continua até que o canal seja fechado, ou até um `break` ou declaração de `return` é alcançada.

Considere o seguinte exemplo:

```go
package main

import (
	"fmt"
	"time"
)

func fib(n int, c chan int) {
	a, b := 1, 1

	for i := 0; i < n; i++ {
		c <- a
		a, b = b, a+b
		time.Sleep(1 * time.Second)
	}

	close(c) // close the channel
}

func main() {
	c := make(chan int)

	go fib(10, c)

	for i := range c { // read from channel until channel is closed
		fmt.Println(i)
	}
	fmt.Println("Exit")
}
```

Neste exemplo, você tem uma função chamada `fib()` que recebe dois argumentos - o número de elementos a serem gerados para a sequência de Fibonacci e o canal para armazenar os números. Cada número da sequência de Fibonacci é calculado e em seguida, enviado para o canal. Eu adicionei um atraso de um segundo para cada número para simular algum atraso. À medida que cada número de Fibonacci é gerado e inserido no canal, a função `fib()` bloqueia até que o valor seja recuperado do canal. Depois de gerar todos os números de Fibonacci necessários, você fecha o canal (usando a função `close()`) para indicar que o canal não está mais aceitando valores, e assim, o valor de `Exit` é impresso na goroutine principal.

*Dica: Para um canal sem buffer, depois que um valor é enviado ao canal, o emissor/sender bloqueia a execução do código seguinte até que o valor seja recuperado do canal por outra goroutine.*

Na função `main()`, você primeiro cria uma instância do canal usando a função `make()`, e então prossegue para chamar a função `fib()` como uma goroutine - gerando os primeiros dez números de Fibonacci.

Você usa a palavra-chave `range` no canal `c` para continuar lendo os valores até que o canal seja fechado. Agora você deve ver a seguinte saída:

```
1
1
2
3
5
8
13
21
34
55
Exit
```

É importante observar que a palavra-chave `range` lerá continuamente os valores de um canal repetidamente até que o canal seja fechado. O não fechamento do canal causará um erro *fatal error: all goroutines are asleep - deadlock!*

### Espera assíncrona entre canais

No mundo real, você pode ter algumas goroutines em execução e, simultaneamente, enviar valores para diferentes canais. Considere o seguinte exemplo:

```go
package main

import (
	"fmt"
	"time"
)

func fib(n int, c chan int) {
	a, b := 1, 1
	for i := 0; i < n; i++ {
		c <- a
		a, b = b, a+b
		time.Sleep(2 * time.Second)
	}

	close(c)
}

func counter(n int, c chan int) {
	for i := 0; i < n; i++ {
		c <- i
		time.Sleep(1 * time.Second)
	}

	close(c)
}

func main() {
	c1 := make(chan int)
	c2 := make(chan int)

	go fib(10, c1)     // generate 10 fibo nums
	go counter(10, c2) // generate 10 numbers

	for i := range c1 {
		fmt.Println("fib()", i)
	}

	for i := range c2 {
		fmt.Println("counter()", i)
	}

	fmt.Println("Exit")
}
```

Neste exemplo, é chamado as funções de `fib()` e `counter()` como goroutines. Essas duas goroutines gravam valores independentemente em dois canais diferentes `c1` e `c2`. Como de costume, inseri instruções de atraso em ambas as funções para simular velocidades diferentes nas quais os valores são gravados nos canais. Quando as funções são concluídas, os canais são fechados.

Em seguida, tem a instrução de `range` para imprimir os valores nos dois canais:

```
fib() 1
fib() 1
fib() 2
fib() 3
fib() 5
fib() 8
fib() 13
fib() 21
fib() 34
fib() 55
counter() 0
counter() 1
counter() 2
counter() 3
counter() 4
counter() 5
counter() 6
counter() 7
counter() 8
counter() 9
Exit
```

Mas há um problema aqui. Observe que todos os números gerados pela função `fib()` são impressos antes que aqueles gerados pela função `counter()` sejam impressos. Isso ocorre porque o primeiro laço `for` precisa ser concluído antes que o segundo laço `for` possa começar.

O ideal é que os números sejam impressos sempre que estiverem disponíveis. Além disso, os números gerados pela função `counter()` devem ser impressos mais cedo porque o atraso é menor do que com a função `fib()`.

Para resolver este problema, você pode usar um loop `for`, junto com a instrução `select`:

```go
func main() {
	c1 := make(chan int)
	c2 := make(chan int)

	go fib(10, c1)     // generate 10 Fibonacci numbers
	go counter(10, c2) // generate 10 numbers

	c1Closed := false
	c2Closed := false

	for {
		select {
		case n, ok := <-c1:
			if !ok {
				// channel closed and drained
				c1Closed = true
				if c1Closed && c2Closed {
					fmt.Println("Exit")

					return
				}
			} else {
				fmt.Println("fib()", n)
			}
		case m, ok := <-c2:
			if !ok {
				// channel closed and drained
				c2Closed = true
				if c1Closed && c2Closed {
					fmt.Println("Exit")

					return
				}
			} else {
				fmt.Println("counter()", m)
			}
		}
	}
}
```

Em cada bloco de `case`, você recupera um valor do respectivo canal. Observe que, neste exemplo, você recupera o valor do canal e o atribui a um par de variáveis:

```go
n, ok := <-c1
```

A primeira variável contém o valor recuperado do canal, enquanto a segunda variável contém um valor booleano para indicar se a recuperação foi bem-sucedida. Se o valor for falso, indica que o canal já está fechado. As instruções anteriores recuperarão os valores de ambos os canais até que ambos sejam fechados. Agora você deve ver os seguintes resultados:

```
counter() 0
fib() 1
counter() 1
counter() 2
fib() 1
counter() 3
fib() 2
counter() 4
counter() 5
fib() 3
counter() 6
counter() 7
fib() 5
counter() 8
counter() 9
fib() 8
fib() 13
fib() 21
fib() 34
fib() 55
Exit
```

Observe que quando ambos os canais são fechados, a função principal será encerrada, conforme indicado pela instrução `return`. Se você quiser realizar algumas outras funções na função `main()` enquanto ao mesmo tempo recupera os valores dos dois canais, execute o loop `for` como uma goroutine:

```go
func main() {
	c1 := make(chan int)
	c2 := make(chan int)

	go fib(10, c1)     // generate 10 Fibonacci numbers
	go counter(10, c2) // generate 10 numbers

	c1Closed := false
	c2Closed := false

	go func() {
		for {
			select {
			case n, ok := <-c1:
				...
			case m, ok := <-c2:
				...
			}
		}
	}()

	fmt.Println("Continue to do something else...")
	fmt.Scanln() // needed here to prevent the program from existing before all the channel values are read
}
```

Executando o código anterior produz a seguinte saída:

```
Continue to do something else...
fib() 1
counter() 0
counter() 1
counter() 2
fib() 1
counter() 3
counter() 4
fib() 2
counter() 5
counter() 6
fib() 3
counter() 7
counter() 8
fib() 5
counter() 9
fib() 8
fib() 13
fib() 21
fib() 34
fib() 55
Exit
```

### Manipulação de conjunto de canais

Uma instrução `select` escolhe qual dentre um conjunto de possíveis operações de envio ou recebimento prosseguirá.

A instrução `select` se parece com uma instrução `switch-case`:

```go
select {
	case x := <-ch1:
	// Received x from ch1
	case y := <-ch2:
	// Received y from ch2
	case ch3 <- z:
	// Sent z to ch3
	default:
		// Optional default, if none of the other
		// operations can proceed
}
```

Em um maior nível, a instrução `select` escolhe uma das operações de envio ou recebimento que pode prosseguir e então executa o bloco correspondente à operação escolhida. No código anterior, o bloco para recepção de `x` de `ch1` é executado somente após `x` ser recebido de `ch1`. *Se houver diversas operações de envio ou recebimento que possam prosseguir, a instrução `select` escolherá uma aleatoriamente.* Se não houver nenhuma, a instrução `select` escolhe a opção `default`. Se uma opção `default` não existir, a instrução `select` será bloqueada até que uma das operações do canal fique disponível.

Usar a opção `default` em uma instrução `select` *é útil para envios e recebimentos sem bloqueio*. A opção `default` só será escolhida quando todas as outras opções não estiverem prontas. A seguir está uma operação de envio sem bloqueio:

```go
select {
	case ch <- x:
		sent = true
	default:
}
```

A instrução `select` anterior testará se o canal `ch` está pronto para enviar dados. Se estiver pronto, o valor `x` será enviado. Caso contrário, a execução continuará com a opção `default`. Observe que isso significa apenas que o canal `ch` não estava pronto para envio quando foi testado. No momento em que a opção `default` começar a ser executada, enviar para `ch` poderá ficar disponível.

Da mesma forma, o seguinte é um recebimento sem bloqueio:

```go
select {
	case x = <-ch:
		received = true
	default:
}
```

A instrução `select` resolve elegantemente um problema comum: se você pode executar duas operações simultâneas, qual você faz primeiro? Vocês não pode favorecer uma operação em detrimento de outras, ou você nunca processará alguns casos. Isso é chamado de *starvation*. A palavra-chave `select` permite que um goroutine leia ou escreva em um de um conjunto de canais. Parece muito com uma instrução switch em branco:

```go
select {
	case v := <-ch:
		fmt.Println(v)
	case v := <-ch2:
		fmt.Println(v)
	case ch3 <-x:
		fmt.Println("wrote", x)
	case <-ch4:
		fmt.Println("got value on ch4, but ignored it")
}
```

Cada caso em uma seleção é uma leitura ou gravação em um canal. Se uma leitura ou gravação for possível para um caso, ele é executado junto com o corpo do caso. Como um switch, cada caso em um `select` cria seu próprio bloco.

O que acontecerá se vários casos tiverem canais que podem ser lidos ou gravados? o algoritmo de seleção é simples: ele escolhe aleatoriamente a partir de qualquer um de seus casos que podem ir para; a ordem não é importante. Isso é muito diferente de uma instrução switch, que sempre escolhe o primeiro caso que resolve como verdadeiro. Ele também resolve o problema de *starvation*, visto que nenhum caso é favorecido em relação ao outro e todos são verificados ao mesmo tempo.

Outra vantagem de `select` escolhendo aleatoriamente é que evita um dos mais causas comuns de deadlocks: aquisição de bloqueios em uma ordem inconsistente. Se você tem dois goroutines que acessam os mesmos dois canais, eles devem ser acessados ​​no mesma ordem em ambos os goroutines, ou eles entrarão em deadlock. Isso significa que ninguém pode continuar porque eles estão esperando um pelo outro. Se cada goroutine em seu aplicativo Go está em um impasse(deadlocked), o runtime do Go mata seu programa (veja o exemplo abaixo).

#### Deadlocking goroutines

```go
func main() {
	ch1 := make(chan int)
	ch2 := make(chan int)

	go func() {
		v := 1
		ch1 <- v
		v2 := <-ch2
		fmt.Println(v, v2)
	}()

	v := 2
	ch2 <- v
	v2 := <-ch1

	fmt.Println(v, v2)
}
```

Se você executar este programa, verá o seguinte erro:

```
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:
main.main()
	/tmp/sandbox1206540161/prog.go:19 +0xaf

goroutine 34 [chan send]:
main.main.func1()
	/tmp/sandbox1206540161/prog.go:13 +0x38
created by main.main
	/tmp/sandbox1206540161/prog.go:11 +0x97
```

Lembre-se de que a `main` está rodando em uma goroutine que é lançada na inicialização pelo runtime do Go. A goroutine que foi lançada não pode prosseguir até que o `ch1` seja lido, e a goroutine principal não pode prosseguir até que o `ch2` seja lido.

Se envolvermos os acessos do canal na goroutine principal em um `select`, evitamos o deadlock:

#### Utilizar o `select` para evitar deadlocks

Com base no código anterior, refatoramos o código para trabalhar com uma instrução de `select` para poder evitar problemas de deadlocks:

```go
func main() {
	ch1 := make(chan int)
	ch2 := make(chan int)

	go func() {
		v := 1
		ch1 <- v
		v2 := <-ch2
		fmt.Println(v, v2)
	}()

	v := 2

	var v2 int

	select {
	case ch2 <- v:
	case v2 = <-ch1:
	}

	fmt.Println(v, v2)
}
```

Se você executar este programa, obterá o resultado:

```
2 1
```

Como um `select` verifica se algum de seus casos pode prosseguir, o deadlock é evitado. o goroutine que lançamos escreveu o valor `1` em `ch1`, então a leitura de `ch1` em `v2` em o goroutine `main` pode ser bem-sucedido.

Uma vez que o `select` é responsável pela comunicação através de uma série de canais, muitas vezes é embutido em um loop `for`:

```go
for {
	select {
		case <-done:
			return
		case v := <-ch:
			fmt.Println(v)
	}
}
```

Isso é tão comum que a combinação costuma ser chamada de loop `for-select`. Ao usar um loop `for-select`, você deve incluir uma maneira de sair do loop.

Assim como as instruções `switch`, uma instrução `select` pode ter uma cláusula `default`. Também apenas como switch, o `default` é selecionado quando não há casos com canais que podem ser lido ou escrito. Se você deseja implementar uma leitura ou gravação sem bloqueio em um canal, use um `select` com um `default`. O código a seguir não espera se não houver valor para ler no `ch`; ele executa imediatamente o corpo do `default`:

```go
select {
	case v := <-ch:
		fmt.Println("read from ch:", v)
	default:
		fmt.Println("no value written to ch")
}
```
### Exemplos

#### Troca de mensagem de saudação

No exemplo abaixo, usaremos uma Goroutine para enviar uma mensagem de saudação e depois receberemos a saudação no processo principal - `main`.

```go
package main

import (
	"log"
)

func greet(ch chan<- string) {
	ch <- "Hello"
}

func main() {
	ch := make(chan string)

	go greet(ch)

	log.Println(<-ch)
}
```

Execute o programa você verá a seguinte saída:

```
2024/04/21 22:16:40 Hello
```

Podemos ver que a mensagem foi entregue à função principal por meio do canal.

#### Troca de mensagens bidirecional

O que queremos agora é enviar mensagens da goroutine principal para a segunda goroutine e, em seguida, receber uma mensagem de volta como resposta. Vamos basear nosso código no anterior e expandi-lo. A goroutine principal enviará uma mensagem "Hello John", enquanto a segunda goroutine retornará "Thanks" pela mensagem recebida, e então adicionará uma mensagem: "Hello David".

```go
package main

import (
	"fmt"
	"log"
)

// The greet() function signature has not changed.
// However, now before sending a message, it will first wait for a message and then reply.
// After receiving the message, this function sends a message back thanking for the
// greeting, and then sends its own greeting.
func greet(ch chan string) {
	msg := <-ch
	time.Sleep(8 * time.Second)
	ch <- fmt.Sprintf("Thanks for %s", msg)
	ch <- "Hello David"
}

func main() {
	ch := make(chan string)

	// Here, the main function is created and a string channel is instantiated.
	// Then, the second Goroutine is started.
	// Next, we need to send the first message from the main routine to the second, which is currently waiting.
	go greet(ch)

	ch <- "Hello John"

	log.Println(<-ch)
	log.Println(<-ch)
}
```

```
2024/04/21 22:27:21 Thanks for Hello John
2024/04/21 22:27:21 Hello David
```

Na saída, você pode ver que ambas as mensagens foram recebidas por meio do canal.

Neste exemplo, você aprendeu como uma goroutine pode enviar e receber mensagens através do mesmo canal e que duas goroutines podem trocar mensagens através do mesmo canal em ambas as direções (enviar e receber).

#### Soma de números de diferentes locais de origem

Imagine que você deseja adicionar alguns números, mas os números vêm de várias fontes. Eles podem vir de um feed ou de um banco de dados; simplesmente não sabemos quais números vamos somar e de onde vêm. No entanto, precisamos adicioná-los todos em um só lugar. Neste exemplo, teremos quatro Goroutines enviando números em intervalos específicos, e a goroutine principal, que calculará sua soma.

```go
package main

import (
	"log"
	"time"
)

func push(from, to int, in chan bool, out chan int) {
	for i := from; i <= to; i++ {
		<-in
		out <- i
		time.Sleep(time.Microsecond)
	}
}

func main() {
	s1 := 0

	in := make(chan bool, 100)
	out := make(chan int, 100)

	go push(1, 25, in, out)
	go push(26, 50, in, out)
	go push(51, 75, in, out)
	go push(76, 100, in, out)

	for c := 0; c < 100; c++ {
		in <- true
		i := <-out
		log.Println(i)
		s1 += i
	}

	log.Println(s1)
}
```

Abaixo, temos a saída truncada depois que você executa o programa:

```
2024/04/21 22:29:20 1
2024/04/21 22:29:20 26
2024/04/21 22:29:20 76
2024/04/21 22:29:20 51
2024/04/21 22:29:20 2
2024/04/21 22:29:20 27
2024/04/21 22:29:20 77
2024/04/21 22:29:20 52
2024/04/21 22:29:20 3
2024/04/21 22:29:20 28
2024/04/21 22:29:20 78
2024/04/21 22:29:20 53
...
2024/04/21 22:29:20 5050
```

Neste exemplo, vimos como podemos dividir algum trabalho computacional em várias goroutines simultâneas e, em seguida, reunir todo o cálculo em uma única goroutine. Cada goroutine executa uma tarefa. Nesse caso, uma envia números, enquanto outro recebe os números e faz uma soma.

#### Outro exemplo para o uso de canais

Considere o programa a seguir, onde você tem duas funções - `sendData()` e `getData()`:

```go
package main

import (
	"fmt"
	"time"
)

// ---send data into a channel---
func sendData(ch chan string) {
	fmt.Println("Sending a string into channel...")
	time.Sleep(2 * time.Second)
	ch <- "Hello"
}

// ---getting data from the channel---
func getData(ch chan string) {
	fmt.Println("String retrieved from channel:", <-ch)
}

func main() {
	ch := make(chan string)

	go sendData(ch)
	go getData(ch)

	fmt.Scanln()
}
```

Na função `main()`, primeiro é criado um canal usando a função `make()`, `ch`, com o tipo de canal especificado (`string`). Isso significa que o canal só pode conter valores do tipo `string`. Em seguida, é chamado as funções `sendData()` e `getData()` como goroutines. Na função `sendData()`, é impresso *sending a string into channel...*. Após um atraso de dois segundos, o código insere uma string no canal usando o operador `<-`.

Ao mesmo tempo, ao executar a função `sendData()`, você também executa a função `getData()`. Com `getData()`, está sendo tentado receber um valor do canal. Como atualmente não há valor no canal (não terá nenhum valor nele até dois segundos depois), a função `getData()` será bloqueada. No momento em que um valor está disponível no canal, a função `getData()` irá desbloquear e recuperar o valor do canal. Portanto, a saída do programa será semelhante a esta:

```
Sending a string into channel...
[After a two-second delay]
String retrieved from channel: Hello
```

Agora vamos fazer algumas alterações no programa:

```go
package main

import (
	"fmt"
	"time"
)

//---send data into a channel---
func sendData(ch chan string) {
	fmt.Println("Sending a string into channel...")
	// comment out the following line
	// time.Sleep(2 * time.Second)
	ch <- "Hello"
	fmt.Println("String has been retrieved from channel...")
}

//---getting data from the channel---
func getData(ch chan string) {
	time.Sleep(2 * time.Second)
	fmt.Println("String retrieved from channel:", <-ch)
}

func main() {
	ch := make(chan string)
	go sendData(ch)
	go getData(ch)
	fmt.Scanln()
}
```

Observe que na função `sendData()`, é tentado imprimir uma frase imediatamente após enviar um valor para o canal. Na função `getData()` , é inserido um atraso de dois segundos antes de recuperar o valor do canal.

Vamos executar o programa e ver o resultado:

```
Sending a string into channel...
[After a two-second delay...]
String retrieved from channel: Hello
String has been retrieved from channel...
```

Observe que imediatamente após enviar um valor para o canal, a função `sendData()` é bloqueada. Ele só será retomado depois que o valor no canal for recuperado pela função `getData()`.

## Utilização de Canais com Buffer

|           | **Unbuffered, open**               | **Unbuffered, closed**                              | **Buffered, open**                    | **Buffered, closed**                                                                                                | **Nil**        |
|-----------|------------------------------------|-----------------------------------------------------|---------------------------------------|---------------------------------------------------------------------------------------------------------------------|----------------|
| **Read**  | *Pause until something is written* | *Return zero value (use comma ok to see if closed)* | *Pause if buffer is empty*            | *Return a remaining value in the buffer. If the buffer is empty, return zero value (use comma ok to see if closed)* | *Hang forever* |
| **Write** | *Pause until something is read*    | `PANIC`                                             | *Pause if buffer is full*             | `PANIC`                                                                                                             | *Hang forever* |
| **Close** | *Works*                            | `PANIC`                                             | *Works, remaining values still there* | `PANIC`                                                                                                             | `PANIC`        |

Até agora, o que foi visto sobre canais foi centrado em *unbuffered channel*(canal sem buffer). Quando você envia um valor para um canal sem buffer, seu código será bloqueado até que o valor seja recebido do canal. Da mesma forma, quando você lê de um canal sem buffer, seu código será bloqueado até que um valor esteja disponível e recuperado do canal.

*Unbuffered channel, por outro lado, permite que vários valores sejam armazenados no canal. Seu código só será bloqueado quando você tentar enviar um valor para um canal que está cheio ou quando você tentar ler de um canal vazio.*

Para criar um *unbuffered channel*(canal sem buffer), forneça o comprimento do buffer como o segundo argumento para a função `make()` ao inicializar um canal:

```go
c := make(chan int, 10)
```

Esta instrução cria um buffered channel com tamanho de buffer `10`. Em outras palavras, o canal pode conter até dez valores antes que o código de envio seja bloqueado. Um canal sem buffer é equivalente a um canal com buffer de comprimento `0`:

```go
c := make(chan int, 0) // unbuffered channel
```

Então, quando você usa um canal com buffer? *Os canais com buffer são úteis se a taxa de envio for maior do que a taxa de recuperação. Ou quando você deseja que o remetente continue a execução após um valor ter sido enviado ao canal, sem esperar que o valor seja recuperado.*

Usando o exemplo abaixo, onde você calcula a soma de uma slice de números:

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func sum(s []int, c chan int) {
	sum := 0
	for _, v := range s {
		sum += v
	}

	c <- sum

	fmt.Println("Done and can continue to do other work")
}

func main() {
	s := []int{}
	sliceSize := 10

	for i := 0; i < sliceSize; i++ {
		s = append(s, rand.Intn(100))
	}

	c := make(chan int, 5) // buffered channel of length 5
	partSize := 2
	parts := sliceSize / partSize
	i := 0

	for i < parts {
		go sum(s[i*partSize:(i+1)*partSize], c)
		i += 1
	}

	i = 0
	total := 0
	time.Sleep(1 * time.Second) // simulate retrieving at a later time

	for i < parts {
		partialSum := <-c // read from channel
		fmt.Println("Partial Sum: ", partialSum)
		total += partialSum
		i += 1
	}

	fmt.Println("Total: ", total)
	fmt.Scanln()
}
```

Ao executar o programa, você verá que depois de somar a parte parcial dos números, a função `sum()` não será bloqueada e pode continuar a executar outras tarefas:

```
Done and can continue to do other work
Done and can continue to do other work
Done and can continue to do other work
Done and can continue to do other work
Done and can continue to do other work
Partial Sum:  128
Partial Sum:  146
Partial Sum:  143
Partial Sum:  56
Partial Sum:  103
Total:  576
```

## Mutex

**Mutex** é a abreviação de *mutual exclusion*. É um mecanismo de sincronização para garantir que apenas uma goroutine possa entrar em uma seção crítica enquanto outras aguardam.

Um *mutex* está pronto para ser usado quando declarado. Uma vez declarado, um *mutex* oferece duas operações básicas: bloquear e desbloquear. Um *mutex* pode ser bloqueado apenas uma vez, portanto, se uma goroutine bloquear um *mutex*, todas as outras goroutines que tentarem bloqueá-lo serão bloqueadas até que o *mutex* seja desbloqueado. Isso garante que apenas uma goroutine entre em uma seção crítica.

Os usos típicos de mutexes são os seguintes:

```go
var m sync.Mutex

func f() {
	m.Lock()

	// Critical section

	m.Unlock()
}

func g() {
	m.Lock()
	defer m.Unlock()

	// Critical section
}
```

Para garantir a exclusão mútua de uma seção crítica, o *mutex* deve ser um objeto compartilhado. Ou seja, um *mutex* definido para uma seção particularmente crítica deve ser compartilhado por todos os goroutines para estabelecer exclusão mútua.

Veremos o uso de *mutexes* com um exemplo realista. Um problema comum que foi resolvido muitas vezes é o problema de cache: certas operações, como cálculos caros, operações de E/S ou trabalho com bancos de dados, são lentas, por isso faz sentido armazenar em cache os resultados depois de obtê-los. Mas, por definição, um cache é compartilhado entre muitas goroutines, portanto deve ser thread-safe. O exemplo a seguir é uma implementação de cache que carrega objetos de um banco de dados e os coloca em um mapa. Caso o objeto não exista no banco de dados, o cache também lembra que:

```go
// Simple cache implementation. This cache may load the sema element
// multiple times. You may get a different number of cache misses
// every time you run it
type Cache struct {
	mu sync.Mutex
	m  map[string]*Data
}

func (c *Cache) Get(ID string) (Data, bool) {
	c.mu.Lock()
	data, exists := c.m[ID]
	c.mu.Unlock()

	if exists {
		if data == nil {
			return Data{}, false
		}

		return *data, true
	}

	data, loaded := retrieveData(ID)
	c.mu.Lock()
	defer c.mu.Unlock()

	d, exists := c.m[data.ID]
	if exists {
		return *d, true
	}

	if !loaded {
		c.m[ID] = nil
		return Data{}, false
	}

	c.m[data.ID] = data

	return *data, true
}
```

A estrutura do `Cache` inclui um *mutex*. O método `Get` começa bloqueando o cache. Isso ocorre porque `Cache.m` é compartilhado entre goroutines, e todas as operações de leitura ou gravação envolvendo `Cache.m` devem ser feitas por apenas uma goroutine. Se houver outras solicitações de cache em andamento naquele momento, esta chamada será bloqueada até que as outras goroutines sejam concluídas.

A primeira seção crítica simplesmente lê o mapa para ver se o objeto solicitado já está no cache. Observe que o cache é desbloqueado assim que a seção crítica é concluída para permitir que outras goroutines entrem em suas seções críticas. Se o objeto solicitado estiver no cache, ou se a inexistência desse objeto estiver registrada no cache, o método retorna. Caso contrário, o método recupera o objeto do banco de dados. Como o bloqueio não é mantido durante esta operação, outras goroutines podem continuar usando o cache. Isso pode fazer com que outras goroutines também carreguem o mesmo objeto. Depois que o objeto é carregado, o cache é bloqueado novamente porque o objeto carregado deve ser colocado no cache. Desta vez, podemos usar `defer c.mu.Unlock()` para garantir que o cache seja desbloqueado assim que o método retornar. Há uma segunda verificação para ver se o objeto já foi colocado no cache por outra goroutine. Isso é possível porque várias goroutines podem solicitar o objeto usando o mesmo `ID` ao mesmo tempo, e muitas goroutines podem prosseguir para carregar o objeto do banco de dados. Verificar isso novamente após adquirir o bloqueio garantirá que se outra goroutine já tiver colocado o objeto no cache, ele não será substituído por uma nova cópia.

Um ponto importante a ser observado aqui é que os mutexes não devem ser copiados. Ao copiar um mutex, você acaba com dois mutexes, o original e a cópia, e bloquear o original não impedirá que as cópias bloqueiem suas cópias também. A ferramenta `go vet` detecta isso. Por exemplo, declarar o método `Get` de cache usando um receptor de valor em vez de um ponteiro copiará a estrutura de cache e o mutex:

```go
func (c Cache) Get(ID string) (Data, bool) {...}
```

Isso copiará o *mutex* em cada chamada, portanto, todas as chamadas `Get` simultâneas entrarão na seção crítica sem exclusão mútua.

Um *mutex* não controla qual goroutine o bloqueou. Isto tem algumas implicações. Primeiro, bloquear um *mutex* duas vezes da mesma goroutine irá travar essa goroutine. Este é um problema comum com múltiplas funções que podem chamar umas às outras e também bloquear o mesmo *mutex*:

```go
var m sync.Mutex

func f() {
	m.Lock()
	defer m.Unlock()
	// process
}

func g() {
	m.Lock()
	defer m.Unlock()
	f() // Deadlock
}
```

No código acima, a função `g()` chama a função `f()`, mas o *mutex* `m` já está bloqueado, então `f` entra em conflito. Uma maneira de corrigir esse problema é declarar duas versões de `f`, uma com bloqueio e outra sem:

```go
func f() {
	m.Lock()
	defer m.Unlock()
	fUnlocked()
}

func fUnlocked() {
	// process
}

func g() {
	m.Lock()
	defer m.Unlock()
	fUnlocked()
}
```

Segundo, não há nada que impeça uma goroutine não relacionada de desbloquear um *mutex* bloqueado por outra goroutine. Essas coisas tendem a acontecer após a refatoração de algoritmos e o esquecimento de alterar os nomes dos mutex durante o processo. Eles criam bugs muito sutis.

A funcionalidade de um *mutex* pode ser replicada usando um canal com tamanho de buffer `1`:

```go
var mutexCh = make(chan struct{},1)

func Lock() {
	mutexCh<-struct{}{}
}

func Unlock() {

	select {
		case <-mutexCh:
		default:
	}
}
```

Muitas vezes, como no exemplo de cache anterior, existem dois tipos de seções críticas: uma para os leitores e outra para os gravadores. A seção crítica para os leitores permite que vários leitores entrem na seção crítica, mas não permite que um escritor entre na seção crítica até que todos os leitores tenham terminado. A seção crítica para escritores exclui todos os outros escritores e todos os leitores. Isto significa que pode haver muitos leitores simultâneos de uma estrutura, mas pode haver apenas um escritor. Para isso, um *mutex* `RWMutex` pode ser usado. Este *mutex* permite que vários leitores ou um único gravador mantenham o bloqueio. O cache modificado é mostrado a seguir:

```go
type Cache struct {
	mu sync.RWMutex // Use read/write mutex
	cache map[string]*Data
}

func (c *Cache) Get(ID string) (Data, bool) {
	c.mu.RLock()
	data, exists := c.m[data.ID]
	c.mu.RUnlock()

	if exists {
		if data == nil {
			return Data{}, false
		}

		return *data, true
	}

	data, loaded = retrieveData(ID)

	c.mu.Lock()
	defer c.mu.Unlock()

	d, exists := c.m[data.ID]

	if exists {
		return *d, true
	}

	if !loaded {
		c.m[ID] = nil
		return Data{}, false
	}

	c.m[data.ID] = data

	return *data, true
}
```

Observe que o primeiro bloqueio é um bloqueio de leitor. Ele permite que muitas goroutines de leitor sejam executadas simultaneamente. Depois que for determinado que o cache precisa ser atualizado, um bloqueio de gravador será usado.

## Wait Groups

Um Wait Group espera que uma coleção de coisas, geralmente goroutines, termine. É essencialmente um contador thread-safe que permite esperar até que o contador chegue a zero. Um padrão comum para seu uso é este:

```go
// Create a waitgroup
wg := sync.WaitGroup{}

for i := 0; i < 10; i++ {
	// Add to the wait group **before** creating the goroutine
	wg.Add(1)
	go func() {
		// Make sure the waitgroup knows about
		// goroutine completion
		defer wg.Done()
		// Do work
	}()
}

// Wait until all goroutines are done
wg.Wait()
```

Quando você cria um `WaitGroup`, ele é inicializado em zero, portanto, uma chamada para `Wait` não esperará por nada. Então, você deve adicionar o número de coisas que ele precisa esperar antes de chamar `Wait`. Para fazer isso, chamamos `Add(n)`, onde `n` é o número de itens a serem adicionados para espera. Torna mais fácil para o leitor chamar `Add(1)` logo antes de criar o item a ser esperado, que é, neste caso, uma goroutine. A goroutine principal então chama `Wait`, que aguardará até que o contador do grupo de espera chegue a zero. Para que isso aconteça, temos que ter certeza de que o método `Done` seja chamado para cada goroutine retornada. Usar uma instrução `defer` é a maneira mais fácil de garantir isso.

‌Um uso comum de `WaitGroup` é em um serviço orquestrador que chama vários serviços e coleta os resultados. O serviço orquestrador precisa aguardar o retorno de todos os serviços para continuar a computação.

Veja o exemplo a seguir:

```go
func orchestratorService() (Result1, Result2) {
	wg := sync.WaitGroup{} // Create a WaitGroup

	wg.Add(1) // Add the first goroutine
	var result1 Result1

	go func() {
		defer wg.Done()          // Make sure waitgroup knows completion
		result1 = callService1() // Call service1
	}()

	wg.Add(1) // Add the second goroutine
	var result2 Result2

	go func() {
		defer wg.Done()          // Make sure waitgroup knows completion
		result2 = callService2() // Call service2
	}()

	wg.Wait() // Wait for both services to return

	return result1, result2 // Return results
}
```

Um erro comum ao trabalhar com um `WaitGroup` é chamar `Add` ou `Done` no lugar errado. Há dois pontos a serem lembrados:

- `Add` deve ser chamado antes que o programa tenha a chance de executar `Wait`. Isso implica que você não pode chamar `Add` dentro da goroutine que você está aguardando usando `WaitGroup`. Não há garantia de que a goroutine será executada antes de `Wait` ser chamado.

- `Done` deve ser chamado eventualmente. A maneira mais segura de fazer isso é usar uma instrução `defer` dentro da goroutine, portanto, se a lógica da goroutine mudar com o tempo ou retornar de forma inesperada (como um `panic`), `Done` será chamado.

Às vezes, usar um grupo de espera e canais juntos podem causar alguns problemas: você precisa fechar um canal após `Wait`, mas `Wait` não terminará a menos que você feche o canal. Veja o seguinte programa:

```go
1: func main() {
2: 		ch := make(chan int)
3: 		var wg sync.WaitGroup
4:		for i := 0; i < 10; i++ {
5:			wg.Add(1)
6:			go func(i int) {
7:				defer wg.Done()
8:				ch<-i
9:			}(i)
10:		}
11:		// There is no goroutine reading from ch
12:		// None of the goroutines will return
13:		// so this will deadlock at Wait below
14:		wg.Wait()
15:		close(ch)
16:		for i := range ch {
17:			fmt.Println(i)
18:		}
19: }
```

Uma possível solução é colocar o loop `for` nas linhas 16-18 em uma goroutine separada antes de `Wait`, para que haja uma leitura da goroutine nos canais. Como os canais serão lidos, todas as goroutines serão encerradas, o que irá liberar o `wg.Wait`, e fechar o canal, encerrando o leitor no loop `for`:

```go
go func() {
	for i := range ch {
		fmt.Println(i)
	}
}()

wg.Wait()
close(ch)
```

Outra solução é a seguinte:

```go
go func() {
	wg.Wait()
	close(ch)
}()

for i := range ch {
	fmt.Println(i)
}
```

O grupo de espera agora está aguardando dentro de outra goroutine e, após todas as goroutines esperadas retornarem, ele fecha o canal.

## Práticas e Alguns Padrões de Concorrência

#### Variáveis com goroutines de closure

Na maioria das vezes, a closure que você usa para iniciar um goroutine não tem parâmetros. Em vez disso, ele captura valores do ambiente onde foi declarado. Há um situação comum em que isso não funciona: ao tentar capturar o índice ou valor de um loop `for`. Este código contém um bug sutil:

```go
func main() {
	a := []int{2, 4, 6, 8, 10}
	ch := make(chan int, len(a))

	for _, v := range a {
		go func() {
			ch <- v * 2
		}()
	}

	for i := 0; i < len(a); i++ {
		fmt.Println(<-ch)
	}
}
```

Lançamos um goroutine para cada valor em `a`. Parece que passamos um valor diferente em para cada goroutine, mas a execução do código mostra algo diferente:

```
20
20
20
20
20
```

A razão pela qual cada goroutine escreveu de `20` a `ch` é que o closure para cada goroutine capturou a mesma variável. As variáveis ​​de índice e valor em um loop `for` são reutilizadas em cada iteração. O último valor atribuído a `v` foi `10`. Quando os goroutines são executados, isso é o valor que eles veem. Esse problema não é exclusivo dos loops for; qualquer hora um goroutine depende de uma variável cujo valor pode mudar, você deve passar o valor para o goroutine. Existem duas maneiras de fazer isso. A primeira é shadow o valor dentro do loop:

```go
for _, v := range a {
	v := v
	go func() {
		ch <- v * 2
	}()
}
```

```
20
4
12
8
16
```

Se quiser evitar shadowing e tornar o fluxo de dados mais óbvio, você também pode passe o valor como um parâmetro para o goroutine:

```go
for _, v := range a {
	go func(val int) {
		ch <- val * 2
	}(v)
}
```

```
20
8
4
12
16
```

*Sempre que sua goroutine usa uma variável cujo valor pode mudar, passe o valor atual da variável para a goroutine.*

#### `Done` channel pattern

O *done channel pattern* fornece uma maneira de sinalizar a uma goroutine que é hora de parar o processamento. Ele usa um canal para sinalizar que é hora de sair. Vejamos um exemplo onde passamos os mesmos dados para várias funções, mas só queremos o resultado da função mais rápida:

```go
func searchData(s string, searchers []func(string) []string) []string {
	done := make(chan struct{})
	result := make(chan []string)

	for _, searcher := range searchers {
		go func(searcher func(string) []string) {
			select {
			case result <- searcher(s):
			case <-done:
			}
		}(searcher)
	}

	r := <-result
	close(done)
	return r
}
```

Em nossa função, declaramos um canal com nome `done` que contém dados do tipo `struct{}`. Usamos uma estrutura vazia para o tipo porque o valor não é importante; nós nunca escrevemos para este canal, apenas fechar. Lançamos um goroutine para cada `searcher` passado. As instruções `select` nas worker goroutines esperam por uma gravação no canal de `result` (quando a função de `searcher` retorna) ou uma leitura no canal `done`. Lembre-se de que uma leitura em um canal aberto pausa até que haja dados disponíveis capaz e que uma leitura em um canal fechado sempre retorna o valor zero para o canal. Isso significa que o caso que lê de `done` ficará pausado até que `done` seja fechado. No `searchData`, lemos o primeiro valor escrito para `result` e, em seguida, fechamos `done`. Esse sinaliza aos goroutines que devem sair, evitando que vazem / leaking.
