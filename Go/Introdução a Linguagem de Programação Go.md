---
id: 9be69c3a-55ce-48b6-b4e7-e078ba73c173
title: Introdução a Linguagem de Programação Go
summary: "Introdução à manipulação de variáveis, manipulação dos tipos primitivos e um pouco de tipos de valor e referência (ponteiro) em Go."
---

## Manipulando Variáveis

Uma variável contém dados temporário para que se possa trabalhar com eles. Quando é declarado uma variável, são necessárias quatro coisas: _um statement da declaração_, _um nome_, _o tipo de dado que ela pode conter_ e um _valor inicial_. Felizmente, **algumas partes são opcionais**, significando também que há mais de uma maneira de definir variáveis.

### `var` x `:=`

*Go* tem várias maneiras de declarar variáveis. Há uma razão para isso; cada estilo de declaração comunica algo sobre como a variável é usada. Examinaremos as maneiras pelas quais pode-se declarar as variáveis e ver quando cada uma é apropriada.

A *maneira mais verbosa* de declarar uma variável em *Go* é usando a palavra-chave `var`, um tipo explícito e uma atribuição. Da seguinte forma:

```go
var x int = 10
```

As partes são `var`, `x`, `int` e `= 10`:

- `var` *é a declaração de que estamos definindo uma variável.*
- `x` *é o nome da variável.*
- `int` *é o tipo da variável.*
- `= 10` *é seu valor inicial.*

Se o tipo da variável do lado direito do sinal `=` for o tipo esperado, você pode não declarar o seu tipo. Como o tipo padrão de um literal inteiro é `int`, a seguinte declaração é a mesma coisa da declaração anterior:

```go
var x = 10
```

Por outro lado, se você deseja declarar uma variável e atribuir a ela **_zero value_**, pode manter o tipo e eliminar seu valor:

```go
var x int
```

Você pode declarar várias variáveis ​​de uma só vez, com uma limitação disso é que, ao declarar o tipo, todos os valores devem ser do mesmo tipo:

```go
// Type only
var x, y int = 10, 20

// Type only - *zero values*:
var start, middle, end float32

// Initial value mixed type
var name, left, right, top, bottom = "one", 1, 1.5, 2, 2.5
```

Existe mais uma maneira de usar o `var`. Se estiver declarando várias variáveis ​​de uma vez, você pode agrupá-las em uma lista de declaração. As variáveis ​​não precisam ser do mesmo tipo e todas podem ter seus próprios valores iniciais.

```go
var (
	x int
	y = 20
	z int = 30
	d, e = 40, "hello"
	f, g string
)
```

*Go* também suporta um formato de declaração curto (**Short Variable Declaration**). Quando estiver dentro de uma função, você pode usar o operador `:=` para **inferência do tipo** e substituir a declaração `var`. *Go* faz isso permitindo que não precisemos usar a palavra-chave `var` e sempre inferindo o tipo a partir de um **valor inicial obrigatório**. As duas instruções a seguir fazem exatamente a mesma coisa, declaram que `x` é um `int` com o valor `10`:

```go
var x = 10
x := 10
```

Assim como `var`, pode-se declarar várias variáveis ​​de uma só vez usando `:=`. *Elas devem estar na mesma linha e cada variável deve ter um valor inicial correspondente*. As seguintes duas linhas atribuem `10` a variável `x` e `"hello"` a variável `y`:

```go
var x, y = 10, "hello"
x, y := 10, "hello"
```

O operador `:=` pode fazer um truque que você não pode fazer com `var`; ele permite atribuir valores a variáveis ​​existentes também. Contanto que haja uma nova variável no lado esquerdo de `:=`, qualquer uma das outras variáveis ​​já pode existir:

```go
x := 10
x, y := 30, "hello"
```

**Existe uma limitação no operador `:=`. Ele não é permitido usar fora das funções ou métodos, assim, se estiver declarando uma variável no nível do pacote, deve-se usar `var`.**

