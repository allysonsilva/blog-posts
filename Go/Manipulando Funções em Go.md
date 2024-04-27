---
id: 9be85450-cc6a-4e27-bd10-a2cf9882d5f8
title: Manipulando Funções em Go
summary: "Saiba como criar e manipular funções em Go."
---

> As funções são uma parte essencial de muitas linguagens e *Go* não é exceção. Uma função é uma seção do código que foi declarada para realizar uma determinada tarefa. As funções em *Go* podem ter zero ou mais entradas e saídas. Um recurso que diferencia o *Go* de outras linguagens de programação são os vários valores de retorno; a maioria das linguagens de programação é limitada a um valor de retorno.

As funções em *Go* são consideradas cidadãos de primeira classe (**first-class**) e funções de ordem superior (**higher-order**). Cidadãos de primeira classe são as funções atribuídas a uma variável. Funções de ordem superior são funções que podem ter uma função como argumento e ter uma função como um tipo de retorno para uma outra função. Os recursos das funções em *Go* permitem que sejam usados ​​em vários segmentos das seguintes maneiras:

- *Funções a serem passadas como um argumento para outra função;*
- *Retorna uma função como um valor de uma função;*
- *Funciona como um tipo;*
- *Closures;*
- *Funções anônimas (Anonymous functions);*
- *Funções atribuídas a uma variável;*

## Partes da Função

Veremos os diferentes componentes envolvidos na **definição de uma função**. A seguir está o layout típico de uma função:

![Different parts of a function](/images/articles/Go/Different%20parts%20of%20a%20function.png?id=6ffb8f8d0f616ef10fde1762f7317f8b "Different parts of a function")

As diferentes partes de uma função são:

- **`func`**: Em *Go*, a declaração da função começa com a palavra-chave `func`.

- ***Identifier***: Também conhecido como *nome da função*. É idiomático em *Go* usar `camelCase` para o *nome da função*. `camelCase` é a prática de ter a primeira letra do *nome da função* em minúscula e a primeira letra de cada palavra seguinte em maiúscula. Exemplos de nomes de funções que seguem esta convenção incluem `calculateTax`, `totalSum` e `fetchId`. O identificador deve ser algo descritivo que torne o código fácil de ler e o propósito da função fácil de entender. O identificador não é obrigatório. Você pode ter uma função sem nome; isso é conhecido como *anonymous function*.

- ***Parameter list***: Os parâmetros *são valores de entrada para uma função*. Um parâmetro são dados exigidos pela função para ajudar a resolver a tarefa da função. Os parâmetros são definidos da seguinte forma: `(nome tipo)`. Um exemplo de lista de parâmetros pode ser (`firstName string, lastName string`). Os parâmetros são variáveis ​​locais para a função. É possível não ter parâmetros para uma função. *Uma função pode ter zero ou mais parâmetros*. Quando dois ou mais parâmetros têm o mesmo tipo, você pode usar o que é chamado de notação de parâmetro abreviada (*shorthand parameter notation*). Isso remove a especificação do mesmo tipo para cada parâmetro. Por exemplo, se seus parâmetros forem (`firstName string, lastName string`), eles podem ser encurtados (*shortened*) para (`firstName, lastName string`). Isso reduz o detalhamento das entradas dos parâmetros e aumenta a legibilidade da lista de parâmetros da função.

- ***Return types***: Os *tipos de retorno* são uma lista de tipos de dados, como booleano, string, mapa ou outra função que pode ser retornada. No contexto da declaração de uma função, nos referimos a esses tipos como *tipos de retorno*. No entanto, no contexto da chamada de uma função, eles são chamados de *valores de retorno*. Os *tipos de retorno* são a saída da função. Frequentemente, eles são o resultado dos argumentos fornecidos para a função. A maioria das linguagens de programação retorna um único tipo; em *Go*, você pode retornar vários tipos.

- ***Function signature***: Uma *function signature* é um termo que faz referência aos parâmetros de entrada combinados com os tipos de retorno. Ambas as unidades constituem uma *function signature*. Frequentemente, ao definir a *function signature* quando ela está sendo usada por outras pessoas, você deseja se esforçar para não fazer alterações nela, pois isso pode impactar negativamente seu código e o código de outras pessoas.

