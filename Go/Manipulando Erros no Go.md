---
id: 9be8ce9d-bf60-4bc5-af88-20955873e4a9
title: Manipulando Erros no Go
summary: "Aprenda a criar, comparar e retornar erros. Além do mais, veja também o uso de Panic e Recover."
---

## Introdução

O que é um erro no *Go*? **Um erro em *Go* é um valor**. Aqui está uma citação de *Rob Pike*, um dos pioneiros do *Go*:

*"Values can be programmed, and since errors are values, errors can be programmed. Errors are not like exceptions. There's nothing special about them, whereas an unhandled exception can crash your program."*

Como os erros são valores, eles podem ser passados ​​para uma função, retornados de uma função e avaliados como qualquer outro valor em *Go*.

**Um erro no *Go* é qualquer coisa que implemente a interface de `error`**. Precisamos examinar alguns aspectos fundamentais que constituem o tipo de `error`. Para ser um tipo de `error`, ele deve primeiro satisfazer o **type error interface**.

`error` é uma *interface* que define um único método:

```go
// https://golang.org/pkg/builtin/#error
type error interface {
	Error() string
}
```

**Qualquer coisa que implemente esta interface é considerada um `error`**.

Existem dois bons motivos pelos quais *Go* usa um `error` retornado em vez de *lançar exceções*. Primeiro, as exceções adicionam pelo menos um novo caminho do código. Esses caminhos às vezes não são claros, especialmente em linguagens cujas funções não incluem uma declaração de que uma exceção é possível. Isso produz código que trava de maneiras surpreendentes quando as exceções não são tratadas adequadamente ou, pior ainda, código que não trava, mas cujos dados não são inicializados, modificados ou armazenados adequadamente.

O segundo motivo é mais sutil, mas demonstra como os recursos do *Go* funcionam juntos. O compilador *Go* requer que todas as variáveis ​​sejam lidas. Fazer *errors* nos valores retornados força os desenvolvedores a verificar e tratar as condições de *error* ou tornar explícito que estão ignorando os *errors* usando um sublinhado (`_`) para o valor de *error* retornado.

O tratamento de exceções pode produzir código mais curto, mas ter menos linhas não torna necessariamente o código mais fácil de entender ou manter. *Go* é idiomático e favorece o código claro, mesmo que tenha mais linhas.

*Outra coisa a observar é como o código flui em Go. O tratamento de erros é indentado dentro de uma instrução `if`. A lógica de negócios, não*. Isso dá uma rápida pista visual de qual código está no *"golden path"* e qual código é a condição excepcional.

É importante entender que um tipo de `error` é um tipo de *interface*. Qualquer valor que seja um `error` pode ser descrito como uma `string`. Ao executar o tratamento de erros no *Go*, *as funções retornarão um valor de erro*. A linguagem *Go* usa isso em toda a biblioteca padrão.

***Go* trata os erros retornando um valor do tipo `error` como o último valor de retorno de função**. *Isso é inteiramente por convenção, mas é uma convenção tão forte que nunca deve ser violada*. Quando uma função é executada conforme o esperado, `nil` é retornado para o parâmetro de `error`. Se algo der errado, um valor de `error` será retornado. A função de chamada então verifica o valor de retorno do `error` comparando-o com `nil`, tratando o erro ou retornando o próprio erro. O código é parecido com este:

```go
func calcRemainderAndMod(numerator, denominator int) (int, int, error) {
	if denominator == 0 {
		return 0, 0, errors.New("denominator is 0")
	}

	return numerator / denominator, numerator % denominator, nil
}
```

Um novo erro é criado a partir de uma `string` chamando a função `New` no pacote de `errors`. *As mensagens de erro não devem ser capitalizadas nem terminar com pontuação ou uma nova linha*. Na maioria dos casos, você deve definir os outros valores de retorno para seus *zero values* quando um erro diferente de `nil` for retornado. Veremos uma exceção a essa regra quando examinarmos os *erros sentinel*.

Ao contrário das linguagens com exceções, *Go não tem construções especiais para detectar se um erro foi retornado*. Sempre que houver um retorno da função, use uma instrução `if` para verificar a variável de erro para ver se ela não é `nil`:

```go
func main() {
	numerator := 20
	denominator := 3

	remainder, mod, err := calcRemainderAndMod(numerator, denominator)

	if err != nil {
		fmt.Println(err)
		os.Exit(1)
	}

	fmt.Println(remainder, mod)
}
```

O motivo pelo qual retornamos `nil` de uma função para indicar que nenhum `error` ocorreu é que **`nil` é o *zero value* para qualquer tipo de interface**.

## Criando Erros

Na biblioteca padrão, o pacote de `error` tem um método que podemos usar para criar nossos próprios erros:

```go
// https://golang.org/src/errors/errors.go
// New returns an error that formats as the given text.
// Each call to New returns a distinct error value even if the text is identical.
func New(text string) error {
	return &errorString{text}
}
```