Como saber qual estilo usar então? Como sempre, escolha o que torna a legibilidade mais clara. *O estilo de declaração mais comum dentro das funções é usar o operador `:=`*.

### Existem algumas situações dentro das funções em que deve ser evitado o uso do operador `:=`:

- Ao inicializar uma variável com seu como *zero value*, use por exemplo `var x int`. Isso deixa claro que o *zero value* é desejado / intencional.

- Ao atribuir uma constante sem tipo ou literal a uma variável e o tipo padrão da constante ou literal não for o tipo desejado, use a forma longa de `var` com o tipo especificado. Embora seja permitido usar uma conversão de tipo para especificar o tipo do valor e usar `:=` para escrever `x := byte(20)`, é mais legível / idiomático escrever `var x byte = 20`.

- Como `:=` permite atribuir variáveis ​​novas e existentes, às vezes cria-se novas variáveis ​​quando você pensa que está reutilizando as existentes. Nessas situações, declare explicitamente todas as suas novas variáveis ​​com `var` para deixar claro quais variáveis ​​são novas e, em seguida, use o operador de atribuição (`=`) para atribuir valores às variáveis ​​novas e antigas.

Você raramente declarará variáveis ​​fora das funções, no que é chamado de *bloco de pacote*. Variáveis ​​no nível do pacote cujos valores mudam são uma má ideia. Quando você tem uma variável fora de uma função, pode ser difícil rastrear suas alterações, o que torna difícil entender como os dados estão fluindo pelo programa. Como regra geral, você só deve declarar variáveis ​​no *bloco de pacote* que são efetivamente imutáveis.

### Mudando o valor de uma variável

Agora que definimos nossas variáveis, vamos ver o que podemos fazer com elas. Primeiro, vamos alterar o valor inicial. Para fazer isso, usamos uma notação semelhante a quando definimos um valor inicial parecido com `<variable> = <value>`.

```go
// Declare a variable:
offset := 5

// Change the value of the variable:
offset = 10
```

### Alterando vários valores de uma vez

Da mesma forma que você pode declarar várias variáveis ​​em uma linha, você também pode alterar o valor de mais de uma variável por vez. A sintaxe também é semelhante; parece com `<var1>, <var2>, …, <varN> = <val1>, <val2>, …, <valN>`.

```go
// Declare our variables with an initial value:
query, limit, offset := "bat", 10, 0

// Change each variable's values using a one-line statement:
query, limit, offset = "ball", offset, 20
```

## Tipos Básicos / Primitivos

*Go* é uma **linguagem compilada e fortemente tipada** e todos os dados recebem um tipo. Esse tipo é fixo e não pode ser alterado. O que você pode e não pode fazer com seus dados é restringido pelos tipos que você atribui. Entender exatamente o que define cada um dos principais tipos do *Go* é fundamental para o sucesso com a linguagem.

**Os tipos básicos do `Go` são:**

### Booleanos - `bool`

> - Variáveis ​​do tipo `bool` podem ter um dos dois valores: `true` ou `false`.
> - *Zero value* para um booleano é `false`.
> - Ao usar um operador de comparação como `==` ou `>`, o resultado dessa comparação é um valor booleano.

```go
var flag bool // no value assigned, set to false (zero value)
// OR
var isAwesome = true
```

No código abaixo, escrevemos um programa para exibir se a complexidade da senha atende a alguns requisitos. Que são:

- Ter uma letra minúscula
- Ter uma letra maiúscula
- Tem um número
- Tem um símbolo
- Ter 8 ou mais caracteres