## Declarando e Chamando Funções

Cada aplicação *Go* tem uma função chamada `main` que é chamada quando a aplicação é executada, e estamos chamando a função `fmt.Println` para exibir a saída na tela. Como **a função `main` não aceitam nenhum parâmetro nem retornam nenhum valor**, vamos ver como fica quando temos uma função que faz algo:

```go
func div(numerator int, denominator int) int {
	if denominator == 0 {
		return 0
	}

	return numerator / denominator
}
```

Assim como outras linguagens, *Go* tem uma palavra-chave `return` para retornar valores de uma função. Se uma função retornar um valor, você deve fornecer um `return`. Se uma função não retornar nada, uma instrução de `return` não será necessária no final da função. *A palavra-chave `return` sem nenhum valor, apenas a palavra-chave, só é necessária em uma função que não retorna nada ou se estiver saindo da função antes da última linha.*

**A função `main` não possui parâmetros ou valores de retorno. Quando uma função não tem parâmetros, use parênteses vazios (`()`). Quando uma função não retorna nada, você não escreve nada antes da chave de abertura do corpo da função**:

```go
func main() {
	result := div(5, 2)

	fmt.Println(result)
}
```

*Dica: quando se tem vários parâmetros do mesmo tipo, pode escrever os parâmetros da seguinte forma: `func div(numerator, denominator int) int`*

### Simulando parâmetros nomeados e opcionais

Antes de chegarmos aos recursos de função exclusivos no *Go*, vamos mencionar dois que o *Go* não possui: **parâmetros nomeados** e **opcionais**. **Você deve fornecer todos os parâmetros para uma função**. Se você deseja *emular parâmetros nomeados e opcionais*, defina uma estrutura que tenha campos que correspondam aos parâmetros desejados e passe a estrutura para sua função. Aqui está um trecho de código que demonstra esse padrão:

```go
type MyFuncOpts struct {
	FirstName string
	LastName string
	Age int
}

func MyFunc(opts MyFuncOpts) error {
	// do something here
}

func main() {
	MyFunc(MyFuncOpts { LastName: "Patel", Age: 50, })
	MyFunc(MyFuncOpts { FirstName: "Joe", LastName: "Smith", })
}
```

*Na prática, não ter parâmetros nomeados e opcionais não é uma limitação. Uma função não deve ter mais do que alguns parâmetros e os parâmetros nomeados e opcionais são úteis principalmente quando uma função tem muitos parâmetros. Se você se encontrar nessa situação, é possível que sua função seja muito complicada.*

### Parâmetros e slices variadic

Uma *função variadic* é uma **função que aceita um número variável de valores de argumento**. É bom usar uma função variável quando o número de argumentos de um tipo especificado é desconhecido.

Os três pontos (`...`) na frente do tipo são chamados de *pack operator*. O *pack operator* é o que o torna uma *função variadic*.

*Variadic parameter deve ser o último (ou único) parâmetro na lista de parâmetros.*

Você indica com três pontos (`...`) antes do tipo. A variável criada na função é uma *`slice`* do tipo especificado.

Você deve usá-lo como qualquer outra *`slice`*. Vamos ver como eles funcionam escrevendo um programa que adiciona um número base a um número variável de parâmetros e retornando o resultado como uma *`slice`* de `int`.

```go
package main

import "fmt"

func addTo(base int, vals ...int) []int {
	out := make([]int, 0, len(vals))

	for _, v := range vals {
		out = append(out, base+v)
	}

	return out
}

func main() {
	fmt.Println(addTo(3))
	fmt.Println(addTo(3, 2))
	fmt.Println(addTo(3, 2, 4, 6, 8))

	a := []int{4, 3}

	fmt.Println(addTo(3, a...))
	fmt.Println(addTo(3, []int{1, 2, 3, 4, 5}...))
}
```