É importante entender que a função `New` pega uma string como argumento e a converte em `*errors.errorString` e retorna como um valor de `error`. O valor subjacente do tipo de `error` que é retornado é do tipo `*errors.errorString`.

Podemos provar isso executando o seguinte código:

```go
package main

import (
	"errors"
	"fmt"
)

func main() {
	ErrBadData := errors.New("Some bad data")
	fmt.Printf("ErrBadData type: %T", ErrBadData)
}
```

Ao executar este código, você obterá a saída:

```
ErrBadData type: *errors.errorString
```

Aqui está um exemplo da biblioteca padrão `http` do *Go*, que usa o pacote de `errors` para criar variáveis ​​no nível do pacote:

```go
var (
	ErrBodyNotAllowed = errors.New("http: request method or response status code does not allow body")
	ErrHijacked = errors.New("http: connection has been hijacked")
	ErrContentLength = errors.New("http: wrote more than the declared ContentLength")
	ErrWriteAfterFlush = errors.New("unused")
)
```

*Ao criar seus próprios erros, é idiomático em Go começar com a variável com `Err`.*

### Exemplo - Criando um aplicativo para calcular o pagamento da semana

Neste exemplo, vamos criar uma função que calcula o pagamento da semana. Esta função aceitará dois argumentos, as horas trabalhadas durante a semana e o valor-hora. A função vai verificar se os dois parâmetros atendem aos critérios de validação. A função precisará calcular o pagamento regular, que é quando as horas é menor ou igual a 40, e o pagamento de horas extras, quando as horas trabalhadas é maior que 40 por semana.

Criaremos dois valores de `error` usando `errors.New()`. O único valor de erro será usado quando houver uma valor-hora inválido. Uma valor-hora inválido em nosso aplicativo é quando é menor que 10 ou maior que 75. O segundo valor de error será quando as horas por semana não estiverem entre 0 e 80.

```go
package main

import (
	"errors"
	"fmt"
)

// Now we have declared our error variables using errors.New().
// We use idiomatic Go for the variable name, starting it with
// Err and camel casing. Our error string is in lowercase with no punctuation:
var (
	ErrHourlyRate  = errors.New("invalid hourly rate")
	ErrHoursWorked = errors.New("invalid hours worked per week")
)

func main() {
	pay, err := payDay(81, 50)
	if err != nil {
		fmt.Println(err)
	}

	pay, err = payDay(80, 5)
	if err != nil {
		fmt.Println(err)
	}

	pay, err = payDay(80, 50)
	if err != nil {
		fmt.Println(err)
	}

	fmt.Println(pay)
}

func payDay(hoursWorked, hourlyRate int) (int, error) {
	if hourlyRate < 10 || hourlyRate > 75 {
		return 0, ErrHourlyRate
	}

	if hoursWorked < 0 || hoursWorked > 80 {
		return 0, ErrHoursWorked
	}

	if hoursWorked > 40 {
		hoursOver := hoursWorked - 40
		overTime := hoursOver * 2
		regularPay := hoursWorked * hourlyRate
		return regularPay + overTime, nil
	}

	return hoursWorked * hourlyRate, nil
}
```

Ao executar este código, você obterá a saída:

```
invalid hours worked per week
invalid hourly rate
4080
```

Neste exemplo, vimos como criar mensagens de `error` personalizadas que podem ser usadas para determinar facilmente por que os dados foram considerados inválidos. Também mostramos como retornar vários valores de uma função e verificar se há erros na função.

## Use Strings Para Erros Simples

A biblioteca padrão de *Go* fornece duas maneiras de criar um erro a partir de uma `string`. A primeira é a função `errors.New`. Ele pega uma string e retorna um `error`. Essa string é retornada quando você chama o método `Error` na instância de `error` retornada. Se você passar um `error` para `fmt.Println`, ele chamará o método `Error` automaticamente:

```go
func doubleEven(i int) (int, error) {
	if i % 2 != 0 {
		return 0, errors.New("only even numbers are processed")
	}

	return i * 2, nil
}
```

A segunda maneira é usar a função `fmt.Errorf`. Esta função permite que você use todos os verbos de formatação para `fmt.Printf` para criar um `error`. Da mesma forma que `errors.New`, essa string é retornada quando você chama o método `Error` na instância de `error` retornada:

```go
func doubleEven(i int) (int, error) {
	if i % 2 != 0 {
		return 0, fmt.Errorf("%d isn't an even number", i)
	}

	return i * 2, nil
}
```

## Erros são Valores

Como o `error` é uma interface, você pode definir seus próprios *erros* que incluem informações adicionais para *logging* ou *error handling*. Por exemplo, você pode incluir um código de status como parte do erro para indicar o tipo de erro que deve ser relatado ao usuário. Isso permite evitar comparações de strings (cujo texto pode mudar) para determinar as causas dos erros. Vamos ver como isso funciona. Primeiro, defina sua própria enumeração para representar os códigos de status:

```go
type Status int

const (
	InvalidLogin Status = iota + 1
	NotFound
)
```

Em seguida, defina um `StatusErr` para manter este valor:

```go
type StatusErr struct {
	Status Status
	Message string
}

func (se StatusErr) Error() string {
	return se.Message
}
```

Agora podemos usar `StatusErr` para fornecer mais detalhes sobre o que deu errado:

```go
func LoginAndGetData(uid, pwd, file string) ([]byte, error) {
	err := login(uid, pwd)
	if err != nil {
		return nil, StatusErr {
			Status: InvalidLogin,
			Message: fmt.Sprintf("invalid credentials for user %s", uid),
		}
	}

	data, err := getData(file)
	if err != nil {
		return nil, StatusErr {
			Status: NotFound,
			Message: fmt.Sprintf("file %s not found", file),
		}
	}

	return data, nil
}
```

Mesmo quando você define seus próprios tipos de erro personalizados, sempre use o `error` como o tipo de retorno para o *resultado do erro*. Isso permite que você retorne diferentes tipos de erros de sua função e permite que os chamadores de sua função escolham não depender do tipo de erro específico.

*Se você estiver usando seu próprio tipo de `error`, certifique-se de não retornar uma instância não inicializada*. Isso significa que você não deve declarar uma variável como o tipo de seu erro personalizado e, em seguida, retornar essa variável. Vamos ver o que acontece se você fizer isso.

```go
package main

import (
	"fmt"
)

type Status int

const (
	InvalidLogin Status = iota + 1
	NotFound
)

type StatusErr struct {
	Status  Status
	Message string
}

func (se StatusErr) Error() string {
	return se.Message
}

func GenerateError(flag bool) error {
	var genErr StatusErr
	if flag {
		genErr = StatusErr{
			Status: NotFound,
		}
	}

	return genErr
}

func main() {
	err := GenerateError(true)
	fmt.Println(err != nil)
	err = GenerateError(false)
	fmt.Println(err != nil)
}
```

A execução deste programa produz a seguinte saída:

```
true
true
```

Esse não é um problema de tipo de ponteiro versus tipo de valor; se declararmos `genErr` como sendo do tipo `*StatusErr`, veremos a mesma saída. A razão pela qual `err` é não-nil é que `error` é uma *interface*. **Para que uma interface seja considerada `nil`, o tipo e o valor subjacente devem ser `nil`**. Se `genErr` é ou não um ponteiro, a parte do tipo subjacente da interface não é `nil`.

Existem duas maneiras de corrigir isso. A abordagem mais comum é retornar explicitamente `nil` para o valor de `error` quando uma função for concluída com sucesso:

```go
func GenerateError(flag bool) error {
	if flag {
		return StatusErr {
			Status: NotFound,
		}
	}

	return nil
}
```

Isso tem a vantagem de não exigir que você leia o código para se certificar de que a variável de `error` na instrução de `return` está definida corretamente.

Outra abordagem é garantir que qualquer variável local que contenha um `error` seja do tipo `error`:

```go
func GenerateError(flag bool) error {
	var genErr error

	if flag {
		genErr = StatusErr {
			Status: NotFound,
		}
	}

	return genErr
}
```

Não use uma *type assertion* ou uma *type switch* para acessar os campos e métodos de um erro personalizado. Em vez disso, use `errors.As`, que veremos em "`Is` e `As`".

## Wrapping Errors

Quando um `error` é retornado por meio de seu código, você geralmente deseja adicionar contexto adicional a ele. Este contexto pode ser o nome da função que recebeu o `error` ou a operação que estava tentando realizar. Quando você preserva um erro enquanto adiciona informações adicionais, isso é chamado de *wrapping* do `error`. Quando você tem uma série de erros agrupados, é chamada de *error chain*.

Há uma função na biblioteca padrão *Go* que envolve os erros, e já vimos isso. A função `fmt.Errorf` possui um verbo especial, `%w`. Use isso para criar um `error` cuja string formatada inclui a string formatada de outro erro e que também contém o erro original. A convenção é escrever `: %w` no final do `error` para string e fazer com que o erro seja empacotado o último parâmetro passado para `fmt.Errorf`.

A biblioteca padrão também fornece uma função para desempacotar erros, a função `Unwrap` no pacote de `errors`. Você passa um `error` e ele retorna o `error` empacotado, se houver. Se não houver, ele retorna `nil`. Aqui está um programa que demonstra o empacotamento com `fmt.Errorf` e o desempacotamento com `errors.Unwrap`.

```go
package main

import (
	"errors"
	"fmt"
	"os"
)

func fileChecker(name string) error {
	f, err := os.Open(name)
	if err != nil {
		return fmt.Errorf("in fileChecker: %w", err)
	}
	f.Close()
	return nil
}

func main() {
	err := fileChecker("not_here.txt")
	if err != nil {
		fmt.Println(err)
		if wrappedErr := errors.Unwrap(err); wrappedErr != nil {
			fmt.Println(wrappedErr)
		}
	}
}
```