```go
package main

import (
	"fmt"
	"unicode"
)

func passwordChecker(pw string) bool {
	// Make it UTF-8 safe
	pwR := []rune(pw)

	if len(pwR) < 8 {
		return false
	}

	hasUpper := false
	hasLower := false
	hasNumber := false
	hasSymbol := false

	for _, v := range pwR {
		if unicode.IsUpper(v) {
			hasUpper = true
		}

		if unicode.IsLower(v) {
			hasLower = true
		}

		if unicode.IsNumber(v) {
			hasNumber = true
		}

		if unicode.IsPunct(v) || unicode.IsSymbol(v) {
			hasSymbol = true
		}
	}

	return hasUpper && hasLower && hasNumber && hasSymbol
}

func main() {
	if passwordChecker("") {
		fmt.Println("password good")
	} else {
		fmt.Println("password bad")
	}

	if passwordChecker("This!I5A") {
		fmt.Println("password good")
	} else {
		fmt.Println("password bad")
	}
}
```

Executar o código anterior mostra a seguinte saída:

```
password bad
password good
```

### Numéricos - `int` `int8` `int16` `int32` `int64` `uint` `uint8` `uint16` `uint32` `uint64` `uintptr`

*Go* tem uma grande quantidade de tipos numéricos que são agrupados em *três categorias*.

#### Inteiros

*Go* tem inteiros *signed* (aceita números negativos) e *unsigned* (apenas números positivos) em uma variedade de tamanhos, *de um a quatro bytes*. Veja a tabela abaixo:

| **Type name** | **Value range**                                 |
|---------------|-------------------------------------------------|
| `int8`        | *–128* to *127*                                 |
| `int16`       | *–32768* to *32767*                             |
| `int32`       | *–2147483648* to *2147483647*                   |
| `int64`       | *–9223372036854775808* to *9223372036854775807* |
| `uint8`       | *0* to *255*                                    |
| `uint16`      | *0* to *65536*                                  |
| `uint32`      | *0* to *4294967295*                             |
| `uint64`      | *0* to *18446744073709551615*                   |

*Zero value para um tipo inteiro é `0` (zero).*

#### Tipos inteiros especiais

*Go* tem alguns nomes especiais para tipos *inteiros*. Um tipo `byte` é um *aliás* para o tipo `uint8`; é correto atribuir, comparar ou realizar operações matemáticas entre um `byte` e um `uint8`.

O segundo nome especial é `int`. Em uma CPU de 32-bits, `int` é um *32-bit signed integer* como um `int32`. Na maioria das CPUs de 64-bit, `int` é um *64-bit signed integer*, deve ser semelhante a um `int64`. Como `int` não é consistente de plataforma para plataforma, é um erro em tempo de compilação atribuir, comparar ou realizar operações matemáticas entre um `int` e um `int32` ou `int64` sem antes convertê-lo primeiro. Os literais *inteiros* são do tipo `int` por padrão.

O terceiro nome especial é `uint`. Ele segue as mesmas regras do `int`, mas são *unsigned* (*os valores são sempre `0` ou positivos*).

Existem outros dois nomes especiais para tipos inteiros, `runa` e `uintptr`.

#### Tipos de ponto flutuante

Existem dois tipos de ponto flutuante no *Go*:

| **Type name** | **Largest absolute value**                     | **Smallest (nonzero) absolute value**          |
|---------------|------------------------------------------------|------------------------------------------------|
| `float32`     | 3.40282346638528859811704183484516925440e+38   | 1.401298464324817070923729583289916131280e-45  |
| `float64`     | 1.797693134862315708145274237317043567981e+308 | 4.940656458412465441765687928682213723651e-324 |

*Como os tipos inteiros, zero value para os tipos de ponto flutuante é `0` (zero).*

### Strings e Runes - `string` `rune`

Como a maioria das linguagens modernas, *Go* inclui `strings` como um tipo embutido. **O zero value de uma string é uma string vazia**. *Go* oferece suporte a *Unicode*; você pode colocar qualquer caractere *Unicode* em uma string. Como inteiros e flutuantes, *Strings* são comparadas quanto à igualdade usando `==`, diferença com `!=`, ou ordenando com `>`, `>=`, `<` ou `<=`. *Strings* são concatenadas usando o operador `+`.

Quando você está escrevendo algum texto para uma *string*, é chamado de *literal de string*. Existem dois tipos de literais de string em *Go*:

