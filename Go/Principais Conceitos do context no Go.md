---
id: 9be8db33-3dfc-418e-af5b-2c02e52d8525
title: "Principais Conceitos do `context` no Go"
summary: "Um estudo sobre o context e seus principais pontos: Cancelamento, Timers e Valores."
---

## Definição

> Um *`context`* é simplesmente uma instância que atende à interface de `Context` definida no pacote de *`context`*. *Go* é idiomático e incentiva a passagem explícita dos dados por meio dos parâmetros da função. O mesmo é verdade para *`context`*. É apenas mais um parâmetro para sua função. Assim como *Go* tem uma convenção de que o último valor de retorno de uma função é um `error`, há outra convenção em *Go* de que o *`context`* é explicitamente passado por seu programa como o primeiro parâmetro da função.

O nome usual para o parâmetro de *`context`* é `ctx`:

```go
func logic(ctx context.Context, info string) (string, error) {
	// do some interesting stuff here
	return "", nil
}
```

Além de definir a interface de `Context`, o pacote de *`context`* também contém várias funções de fábrica para criar e agrupar *contexts*. Quando você não tem um *`context`* existente, como no *entry point* para um programa CLI, crie um *`context`* inicial vazio com a função `context.Background`. Isso retorna uma variável do tipo `context.Context`.

Um *context* vazio é um ponto de partida; cada vez que você adiciona metadados ao *context*, você o faz envolvendo o *`context`* existente usando uma das funções de fábrica do pacote `context`:

```go
ctx := context.Background()
result, err := logic(ctx, "a string")
```

Ao manipular um *servidor HTTP*, você usa um padrão ligeiramente diferente para adquirir e passar o *context* por meio de camadas de *middleware* para o top-level `http.Handler`. Infelizmente, o *`context`* foi adicionado às *APIs* do *Go* muito depois que o pacote `net/http` foi criado. Devido à promessa de compatibilidade, não havia como alterar a interface `http.Handler` para adicionar um parâmetro `context.Context`.

*A promessa de compatibilidade permite que novos métodos sejam adicionados aos tipos existentes*, e foi isso que a equipe do *Go* fez. Existem dois métodos relacionados ao context em `http.Request`:

- *`Context` retorna o `context.Context` associado à solicitação;*
- *`WithContext` recebe um `context.Context` e retorna um novo `http.Request` com o estado da solicitação anterior combinado com o `context.Context` fornecido;*

Aqui está o padrão geral:

```go
func Middleware(handler http.Handler) http.Handler {
	return http.HandlerFunc(func(rw http.ResponseWriter, req *http.Request) {
		ctx := req.Context()
		// wrap the context with stuff -- we'll see how soon!
		req = req.WithContext(ctx)
		handler.ServeHTTP(rw, req)
	})
}
```

A primeira coisa que fazemos em nosso *middleware* é extrair o *context* existente da solicitação usando o método `Context`. Depois de colocar os valores no *context*, criamos uma nova solicitação com base na solicitação antiga e no *context* agora preenchido, usando o método `WithContext`. Por fim, chamamos o manipulador e passamos nossa nova solicitação e o `http.ResponseWriter` existente.

Quando o fluxo chega no método `handler`, é extraído o *`context`* da solicitação usando o método `Context` e chama a lógica de negócios no método `logic` com o *`context`* como o primeiro parâmetro, assim como vimos anteriormente:

```go
func handler(rw http.ResponseWriter, req *http.Request) {
	ctx := req.Context()
	err := req.ParseForm()

	if err != nil {
		rw.WriteHeader(http.StatusInternalServerError)
		rw.Write([]byte(err.Error()))
		return
	}

	data := req.FormValue("data")
	result, err := logic(ctx, data)

	if err != nil {
		rw.WriteHeader(http.StatusInternalServerError)
		rw.Write([]byte(err.Error()))
		return
	}

	rw.Write([]byte(result))
}
```