Ao executar este programa, você vê a seguinte saída:

```
in fileChecker: open not_here.txt: no such file or directory
open not_here.txt: no such file or directory
```

*DICA: Você geralmente não chama `errors.Unwrap` diretamente. Em vez disso, você usa `errors.Is` e `errors.As` para localizar um error específico empacotado.*

Se você quiser envolver um erro com seu tipo de erro personalizado, seu tipo de erro precisa implementar o método `Unwrap`. Este método não aceita parâmetros e retorna um `error`. Aqui está uma atualização do erro que definimos anteriormente para demonstrar como isso funciona:

```go
type StatusErr struct {
	Status Status
	Message string
	Err error
}

func (se StatusErr) Error() string {
	return se.Message
}

func (se StatusErr) Unwrap() error {
	return se.Err
}
```

Agora podemos usar `StatusErr` para envolver os erros subjacentes:

```go
func LoginAndGetData(uid, pwd, file string) ([]byte, error) {
	err := login(uid,pwd)
	if err != nil {
		return nil, StatusErr {
			Status: InvalidLogin,
			Message: fmt.Sprintf("invalid credentials for user %s", uid),
			Err: err,
		}
	}

	data, err := getData(file)
	if err != nil {
		return nil, StatusErr {
			Status: NotFound,
			Message: fmt.Sprintf("file %s not found",file),
			Err: err,
		}
	}

	return data, nil
}
```

Nem todos os erros precisam ser agrupados/wrapped. Uma biblioteca pode retornar um erro que significa que o processamento não pode continuar, mas a mensagem de erro contém detalhes de implementação que não são necessários em outras partes do seu programa. Nesta situação é perfeitamente aceitável para criar um erro totalmente novo e retorná-lo em seu lugar. Compreenda a situação e determine o que precisa ser devolvido.

Se você deseja criar um novo erro que contém a mensagem de outro erro, mas não deseja envolvê-lo/wrap, use `fmt.Errorf` para criar um erro, mas use o verbo `%v` em vez de `%w`:

```go
err := internalFunction()
if err != nil {
	return fmt.Errorf("internal failure: %v", err)
}
```

### Diretrizes ao trabalhar com Errors

As diretrizes são apenas para orientação. Elas não são gravados na pedra. Isso significa que na maioria das vezes você deve seguir as diretrizes; no entanto, pode haver exceções.

- Ao declarar nosso próprio tipo de `error`, a variável precisa começar com `Err`. Ele também deve seguir a convenção da nomenclatura de camel case.
	```go
	var ErrExampleNotAllowd= errors.New("error example text")
	```

- A string de `error` deve começar com letras minúsculas e não terminar com pontuação. Um dos motivos para essa diretriz é que o erro pode ser retornado e concatenado com outras informações relevantes ao erro.

## `Is` e `As`

Encapsular/Wrapping os erros é uma maneira útil de obter informações adicionais sobre um erro, mas apresenta problemas. Se um *erro sentinel* for encapsulado/wrapped, você não pode usar `==` para verificá-lo, nem pode usar uma *type assertion* ou *type switch* para corresponder a um erro personalizado encapsulado/wrapped. *Go* resolve esse problema com duas funções no pacote de `errors`, `Is` e `As`.

Para verificar se o erro retornado ou quaisquer erros que ele envolve correspondem a uma instância específica de *erro sentinel*, use `errors.Is`. Recebe dois parâmetros, o erro que está sendo verificado e a instância com a qual você está comparando. A função `errors.Is` retorna `true` se houver um erro na cadeia de erros que corresponda ao *erro sentinel*. Vamos escrever um pequeno programa para ver `errors.Is` em ação.

```go
package main

import (
	"errors"
	"fmt"
	"os"
)

func fileChecker(name string) error {
	f, err := os.Open(name)
	if err != nil {
		return fmt.Errorf("in fileChecker: %w", err)
	}
	f.Close()
	return nil
}

func main() {
	err := fileChecker("not_here.txt")
	if err != nil {
		if errors.Is(err, os.ErrNotExist) {
			fmt.Println("That file doesn't exist")
		}
	}
}
```

A execução deste programa produz a saída:

```
That file doesn't exist
```

Por padrão, `errors.Is` usa `==` para comparar cada erro empacotado/wrapped com o erro especificado. Se isso não funcionar para um tipo de erro que você definir (por exemplo, se o seu erro for um tipo não comparável), implemente o método `Is` em seu `error`:

```go
type MyErr struct {
	Codes []int
}

func (me MyErr) Error() string {
	return fmt.Sprintf("codes: %v", me.Codes)
}

func (me MyErr) Is(target error) bool {
	if me2, ok := target.(MyErr); ok {
		return reflect.DeepEqual(me, me2)
	}

	return false
}
```