- **Raw** - definido envolvendo o texto em `` ` ``
- **Interpreted** - definido envolvendo o texto em `"`

Com o **raw**, o texto que está na variável é justamente o que você vê na tela. Com **interpreted**, *Go* verifica o valor que está na variável, em seguida, aplica transformações com base em seu próprio conjunto de regras.

Vendo o seguinte código:

```go
package main

import "fmt"

func main() {
	comment1 := `This is the BEST
thing ever!`
	comment2 := `This is the BEST\nthing ever!`
	comment3 := "This is the BEST\nthing ever!"

	fmt.Print(comment1, "\n\n")
	fmt.Print(comment2, "\n\n")
	fmt.Print(comment3, "\n")
}
```

Executar o código anterior fornece a seguinte saída:

```
This is the BEST
thing ever!

This is the BEST\nthing ever!

This is the BEST
thing ever!
```

Em uma string interpretada, `\n` representou uma nova linha. Na string *raw*, `\n` não faz nada. Para obter uma nova linha na string *raw*, devemos adicionar uma nova linha real em nosso código. A string interpretada deve usar `\n` para obter uma nova linha, já que não é permitido ter uma nova linha real em uma string interpretada.

Os literais de string interpretadas são o tipo mais comum nos códigos, mas os literais *raw* têm sua importância. Se você quiser copiar e colar algum texto que contenha muitas linhas novas com `"` ou `\`, é mais fácil usar *raw*.

No exemplo a seguir, você pode ver como o uso de *raw* torna o código mais legível:

```go
package main

import "fmt"

func main() {
	comment1 := `In "Windows" the user directory is "C:\Users\"`
	comment2 := "In \"Windows\" the user directory is \"C:\\Users\\\""

	fmt.Println(comment1)
	fmt.Println(comment2)
}
```

Executar o código anterior mostra a seguinte saída:

```
In "Windows" the user directory is "C:\Users\"
In "Windows" the user directory is "C:\Users\"
```

Uma coisa que você não pode ter em um literal raw é `` ` ``. Se você precisa de um literal com `` ` ``, você deve usar um literal de string interpretada.

Literais de *string* são apenas maneiras de inserir algum texto em uma variável do tipo *string*. Depois de ter o valor na variável, não há diferenças.

Strings em *Go* são imutáveis; você pode re-atribuir o valor de uma variável de *string*, mas não pode alterar o valor da *string* atribuída a ela.

O tipo `rune` é um apelido para o tipo `int32`, assim como `byte` é um apelido para `uint8`. O tipo padrão de um literal de `rune` é um `rune`, e o tipo padrão de um literal de *string* é uma *string*.

### O valor `nil`

`nil` não é um tipo, mas um valor especial em *Go*. **Ele representa um valor vazio de nenhum tipo**. Ao trabalhar com *ponteiros*, *mapas* e *interfaces*, você precisa ter certeza de que eles não são `nil`. Se você tentar interagir com um valor `nil`, seu código irá travar / crash.

Se você não tem certeza se um valor é `nil` ou não, você pode verificá-lo assim:

```go
package main

import "fmt"