Há mais uma situação em que você usa o método `WithContext`: ao fazer uma *chamada HTTP* de seu aplicativo para outro serviço *HTTP*. Assim como fizemos ao passar um *`context`* por meio de middleware, você define o *`context`* na solicitação de saída usando `WithContext` :

```go
type ServiceCaller struct {
	client *http.Client
}

func (sc ServiceCaller) callAnotherService(ctx context.Context, data string) (string, error) {
	req, err := http.NewRequest(http.MethodGet, "http://example.com?data=" + data, nil)

	if err != nil {
		return "", err
	}

	req = req.WithContext(ctx)
	resp, err := sc.client.Do(req)

	if err != nil {
		 return "", err
	}

	defer resp.Body.Close()

	if resp.StatusCode != http.StatusOK {
		return "", fmt.Errorf("Unexpected status code %d", resp.StatusCode)
	}

	// do the rest of the stuff to process the response
	id, err := processResponse(resp.Body)
	return id, err
}
```

Agora que sabemos como adquirir e passar um *`context`*, é hora de torná-los úteis. Começando com **cancelamento**.

## Cancelamento do Contexto

Imagine que você tenha uma solicitação que gera vários *goroutines*, cada uma chamando um serviço *HTTP* diferente. Se um serviço retorna um erro que o impede de retornar um resultado válido, não há nenhum ponto/porque em continuar processando as outras *goroutines*. Em *Go*, isso é chamado de *cancellation* e o *`context`* fornece o mecanismo para implementação.

Para criar um *`context`* cancelável, use a função `context.WithCancel`. Ele recebe um `context.Context` como parâmetro e retorna um `context.Context` e um `context.CancelFunc`. O `context.Context` devolvido não é o mesmo *context* que foi passado para a função. Em vez disso, é um *context* filho que envolve o context pai `context.Context` passado. Um `context.CancelFunc` é uma função que cancela o *`context`*, informando a todo o código que está ouvindo o **cancelamento** em potencial que é **hora de interromper o processamento**.

Vamos dar uma olhada em como isso funciona. Como esse código configura um servidor. Primeiro, vamos configurar dois servidores em um arquivo chamado `servers.go`:

```go
package main

import (
	"net/http"
	"net/http/httptest"
	"time"
)

func slowServer() *httptest.Server {
	s := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		time.Sleep(10 * time.Second)
		w.Write([]byte("slow response"))
	}))

	return s
}

func fastServer() *httptest.Server {
	s := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		if r.URL.Query().Get("error") == "true" {
			w.Write([]byte("error"))
			return
		}

		w.Write([]byte("ok"))
	}))
	return s
}
```

Essas funções iniciam servidores *HTTP* quando são chamadas. Um servidor dorme/aguarda por dez segundos e, em seguida, retorna a mensagem `slow response`. O outro verifica se há um parâmetro na query string da URL de `error` definido como `true`. Se houver, ele retornará a mensagem de `error`. Caso contrário, retorna a mensagem `ok`.

*É usando o `httptest.Server`, o que torna mais fácil escrever testes unitários para o código que se comunica com servidores HTTP remotos.*

A seguir, vamos escrever a parte do código do *client* em um arquivo chamado `client.go`:

```go
package main

import (
	"context"
	"errors"
	"fmt"
	"io/ioutil"
	"net/http"
	"sync"
)

var client = http.Client{}

func callBoth(ctx context.Context, errVal string, slowURL string, fastURL string) {
	ctx, cancel := context.WithCancel(ctx)
	defer cancel()

	var wg sync.WaitGroup
	wg.Add(2)

	go func() {
		defer wg.Done()
		err := callServer(ctx, "slow", slowURL)
		if err != nil {
			cancel()
		}
	}()

	go func() {
		defer wg.Done()
		err := callServer(ctx, "fast", fastURL+"?error="+errVal)
		if err != nil {
			cancel()
		}
	}()

	wg.Wait()
	fmt.Println("done with both")
}

func callServer(ctx context.Context, label string, url string) error {
	req, err := http.NewRequestWithContext(ctx, http.MethodGet, url, nil)
	if err != nil {
		fmt.Println(label, "request err:", err)
		return err
	}

	resp, err := client.Do(req)
	if err != nil {
		fmt.Println(label, "response err:", err)
		return err
	}

	data, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Println(label, "read err:", err)
		return err
	}

	result := string(data)
	if result != "" {
		fmt.Println(label, "result:", result)
	}

	if result == "error" {
		fmt.Println("cancelling from", label)
		return errors.New("error happened")
	}

	return nil
}
```