Como você pode ver, você pode fornecer quantos valores quiser para o parâmetro *variadic* ou nenhum valor. Como o parâmetro *variadic* é convertido em uma *`slice`*, você pode fornecer um outro *`slice`*. No entanto, você deve colocar três pontos (`...`) após a variável ou literal de *`slice`*. Se você não fizer isso, gerará um erro de compilação.

Ao construir e executar este programa, você obtém:

```
[]
[5]
[5 7 9 11]
[7 6]
[4 5 6 7 8]
```

As *funções variadic* ​​podem ter outros parâmetros. No entanto, **se sua função requer vários parâmetros, o parâmetro *variadic* deve ser o último parâmetro na função**. Além disso, **só pode haver uma variável *variadic* por função**. A função a seguir está incorreta e resultará em um erro no momento da compilação.

**Função incorreta:**

```go
func main() {
	nums(99, 100, "James")
}

func nums(i ...int, str person) {
	fmt.Println(str)
	fmt.Println(i)
}
```

```
./prog.go:12:11: syntax error: cannot use ... with non-final parameter i
```

**Função correta:**

```go
func main() {
	nums("Jonh", 99, 100)
}

func nums(str string, i ...int) {
	fmt.Println(str)
	fmt.Println(i)
}
```

```
Jonh
[99 100]
```

O tipo real de `Type` dentro da função é uma *`slice`*. A função pega os argumentos que estão sendo passados ​​e os converte em uma *`slice`*. Por exemplo, se o tipo de variável for `int`, uma vez que você estiver dentro da função, *Go* converterá essa variável `int` em uma *`slice`* de números inteiros:

![Conversion of a variadic int into a slice of integers](/images/articles/Go/Conversion%20of%20a%20variadic%20int%20into%20a%20slice%20of%20integers.png?id=29d5f96e343d6ce060db610e6bb54601 "Conversion of a variadic int into a slice of integers")


```go
package main

import "fmt"

func main() {
	nums(99, 100)
}

func nums(i ...int) {
	fmt.Println(i)
	fmt.Printf("%T\n", i)
	fmt.Printf("Len: %d\n", len(i))
	fmt.Printf("Cap: %d\n", cap(i))
}
```

A saída da *função variadic* é a seguinte:

```
[99 100]
[]int
Len: 2
Cap: 2
```

*Go* tem um mecanismo para passar uma *`slice`* para uma função *variadic*. Precisamos usar o **operador de unpack** (descompactação); são três pontos (`...`). Quando você chama uma função *variadic* e deseja passar uma *`slice`* como argumento para um parâmetro *variadic*, é necessário colocar os três pontos antes da variável:

```go
package main

import "fmt"

func main() {
	i := []int{5, 10, 15}
	nums(i...)
}

func nums(i ...int) {
	fmt.Println(i) // [5 10 15]
}
```

Os três pontos são colocados depois que da variável `i`. Isso permite que uma *`slice`* seja passada para a *função variadic*.

No exemplo de código abaixo, vamos somar um número variável de argumentos. Vamos passar os argumentos como uma lista de argumentos e como uma *`slice`*. O valor de retorno será um `int`, a soma dos valores que passamos para a função.

```go
package main

import (
	"fmt"
)

func main() {
	i := []int{5, 10, 15}
	fmt.Println(sum(5, 4))
	fmt.Println(sum(i...))
}

func sum(nums ...int) int {
	total := 0

	for _, num := range nums {
		total += num
	}

	return total
}
```

A saída esperada é a seguinte:

```
9
30
```

No código acima, vimos que, usando um parâmetro *variadic*, podemos aceitar um número desconhecido de argumentos. Nossa função nos permite somar qualquer número de inteiros. **Podemos ver que parâmetros *variadic* podem ser utilizados para resolver problemas específicos em que o número de valores do mesmo tipo transmitidos como um argumento é desconhecido**.

### Múltiplos valores de retorno

As funções geralmente aceitam entradas, realizam alguma ação nessas entradas e, em seguida, retornam os resultados dessas entradas. A maioria das linguagens de programação retornam apenas um valor. ***Go* permite que você retorne vários valores de uma função.**

A primeira diferença entre *Go* e outras linguagens é que ***Go* permite vários valores de retorno**. Vamos adicionar um pequeno recurso ao programa de divisão. Vamos devolver o dividendo e o restante do valor na função:

```go
func divAndRemainder(numerator int, denominator int) (int, int, error) {
	if denominator == 0 {
		return 0, 0, errors.New("cannot divide by zero")
	}

	return numerator / denominator, numerator % denominator, nil
}
```

Existem algumas alterações para oferecer suporte a vários valores de retorno. Quando uma função *Go* retorna vários valores, **os tipos dos valores de retorno são listados entre parênteses, separados por vírgulas**. Além disso, se uma função retornar vários valores, você deve retornar todos eles, separados por vírgulas. Não coloque parênteses em torno dos valores retornados; isso é um erro de compilação.

Executando a função:

```go
func main() {
	result, remainder, err := divAndRemainder(5, 2)

	if err != nil {
		fmt.Println(err)
		os.Exit(1)
	}

	fmt.Println(result, remainder) // 2 1
}
```

No lado direito de `:=`, chamamos a função `divAndRemainder` com os valores `5` e `2`. No lado esquerdo, atribuímos os valores retornados às variáveis `​​result`, `remainder` e `err`. Verificamos se houve um erro comparando `err` com `nil`.

### Ignorando valores retornados (*naked returns*)

Mas e se você chamar uma função e não quiser usar todos os valores retornados? ***Go* não permite variáveis ​​não utilizadas**. Se uma função retornar vários valores, mas você não precisar ler um ou mais dos valores, atribua os valores não usados ​​a `_`. Por exemplo, se não fosse ler o `remainder`, escreveríamos a atribuição como `result, _ := divAndRemainder(5, 2)`

Surpreendentemente, *Go* permite que você ignore implicitamente todos os valores de retorno de uma função. Você pode escrever `_ := divAndRemainder(5, 2)` e os valores retornados são descartados. Você deve deixar explícito que está ignorando os valores de retorno usando sublinhados.

*Dica: use sempre `_` quando não precisar ler um valor retornado por uma função.*

### Valores de retorno nomeados

Além de permitir que você retorne mais de um valor em uma função, *Go* também permite que você especifique **nomes para seus valores de retorno**. Vamos reescrever nossa função `divAndRemainder` mais uma vez, desta vez usando valores de retorno nomeados:

```go
func divAndRemainder(numerator int, denominator int) (result int, remainder int, err error) {

	if denominator == 0 {
		err = errors.New("cannot divide by zero")
		return result, remainder, err
	}

	result, remainder = numerator/denominator, numerator%denominator

	return result, remainder, err
}
```

Quando você fornece nomes para seus valores de retorno, o que você está fazendo é pré-declarar variáveis ​​que você usa dentro da função para manipular os valores de retorno. Eles são criados como uma lista separada por vírgulas entre parênteses. *Você deve colocar os valores de retorno nomeados entre parênteses, mesmo se houver apenas um único valor de retorno. Os valores de retorno nomeados são criados inicializados com seus zero values*. Isso significa que podemos devolvê-los antes de qualquer uso ou atribuição explícita.

*Os parâmetros de retorno nomeados fornecem uma maneira de declarar a intenção de usar variáveis ​​para conter os valores de retorno, mas não exige/obriga que você os use na instrução de `return`.*

Uma coisa importante a observar: **o nome que é usado para um valor retornado nomeado é local para a função**; *eles não impõem nenhum nome fora da função*. É perfeitamente legal atribuir os valores de retorno a variáveis ​​de nomes diferentes:

```go
func main() {
	x, y, z := divAndRemainder(5, 2)

	fmt.Println(x, y, z)
}
```

*Dica: Se você deseja nomear apenas alguns dos valores de retorno, pode ser usando `_` como o nome de quaisquer valores de retorno que você deseja que permaneçam sem nome.*

## Funções São Valores

Assim como em muitas outras linguagens, **as funções em *Go* são valores**. O tipo de uma função é construído a partir da palavra-chave `func`, com os tipos dos parâmetros e valores de retorno. Essa combinação é chamada de *assinatura da função*. **Qualquer função que tenha exatamente o mesmo número e tipo de parâmetros e retorno atende à assinatura do tipo da função**.