func main() {
	var message *string

	if message == nil {
		fmt.Println("error, unexpected nil value")
		return
	}

	fmt.Println(&message)
}
```

Executar o código anterior mostra a seguinte saída:

```
error, unexpected nil value
```

### *Zero Value*

> *Go*, como a maioria das linguagens modernas, *atribui um valor padrão a variáveis declaradas sem um valor inicial explícito*.

Os valores padrões para *Zero Value* são:

| **Type**                                                      | **Zero Value**      |
|---------------------------------------------------------------|---------------------|
| `bool`                                                        | `false`             |
| `Numbers (integers e floats)`                                 | 0                   |
| `String`                                                      | `""` (empty string) |
| `pointers, functions, interfaces, slices, channels, e maps`   | `nil`               |

### Conversão de tipos

A maioria das linguagens que possuem vários tipos numéricos são convertidos automaticamente de um para outro quando necessário. Isso é chamado de *automatic type promotion* e, embora pareça muito conveniente, as regras para converter corretamente um tipo em outro podem se tornar complicadas e produzir resultados inesperados. Como uma linguagem que valoriza a *clareza de intenção* e *legibilidade*, *Go* não permite *automatic type promotion* entre variáveis. Você deve usar *type conversion* quando os tipos de variáveis ​​não correspondem. Até mesmo números inteiros e flutuantes de tamanhos diferentes devem ser convertidos para o mesmo tipo para manipulação.

Isso deixa claro exatamente o tipo que se deseja, sem ter que saber nenhuma regra da linguagem de *type conversion*.

```go
var x int = 10
var y float64 = 30.2
var z float64 = float64(x) + y
var d int = x + int(y)
```

No código acima, definimos 4 variáveis. `x` é um `int` com o valor `10` e `y` é um `float64` com o valor `30.2`. Como esses tipos não são idênticos, é necessário convertê-los para manipulação. Para a variável `z`, convertemos `x` em um `float64` usando *type conversion* e para a variável `d` convertemos `y` em um `int`.

Essa rigidez em torno dos tipos tem outras implicações. Como todas as conversões de tipo em *Go* são explícitas, não é possível tratar outro tipo *Go* como *booleano*. Em muitas linguagens, um número diferente de zero ou uma string não vazia pode ser interpretado como um *booleano* `true`. Assim como *type promotion*, as regras para valores verdadeiros variam de linguagem para linguagem e podem ser confusas. Assim, nenhum outro tipo pode ser convertido em `bool`, implícito ou explicitamente. Se você deseja converter de outro tipo para *booleano*, deve usar um dos operadores de comparação (`==`, `!=`, `>`, `<`, `<=` ou `>=`). Por exemplo, para verificar se a variável `x` é igual a `0`, você deve escrever `x == 0`. Se você quiser verificar se a string `s` está vazia, você deve escrever `s == ""`.

## `const`

Muitas linguagens têm uma maneira de declarar que um valor é imutável. Em *Go*, isso é feito com a palavra-chave `const`. Como no código abaixo:

```go
package main

import "fmt"

const x int64 = 10

const (
	idKey   = "id"
	nameKey = "name"
)

const z = 20 * 10

func main() {
	const y = "hello"

	fmt.Println(x)
	fmt.Println(y)

	x = x + 1
	y = "bye"

	fmt.Println(x)
	fmt.Println(y)
}
```

Se você tentar executar este código, as compilações falharão com as seguintes mensagens de erro:

```
./prog.go:20:2: cannot assign to x (declared const)
./prog.go:21:2: cannot assign to y (declared const)
```

*Você declara uma constante no nível do pacote ou dentro de uma função. Assim como `var`, você pode (e deve) declarar um grupo de constantes relacionadas dentro de parênteses.*

No entanto, `const` em *Go* é muito limitado. Constantes em *Go* são uma forma de dar nomes a literais. ***Eles só podem conter valores que o compilador pode calcular em tempo de compilação***. Isso significa que eles podem ser atribuídos:

- Literais do tipo numérico
- true e false
- Strings
- Runes

*Go* não fornece uma maneira de especificar que um valor calculado em tempo de execução é imutável. Não há arrays, slices, maps ou structs imutáveis e não há como declarar que um campo em uma estrutura é imutável.

## Go é Chamado Por Valor

***Go* é uma linguagem chamada por valor. Isso significa que quando você fornece uma variável para um parâmetro de uma função, *Go* sempre faz uma cópia do valor da variável**. Vamos dar uma olhada.

```go
package main

import "fmt"

type person struct {
	age  int
	name string
}

func modifyFails(i int, s string, p person) {
	i = i * 2
	s = "Goodbye"
	p.name = "Jonh"
}