Todas as coisas interessantes estão neste arquivo. Primeiro, a função `callBoth` cria um *`context`* cancelável e uma função de cancelamento a partir do *`context`* passado. Por convenção, essa variável de função é chamada de `cancel`. **É importante lembrar que sempre que você criar um *`context`* cancelável, deverá chamar a função `cancel`**. *Não tem problema chamá-lo mais de uma vez; cada invocação após a primeira é ignorada*. Usamos `defer` para garantir que ele seja eventualmente chamado. Em seguida, configuramos duas *goroutines* e passamos o *`context`* cancelável, uma label e a URL para `callServer` e esperamos que ambos sejam concluídos. Se qualquer chamada para `callServer` retornar um erro, chamamos a função de `cancel`.

A função `callServer` é um simples *client*. Criamos as requests com o *`context`* cancelável e fazemos uma chamada. Se ocorrer um erro, ou se obtivermos o `error` da query string, retornamos o erro.

Por fim, temos a função `main`, que dá início ao programa, no arquivo `main.go`:

```go
package main

import (
	"context"
	"os"
)

func main() {
	ss := slowServer()
	defer ss.Close()
	fs := fastServer()
	defer fs.Close()

	ctx := context.Background()
	callBoth(ctx, os.Args[1], ss.URL, fs.URL)
}
```

Na função `main`, é iniciado os servidores HTTP, criado um *`context`*, e depois chamar os *clients* com o *context* como primeiro argumento para o programa, e as *URLs* para os servidores.

Veja o que acontece se você executar sem erros:

```bash
make run-ok
# fast result: ok
# slow result: slow response
# done with both
```

E aqui está o que acontece se um erro for acionado:

```bash
make run-cancel
# fast result: error
# cancelling from fast
# slow response err: Get "http://127.0.0.1:53053": context canceled
# done with both
```

*Sempre que você criar um context que tenha uma função de cancelamento associada, deverá chamar essa função de cancelamento ao terminar o processamento, quer o processamento termine ou não em erro. Se você não fizer isso, seu programa vazará(leak) recursos (memória e goroutines) e, eventualmente, ficará lento ou travará(crash). Não haverá erro se você chamar a função `cancel` mais de uma vez; qualquer invocação após a primeira não faz nada. A maneira mais fácil de ter certeza de chamar a função `cancel` é usar `defer`.*

Embora o cancelamento manual seja útil, não é sua única opção. No próxima tópico, veremos como automatizar o cancelamento com tempos limite.

## Timers

Uma das tarefas mais importantes para um servidor *HTTP* é gerenciar as *requests*. Um programador novato frequentemente pensa que um servidor deve receber tantas *requests* quanto possível e manipular elas pelo tempo que puder até retornar um resultado para cada *client*.

O problema é que essa abordagem não é escalável. Um *server HTTP* é um recurso compartilhado. Como todos os recursos compartilhados, cada usuário deseja obter o máximo possível deles e não tem nenhuma preocupação com as necessidades dos outros usuários. *É responsabilidade do recurso compartilhado gerenciar a si mesmo de forma que forneça uma quantidade razoável de tempo a todos os seus usuários.*

Geralmente, há quatro coisas que um servidor *HTTP* pode fazer para gerenciar sua carga:

- *Limitar requests simultâneas;*
- *Limitar o número de request que estão na fila à espera de execução;*
- *Limite por quanto tempo uma request pode ser executada;*
- *Limitar os recursos que uma request pode usar (como memória ou espaço em disco);*