### Declarações de *function type*

Como vimos até agora, *Go* oferece suporte a recursos avançados para funções. **Em *Go*, as funções também são tipos**, assim como `int`, `string` e `bool`. **Isso significa que podemos passar funções como argumentos para outras funções, funções podem ser retornadas de uma função e funções podem ser atribuídas a variáveis**. Podemos até definir nossos próprios tipos de função. A assinatura do tipo de uma função define os tipos dos parâmetros e valores de retorno. **Para que uma função seja do tipo de outra função, ela deve ter a assinatura exata da função de tipo declarada**. Vamos analisar alguns tipos de função:

```go
type message func()
```

O código anterior cria um novo tipo de função chamado `message`. Ele não possui parâmetros de entrada e não possui nenhum tipo de retorno.

Vamos analisar outro:

```go
type calc func(int, int) string
```

O código anterior cria um novo tipo de função chamado `calc`. Ele aceita dois argumentos do tipo `int` e seu valor de retorno é do tipo `string`.

Agora que temos uma compreensão fundamental dos *tipos de função*, podemos escrever alguns códigos para demonstrar seus usos:

```go
type calc func(int, int) string

func main() {
	calculator(add, 5, 6) // Added 5 + 6 = 11
}

func add(i, j int) string {
	result := i + j
	return fmt.Sprintf("Added %d + %d = %d", i, j, result)
}

func calculator(f calc, i, j int) {
	fmt.Println(f(i, j))
}
```

Vejamos o código pela linha:

```go
type calc func(int, int) string
```

o tipo `calc` declara `calc` como sendo do tipo `func`, determinando que ele recebe dois inteiros como argumentos e retorna uma string:

```go
func add(i, j int) string {
	result := i + j
	return fmt.Sprintf("Added %d + %d = %d", i, j, result)
}
```

`func add(i, j int) string` tem a mesma assinatura do tipo `calc`. Ele recebe dois inteiros como argumentos e retorna uma string informando "`Added i + j = result`". As funções podem ser passadas para outras funções como qualquer outro tipo em *Go*:

```go
func calculator(f calc, i, j int) {
	fmt.Println(f(i, j))
}
```

`func calculator(f calc, i, j int)` aceita o tipo `calc` como entrada. O tipo `calc` , é um tipo de função que possui dois parâmetros do tipo `int` e um tipo de `string` como retorno. *Qualquer coisa que corresponda a essa assinatura pode ser passada para a função*. A função `func calculator` retorna o resultado da função do tipo `calc`.

Na função principal, chamamos `calculator(add, 5, 6)`. Estamos passando a função `add`. `add` satisfaz a assinatura do tipo `calc func`.

A imagem abaixo resume cada uma das funções anteriores e como elas se relacionam entre si. A imagem mostra como `func add` é do tipo `func calc`, que então permite que seja passado como um argumento para `func calculator`:

![Function types and uses](/images/articles/Go/Function%20types%20and%20uses.png?id=1c2094d9a5ac17f9f2368233afd4e3b3 "Function types and uses")

Acabamos de ver como criar um tipo de função e passá-lo como um argumento para outra função. Não é muito difícil passar uma função como parâmetro para outra função. Vamos mudar nosso exemplo anterior ligeiramente para refletir a passagem de uma função como um parâmetro:

```go
func main() {
	calculator(add, 5, 6)       // 11
	calculator(subtract, 10, 5) // 5
}

func calculator(f func(int, int) int, i, j int) {
	fmt.Println(f(i, j))
}

func add(i, j int) int {
	return i + j
}

func subtract(i, j int) int {
	return i - j
}
```

*A capacidade de passar funções como um tipo é um recurso muito poderoso que pode passar funções diferentes para outras funções, desde que suas assinaturas correspondam aos parâmetros da função passada*. É muito simples quando você pensa sobre isso. Um tipo inteiro para uma função pode ser qualquer valor, desde que seja um inteiro. O mesmo se aplica à passagem de funções: uma função pode ser qualquer valor, desde que seja do tipo correto (mesma assinatura).