func main() {
	p := person{}
	i := 2
	s := "Hello"
	modifyFails(i, s, p)
	fmt.Println(i, s, p)
}
```

A execução deste código mostra que a função não altera os valores dos parâmetros passados para ela:

```
2 Hello {0 }
```

Eu incluí a estrutura de `person` para mostrar que isso não é obrigatório para tipos primitivos. Se você tem experiência em programação em Java, JavaScript, Python ou Ruby, pode achar o comportamento da estrutura muito estranho. Afinal, essas linguagens permitem modificar os campos de um objeto quando você passa um objeto como parâmetro para uma função. O motivo da diferença é algo que abordaremos quando falarmos sobre ponteiros.

O comportamento é um pouco diferente para *`maps`* e *`slices`*. Vamos ver o que acontece quando tentamos modificá-los dentro de uma função. Vamos escrever uma função para modificar um parâmetro de um *map* e uma função para modificar um parâmetro da *slice*:

```go
package main

import "fmt"

func modMap(m map[int]string) {
	m[2] = "hello"
	m[3] = "goodbye"

	delete(m, 1)
}

func modSlice(s []int) {
	for k, v := range s {
		s[k] = v * 2
	}

	s = append(s, 10)
}

func main() {
	m := map[int]string{
		1: "first",
		2: "second",
	}

	modMap(m)
	fmt.Println(m)

	s := []int{1, 2, 3}
	modSlice(s)
	fmt.Println(s)
}
```

Ao executar este código, você verá algo interessante:

```
map[2:hello 3:goodbye]
[2 4 6]
```

Para o map, é fácil explicar o que acontece: *quaisquer alterações feitas em um parâmetro do map são refletidas na variável passada para a função*. Para uma slice, é mais complicado. *Você pode modificar qualquer elemento na slice, mas não pode adicionar novos*. Isso acontece para maps e slices que são passados para a função, bem como campos de maps e slices em estruturas.

Este programa leva à pergunta: por que maps e slices se comportam de maneira diferente de outros tipos? **É porque maps e slices são implementados com ponteiros**.

Chamada por valor é uma das razões pelas quais o suporte limitado de *Go* para constantes é apenas uma pequena desvantagem. **Como as variáveis ​​são passadas por valor, você pode ter certeza de que chamar uma função não modifica a variável cujo valor foi passado (a menos que a variável seja uma *`slice`* ou *`map`*)**. Em geral, isso é uma coisa boa. Torna mais fácil entender o fluxo de dados por meio de seu programa quando as funções não modificam seus parâmetros e, em vez disso, retornam valores recém-manipulados.

Embora essa abordagem seja fácil de entender, há casos em que você precisa passar algo mutável para uma função. O que você faz então? É quando você precisa de ponteiros (valores por referência).

## Value vs Pointer

Quando você passa valores como `int`, `bool` e `string` para uma função, *Go* faz uma **cópia do valor**, e é a cópia que é usada na função. **Essa cópia significa que uma alteração feita no valor do parâmetro da função de dentro da função, não afeta o valor da variável que você usou ao chamar a função**.

*A passagem de valores por meio de cópia tende a resultar em código com menos bugs*. A desvantagem é que a cópia usa cada vez mais memória à medida que os valores são passados de função para função. No código nas aplicações, as funções tendem a ser pequenas e os valores são passados para várias funções, portanto, usar parâmetros por valor pode às vezes acabar usando muito mais memória do que o necessário.

Existe uma alternativa para a cópia que usa menos memória. Em vez de passar um valor, criamos algo chamado de **ponteiro** e, em seguida, passamos isso para as funções. **Um ponteiro não é um valor em si, e você não pode fazer nada com um ponteiro, exceto obter um valor real usando-o**. **Você pode pensar em um ponteiro como direções para um valor que deseja e, para chegar ao valor, deve seguir as referências**. Se você usar um ponteiro, *Go* não fará uma cópia do valor ao passar um ponteiro para uma função.

Embora os benefícios de usar um ponteiro sobre um valor que é passado para várias funções sejam claros para o uso da memória, não são tão claros para o uso da CPU. Quando um valor é copiado, *Go* precisa de ciclos de CPU para obter essa memória e liberá-la mais tarde. Usar um ponteiro evita esse uso da CPU ao passá-lo para uma função. Por outro lado, ter um valor no *heap* significa que ele precisa ser gerenciado pelo complexo processo do *garbage collection*. Esse processo pode se tornar um gargalo na CPU em certas situações, por exemplo, se houver muitos valores no *heap*. Quando isso acontece, o *garbage collection* tem que fazer muitas verificações, o que consome ciclos de CPU. Não há uma resposta correta, e a melhor abordagem é a clássica de otimização de desempenho. Primeiro, não otimize prematuramente. Quando você tiver um problema de desempenho, meça antes de fazer uma mudança e, em seguida, meça depois de fazer uma mudança.

Além do desempenho, você pode usar ponteiros para alterar o design do seu código. Às vezes, usar ponteiros permite uma interface mais limpa e simplifica seu código. Por exemplo, se você precisa saber se um valor está presente ou não, um valor sem ponteiro sempre tem pelo menos seu *zero value*, o que pode ser válido em sua lógica. Você pode usar um ponteiro para permitir um estado não definido, bem como manter um valor. Isso ocorre porque os ponteiros, além de manter o endereço para um valor, também podem ser `nil`, o que significa que não há valor. Em *Go*, `nil` é um tipo especial que representa algo sem valor.

A capacidade de um ponteiro ser `nil` também significa que é possível obter o valor de um ponteiro quando ele não tem um valor associado a ele, o que significa que você obterá um erro em runtime (tempo de execução). Para evitar erros de runtime, você pode comparar um ponteiro com `nil` antes de tentar obter seu valor. Usando como exemplo `<pointer> != nil`. *Você pode comparar ponteiros com outros ponteiros do mesmo tipo, mas eles só resultam em verdadeiro se você estiver comparando um ponteiro a outro do mesmo tipo*. Nenhuma comparação dos valores associados é feita.

### Obtendo um Ponteiro

Para obter um ponteiro, você tem algumas opções. Você pode declarar uma variável como sendo um tipo de ponteiro usando `var`. Você pode fazer isso adicionando um `*` na frente da maioria dos tipos. Esta notação se parece com `var <name> *<type>`. O valor inicial de uma variável que usa esse método é `nil`. Você pode usar a função `new` para fazer isso. Esta função deve ser usada para obter alguma memória para um tipo e retornar um ponteiro desse endereço. A notação é semelhante a `<name> := new(<type>)`. A função `new` também pode ser usada com `var`. Você também pode obter um ponteiro de uma variável existente usando `&`. Se parece com `<var1> := &<var2>`.

Veja o seguinte código:

```go
package main