*Go* fornece ferramentas para lidar com os três primeiros. Ao limitar o número de *goroutines*, um servidor gerencia a carga simultânea. O tamanho da fila de espera é controlado por meio de canais *buffered*.

O *`context`* fornece uma maneira de controlar por quanto tempo uma solicitação é executada. Ao construir um aplicativo, você deve ter uma ideia de seu desempenho: *quanto tempo você tem para que sua solicitação seja concluída antes que o usuário tenha uma experiência insatisfatória*. Se você souber a quantidade máxima de tempo que uma solicitação pode ser executada, poderá aplicá-la usando o *`context`*.

Você pode usar uma das duas funções diferentes para criar um *`context`* de tempo limitado. O primeiro é `context.WithTimeout`. Leva dois parâmetros, um *`context`* existente e `time.Duration` que especifica a duração até que o *`context`* seja cancelado automaticamente. Ele retorna um *`context`* que dispara automaticamente um cancelamento após a duração especificada, bem como uma função de cancelamento que é chamada para cancelar o *`context`* imediatamente.

A segunda função é `context.WithDeadline`. Esta função considera um *`context`* existente e um `time.Time` que especifica a hora em que o *`context`* é cancelado automaticamente. Como `context.WithTimeout`, ele retorna um *`context`* que dispara automaticamente um cancelamento após o tempo especificado ter decorrido, bem como uma função de cancelamento.

Se você quiser saber quando um *`context`* será cancelado automaticamente, use o método `Deadline` em `context.Context`. Ele retorna um `time.Time` que indica o tempo e um `bool` que indica se houve um tempo limite definido. Isso reflete o *comma ok idiom* que é usado ao ler mapas ou canais.

Ao definir um limite de tempo para a duração da solicitação, você pode querer subdividir esse tempo. E se você chamar outro serviço a partir do seu serviço, pode limitar o tempo de execução da chamada de rede, reservando algum tempo para o restante do processamento ou para outras chamadas de rede. Você controla quanto tempo uma chamada individual leva criando um *`context`* filho que envolve um *`context`* pai usando `context.WithTimeout` ou `context.WithDeadline`.

Qualquer tempo limite definido no *`context`* filho é limitado pelo tempo limite definido no *`context`* pai; se um *`context`* pai expira em dois segundos, você pode declarar que um *`context`* filho expira em três segundos, mas quando o *`context`* pai expira depois de dois segundos, o filho também expirará.