Uma função também pode ser retornada de outra função:

```go
func main() {
	v := square(9)
	fmt.Println(v())
	fmt.Printf("Type of v: %T", v)
}

func square(x int) func() int {
	f := func() int {
		return x * x
	}

	return f
}
```

Executar o código anterior, produz a seguinte saída:

```
81
Type of v: func() int
```

Vamos criar várias funções. Precisamos saber calcular o salário de um desenvolvedor e de um gerente. Queremos que esta solução seja extensível para futuras possibilidades de cálculos de outros salários. Estaremos criando funções para calcular o salário do desenvolvedor e gerente. Em seguida, criaremos outra função que usará a função mencionada anteriormente como parâmetro de entrada.

```go
package main

import (
	"fmt"
)

func main() {
	devSalary := salary(50, 2080, developerSalary)
	bossSalary := salary(150000, 25000, managerSalary)

	fmt.Printf("Boss salary: %d\n", bossSalary)
	fmt.Printf("Developer salary: %d\n", devSalary)
}

func salary(x, y int, f func(int, int) int) int {
	pay := f(x, y)

	return pay
}

func managerSalary(baseSalary, bonus int) int {
	return baseSalary + bonus
}

func developerSalary(hourlyRate, hoursWorked int) int {
	return hourlyRate * hoursWorked
}
```

A saída esperada é a seguinte:

```
Boss salary: 175000
Developer salary: 104000
```

No exemplo de código acima, vimos como um tipo de função pode ser um parâmetro para outra função. Isso permite que uma função seja um argumento para outra função. Este exemplo mostrou como o código pode ser simplificado por ter uma função `salary`. Se, no futuro, precisarmos calcular o salário para uma posição de tester, precisaremos apenas criar uma função que corresponda ao tipo de função de `salary` e passá-la como um argumento. A flexibilidade que isto proporciona é que não temos que alterar a implementação da nossa função `salary`.

## Closures

As funções declaradas dentro de funções são especiais; eles são *closures*. Esta é uma palavra da ciência da computação que significa que funções declaradas dentro de funções são capazes de acessar e modificar variáveis ​​declaradas na função externa.

**As *closures* são uma forma de funções anônimas**. *As funções regulares não podem fazer referência a variáveis ​​fora delas*; no entanto, uma função anônima pode fazer referência a variáveis ​​externas à sua definição. Uma closure pode usar variáveis ​​declaradas no mesmo nível que a função anônima declarada. Essas variáveis ​​não precisam ser passadas como parâmetros. A função anônima tem acesso a essas variáveis ​​quando é chamada:

```go
func main() {
	i := 0

	incrementor := func() int {
		i += 1
		return i
	}

	fmt.Println(incrementor())  // 1
	fmt.Println(incrementor())  // 2

	i += 10

	fmt.Println(incrementor())  // 13
}
```

Um problema com o exemplo anterior que observamos é que qualquer código na função principal tem acesso a variável `i`. Como vimos no exemplo, `i` pode ser acessado e alterado fora da função. Este não é o comportamento desejado; queremos que o `incrementor` seja o único a alterar esse valor. Em outras palavras, queremos proteger `i` de acesso externo a função `incrementor`. A única função que deveria estar mudando é nossa função anônima quando a chamamos:

```go
func main() {
	increment := incrementor()

	fmt.Println(increment())    // 1
	fmt.Println(increment())    // 2
}

func incrementor() func() int {
	i := 0

	return func() int {
		i += 1
		return i
	}
}
```

Todas essas funções internas e coisas de *closure* podem não parecer muito interessantes à primeira vista. Qual é o benefício de criar minifunções dentro de uma função maior? Por que *Go* tem esse recurso?

*Uma coisa que as closures permitem que você faça é limitar o escopo de uma função*. Se uma função vai ser chamada apenas de uma outra função, mas é chamada várias vezes, você pode usar uma função interna para "ocultar" a função chamada. Isso reduz o número de declarações no nível do pacote, o que pode facilitar a localização de um nome não utilizado.