import (
	"fmt"
)

func main() {
	var count1 *int
	// Its initial value is the zero value of the variable's type
	count2 := new(int)
	countTemp := 5
	// Using &, create a pointer from the existing variable:
	count3 := &countTemp

	fmt.Printf("count1: %#v\n", count1)
	fmt.Printf("count2: %#v\n", count2)
	fmt.Printf("count3: %#v\n", count3)
}
```

O resultado é o seguinte:

```
count1: (*int)(nil)
count2: (*int)(0xc000012028)
count3: (*int)(0xc000012050)
```

No código acima, vimos três maneiras diferentes de criar um ponteiro. Cada um é útil, dependendo das necessidades de seu código. Com a instrução `var`, o ponteiro tem um valor `nil`, enquanto os outros já têm um endereço de valor associado a eles.

A seguir, veremos como podemos obter um valor de um ponteiro.

### Obtendo um valor de um ponteiro

Na seção anterior, quando imprimimos as variáveis de ponteiros `int` no console, não obtivemos ou vimos seu valor real. Para obter o valor ao qual um ponteiro está associado, **desreferencie** o valor usando `*` na frente do nome da variável. Isso se parece com `fmt.Println(*<val>)`.

**Desreferenciar um ponteiro `nil`** dará um bug no *Go*, pois o compilador não pode avisá-lo sobre isso, porque isso acontece quando o aplicativo está em execução. Portanto, é sempre uma prática recomendada verificar se um ponteiro não é `nil` antes de *desreferenciá-lo*, a menos que você tenha certeza de que não é `nil`.

Você nem sempre precisa *desreferenciar*; por exemplo, quando uma propriedade ou função está em uma estrutura. Não se preocupe muito sobre quando você não deve *desreferenciar*, pois *Go* apresenta erros claros sobre quando você pode ou não pode *desreferenciar* um valor.

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	// Our pointers are declared in the same way as they were previously:
	var count1 *int
	count2 := new(int)
	countTemp := 5
	count3 := &countTemp
	t := &time.Time{}

	// For count 1, 2, and 3, we need to add a nil check and add * in front of the variable name:
	if count1 != nil {
		fmt.Printf("count1: %#v\n", *count1)
	}

	if count2 != nil {
		// We'll dereference the variable using *, just like we did with the count variables:
		fmt.Printf("count2: %#v\n", *count2)
	}

	if count3 != nil {
		fmt.Printf("count3: %#v\n", *count3)
	}

	if t != nil {
		fmt.Printf("time  : %#v\n", *t)
		// Here, we're calling a function on our time variable. This time, we don't need to dereference it:
		fmt.Printf("time  : %#v\n", t.String())
	}
}
```