Outro uso para definir seu próprio método `Is` é permitir comparações com *erros* que não são instâncias idênticas. Você pode querer padronizar a correspondência de seus *erros*, especificando uma instância de filtro que corresponda aos *erros* que possuem alguns dos mesmos campos. Vamos definir um novo tipo de erro, `ResourceErr`:

```go
type ResourceErr struct {
	Resource string
	Code int
}

func (re ResourceErr) Error() string {
	return fmt.Sprintf("%s: %d", re.Resource, re.Code)
}
```

Se quisermos que duas instâncias `ResourceErr` correspondam quando um dos campos for definido, podemos fazer isso escrevendo um método `Is` personalizado:

```go
func (re ResourceErr) Is(target error) bool {
	if other, ok := target.(ResourceErr); ok {
		ignoreResource := other.Resource == ""
		ignoreCode := other.Code == 0
		matchResource := other.Resource == re.Resource
		matchCode := other.Code == re.Code

		return  matchResource && matchCode ||
				matchResource && ignoreCode ||
				ignoreResource && matchCode
	}

	return false
}
```

Agora podemos encontrar, por exemplo, todos os erros que se referem ao banco de dados, não importa o código:

```go
package main

import (
	"errors"
	"fmt"
)

type ResourceErr struct {
	Resource string
	Code     int
}

func (re ResourceErr) Error() string {
	return fmt.Sprintf("%s: %d", re.Resource, re.Code)
}

func (re ResourceErr) Is(target error) bool {
	if other, ok := target.(ResourceErr); ok {
		ignoreResource := other.Resource == ""
		ignoreCode := other.Code == 0
		matchResource := other.Resource == re.Resource
		matchCode := other.Code == re.Code

		return matchResource && matchCode ||
			matchResource && ignoreCode ||
			ignoreResource && matchCode
	}

	return false
}

func main() {
	err := ResourceErr{
		Resource: "Database",
		Code:     123,
	}

	err2 := ResourceErr{
		Resource: "Network",
		Code:     456,
	}

	if errors.Is(err, ResourceErr{Resource: "Database"}) {
		fmt.Println("The database is broken:", err)
	}

	if errors.Is(err2, ResourceErr{Resource: "Database"}) {
		fmt.Println("The database is broken:", err2)
	}

	if errors.Is(err, ResourceErr{Code: 123}) {
		fmt.Println("Code 123 triggered:", err)
	}

	if errors.Is(err2, ResourceErr{Code: 123}) {
		fmt.Println("Code 123 triggered:", err2)
	}

	if errors.Is(err, ResourceErr{Resource: "Database", Code: 123}) {
		fmt.Println("Database is broken and Code 123 triggered:", err)
	}

	if errors.Is(err, ResourceErr{Resource: "Network", Code: 123}) {
		fmt.Println("Network is broken and Code 123 triggered:", err)
	}
}
```

A execução do código produz a seguinte saída:

```
The database is broken: Database: 123
Code 123 triggered: Database: 123
Database is broken and Code 123 triggered: Database: 123
```

A função `errors.As` permite que você verifique se um erro retornado (ou qualquer erro que wraps/envolva) corresponde a um tipo específico. Leva dois parâmetros. O primeiro é o erro que está sendo examinado e o segundo é um ponteiro para uma variável do tipo que você está procurando. Se a função retornar verdadeiro, foi encontrado um erro na cadeia de erro correspondente, e esse erro correspondente é atribuído ao segundo parâmetro. Se a função retornar falso, nenhuma correspondência foi encontrada na cadeia de erro. Vamos experimentar com `MyErr`:

```go
err := AFunctionThatReturnsAnError()
var myErr MyErr
if errors.As(err, &myErr) {
	fmt.Println(myErr.Code)
}
```

Observe que você usa `var` para declarar uma variável de um tipo específico definido com *zero value*. Em seguida, você passa um ponteiro para essa variável em `errors.As`.

Você não precisa passar um ponteiro para uma variável de um tipo de erro como o segundo parâmetro para `errors.As`. Você pode passar um ponteiro para uma interface para encontrar um erro que corresponda à interface:

```go
err := AFunctionThatReturnsAnError()

var coder interface {
	Code() int
}

if errors.As(err, &coder) {
	fmt.Println(coder.Code())
}
```

Estamos usando uma interface anônima aqui, mas qualquer tipo de interface é aceitável.

Assim como você pode substituir a comparação padrão de `errors.Is` com um método `Is`, você pode substituir o padrão `errors.As` comparação com um método `As` em seu erro. A implementação do método `As` é não-trivial e exige reflexão. Você só deve fazer isso em circunstâncias incomuns, como quando deseja corresponder um erro de um tipo e retornar outro.

*DICA: Use `errors.Is` quando estiver procurando uma instância específica ou valores específicos. Use `errors.As` quando estiver procurando por um tipo específico.*