*Closures* realmente se tornam interessantes quando são passados ​​para outras funções ou retornados de uma função. Eles permitem que você pegue as variáveis ​​dentro de sua função e use esses valores fora de sua função.

### Passando *Closures* como parâmetros

Uma vez que funções são valores e você pode especificar o tipo de uma função usando seus parâmetros e o tipo de retorno, você pode passar funções como parâmetros em outras funções. Se você não está acostumado a tratar funções como dados, pode precisar de um momento para pensar sobre as implicações de criar uma *closure* que faça referência a variáveis ​​locais e, em seguida, passar esse *closure* para outra função. É um padrão muito útil e aparece várias vezes na biblioteca do *Go*.

Um exemplo é a classificação de *slices*. Há uma função no pacote `sort` chamada `sort.Slice`. Ele recebe qualquer *slice* e uma *closure*/função que é usada para classificar a *slice* passada. Vamos ver como funciona classificando uma *slice* de uma estrutura usando dois campos diferentes.

Vamos ver como usamos *closures* para classificar os mesmos dados de maneiras diferentes.

```go
package main

import (
	"fmt"
	"sort"
)

func main() {
	type Person struct {
		FirstName string
		LastName  string
		Age       int
	}

	people := []Person {
		{"Firts 1", "Last 3", 2},
		{"Firts 2", "Last 2", 1},
		{"Firts 3", "Last 1", 3},
	}

	fmt.Println(people) // [{Firts 1 Last 3 2} {Firts 2 Last 2 1} {Firts 3 Last 1 3}]

	// Sort by last name
	sort.Slice(people, func(i int, j int) bool {
		return people[i].LastName < people[j].LastName
	})

	fmt.Println(people) // [{Firts 3 Last 1 3} {Firts 2 Last 2 1} {Firts 1 Last 3 2}]

	// Sort by age
	sort.Slice(people, func(i int, j int) bool {
		return people[i].Age < people[j].Age
	})

	fmt.Println(people) // [{Firts 2 Last 2 1} {Firts 1 Last 3 2} {Firts 3 Last 1 3}]
}
```

A *closure* que é passada para `sort.Slice` tem dois parâmetros, `i` e `j`, mas dentro da *closure*, podemos nos referir a `people` para que possamos classificá-lo pelo campo `LastName`. Em termos de ciência da computação, `people` são capturadas pela *closure*.

A execução desse código fornece a seguinte saída:

```
[{Firts 1 Last 3 2} {Firts 2 Last 2 1} {Firts 3 Last 1 3}]
[{Firts 3 Last 1 3} {Firts 2 Last 2 1} {Firts 1 Last 3 2}]
[{Firts 2 Last 2 1} {Firts 1 Last 3 2} {Firts 3 Last 1 3}]
```

A *slice* de `people` é alterada pela chamada para `sort.Slice`.

*DICA: Passar funções como parâmetros para outras funções geralmente é útil para realizar diferentes operações no mesmo tipo de dados.*

### Retornando *Closure* de funções

Você não só pode usar uma *closure* para passar algum estado de função para outra função, como também retornar uma *closure* de uma função. Vamos mostrar isso escrevendo uma função que retorna uma função multiplicadora.

```go
package main

import "fmt"

func makeMult(base int) func(int) int {
	return func(factor int) int {
		return base * factor
	}
}

func main() {
	twoBase := makeMult(2)
	threeBase := makeMult(3)

	for i := 0; i < 3; i++ {
		fmt.Println(twoBase(i), threeBase(i))
	}
}
```

A execução deste programa fornece a seguinte saída:

```
0 0
2 3
4 6
```

## `defer`

Os programas geralmente criam recursos temporários, como arquivos ou conexões de rede, que precisam ser fechadas / limpas. Essa limpeza deve acontecer, não importa quantos pontos de saída uma função tenha ou se uma função foi concluída com êxito ou não. Em *Go*, o código de limpeza é anexado à função com a palavra-chave `defer`.

*A instrução `defer` adia a execução de uma instrução até que a função que está sendo executando atualmente seja concluída. Essa instrução será executada essencialmente logo antes que a função em que você está atualmente seja concluída.* Ainda confuso? Talvez um exemplo torne este conceito um pouco mais claro:

```go
func main() {
	defer done()
	fmt.Println("Main: Start")
	fmt.Println("Main: End")
}

func done() {
	fmt.Println("Now I am done")
}
```

A saída para o exemplo do código acima é a seguinte:

```
Main: Start
Main: End
Now I am done
```

Dentro da função `main()`, temos uma função adiada, `defer done()`. Observe que a função `done()` não possui sintaxe nova ou especial. Ele apenas tem uma simples impressão em `stdout`.

A seguir, temos duas instruções de impressão. Os resultados são interessantes. As duas instruções *print* na função `main()` são impressas primeiro. Mesmo que a função adiada tenha sido a primeira instrução em `main()`, ela foi impressa por último. Não é interessante? *Sua posição na `main()` não dita a sua ordem de execução.*

*As funções adiadas são comumente usadas para executar atividades de "limpeza". Isso incluiria a liberação de recursos, o fechamento de arquivos, o fechamento de conexões de banco de dados e a remoção de arquivos de configuração/temp criados por um programa. As funções `defer` também são usadas para se recuperar de um pânico.*

O uso da instrução `defer` não se limita apenas a funções nomeadas. Na verdade, você pode utilizar a instrução `defer` com funções anônimas. Pegando nosso snippet de código anterior, vamos transformá-lo em uma chamada adiada com uma função anônima:

```go
func main() {
	defer func() {
		fmt.Println("Now I am done")
	}()

	fmt.Println("Main: Start")
	fmt.Println("Main: End")
}
```

- Não há muita coisa que mudou em relação ao código anterior. Pegamos o código que estava na função `done` e criamos uma função anônima adiada.
- A instrução `defer` é colocada antes da palavra-chave `func()`. Nossa função não tem um nome. Como você deve se lembrar, uma função sem nome é uma *anonymous function* (função anônima).
- Os resultados são iguais aos do exemplo anterior. A legibilidade, até certo ponto, é mais fácil do que ter a função adiada declarada como uma função nomeada, como no exemplo anterior.

Também é possível e comum ter várias instruções `defer` em uma função. No entanto, eles podem não ser executados na ordem que você espera. Ao usar instruções `defer` na frente das funções, a execução segue a ordem de *First In Last Out (FILO)*. O primeiro item a ser retirado da pilha é o último item colocado na pilha. Vejamos um exemplo que declara várias funções anônimas com a instrução `defer` colocada na frente delas:

```go
func main() {
	defer func() {
		fmt.Println("I was declared first.")
	}()

	defer func() {
		fmt.Println("I was declared second.")
	}()

	defer func() {
		fmt.Println("I was declared third.")
	}()

	f1 := func() {
		fmt.Println("Main: Start")
	}

	f2 := func() {
		fmt.Println("Main: End")
	}

	f1()
	f2()
}
```

A saída do código acima é:

```
Main: Start
Main: End
I was declared third.
I was declared second.
I was declared first.
```

- As três primeiras funções anônimas estão tendo sua execução adiada.
- Declaramos `f1` e `f2` do tipo `func()`. Essas duas funções são funções anônimas.
- Como você pode ver, `f1()` e `f2()` foram executados conforme o esperado, mas a ordem das várias instruções `defer` foram executadas na ordem inversa de como foram declaradas no código. O primeiro adiamento foi o último a ser executado e o último adiamento foi o primeiro a ser executado.

Uma consideração cuidadosa deve ser dada ao usar declarações `defer`. Uma situação que você deve considerar é quando você usa instruções `defer` em conjunto com variáveis. *Quando uma variável é passada para uma função adiada, o valor da variável naquele momento é o que será usado na função adiada. Se essa variável for alterada após a função adiada, ela não será refletida quando a função adiada for executada*:

```go
func main() {
	age := 25
	name := "John"
	defer personAge(name, age)
	age *= 2
	fmt.Printf("Age double %d.\n",age)
}

func personAge(name string, i int) {
	fmt.Printf("%s is %d.\n", name, i)
}
```

A saída seria a seguinte:

```
Age double 50.
John is 25.
```