O resultado é o seguinte:

```
count2: 0
count3: 5
time  : time.Date(1, time.January, 1, 0, 0, 0, 0, time.UTC)
time  : "0001-01-01 00:00:00 +0000 UTC"
```

No código anterior, usamos desreferenciamento para obter os valores de nossos ponteiros. Também usamos verificações `nil` para evitar erros de desreferenciação. A partir da saída do código, podemos ver que `count1` era um valor `nil` e que teríamos obtido um erro se tentássemos desreferenciar. `count2` foi criado usando `new` e seu valor é um *zero value* para seu tipo. `count3` também tinha um valor que corresponde ao valor da variável da qual obtemos o ponteiro. Com nossa variável de `time`, fomos capazes de desreferenciar toda a estrutura, e é por isso que nossa saída não começa com um `&`.

A seguir, veremos como o uso de um ponteiro nos permite alterar o design de nosso código.

### Design de função com ponteiros

*Se você tiver uma variável de ponteiro ou tiver passado um ponteiro de uma variável para uma função, quaisquer alterações feitas no valor da variável na função também afetarão o valor da variável fora da função.*

Veja o seguinte código:

```go
package main

import "fmt"

// Create a function that takes an int as an argument:
func add5Value(count int) {
	count += 5
	fmt.Println("add5Value     :", count)
}

// Create function that takes an int pointer:
func add5Point(count *int) {
	// Dereference the value and add 5 to it:
	*count += 5
	// Print out the updated value of count and dereference it:
	fmt.Println("add5Point     :", *count)
}

func main() {
	var count int
	add5Value(count)
	fmt.Println("add5Value post:", count)
	// Call the second function. This time, you'll need to use & to pass a pointer to the variable:
	add5Point(&count)
	fmt.Println("add5Point post:", count)
}
```

O resultado é o seguinte:

```
add5Value     : 5
add5Value post: 0
add5Point     : 5
add5Point post: 5
```

No código anterior, mostramos que passar valores por um ponteiro pode afetar as variáveis ​​de valor que são passadas a eles. Vimos que, ao passar por valor, as mudanças feitas no valor em uma função não afetam o valor da variável que é passada para a função, enquanto passar um ponteiro para um valor muda o valor da variável passada para a função.

Você pode usar esse fato para superar problemas de design estranhos e, às vezes, simplificar o design do seu código. A passagem de valores por um ponteiro tem tradicionalmente se mostrado mais sujeita a erros, portanto, use esse design com moderação. Também é comum usar ponteiros em funções para criar um código mais eficiente, o que a biblioteca padrão do *Go* faz muito.