Podemos ver isso com um simples programa:

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func main() {
	ctx := context.Background()
	parent, cancel := context.WithTimeout(ctx, 2*time.Second)
	defer cancel()
	child, cancel2 := context.WithTimeout(parent, 3*time.Second)
	defer cancel2()
	start := time.Now()
	<-child.Done()
	end := time.Now()
	fmt.Println(end.Sub(start)) // 2s
}
```

Neste exemplo, especificamos um tempo limite de dois segundos no *`context`* pai e um tempo limite de três segundos no *`context`* filho. Em seguida, esperamos que o *`context`* filho seja concluído, aguardando no canal retornado do método `Done` no *`context`* filho de `context.Context`.

Você pode executar este código e verá o seguinte resultado: `2s`

## Cancelamento de `context`

Na maioria das vezes, você não precisa se preocupar com tempos limite ou *cancelamento* dentro do seu próprio código; ele simplesmente não funciona por tempo suficiente. Sempre que você chamar outro serviço *HTTP* ou o banco de dados, deve transmitir o *`context`*; essas bibliotecas tratam adequadamente o *cancelamento* por meio do *`context`*.

Se você escrever um código que deve ser interrompido por um *cancelamento* de *`context`*, você implementa as verificações de *cancelamento* usando os recursos de *simultaneidade*. O tipo `context.Context` possui dois métodos que são usados ​​ao gerenciar o *cancelamento*.

O método `Done` retorna um canal de `struct{}`. (*O motivo pelo qual este é o tipo de retorno escolhido é que uma estrutura vazia não usa memória*). O canal é fechado quando o *`context`* é cancelado devido a um tempo limite especificado ou a função de *`cancel`* que está sendo chamada. Lembre-se de que um canal fechado sempre retorna imediatamente seu *zero value* quando você tenta lê-lo.

*Se você chamar `Done` em um `context` que não pode ser cancelado, ele retornará `nil`. Como sabemos, uma leitura de um canal `nil` nunca retorna. Se isso não for feito dentro de um `case` em uma instrução `select`, seu programa irá travar.*

O método `Err` retorna `nil` se o *`context`* ainda estiver ativo, ou retorna um dos dois erros se o *`context`* foi cancelado: `context.Canceled` e `context.DeadlineExceeded`. O primeiro é retornado após o cancelamento explícito e o segundo é retornado quando um cancelamento acionado por tempo limite.

Este é o padrão para suportar o cancelamento de um *`context`* em seu código:

```go
func longRunningThingManager(ctx context.Context, data string) (string, error) {
	type wrapper struct {
		result string
		err error
	}

	ch := make(chan wrapper, 1)

	go func() {
		// do the long running thing
		result, err := longRunningThing(ctx, data)
		ch <- wrapper{result, err}
	}()

	select {
		case data := <-ch:
			return data.result, data.err
		case <-ctx.Done():
			return "", ctx.Err()
	}
}
```

No código acima, precisamos colocar os dados retornados da função de longa duração em uma estrutura, para que possamos passá-los em um canal. Em seguida, criamos um canal do tipo `wrapper` com tamanho de *buffer* `1`. Armazenando o canal em *buffer*, permitimos que a *goroutine* se encerre, mesmo se o valor *buffered* nunca for lido devido ao cancelamento.

Na *goroutine*, recuperamos a saída da função de longa duração e a colocamos no canal *buffered*. Temos então um `select` com dois `case`. Em nosso primeiro `case` do `select`, lemos os dados da função de longa duração e os retornamos. Este é o caso que é acionado se o *`context`* não for cancelado devido ao tempo limite ou invocação da função `cancel`. O segundo `case` do `select` é acionado se o *`context`* for cancelado. Retornamos o *zero value* para os dados e o erro do *`context`* para nos dizer por que ele foi cancelado.

## Values

Existe mais um uso para o *`context`*. Ele também fornece uma maneira de passar metadados pela solicitação por meio do programa.

Por padrão, você deve preferir passar os dados por meio de parâmetros explícitos. Como foi mencionado antes, o *Go* é idiomático a favorece o explícito sobre o implícito, e isso inclui a passagem de dados explícita. Se uma função depende de alguns dados, deve ficar claro de onde ele veio.

Assim como há métodos de fábrica no pacote de *`context`* para criar *contexts timed* e *cancellable*, há um método de fábrica para colocar valores no context, `context.WithValue`. Ele possui três parâmetros: um *`context`* para ser manipulado, uma chave para pesquisar o valor e o próprio valor. Ele retorna um *`context`* filho que contém os dados de chave-valor. O tipo da chave e os parâmetros de valor são declarados como *interfaces vazias* (`interface{}`).

Para verificar se um valor está em um *`context`* ou em algum de seus *parents*, use o método `Value` em `context.Context`. Este método recebe uma chave e retorna o valor associado à chave. Novamente, o parâmetro da chave e o resultado do valor são declarados do tipo `interface{}`. Se nenhum valor for encontrado para a chave fornecida, `nil` será retornado. Use *comma ok idiom* com *type assert* do valor retornado para o tipo correto.

Embora o valor armazenado no *`context`* possa ser de qualquer tipo, existe um padrão idiomático usado para garantir a exclusividade da chave. Da mesma forma que a chave para um mapa, a chave para o valor do *`context`* deve ser comparável. Crie um novo tipo não exportado para a chave, com base em um `int`:

```go
type userKey int
```

Se você usar uma string ou outro tipo público para o tipo da chave, diferentes pacotes podem criar chaves idênticas, resultando em colisões. Isso causa problemas difíceis de depurar, como um pacote gravando dados no *`context`* que mascara os dados gravados por outro pacote ou lendo dados do *`context`* gravado por outro pacote.

Depois de declarar seu tipo de chave não exportada, você declara uma constante não exportada desse tipo:

```go
const key userKey = 1
```

Com o tipo e a constante da chave não sendo exportados, nenhum código de fora de seu pacote pode colocar dados no *`context`* que causaria uma colisão. Se o seu pacote precisar colocar vários valores no *`context`*, defina uma chave diferente do mesmo tipo para cada valor, usando o padrão `iota`. Como nos preocupamos apenas com o valor da constante como forma de diferenciar entre várias chaves, esse é um uso perfeito para `iota`.

Em seguida, construa uma API para colocar e ler um valor no *`context`*. Torne essas funções públicas apenas se o código fora de seu pacote puder ler e gravar seus valores no *`context`*. O nome da função que cria um *`context`* com o valor deve começar com `ContextWith`. A função que retorna o valor do *`context`* deve ter um nome que termina com `FromContext`. Aqui estão as implementações de nossas funções para obter e ler o usuário a partir do *`context`*:

```go
func ContextWithUser(ctx context.Context, user string) context.Context {
	return context.WithValue(ctx, key, user)
}