## Empacotando erros com `defer`

Às vezes, acontece de envolver vários erros na mesma mensagem:

```go
func DoSomeThings(val1 int, val2 string) (string, error) {
	val3, err := doThing1(val1)

	if err != nil {
		return "", fmt.Errorf("in DoSomeThings: %w", err)
	}

	val4, err := doThing2(val2)
	if err != nil {
		return "", fmt.Errorf("in DoSomeThings: %w", err)
	}

	result, err := doThing3(val3, val4)
	if err != nil {
		return "", fmt.Errorf("in DoSomeThings: %w", err)
	}

	return result, nil
}
```

Podemos simplificar esse código usando `defer`:

```go
func DoSomeThings(val1 int, val2 string) (_ string, err error) {
	defer func() {
		if err != nil {
			err = fmt.Errorf("in DoSomeThings: %w", err)
		}
	}()

	val3, err := doThing1(val1)
	if err != nil {
		return "", err
	}

	val4, err := doThing2(val2)
	if err != nil {
		return "", err
	}

	return doThing3(val3, val4)
}
```

Temos que nomear nossos valores de retorno para que possamos nos referir a `err` na função *deferred*. Se você nomear um único valor de retorno, deve nomear todos eles, então usamos um underscore para o valor de retorno da string, uma vez que não atribuímos explicitamente a ele.

No closure de `defer`, verificamos se um erro foi retornado. Nesse caso, reatribuímos o erro a um novo erro que envolve o erro original com uma mensagem que indica qual função detectou o erro.

Esse padrão funciona bem quando você envolve todos os erros com a mesma mensagem. Se você quiser personalizar o wrapping erro para fornecer mais contexto sobre o que causou o erro, coloque a mensagem específica e a geral em cada `fmt.Errorf`.

## Panic

Diversas linguagens usam `exceptions` para lidar com erros. No entanto, ***Go* não usa `exceptions`, ele usa algo chamado `panic`**. `panic` é uma função que faz com que o programa crash (trave). *Ele interrompe a execução normal da Goroutine*.

Em *Go*, *panic* não é a norma, ao contrário de outras linguagens em que uma exception é a norma. Um sinal de *panic* indica que algo anormal está ocorrendo em seu código. Normalmente, quando o *panic* é iniciado pelo runtime ou pelo desenvolvedor, é para proteger a integridade do programa.

*Errors* e *panics* diferem em seus propósitos e em como são tratados pelo runtime do *Go*. Um erro no *Go* indica que algo inesperado ocorreu, mas não afetará negativamente a integridade do programa. Go espera que o desenvolvedor lide com o erro de maneira adequada. A função ou outros programas normalmente não travarão se você não resolver o erro. No entanto, os *panics* diferem a esse respeito. Quando o *panic* ocorre, ele acaba travando o sistema, a menos que haja manipuladores para lidar com o *panic*. Se não houver manipuladores para o *panic*, ele irá subir na stack e travar o programa.

Um exemplo que examinaremos posteriormente é tentar acessar um índice de uma coleção que não existe. Se Go não entrar em *panic* nesse caso, isso poderá ter um impacto adverso na integridade do programa, como outras partes do programa tentando armazenar ou recuperar dados que não estão na coleção.

*`panics`* podem ser iniciados pelo desenvolvedor e podem ser causados ​​durante a execução do programa por erros de runtime. Uma função de `panic()` aceita uma interface vazia. Isso significa que pode aceitar qualquer coisa como argumento. No entanto, na maioria dos casos, você deve passar um tipo de `error` para a função `panic()`. É mais intuitivo para o tratamento ter alguns detalhes sobre o que causou o *panic*. Passar um `error` para a função de *panic* também é idiomático no *Go*. Também veremos como a recuperação de um *panic* que possui um tipo de `error` passado nos dá algumas opções diferentes ao lidar com isso. Quando ocorre o *panic*, geralmente ocorre estas etapas:

- *The execution is stopped;*
- *Any deferred functions in the panicking function will be called;*
- *Any deferred functions in the stack of the panicking function will be called;*
- *It will continue all the way up the stack until it reaches `main()`;*
- *Statements after the panicking function will not execute;*
- *The program then crashes;*

É assim que *`panic`* funciona:

<img src="{{ mix('/images/articles/Go/The working of panic.png') }}" alt="The working of panic">

O diagrama anterior ilustra o código da função `main` que chama a função `a()`. A função `a()` então chama a função `b()`. Dentro da função `b()`, ocorre um *`panic`*. A função `panic()` não é controlada por nenhum código anterior (função `a()` ou a função `main()`), então o programa irá travar a função `main()`.

Aqui está um exemplo de *`panic`* que ocorre no *Go*. Tente determinar por que esse programa entra em *`panic`*.

```go
func main() {
	nums := []int{1, 2, 3}

	for i := 0; i <= 10; i++ {
		fmt.Println(nums[i])
	}
}
```