func UserFromContext(ctx context.Context) (string, bool) {
	user, ok := ctx.Value(key).(string)

	return user, ok
}
```

Agora que escrevemos nosso código de gerenciamento de usuário, vamos ver como usá-lo. Vamos escrever um middleware que extraia o ID do usuário de um *cookie*:

```go
// a real implementation would be signed to make sure
// the user didn't spoof their identity

func extractUser(req *http.Request) (string, error) {
	userCookie, err := req.Cookie("user")

	if err != nil {
		return "", err
	}

	return userCookie.Value, nil
}

func Middleware(h http.Handler) http.Handler {
	return http.HandlerFunc(func(rw http.ResponseWriter, req *http.Request) {
		user, err := extractUser(req)
		if err != nil {
			rw.WriteHeader(http.StatusUnauthorized)
			return
		}

		ctx := req.Context()
		ctx = ContextWithUser(ctx, user)
		req = req.WithContext(ctx)
		h.ServeHTTP(rw, req)
	})
}
```

No middleware, primeiro obtemos o valor do usuário. Depois, extraímos o *`context`* da *request* com o método `Context` e criamos um novo *`context`* que contém o usuário com a função `ContextWithUser`. Em seguida, criamos uma nova *request* a partir da *request* antiga e do novo *`context`* usando o método `WithContext`. Finalmente, chamamos a próxima função em nossa cadeia de manipuladores com a nova *request* e o `http.ResponseWriter` do método.

Na maioria dos casos, você deseja extrair o valor do *`context`* em seu manipulador da *request* e passá-lo para sua lógica de negócios explicitamente. As funções *Go* têm parâmetros explícitos e você não deve usar o *`context`* como uma forma de passar os valores pela *API*:

```go
func (c Controller) handleRequest(rw http.ResponseWriter, req *http.Request) {
	ctx := req.Context()
	user, ok := identity.UserFromContext(ctx)

	if !ok {
		rw.WriteHeader(http.StatusInternalServerError)
		return
	}

	data := req.URL.Query().Get("data")
	result, err := c.Logic.businessLogic(ctx, user, data)

	if err != nil {
		rw.WriteHeader(http.StatusInternalServerError)
		rw.Write([]byte(err.Error()))
		return
	}

	rw.Write([]byte(result))
}
```

Nosso manipulador obtém o *`context`* usando o método `Context` na *request*, extrai o usuário do *`context`* usando a função `UserFromContext` e chama a lógica de negócios.

Existem algumas situações em que é melhor manter um valor no *`context`*. O *GUID* de rastreamento é uma delas. Essas informações são destinadas ao gerenciamento do aplicativo; não faz parte do estado de negócios. Passá-lo explicitamente por meio do código adiciona parâmetros adicionais e impede a integração com bibliotecas de terceiros que não sabem sobre sua metainformação. Ao deixar um *GUID* de rastreamento no *`context`*, ele passa invisivelmente pela lógica de negócios que não precisa saber sobre rastreamento e está disponível quando seu programa grava uma mensagem de log ou se conecta a outro servidor.

Aqui está uma implementação *GUID* simples com base no *`context`* que rastreia de serviço a serviço e cria *logs* com o *GUID* incluído:

```go
package tracker