Um exemplo de *panic* é o seguinte:

```
1
2
3
panic: runtime error: index out of range [3] with length 3

goroutine 1 [running]:
main.main()
	/tmp/sandbox2755820320/prog.go:11 +0xc5
```

O erro de *`panic`* em runtime comum que você encontrará durante o desenvolvimento. É um error de `index out of range`. *Go* gerou esse *panic* porque estamos tentando iterar em uma *`slice`* mais vezes do que o número de elementos. *Go* viu que esse é um motivo para *`panic`*, porque coloca o programa em uma condição anormal.

Aqui está um trecho de código que demonstra os fundamentos do uso de um *`panic`* e uma instrução `defer` funcionando juntos:

```go
package main

import (
	"errors"
	"fmt"
)

func main() {
	test()

	fmt.Println("This line will not get printed")
}

func test() {
	n := func() {
		fmt.Println("Defer in test")
	}

	defer n()

	message("good-bye")
}

func message(msg string) {
	f := func() {
		fmt.Println("Defer in message func")
	}

	defer f()

	if msg == "good-bye" {
		panic(errors.New("something went wrong"))
	}
}
```

A saída do exemplo é a seguinte:

```
Defer in message func
Defer in test
panic: something went wrong

goroutine 1 [running]:
main.message({0x49b165?, 0xc0000061c0?})
	/tmp/sandbox1289070679/prog.go:32 +0x8e
main.test()
	/tmp/sandbox1289070679/prog.go:21 +0x3c
main.main()
	/tmp/sandbox1289070679/prog.go:9 +0x13
```

Vamos entender o código em partes:

- Começaremos examinando o código da função `message()`, pois é aí que começa o `panic`. Quando o pânico ocorre, ele executa a instrução `defer` dentro da função que entrará em pánico, `message()`.
- A função adiada, `func f()`, é executada na função `message()`.
- Subindo nas chamadas da stack, a próxima função é a função `test()`, e sua função adiada, `n()`, será executada.
- Finalmente, chegamos à função `main()` onde a execução é interrompida pela função que gerou o pânico (`message()`). A instrução `fmt.Println` em `main()` não é executada.

Seguindo com outro exemplo. A função `payDay()` retornará o valor do pagamento e nenhum erro. Quando os argumentos fornecidos para a função são inválidos, em vez de retornar um erro, a função entrará em pânico. Isso fará com que o programa pare imediatamente e, portanto, não processará o cheque de pagamento.

```go
package main

import (
	"errors"
	"fmt"
)

var (
	ErrHourlyRate  = errors.New("invalid hourly rate")
	ErrHoursWorked = errors.New("invalid hours worked per week")
)

func main() {
	pay := payDay(81, 50)
	fmt.Println(pay)
}

func payDay(hoursWorked, hourlyRate int) int {
	report := func() {
		fmt.Printf("HoursWorked: %d\nHourldyRate: %d\n", hoursWorked, hourlyRate)
	}

	defer report()

	if hourlyRate < 10 || hourlyRate > 75 {
		panic(ErrHourlyRate)
	}

	if hoursWorked < 0 || hoursWorked > 80 {
		panic(ErrHoursWorked)
	}

	return hoursWorked * hourlyRate
}
```

A saída é a seguinte:

```
HoursWorked: 81
HourldyRate: 50
panic: invalid hours worked per week

goroutine 1 [running]:
main.payDay(0x0?, 0x60?)
	/tmp/sandbox1585701503/prog.go:30 +0xae
main.main()
	/tmp/sandbox1585701503/prog.go:14 +0x1d
```

No exemplo do código acima, vimos como executar `panic` e passar um `error` para a função `panic()`. Isso ajuda o usuário a compreender a causa do pânico. No próximo tópico, veremos como recuperar o controle do programa após a ocorrência de um pânico usando **Recover**.

## Recover

Go nos dá a capacidade de recuperar o controle depois que o `panic` ocorre. `recover` é uma função usada para recuperar o controle de um *Goroutine* que ocorreu um `panic`.

A assinatura da função `recover()` é a seguinte:

```go
func recover() interface{}
```

A função `recover()` não aceita argumentos e retorna uma interface vazia `interface{}`. Uma interface vazia `interface{}` indica que qualquer tipo pode ser retornado. A função `recover()` retornará o valor enviado para a função `panic()`.

A função `recover()` só é útil dentro de uma função adiada (*deferred*). Como você deve se lembrar, uma função adiada (*deferred*) é executada antes que a função a qual está definida termine. Executar uma chamada para a função `recover()` dentro de uma função adiada (*deferred*) interrompe o `panic`, restaurando a execução normal. **Se a função `recover()` for chamada fora de uma função adiada, ela não interromperá o `panic`**.

O diagrama a seguir mostra as etapas que um programa executaria ao usar `panic()`, `recover()` e uma função `defer()`:

<img src="{{ mix('/images/articles/Go/Recover function flow.png') }}" alt="Recover function flow">

As etapas seguidas no diagrama podem ser explicadas da seguinte forma:

- *A função `main()` chama `func a()`;*
- *`func a()` chama `func b()`;*
- *Dentro de `func b()`, existe um `panic`;*
- *A função `panic()` é tratada por uma função adiada que está usando a função `recover()`;*
- *A função adiada é a última função a ser executada dentro de `func b()`;*
- *A função adiada chama a função `recover()`;*
- *A chamada para `recover()` causa o fluxo normal de volta para o chamador, `func a()`;*
- *O fluxo normal continua e o controle finalmente está de volta para a função `main()`;*

O seguinte código imita o comportamento do diagrama anterior:

```go
package main

import (
	"errors"
	"fmt"
)

func main() {
	a()

	fmt.Println("This line will now get printed from main() function")
}

func a() {
	b("good-bye")

	fmt.Println("Back in function a()")
}

func b(msg string) {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("error in func b()", r)
		}
	}()

	if msg == "good-bye" {
		panic(errors.New("something went wrong"))
	}

	fmt.Print(msg)
}
```

Ao executar este código, você obterá a saída:

```
error in func b() something went wrong
Back in function a()
This line will now get printed from main() function
```

**Code Synopsis**

- A função `main()` chama a função `a()`. A função `a()` chama a função `b()`.

- A função `b()` aceita um tipo de string e o atribui à variável `msg`. Se `msg` for avaliada como verdadeira na instrução `if`, ocorrerá um `panic`.

- O argumento para `panic` é um novo `error` criado pela função `errors.New()`:
	```go
	if msg == "good-bye" {
		panic(errors.New("something went wrong"))
	}
	```

Assim que *panic* ocorrer, a próxima chamada será para a função adiada (*deferred*).

A função adiada (*deferred*) usa a função `recover()`. O valor de `panic` é devolvido para `recover`; neste caso, o valor de `r` é um tipo de `error`. Em seguida, a função imprime alguns detalhes:

```go
defer func() {
	if r := recover(); r != nil {
		fmt.Println("error in func b()", r)
	}
}()
```

- O fluxo de controle volta para a função `a()`. A função `a()` imprime alguns detalhes.

- Em seguida, o controle volta para a função `main()` e imprime alguns detalhes e termina:
	```
	error in func b() something went wrong
	Back in function a()
	This line will now get printed from main() function
	```

No exemplo a seguir, vamos aprimorar a função `payDay()` para se recuperar de um pânico. Quando `payDay()` entrar em pânico, inspecionaremos o `error` desse pânico. Então, dependendo do `error`, imprimiremos uma mensagem informativa ao usuário.

```go
package main

import (
	"errors"
	"fmt"
)

var (
	ErrHourlyRate  = errors.New("invalid hourly rate")
	ErrHoursWorked = errors.New("invalid hours worked per week")
)

func main() {
	pay := payDay(100, 25)
	fmt.Println(pay)

	pay = payDay(100, 200)
	fmt.Println(pay)

	pay = payDay(60, 25)
	fmt.Println(pay)
}

func payDay(hoursWorked, hourlyRate int) int {
	defer func() {
		if r := recover(); r != nil {
			if r == ErrHourlyRate {
				fmt.Printf("hourly rate: %d\nerr: %v\n\n", hourlyRate, r)
			}

			if r == ErrHoursWorked {
				fmt.Printf("hours worked: %d\nerr: %v\n\n", hoursWorked, r)
			}
		}

		fmt.Printf("Pay was calculated based on:\nhours worked: %d\nhourly Rate: %d\n", hoursWorked, hourlyRate)
	}()

	if hourlyRate < 10 || hourlyRate > 75 {
		panic(ErrHourlyRate)
	}

	if hoursWorked < 0 || hoursWorked > 80 {
		panic(ErrHoursWorked)
	}

	if hoursWorked > 40 {
		hoursOver := hoursWorked - 40
		overTime := hoursOver * 2
		regularPay := hoursWorked * hourlyRate

		return regularPay + overTime
	}

	return hoursWorked * hourlyRate
}
```

A saída esperada é a seguinte:

```
hours worked: 100
err: invalid hours worked per week

Pay was calculated based on:
hours worked: 100
hourly Rate: 25
0
hourly rate: 200
err: invalid hourly rate

Pay was calculated based on:
hours worked: 100
hourly Rate: 200
0
Pay was calculated based on:
hours worked: 60
hourly Rate: 25
1540
```

Nos exemplo anterior, vimos a progressão da criação de um erro personalizado e do retorno desse erro. A partir disso, fomos capazes de travar programas quando necessário usando `panic`. No exemplo anterior, demonstramos a capacidade de se recuperar de pânicos e exibir mensagens de erro com base no tipo de erro que foi passado para a função `panic()`.