import (
	"context"
	"fmt"
	"net/http"

	"github.com/google/uuid"
)

type guidKey int

const key guidKey = 1

func contextWithGUID(ctx context.Context, guid string) context.Context {
	return context.WithValue(ctx, key, guid)
}

func guidFromContext(ctx context.Context) (string, bool) {
	g, ok := ctx.Value(key).(string)

	return g, ok
}

func Middleware(h http.Handler) http.Handler {
	return http.HandlerFunc(func(rw http.ResponseWriter, req *http.Request) {
		ctx := req.Context()
		if guid := req.Header.Get("X-GUID"); guid != "" {
			ctx = contextWithGUID(ctx, guid)
		} else {
			ctx = contextWithGUID(ctx, uuid.New().String())
		}

		req = req.WithContext(ctx)
		h.ServeHTTP(rw, req)
	})
}

type Logger struct{}

func (Logger) Log(ctx context.Context, message string) {
	if guid, ok := guidFromContext(ctx); ok {
		message = fmt.Sprintf("GUID: %s - %s", guid, message)
	}

	// do logging
	fmt.Println(message)
}

func Request(req *http.Request) *http.Request {
	ctx := req.Context()

	if guid, ok := guidFromContext(ctx); ok {
		req.Header.Add("X-GUID", guid)
	}

	return req
}
```

A função `Middleware` extrai o *GUID* da request de entrada ou gera um novo *GUID*. Em ambos os casos, ele coloca o *GUID* no *`context`*, cria uma nova request com o *`context`* atualizado e continua a cadeia de chamadas.

A seguir, veremos como esse *GUID* é usado. A estrutura `Logger` fornece um método de registro `Log` que tem como parâmetro um *`context`* e uma string. Se houver um *GUID* no *`context`*, ele o anexará ao início da mensagem de *log*. A função `Request` é usada quando este serviço faz uma chamada para outro serviço. Ele recebe um `*http.Request`, adiciona um cabeçalho com o *GUID* se ele existir no *`context`* e retorna o `*http.Request`.

Assim que tivermos esse pacote, podemos usar as técnicas de injeção de dependência para criar uma lógica de negócios que não tem conhecimento de nenhuma informação de rastreamento. Primeiro, declaramos uma interface para representar o `logger`, um tipo de função para representar um `Decorator` da request e uma estrutura da lógica de negócios que depende deles:

```go
type Logger interface {
	Log(context.Context, string)
}

type RequestDecorator func(*http.Request) *http.Request

type BusinessLogic struct {
	RequestDecorator RequestDecorator
	Logger Logger
	Remote string
}
```

Em seguida, implementamos a lógica de negócios:

```go
func (bl BusinessLogic) businessLogic(ctx context.Context, user string, data string) (string, error) {
	bl.Logger.Log(ctx, "starting businessLogic for " + user + " with " + data)

	req, err := http.NewRequestWithContext(ctx, http.MethodGet, bl.Remote+"?query="+data, nil)

	if err != nil {
		bl.Logger.Log(ctx, "error building remote request:" + err)
		return "", err
	}

	req = bl.RequestDecorator(req)
	resp, err := http.DefaultClient.Do(req)

	// processing continues
}
```

O *GUID* é passado para o criador de *logs* e o Decorator da request sem que a lógica de negócios saiba, separando os dados necessários da lógica do programa dos dados necessários para o gerenciamento do programa. O único lugar que está ciente da associação é o código `main` que conecta nossas dependências:

```go
bl := BusinessLogic {
	RequestDecorator: tracker.Request,
	Logger: tracker.Logger{},
	Remote: "http://www.example.com/query",
}
```
