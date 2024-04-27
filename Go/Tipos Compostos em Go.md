---
id: 9be858b9-f082-4703-9434-4336b08966c6
title: Tipos Compostos em Go
summary: "Manipulando Arrays, Slices, Maps e Structs."
---

## *Arrays*

> - Em *Go*, um *array* é uma sequência numerada de itens do mesmo tipo de tamanho específico.
> - Os valores que um *array* contém são conhecidos como: itens ou elementos. Todos os elementos em um *array* devem ser do mesmo tipo. Depois que um *array* é criado, seu tamanho não pode ser alterado - ele não pode crescer ou diminuir.

A coleção de dados mais básica no *Go* é um *array*. Ao definir uma *array*, você deve especificar qual o tipo de dado que ele pode conter e o seu tamanho no seguinte formato: `[<size>]<type>`. Por exemplo, `[10]int` é um *array* que pode conter até 10 números inteiros, enquanto `[5]string` é uma *array* de tamanho 5 que contém strings.

O ponto-chave para fazer um *array*, ser um *array*, é especificar o seu tamanho. Se na sua definição não tiver o seu tamanho, parece que funciona como um *array*, mas não seria - seria uma *slice*. Uma *slice* é um tipo de coleção diferente e mais flexível que falaremos após os *arrays*. Você pode definir os valores dos elementos de qualquer tipo, incluindo ponteiros e outros *arrays*.

Você pode inicializar *arrays* usando o seguinte formato: `[<size>]<type>{<value1>,<value2>,…<valueN>}`. Por exemplo, `[5]int{1}` inicializa o *array* com o primeiro valor de 1, enquanto que `[5]int{9,9,9,9,9}` preenche o *array* com o valor 9 para cada elemento. Ao inicializar o *array*, você pode fazer com que *Go* defina seu tamanho com base no número de elementos com os quais você o inicializa. Você pode tirar proveito disso, substituindo o número do seu tamanho por `...`. Por exemplo, `[...]int{9,9,9,9,9}` cria um *array* de tamanho 5 porque o inicializamos com 5 elementos. Assim como todos os *arrays*, o tamanho é definido em tempo de compilação e não pode ser alterado em runtime (tempo de execução).

Para declarar um array, você usa a seguinte sintaxe:

```
var array_name [size_of_array]data_type
```

Vamos definir um simples alguns array de tamanho 10 que contém números inteiros.

```go
package main

import "fmt"

func defineArray() ([10]int, [10]int, [10]int) {
	var arr1 [10]int
	var arr2 [10]int = [10]int{1, 2, 3, 4}
	arr3 := [10]int{4, 5, 6}

	return arr1, arr2, arr3
}

func main() {
	arr1, arr2, arr3 := defineArray()

	fmt.Printf("%#v\n", arr1)
	fmt.Printf("%#v\n", arr2)
	fmt.Printf("%#v\n", arr3)
}
```

A execução do código anterior fornece a seguinte saída:

```
[10]int{0, 0, 0, 0, 0, 0, 0, 0, 0, 0}
[10]int{1, 2, 3, 4, 0, 0, 0, 0, 0, 0}
[10]int{4, 5, 6, 0, 0, 0, 0, 0, 0, 0}
```

No código acima, definimos três array, o primeiro sem ser inicializado, o segundo inicializando preenchendo apenas os 4 primeiros elementos/itens, e o terceiro preenchendo apenas os três primeiros itens. Como todos os *arrays* têm um tamanho fixo, quando o primeiro *array* foi impresso, ele continha 10 valores inicializadas com *zero values* para um `int`, que é `0`. O segundo *array* continha apenas os 6 últimos elementos com seus *zero values*, e o terceiro os últimos 7 elementos preenchidos com *zero values* para um inteiro.

**Todos os elementos no array devem ser do mesmo tipo especificado**. Existem alguns estilos de declarações diferentes. No primeiro, você especifica o tamanho da *array* e o tipo dos elementos no *array*:

```go
var x [3]int
```

Isso cria uma *array* de 3 inteiros. Como nenhum valor foi especificado, todas as posições (`x[0]`, `x[1]` e `x[2]`) são inicializadas com *zero value* para um `int`, que é `0`. Se você tiver valores iniciais para o *array*, você os especifica com um *literal de array*:

```go
var x = [3]int{10, 20, 30}
```

Ao usar um *literal de array* para sua inicialização, você pode omitir o número do tamanho do *array* e usar `...`:

```go
var x = [...]int{10, 20, 30}
```

**O tamanho do *array* faz parte da sua definição de tipo**. Se você tiver dois *arrays* que aceitam o mesmo tipo, mas têm tamanhos diferentes, eles não são compatíveis e não são comparáveis entre si. *Arrays* de tamanhos diferentes que não são do mesmo tipo não podem ser comparados uns com os outras. Você pode usar `==` e `!=` Para comparar *arrays*:

```go
package main

import "fmt"

func compArrays() (bool, bool, bool, bool, bool) {
	var arr1 [5]int
	arr2 := [5]int{0}
	arr3 := [...]int{0, 0, 0, 0, 0}
	arr4 := [5]int{0, 0, 0, 0, 9}
	arr5 := [...]int{1, 2, 3}
	arr6 := [3]int{1, 2, 3}
	arr7 := [3]int{1, 1, 3}

	return arr1 == arr2, arr1 == arr3, arr1 == arr4, arr5 == arr6, arr5 == arr7
}

func main() {
	comp1, comp2, comp3, comp4, comp5 := compArrays()

	fmt.Println("[5]int == [5]int{0}              :", comp1)
	fmt.Println("[5]int == [...]int{0, 0, 0, 0, 0}:", comp2)
	fmt.Println("[5]int == [5]int{0, 0, 0, 0, 9}  :", comp3)
	fmt.Println("[...]int{1, 2, 3} == [3]int{1, 2, 3}  :", comp4)
	fmt.Println("[...]int{1, 2, 3} == [3]int{1, 1, 3}  :", comp5)
}
```

Executar o código anterior produz a seguinte saída:

```
[5]int == [5]int{0}              : true
[5]int == [...]int{0, 0, 0, 0, 0}: true
[5]int == [5]int{0, 0, 0, 0, 9}  : false
[...]int{1, 2, 3} == [3]int{1, 2, 3}  : true
[...]int{1, 2, 3} == [3]int{1, 1, 3}  : false
```

Como a maioria das linguagens, os arrays em *Go* são lidos e escritos usando a sintaxe `<array>[<index>]`. Por exemplo, isso acessa o primeiro elemento de um array, `arr[0]`. Depois que o array é definido, você pode fazer alterações em elementos individuais usando seu índice, com a sintaxe `<array>[<index>] = <value>`.

No código abaixo, definiremos um array e o inicializaremos com algumas palavras. Então, vamos ler as palavras na forma de uma mensagem e imprimi-la.


```go
package main

import "fmt"

func message() string {
	arr := [...]string{"ready", "Get", "Go", "to"}
	arr[1] = "It's"
	arr[0] = "time"

	return fmt.Sprintln(arr[1], arr[0], arr[3], arr[2])
}

func main() {
	fmt.Print(message())
}
```

Executar o código anterior produz a seguinte saída:

```
It's time to Go
```

Os arrays em *Go* raramente são usados explicitamente. Isso ocorre porque eles vêm com uma limitação incomum: ***Go* considera o tamanho do array como parte do tipo do array**. Isso faz com que um array declarado como `[3]int` um tipo diferente de uma array declarada como `[4]int`. Isso também significa que não pode ser usado uma variável para especificar o tamanho de um array, ***porque os tipos devem ser resolvidos em tempo de compilação, não em tempo de execução***.

Além do mais, **não pode ser usado conversão de tipo para converter arrays de tamanhos diferentes em tipos idênticos**. *Porque você não pode converter arrays de tamanhos diferentes entre si, você não pode escrever uma função que funcione com arrays de qualquer tamanho e você não pode atribuir arrays de tamanhos diferentes para a mesma variável.*

> **Devido a essas restrições, não use arrays a menos que você saiba o tamanho que você precisa em tempo de execução.**

*Isso levanta uma questão: por que uma feature tão limitada na linguagem? A principal razão pela qual os arrays estão presentes no Go é fornecer o armazenamento de apoio para os **slices**, que são uma das features mais úteis do Go.*

## *Slices*

Os *arrays* são importante, mas sua inflexibilidade em relação ao seu tamanho pode causar problemas. Se você quisesse criar uma função que tem um parâmetro de *array* para classificação dos dados, ela só funcionaria com tamanho específico do *array*. Isso requer que você crie uma função para cada tamanho do *array*. Essa restrição faz com que a manipulação de *arrays* pareça complicado. O outro lado dos *arrays* é que eles são uma maneira eficiente de gerenciar coleções de dados classificados. Não seria ótimo se houvesse uma maneira de obter a eficiência dos *arrays*, mas com mais flexibilidade? *Go* oferece isso na forma de *`slices`*.

Os *`slices`* são construidos a partir dos *arrays* e permite que você tenha uma coleção indexada numérica classificada sem que você precise se preocupar com o tamanho. *Go* gerencia todos os detalhes, como o tamanho do *array* a ser usado. Você usa um *`slice`* exatamente como faria com um *array*; ele contém apenas valores do mesmo tipo, você pode ler e gravar em cada elemento, e eles são fáceis de executar em loops.

A outra coisa que uma *`slice`* pode fazer é ser facilmente ampliada usando a função `append`. Esta função aceita sua *`slice`* e os valores que você gostaria de adicionar e retorna uma nova *`slice`* com tudo mesclado. É comum começar com uma *`slice`* vazia e aumenta-la conforme necessário.

No código do mundo real, você deve usar *`slices`* como seu destino para todas as coleções classificadas. Você será mais produtivo porque não precisará escrever tanto código quanto faria
com um *array*. A maioria dos códigos que você verá em projetos do mundo real usa muitas *slices* e raramente usa *arrays*.

Na maioria das vezes, quando você deseja uma estrutura de dados que contenha uma sequência de valores, um *`slice`* é o que você usará. **O que torna os *slices* tão úteis é que o tamanho não faz parte do seu tipo**. Isso remove as limitações dos *arrays*. Podemos escrever uma única função que processa *`slices`* de qualquer tamanho e podemos aumentá-los conforme necessário.

No exemplo de código a seguir, mostraremos como os *`slices`* são flexíveis lendo alguns dados, passando uma *`slice`* para uma função, fazendo loop, lendo valores e acrescentando valores ao final de uma *`slice`*.

```go
package main

import (
	"fmt"
	"os"
)

func getPassedArgs(minArgs int) []string {
	if len(os.Args) < minArgs {
		fmt.Printf("At least %v arguments are needed\n", minArgs)
		os.Exit(1)
	}

	var args []string

	for i := 1; i < len(os.Args); i++ {
		args = append(args, os.Args[i])
	}

	return args
}

func findLongest(args []string) string {
	var longest string

	for i := 0; i < len(args); i++ {
		if len(args[i]) > len(longest) {
			longest = args[i]
		}
	}

	return longest
}

func main() {
	if longest := findLongest(getPassedArgs(3)); len(longest) > 0 {
		fmt.Println("The longest word passed was:", longest)
	} else {
		fmt.Println("There was an error")
		os.Exit(1)
	}
}
```

Salve o arquivo. Em seguida, na pasta em que está salvo, execute o código usando o seguinte comando:

```go
go run . Get ready to Go
```

Executar o código anterior produz a seguinte saída:

```
The longest word passed was: ready
```

No código acima, pudemos ver como os *`slices`* são flexíveis e, ao mesmo tempo, como funcionam como *arrays*. Essa maneira de trabalhar com *`slices`* é outra razão pela qual *Go* tem a sensação de uma linguagem dinâmica.

Manipular os *`slices`* é muito parecido como manipular os arrays, mas existem diferenças sutis. A primeira coisa a notar é que não especificamos o tamanho do *`slice`* quando é declarado:

```go
var x = []int{10, 20, 30}
```

Isso cria um *`slice`* de 3 *ints* usando um literal de *`slice`*. Assim como os *arrays*, também podemos especificar apenas os índices com valores no literal de *`slice`*:

```go
var x = []int{1, 5: 4, 6, 10: 100, 15}
```

Isso cria um *`slice`* de 12 *ints* com os seguintes valores: `[1 0 0 0 0 4 6 0 0 0 100 15]`.

Você lê e grava *`slice`* usando a sintaxe de colchetes:

```go
x[0] = 10
fmt.Println(x[2])
```

Até agora, os *`slice`* pareciam idênticos aos *arrays*. Começamos a ver as diferenças entre *arrays* e *`slice`* quando examinamos a declaração do *`slice`* sem usar um literal.

```go
var x []int
```

Isso cria um *`slice`* de *ints*. Uma vez que nenhum valor é atribuído, `x` é atribuído para `zero value` como `nil`. Em *Go*, *`nil` é um identificador que representa a falta de um valor para alguns tipos*. Um *`slice`* com seu valor de `nil` não contém nada.

**Uma *`slice`* é um tipo que não é comparável**. É um erro em tempo de compilação usar `==` para ver se dois *`slices`* são idênticos ou `!=` para ver se são diferentes. A única coisa com a qual você pode comparar um *`slice`* é com `nil`.

```go
fmt.Println(x == nil) // prints true
```

### `len`

*Go* fornece várias funções integradas para trabalhar com seus tipos. A função `len` é utilizada para saber o tamanho do *`slice`*. Quando você passa um *`slice`* como `nil` para `len`, ele retorna `0`.

### `append`

A função `append` é usada para adicionar mais de um valor a um *`slice`*.

```go
var x []int
x = append(x, 10)
```

A função `append` leva pelo menos dois parâmetros, um *`slice`* de qualquer tipo e um valor desse tipo. Ele retorna um *`slice`* do mesmo tipo. O *`slice`* retornado é atribuído de volta ao *`slice`* que foi passado. No exemplo acima, estamos adicionando a um *`slice`* `nil`, mas você pode adicionar a uma *`slice`* que já possui elementos:

```go
var x = []int{1, 2, 3}
x = append(x, 4)
```

Você pode adicionar mais de um valor:

```go
x = append(x, 5, 6, 7)
```

Um *`slice`* pode ser adicionado a outro usando o operador ***variadic*** `...` para expandir o *`slice`* de origem em valores individuais:

```go
y := []int{20, 30, 40}
x = append(x, y...)
```

No código abaixo, usaremos o parâmetro ***variadic*** em `append` para adicionar vários valores na forma de dados predefinidos a uma *`slice`*. Em seguida, adicionaremos uma quantidade dinâmica de dados com base na entrada do usuário para a mesma *`slice`*:

```go
package main

import (
	"fmt"
	"os"
)

func getPassedArgs() []string {
	var args []string

	for i := 1; i < len(os.Args); i++ {
		args = append(args, os.Args[i])
	}

	return args
}

func getLocals(extraLocals []string) []string {
	var locales []string

	locales = append(locales, "en_US", "fr_FR")
	locales = append(locales, extraLocals...)

	return locales
}

func main() {
	locales := getLocals(getPassedArgs())

	fmt.Println("Locales to use:", locales)
}
```

Salve o arquivo. Em seguida, na pasta criada aonde está o arquivo, execute o código usando o seguinte comando:

```go
go run . fr_CN en_AU
```

Executar o código anterior produz a seguinte saída:

```
Locales to use: [en_US fr_FR fr_CN en_AU]
```

No código acima, usamos dois métodos para adicionar vários valores a uma *`slice`*. Você também usaria essa técnica se precisasse mergear duas *`slices`*.

### `cap`

Como explicado acima, um *`slice`* é uma sequência de valores. Cada elemento em uma *`slice`* é atribuído a locais de memória consecutivos, o que torna mais rápido ler ou gravar esses valores. Cada *`slice`* tem uma capacidade, que é o número de locais de memória consecutivos reservados. Isso pode ser maior do que o tamanho. Cada vez que a função `append` é chamada em um *`slice`*, um ou mais valores são adicionados ao seu final. Cada valor adicionado aumenta o tamanho em um. Quando o tamanho atinge a capacidade, não há mais espaço para colocar valores. Se você tentar adicionar valores quando o tamanho for igual à capacidade, a função `append` usará o *runtime* do *Go* para alocar uma nova *`slice`* com uma capacidade maior. Os valores da *`slice`* original são copiados para a nova *`slice`*, os novos valores são adicionados ao final e a nova *`slice`* é retornada.

Quando uma *`slice`* cresce por meio da função `append`, leva tempo para o *runtime* do *Go* alocar nova memória e copiar os dados existentes da memória antiga para a nova. A memória antiga também precisa ser *garbage collected*. Por esse motivo, o *runtime* do *Go* geralmente aumenta uma *`slice`* em mais de uma cada vez que fica sem capacidade. As regras do `Go 1.16` são dobrar o tamanho da *`slice`* quando a capacidade for inferior a 1024 e, em seguida, crescer por pelo menos 25%.

Assim como a função `len` retorna o tamanho atual de um *`slice`*, a função `cap` retorna a capacidade atual de uma *`slice`*. É usado com muito menos frequência do que `len`. Na maioria das vezes, `cap` é usado para verificar se uma *`slice`* é grande o suficiente para conter novos dados ou se uma chamada a `make` é necessária para criar uma nova *`slice`*.

Você também pode passar um *array* para a função `cap`, mas `cap` sempre retorna o mesmo valor que `len` para os *arrays*.

Vamos dar uma olhada em como adicionar elementos a uma *`slice`* alterando o tamanho e a capacidade.

```go
package main

import "fmt"

func main() {
	var x []int
	fmt.Println(x, len(x), cap(x))
	x = append(x, 10)
	fmt.Println(x, len(x), cap(x))
	x = append(x, 20)
	fmt.Println(x, len(x), cap(x))
	x = append(x, 30)
	fmt.Println(x, len(x), cap(x))
	x = append(x, 40)
	fmt.Println(x, len(x), cap(x))
	x = append(x, 50)
	fmt.Println(x, len(x), cap(x))
}
```

Ao executar o código, você verá a seguinte saída. Observe como e quando a capacidade aumenta:

```
[] 0 0
[10] 1 1
[10 20] 2 2
[10 20 30] 3 4
[10 20 30 40] 4 4
[10 20 30 40 50] 5 8
```

Embora seja bom que as *`slices`* cresçam automaticamente, é muito mais eficiente dimensioná-las apenas uma vez. Se você sabe quantos elementos planeja colocar em uma *`slice`*, crie a *`slice`* com a capacidade inicial correta. Fazemos isso com a função `make`.

### `make`

A função `make` do *Go* permite que você defina o tamanho e a capacidade de uma *`slice`* ao criá-la. A sintaxe é semelhante a `make(<sliceType>, <length>, <capacity>)`. Ao criar uma *`slice`* usando `make`, o argumento `<capacity>` é opcional, mas o `<length>` é obrigatório.

Já vimos duas maneiras de declarar uma *`slice`*, usando um literal de *`slice`* ou `nil` *zero value*. Embora útil, nenhuma dessas maneira permite criar uma *`slice`* vazia que já tenha um tamanho ou capacidade especificada. Esse é o trabalho da função `make`. Permite-nos especificar o tipo, tamanho e, opcionalmente, a capacidade. Vamos dar uma olhada:

```go
x := make([]int, 5)
```

Isso cria uma *`slice`* com tamanho de 5 e capacidade de 5. Como tem tamanho de 5, `x[0]` a `x[4]` são elementos válidos e todos são inicializados com `0`.

Usando a função `make`, criaremos algumas *`slices`* e exibiremos seu tamanho e capacidade:

```go
package main

import "fmt"

func genSlices() ([]int, []int, []int) {
	var s1 []int

	// Define a slice using make and set only the length:
	s2 := make([]int, 10)
	// Define a slice that uses both the length and capacity of the slices:
	s3 := make([]int, 10, 50)

	return s1, s2, s3
}

func main() {
	s1, s2, s3 := genSlices()

	fmt.Printf("s1: len = %v cap = %v\n", len(s1), cap(s1))
	fmt.Printf("s2: len = %v cap = %v\n", len(s2), cap(s2))
	fmt.Printf("s3: len = %v cap = %v\n", len(s3), cap(s3))
}
```

Executar o código anterior produz a seguinte saída:

```
s1: len = 0 cap = 0
s2: len = 10 cap = 10
s3: len = 10 cap = 50
```

No código anterior, usamos `make`, `len` e `cap` para controlar e exibir o tamanho e a
capacidade de uma *`slice`* ao definir uma.

Um erro comum é tentar preencher esses elementos iniciais usando `append`.

```go
x := make([]int, 5) // [0 0 0 0 0]
x = append(x, 10)
```

O número 10 é colocado no final da *`slice`*, após os *zero values* nas posições `0-4` porque `append` sempre aumenta o tamanho de uma *`slice`*. O valor de `x` agora é `[0 0 0 0 0 10]`, com tamanho de 6 e capacidade de 10 (a capacidade foi dobrada assim que o 6º elemento foi anexado).

Também podemos especificar uma capacidade inicial com `make`:

```go
x := make([]int, 5, 10)
```

Isso cria uma *`slice`* de inteiros com tamanho de 5 e capacidade de 10.

Você também pode criar uma *`slice`* com tamanho zero, mas uma capacidade maior que zero:

```go
x := make([]int, 0, 10)
```

Nesse caso, temos uma *`slice`* non-nil com tamanho de zero, mas capacidade de 10. Como o tamanho é 0, não podemos indexar diretamente, mas podemos acrescentar valores:

```go
x := make([]int, 0, 10)
x = append(x, 5, 6, 7, 8)
```

O valor da variável `x` agora é `[5 6 7 8]`, com tamanho de 4 e capacidade de 10.

#### Mais de `make`

Se você tem uma boa ideia do tamanho que sua *`slice`* precisa ter, mas não sabe quais serão esses valores quando você estiver escrevendo o programa, use `make`. A questão então é se você deve especificar um comprimento diferente de zero na chamada para `make` ou especificar um comprimento zero e uma capacidade diferente de zero. Existem três possibilidades:

1. Se você estiver usando uma *`slice`* como *buffer*, especifique um comprimento diferente de zero.

2. Se você tiver certeza de que sabe o tamanho exato que deseja, pode especificar o comprimento da *`slice`* para definir os valores. Isso geralmente é feito ao transformar valores em uma *`slice`*. A desvantagem dessa abordagem é que, se você tiver o tamanho errado, acabará com *zero values* no final da *`slice`* ou erros de *panic* ao tentar acessar elementos que não existem.

3. Em outras situações, use `make` com comprimento zero e uma capacidade especificada. Isso permite que você use `append` para adicionar itens à *`slice`*. Se o número de itens acabar sendo menor, você não terá *zero value* no final. Se o número de itens for maior, seu código não entrará em *panic*.

### Slicing *Slices* - Criação de *Slices* a partir de *Slices* e *Arrays*

Uma expressão de *`slice`* cria uma *`slice`* de uma *`slice`*. É escrito entre colchetes e consiste em um offset inicial e um offset final, separados por dois pontos (`:`). A notação mais comum é `[:]`. Se você não especificar o valor do offset inicial, `0` será assumido. Da mesma forma, se você não especificar valor do offset final, até o final da *`slice`* será utilizado. Você pode ver como isso funciona executando o seguinte código:

```go
package main

import "fmt"

func main() {
	x := []int{1, 2, 3, 4}
	y := x[:2]
	z := x[1:]
	d := x[1:3]
	e := x[:]
	fmt.Println("x:", x)
	fmt.Println("y:", y)
	fmt.Println("z:", z)
	fmt.Println("d:", d)
	fmt.Println("e:", e)
}
```

Ele fornece a seguinte saída:

```
x: [1 2 3 4]
y: [1 2]
z: [2 3 4]
d: [2 3]
e: [1 2 3 4]
```

No código abaixo, usaremos a notação de intervalo de *`slice`* para criar *`slices`* com uma variedade de valores. Normalmente, no código das aplicações, você precisa trabalhar apenas com uma pequena parte de um *`slice`* ou *array*. A notação de intervalo é uma maneira rápida e direta de obter apenas os dados de que você precisa.

```go
package main

import "fmt"

func message() string {
	s := []int{1, 2, 3, 4, 5, 6, 7, 8, 9}

	m := fmt.Sprintln("First   :", s[0], s[0:1], s[:1])
	m += fmt.Sprintln("Last    :", s[len(s)-1], s[len(s)-1:len(s)], s[len(s)-1:])
	m += fmt.Sprintln("First 5 :", s[:5])
	m += fmt.Sprintln("Last 4  :", s[5:])
	m += fmt.Sprintln("Middle 5:", s[2:7])

	return m
}

func main() {
	fmt.Print(message())
}
```

Executar o código anterior produz a seguinte saída:

```
First   : 1 [1] [1]
Last    : 9 [9] [9]
First 5 : [1 2 3 4 5]
Last 4  : [6 7 8 9]
Middle 5: [3 4 5 6 7]
```

No código anterior, tentamos algumas maneiras de criar *`slices`* a partir de outra *`slice`*. Você também pode usar essas mesmas técnicas em um *array* como outro *array*. Vimos que os índices inicial e final são opcionais. Se você não tiver um índice inicial, ele começará no início da *`slice`* ou *array* de origem. Se você não tiver o índice final, ele irá até no final do *array*. Se você não especificar os índices inicial e final, será feita uma cópia da *`slice`* ou *array*. Este truque é útil para transformar *arrays* em um *slice*, mas não é útil para copiar *`slices`* porque as duas *`slices`* compartilham o mesmo endereço de memória.

#### Slices share storage sometimes

Quando você cria uma *`slice`* de uma outra *`slice`*, não está fazendo uma cópia dos dados. Em vez disso, agora você tem duas variáveis ​​que estão compartilhando o mesmo endereço de memória. Isso significa que as alterações em um elemento de uma *`slice`* afetam todas as *`slices`* que compartilham esse elemento. Vamos ver o que acontece quando alteramos os valores.

```go
package main

import "fmt"

func main() {
	x := []int{1, 2, 3, 4}
	y := x[:2]
	z := x[1:]
	x[1] = 20
	y[0] = 10
	z[1] = 30
	fmt.Println("x:", x)
	fmt.Println("y:", y)
	fmt.Println("z:", z)
}
```

Você obtém a seguinte saída:

```
x: [10 20 30 4]
y: [10 20]
z: [20 30 4]
```

Como você pode ver, alterar `x` modificou `y` e `z`, enquanto alterações em `y` e `z` modificaram `x`.

### Convertendo *Arrays* em *Slices*

*Slices* não são a única coisa que você pode fatiar. Se você tiver um *array*, poderá obter uma *`slice`* usando uma *slice expression*. Esta é uma maneira útil de conectar um *array* a uma função que espera receber *`slices`*. No entanto, esteja ciente de que tirar uma *`slice`* de uma *array* tem as mesmas propriedades de compartilhamento de memória que tirar uma *`slice`* de uma outra *`slice`*.

```go
package main

import (
	"fmt"
)

func main() {
	x := [4]int{5, 6, 7, 8}
	y := x[:2]
	z := x[2:]
	x[0] = 10
	fmt.Println("x:", x)
	fmt.Println("y:", y)
	fmt.Println("z:", z)
}
```

Você obtém o resultado:

```
x: [10 6 7 8]
y: [10 6]
z: [7 8]
```

### `copy`

Se você precisar criar uma nova *`slice`* independente do original, use a função de `copy`. Vamos dar uma olhada em um simples exemplo.

```go
package main

import "fmt"

func main() {
	x := []int{1, 2, 3, 4}
	y := make([]int, 4)
	num := copy(y, x)
	fmt.Println(y, num)
}
```

Você obtém o resultado:

```
[1 2 3 4] 4
```

A função `copy` leva dois parâmetros. O primeiro é a *`slice`* de destino e o segundo é a *`slice`* de origem. Ele copia quantos valores puder da origem ao destino, limitado por qualquer *`slice`* que seja menor, e retorna o número de elementos copiados. A capacidade de `x` e `y` não importa, é o tamanho/comprimento que importa.

Você não precisa copiar uma *`slice`* inteira. O código a seguir copia os dois primeiros elementos de uma *`slice`* de quatro elementos de uma *`slice`* de dois elementos:

```go
x := []int{1, 2, 3, 4}
y := make([]int, 2)
num = copy(y, x)
```

A variável `y` é definida como `[1 2]` e `num` é definido como 2.

Você também pode copiar do meio da *`slice`* de origem:

```go
x := []int{1, 2, 3, 4}
y := make([]int, 2)
copy(y, x[2:])
```

Estamos copiando o terceiro e o quarto elementos em `x`, obtendo uma *`slice`* da *`slice`*. Observe também que não atribuímos a saída da `copy` a uma variável. Se você não precisa do número de elementos copiados, não precisa atribuí-lo.

A função de `copy` permite que você copie entre duas *`slices`* que cobrem as seções sobrepostas de uma *`slice`* subjacente:

```go
x := []int{1, 2, 3, 4}
num = copy(x[:3], x[1:])
fmt.Println(x, num)
```

Neste caso, estamos copiando os últimos três valores em `x` sobre os três primeiros valores de `x`. Isso imprime `[2 3 4 4] 3`.

Você pode usar `copy` com *array* pegando uma parte do *array*. Você pode tornar o *array* a origem ou o destino de `copy`.

```go
x := []int{1, 2, 3, 4}
d := [4]int{5, 6, 7, 8}
y := make([]int, 2)
copy(y, d[:])
fmt.Println(y)
copy(d[:], x)
```

A primeira `copy` copia os dois últimos valores do *array* `d` para a *`slice`* `y`. A segunda copia todos os valores da *`slice`* `x` para o *array* `d`. Isso produz a saída:

```
[5 6]
[1 2 3 4]
```

## *Maps*

As *`slices`* são úteis quando você tem dados sequenciais. Como a maioria das linguagens, *Go* fornece um tipo de dado para situações em que você deseja associar um valor a outro. O tipo de *`map`* é escrito usando a notação `map[<keyType>]<valueType>`. Vamos dar uma olhada em algumas maneiras diferentes de declarar *`maps`*. Primeiro, você pode usar uma declaração `var` para criar uma variável de *`map`* definida com *zero value*:

```go
var nilMap map[string]int
```

Nesse caso, a variável `nilMap` declara um *`map`* com chaves de `string` e valores `int`. O *zero value* para um *`map`* é `nil`. Um *`map`* `nil` tem um tamanho de `0`. A tentativa de ler um mapa `nil` sempre retorna *zero value* para o tipo de valor do mapa. No entanto, a tentativa de gravar em uma variável de mapa como `nil` causa pânico.

Podemos usar a declaração `:=` para criar uma variável de *`map`* atribuindo a ela um *map literal*:

```go
totalWins := map[string]int{}
```

Nesse caso, estamos usando um *map literal* vazio. Não é o mesmo que um *`map`* como `nil`. Ele tem tamanho zero, mas você pode ler e escrever um *`map`* criado como um *map literal* vazio. Esta é a aparência de um *map literal* não vazio:

```go
teams := map[string][]string {
	"Orcas": []string{"Fred", "Ralph", "Bijou"},
	"Lions": []string{"Sarah", "Peter", "Billie"},
	"Kittens": []string{"Waldo", "Raul", "Ze"},
}
```

O corpo de um *map literal* é escrito com a chave, seguido por dois pontos (`:`) e o seu valor. Há uma vírgula separando cada par de chave-valor no mapa, mesmo na última linha. Neste exemplo, o valor é uma *`slice`* de *strings*. *O tipo do valor em um mapa pode ser qualquer coisa*. Existem algumas restrições aos tipos de chaves que discutiremos em breve.

Se você sabe quantos pares de chave-valor pretende colocar no *`map`*, mas não sabe os valores exatos, pode usar `make` para criar um *`map`* com um tamanho padrão:

```go
ages := make(map[int][]string, 10)
```

Os *`maps`* criados com `make` ainda têm tamanho de `0` e podem crescer além do tamanho especificado inicialmente.

Os *`maps`* são como *`slices`* de várias maneiras:

- Os *`maps`* aumentam automaticamente à medida que você adiciona pares de chave-valor a eles.
- Se você sabe quantos pares de chave-valor planeja inserir em um *`map`*, você pode usar `make` para criar um *`map`* com um tamanho inicial específico.
- Passar um *`map`* para a função `len`, informará o número de pares de chave-valor.
- O *zero value* para um *`map`* é `nil`.
- Os *`maps`* não são comparáveis. Você pode verificar se eles são iguais a `nil`, mas não pode verificar se dois *`maps`* têm chaves e valores idênticos usando `==` ou diferem usando `!=`.

A chave de um *`map`* pode ser qualquer tipo comparável. Isso significa que você não pode usar uma *`slice`* ou outro *`map`* como chave para um *`map`*.

Quando você deve usar um *`map`* e quando você deve usar uma *`slice`*? As *`slices`* são para listas de dados, especialmente para dados que são processados ​​sequencialmente. Os *`maps`* são úteis quando você tem dados organizados/relacionados de acordo com um valor e que a ordem não importa.

*DICA: use um `map` quando a ordem dos elementos não for importante. Use uma `slice` quando a ordem dos elementos for importante.*

### Ler e escrever

Você nem sempre saberá se existe uma chave em um *`map`* antes de precisar usá-la para obter um valor. Quando você está obtendo um valor para uma chave que não existe em um *`map`*, *Go* retorna o *zero value* para o tipo de valor do *`map`*. Ter uma lógica que manipule o *zero values* é uma forma válida de programar em *Go*, mas nem sempre é possível. Se você não puder usar a lógica de verificação do *zero value*, os *`maps`* podem retornar um valor de retorno extra. A notação parece com `<value>, <exists_value> := <map>[<key>]`.`<exists_value>` é um valor *booleano* que é verdadeiro se houver uma chave no *`map`*; caso contrário, é falso. Ao fazer um loop, você deve usar a palavra-chave `range`. Ao percorrer um *`map`*, nunca confie na ordem dos itens nele. *Go* não garante a ordem dos itens em um *`map`*. Para garantir que ninguém tenha responsabilidade sobre a ordem dos elementos, *Go* propositadamente randomiza a ordem deles quando é percorrido. Se você precisar fazer um loop sobre os elementos de seu *`map`* em uma ordem específica, precisará usar um *array* ou *`slice`* para ajudá-lo com isso.

Vejamos um pequeno programa que declara, escreve e lê de um mapa:

```go
package main

import "fmt"

func main() {
	totalWins := map[string]int{}
	totalWins["Orcas"] = 1
	totalWins["Lions"] = 2
	fmt.Println(totalWins["Orcas"])
	fmt.Println(totalWins["Kittens"])
	totalWins["Kittens"]++
	fmt.Println(totalWins["Kittens"])
	totalWins["Lions"] = 3
	fmt.Println(totalWins["Lions"])
}
```

Ao executar o código acima, você verá a seguinte saída:

```
1
0
1
3
```

Atribuímos um valor a uma chave do *`map`* colocando a chave entre colchetes e usando o sinal de atribuição `=` para especificar o valor e lemos o valor do *`map`* colocando a chave entre colchetes. Observe que você não pode usar `:=` para atribuir um valor a uma chave do *`map`*.

Quando tentamos acessar uma chave do *`map`* que nunca foi definida, o *`map`* retorna o *zero value* para o tipo de seu valor. Nesse caso, o tipo do valor é um `int`, então obtemos um `0`. Você pode usar o operador `++` para incrementar um valor numérico. Como um *`map`* retorna o *zero value* por padrão, isso funciona mesmo quando não há nenhum valor existente associado à chave.

### *Comma ok Idiom*

Como vimos, um *`map`* retorna o *zero value* se você solicitar o valor associado a uma chave que não está no *`map`*. Isso é útil ao implementar coisas como o contador que vimos anteriormente. No entanto, às vezes você precisa descobrir se uma chave existe em um *`map`*. *Go* fornece o *comma ok idiom* para dizer a diferença entre uma chave que está associada a um *zero value* e uma chave que não está no *`map`*. Use a seguinte notação para fazer isso: `<value>, <exists_value> := <map>[<key>]`.

```go
m := map[string]int{
	"hello": 5,
	"world": 0,
}

v, ok := m["hello"]
fmt.Println(v, ok)  // 5 true

v, ok = m["world"]
fmt.Println(v, ok)  // 0 true

v, ok = m["goodbye"]
fmt.Println(v, ok)  // 0 false
```

Em vez de atribuir o resultado de uma leitura do *`map`* a uma única variável, com o *comma ok idiom* você atribui os resultados de uma leitura do *`map`* a duas variáveis. O primeiro obtém o valor associado à chave. O segundo valor retornado é um `bool`. Geralmente é denominado `ok`. Se `ok` for verdadeiro, a chave está presente no *`map`*. Se `ok` for falso, a chave não está presente. Neste exemplo, o código imprime `5 true, 0 true, and 0 false`.

Vamos ver outro exemplo, criando uma lógica para imprimir os dados do *`map`* com base na chave que é passada para o programa.

```go
package main

import (
	"fmt"
	"os"
)

// Define a map and initialize it with data. Then, return the map and close the function:
func getUsers() map[string]string {
	return map[string]string{
		"305": "User 1",
		"204": "User 2",
		"631": "User 3",
		"073": "User 4",
	}
}

func getUser(id string) (string, bool) {
	users := getUsers()

	user, exists := users[id]

	return user, exists
}

func main() {
	if len(os.Args) < 2 {
		fmt.Println("User ID not passed")
		os.Exit(1)
	}

	userID := os.Args[1]
	name, exists := getUser(userID)

	// If the key is not found, print a message, and then print all the users using a range loop. After that, exit:
	if !exists {
		fmt.Printf("Passed user ID (%v) not found.\nUsers:\n", userID)
		for key, value := range getUsers() {
			fmt.Println("    ID:", key, "Name:", value)
		}
		os.Exit(1)
	}

	fmt.Println("Name:", name)
}
```

Executar o código com o seguinte comando:

```go
go run main.go 123
```

Produz a seguinte saída:

```
Passed user ID (123) not found.
Users:
	ID: 073 Name: User 4
	ID: 305 Name: User 1
	ID: 204 Name: User 2
	ID: 631 Name: User 3
exit status 1
```

Executar o código com o seguinte comando:

```go
go run main.go 305
```

Produz a seguinte saída:

```
Name: User 1
```

Neste código acima, aprendemos como podemos verificar se existe uma chave em um *`map`*. Pode parecer um pouco estranho vindo de outras linguagens que exigem que você verifique a existência de uma chave antes de obter o valor, não depois. Essa maneira de fazer as coisas significa que há muito menos chance de erros em runtime. Se um *zero value* não for possível em sua lógica de domínio, você pode usar esse forma para verificar se existe uma chave.

Usamos um loop com `range` para imprimir todos os usuários no *`map`*. Sua saída provavelmente está em uma ordem diferente da saída mostrada do resultado acima. Isso se deve ao fato de *Go* randomiza a ordem dos elementos em um *`map`* quando você usa `range`.

### Removendo chaves

Para remover um elemento no *`map`*, precisamos usar a função `delete`. A assinatura da função de `delete`, quando usada com *`maps`*, é `delete(<map>, <key>)`. A função não retorna nada e se uma chave não existe, nada acontece.

```go
m := map[string]int{
	"hello": 5,
	"world": 10,
}

delete(m, "hello")
```

A função `delete` recebe um *`map`* e uma chave e, em seguida, remove o elemento com a chave especificada. Se a chave não estiver presente no *`map`* ou se o *`map`* for `nil`, nada acontece. A função de `delete` não retorna um valor.

No código abaixo, definiremos um *`map`* e, em seguida, excluiremos um elemento dele usando a entrada do usuário. Em seguida, imprimiremos o *`map`* no console sem o usuário que foi deletado:

```go
package main

import (
	"fmt"
	"os"
)

var users = map[string]string{
	"305": "User 1",
	"204": "User 2",
	"631": "User 3",
	"073": "User 4",
}

func deleteUser(id string) {
	delete(users, id)
}

func main() {
	if len(os.Args) < 2 {
		fmt.Println("User ID not passed")
		os.Exit(1)
	}

	userID := os.Args[1]

	deleteUser(userID)
	fmt.Println("Users:", users)
}
```

Execute o código usando o seguinte comando:

```go
go run . 305
```

Execução do código anterior produz a seguinte saída:

```
Users: map[073:User 4 204:User 2 631:User 3]
```

No código acima, usamos a função `delete` para remover um elemento de um *`map`*. Este requisito é exclusivo para mapas; você não pode usar a função `delete` em *arrays* ou *slices*.

## Simple Custom Types

*Go* permite que você declare um tipo em qualquer nível, do bloco de pacote para baixo. No entanto, você só pode acessar o tipo de dentro de seu escopo. As únicas exceções são os tipos de nível de bloco de pacote exportados.

Você pode criar tipos personalizados usando os tipos simples do *Go* como ponto de partida. A notação é `type <name> <type>`. Se tivéssemos que criar um tipo de ID baseado em uma string, isso se pareceria com `type id string`. O tipo personalizado age da mesma forma que o tipo no qual você o baseou, incluindo obter o mesmo *zero value* e ter as mesmas habilidades para comparar com outros valores do mesmo tipo. Um tipo personalizado não é compatível com seu tipo base, mas você pode converter seu tipo personalizado de volta no tipo base para permitir a interação.

No código abaixo, definiremos um *`map`* e, em seguida, excluiremos um elemento usando o input do usuário. Em seguida, imprimiremos o *`map`* menor no console.

```go
package main

import "fmt"

type id string

func getIDs() (id, id, id) {
	var id1 id
	var id2 id = "1234-5678"
	var id3 id
	id3 = "1234-5678"

	return id1, id2, id3
}

func main() {
	id1, id2, id3 := getIDs()

	fmt.Println("id1 == id2        :", id1 == id2)
	fmt.Println("id2 == id3        :", id2 == id3)
	fmt.Println("id2 == \"1234-5678\":", string(id2) == "1234-5678")
}
```

Executar o código anterior produz a seguinte saída:

```
id1 == id2        : false
id2 == id3        : true
id2 == "1234-5678": true
```

Neste exemplo, criamos um tipo personalizado, configuramos seus dados, em seguida, comparamos com valores do mesmo tipo e com seu tipo base.

*Tipos personalizados são uma parte fundamental da modelagem dos problemas de dados que você verá nas aplicações. Ter tipos projetados para refletir os dados com os quais você precisa trabalhar ajuda a manter seu código fácil de entender e manter.*

## Structs

Os mapas são uma maneira conveniente de armazenar alguns tipos de dados, mas eles têm limitações. Eles não definem uma API, pois não há como restringir um mapa para permitir apenas certas chaves. Todos os valores em um mapa também devem ser do mesmo tipo. Por essas razões, os mapas não são a maneira ideal de passar dados de função para função. Quando você tiver dados relacionados que deseja agrupar, deve definir uma *`struct`*.

As coleções são perfeitas para agrupar valores do mesmo tipo e finalidade. Há uma outra maneira de agrupar dados no *Go* para diferentes finalidades. Frequentemente, uma *string* simples, número ou booleano não captura totalmente a essência dos dados que você terá.

Por exemplo, para o exemplo acima do mapa de usuário, um usuário era representado por seu ID exclusivo e seu primeiro nome. Raramente haverá detalhes suficientes para trabalhar com os registros do usuário. Os dados que você pode capturar sobre uma pessoa são quase infinitos. Seria possível armazenar esses dados em vários mapas, todos com a mesma chave, mas isso é difícil de trabalhar e manter.

A coisa ideal a fazer é coletar todos esses diferentes bits de dados em uma única estrutura de dados que você pode projetar e controlar. Esse é o tipo *`struct`* do *Go*: **é um tipo personalizado que você pode nomear e especificar as propriedades com seus tipos**.

*NOTA: Se você já conhece uma linguagem orientada a objetos, pode estar se perguntando sobre a diferença entre classes e `structs`. A diferença é simples: Go não tem classes, porque não tem herança. Isso não significa que Go não tenha alguns dos recursos das linguagens orientadas a objetos, ele deve fazer as coisas de uma maneira um pouco diferente.*

A notação das *`structs`* é a seguinte:

```
type <name> struct {
	<fieldName1> <type>
	<fieldName2> <type>
	…
	<fieldNameN> <type>
}
```

**Os nomes dos campos (`fieldName1`, `fieldName2` ...) devem ser exclusivos dentro de uma *`struct`***. Você pode usar qualquer tipo para um campo, incluindo ponteiros, coleções e outras *`structs`*.

Você pode acessar um campo em uma *`struct`* usando a seguinte notação: `<structValue>.<fieldName>`. Para definir o valor, você usa a notação: `<structValue>.<fieldName> = <value>`. Para ler um valor, você usa a seguinte notação: `value = <structValue>.<fieldName>`.

*Structs* é a coisa mais próxima que *Go* tem do que são chamadas de classes em outras linguagens, mas *`structs`* foram propositalmente mantidos pequenos pelos designers do *Go*. A principal diferença é que as *`structs`* não têm nenhuma forma de herança. Os designers do *Go* sentem que a herança causa mais problemas do que resolve no código.

Depois de definir seu tipo de *`struct`* personalizada, você pode usá-lo para criar um valor. Você tem várias maneiras de criar valores a partir de tipos de *`struct`*.

A maioria das linguagens tem um conceito semelhante a *`struct`* e a sintaxe que *Go* usa para ler e escrever *`structs`* deve ser familiar.

```go
type person struct {
	name string
	age int
}
```

Depois que um tipo de estrutura é declarado, podemos definir variáveis ​​desse tipo.

```go
var jonh person
```

Estamos usando uma declaração `var`. Como nenhum valor é atribuído a `jonh`, ele obtém o *zero value* para o tipo da *`struct`* de `person`. Uma *`struct`* *zero value* tem cada campo definido com o seu próprio *zero value*.

Um literal de *`struct`* também pode ser atribuído a uma variável.

```go
bob := person{}
```

Ao contrário dos mapas, não há diferença entre atribuir um literal de struct vazia e não atribuir um valor. Ambos inicializam todos os campos na *`struct`* com os *zero values* dos campos. Existem dois estilos diferentes para um literal de *`struct`* não vazio. Um literal de *`struct`* pode ser especificado como uma lista de valores separados por vírgulas para os campos entre colchetes:

```go
jonh := person {
	"Jonh",
	30,
}
```

Ao usar este formato literal de *`struct`*, um valor para cada campo na *`struct`* deve ser especificado e os valores são atribuídos aos campos na ordem em que foram declarados na definição da *`struct`*.

O segundo estilo literal de *`struct`* se parece com o estilo literal de *`map`*:

```go
nick := person {
	age: 30,
	name: "Nick",
}
```

Você usa os nomes dos campos na *`struct`* para especificar os valores. Qualquer campo não especificado é definido com seu *zero value*. Você não pode misturar os dois estilos literais de *`struct`*; ou todos os campos são especificados com chaves ou nenhum deles é. Para pequenas *`structs`* onde todos os campos são sempre especificados, o estilo literal de estrutura mais simples é adequado. Em outros casos, use os nomes das chaves. É mais detalhado, mas deixa claro qual valor está sendo atribuído a qual campo sem precisar fazer referência à definição da *`struct`*. Também é mais fácil de manter. Se você inicializar uma *`struct`* sem usar os nomes de campo e uma versão futura precisar adicionar campos adicionais, seu código não será mais compilado.

Agora, se você deseja acessar as diferentes partes dos dados armazenados na *`struct`*, você pode simplesmente usar o operador de ponto "`.`":

```go
bob.name = "Bob"
fmt.Println(bob.name)
```

Assim como usamos colchetes para ler e escrever em um *`map`*, usamos a notação de ponto para ler e escrever nos campos da *`struct`*.

No exemplo de código abaixo, vamos definir uma *`struct`* de usuário. Vamos definir alguns campos de diferentes tipos. Em seguida, criaremos alguns valores da *`struct`* usando alguns métodos diferentes.

```go
package main

import "fmt"

type user struct {
	name    string
	age     int
	balance float64
	member  bool
}

func getUsers() []user {
	u1 := user{
		name:    "User 1",
		age:     20,
		balance: 78.43,
		member:  true,
	}

	u2 := user{
		age:  19,
		name: "User 2",
	}

	u3 := user{
		"User 3",
		25,
		0,
		false,
	}

	var u4 user
	u4.name = "User 4"
	u4.age = 31
	u4.member = true
	u4.balance = 17.09

	return []user{u1, u2, u3, u4}
}

func main() {
	for key, user := range getUsers() {
		fmt.Printf("%v: %#v\n", key, user)
	}
}
```

Executar o código anterior produz a seguinte saída:

```
0: main.user{name:"User 1", age:20, balance:78.43, member:true}
1: main.user{name:"User 2", age:19, balance:0, member:false}
2: main.user{name:"User 3", age:25, balance:0, member:false}
3: main.user{name:"User 4", age:31, balance:17.09, member:true}
```

No código anterior, você definiu um tipo de *`struct`* personalizada que continha vários campos, cada um de um tipo diferente. Em seguida, criamos valores dessa estrutura usando alguns métodos diferentes. Cada um desses métodos é válido e útil em diferentes contextos.

Definimos a *`struct`* no escopo do pacote e, embora não seja típico, você pode definir os tipos de *`struct`* no escopo da função também. Se você definir uma *`struct`* em uma função, ela só será válida para uso nessa função. Ao definir um tipo no nível do pacote, ele está disponível para uso em todo o pacote.

**Transformando uma *`struct`* por meio de seu ponteiro passado para uma função**:

```go
type User struct {
	name string
	age  int
}

func main() {
	newUser := User{"User 1", 19}
	fmt.Println("Old Age", newUser.age)
	incrementAge(&newUser)
	fmt.Println("Age", newUser.age)
}

func incrementAge(user *User) {
	user.age++
	fmt.Println("New Age", user.age)
}
```

```
Old Age 19
New Age 20
Age 20
```

### Estruturas anônimas

Você também pode declarar que uma variável implementa um tipo de *`struct`* sem primeiro dar um nome ao tipo da *`struct`*. Isso é chamado de *estrutura anônima*.

```go
var person struct {
	name string
	age int
}

person.name = "bob"
person.age = 50

pet := struct {
	name string
	kind string
} {
	name: "Dog name",
	kind: "dog",
}
```

Neste exemplo, os tipos das variáveis `person` e `pet` são **estruturas anônimas**. Você atribui (e lê) campos em uma *estrutura anônima* da mesma forma que uma estrutura nomeada. Assim como você pode inicializar uma instância de uma estrutura nomeada com uma estrutura literal, você também pode fazer o mesmo para uma *estrutura anônima*.

Você pode se perguntar quando é útil ter tipo de dados que está associado apenas a uma única instância. Existem duas situações comuns em que **estruturas anônimas** são úteis. A primeira é quando você converte dados externos em uma estrutura ou uma estrutura em dados externos (como JSON ou buffers). Isso é chamado de **marshaling** (empacotamento) e **unmarshaling** (desempacotamento) de dados. Escrever testes é outro lugar onde **estruturas anônimas** aparecem.

### Comparando e convertendo estruturas

> Se uma *`struct`* é ou não comparável depende de seus campos. As *`structs`* inteiramente compostas de tipos comparáveis ​​são comparáveis; aqueles com campos de *`slices`* ou *`maps`* não são. Se a *`struct`* foi definida anonimamente e tem a mesma estrutura de uma *`struct`* nomeada, *Go* permite a comparação.

No exemplo de código abaixo, definiremos uma *`struct`* comparável e criaremos um valor com ela. Também definiremos e criaremos valores com *`structs`* anônimas que têm a mesma estrutura de nossa *`struct`* nomeada. Finalmente, iremos compará-los e imprimir os resultados no console.

```go
package main

import "fmt"

type point struct {
	x int
	y int
}

func compare() (bool, bool) {
	point1 := struct {
		x int
		y int
	}{
		10,
		10,
	}

	point2 := struct {
		x int
		y int
	}{}

	point2.x = 10
	point2.y = 5

	point3 := point{10, 10}

	return point1 == point2, point1 == point3
}

func main() {
	a, b := compare()

	fmt.Println("point1 == point2:", a)
	fmt.Println("point1 == point3:", b)
}
```

Executar o código anterior produz a seguinte saída:

```
point1 == point2: false
point1 == point3: true
```

No código anterior, vimos que podemos trabalhar com valores de *`struct`* anônimas da mesma maneira que tipos de *`struct`* nomeadas, incluindo suas comparações. Com tipos nomeados, você só pode comparar *`structs`* do mesmo tipo. Quando você compara tipos em *Go*, *Go* compara todos os campos para verificar se há uma correspondência. *Go* está permitindo que uma comparação dessas *`structs`* anônimas seja feita porque os nomes e tipos de campo correspondem. *Go* é um pouco flexível ao comparar *`structs`* como esta.

**Em *Go* não é permito comparações entre variáveis ​​de tipos primitivos diferentes**, *Go não permite comparações entre variáveis ​​que representam `structs` de tipos diferentes*. *Go* permite que você execute uma conversão de tipo, de um tipo de *`struct`* para outro se os campos de ambas as *`structs`* tiverem os mesmos nomes, ordem e tipos. Vamos ver o que isso significa. Dada esta *`struct`*:

```go
type firstPerson struct {
	name string
	age int
}
```

Podemos usar uma conversão de tipo para converter uma instância de `firstPerson` em `secondPerson`, mas não podemos usar `==` para comparar uma instância de `firstPerson` e uma instância de `secondPerson`, porque são tipos diferentes:

```go
type secondPerson struct {
	name string
	age int
}
```

Não podemos converter uma instância de `firstPerson` em `thirdPerson`, porque os campos estão em uma ordem diferente:

```go
type thirdPerson struct {
	age int
	name string
}
```

Não podemos converter uma instância de `firstPerson` em `fourthPerson` porque os nomes dos campos não correspondem:

```go
type fourthPerson struct {
	firstName string
	age int
}
```

Por fim, não podemos converter uma instância de `firstPerson` em `fifthPerson` porque há um campo adicional:

```go
type fifthPerson struct {
	name string
	age int
	favoriteColor string
}
```

As *`structs`* anônimas adicionam uma pequena diferença a isso: se duas variáveis ​​de *`struct`* estão sendo comparadas e pelo menos uma delas tem um tipo que é uma *`struct`* anônima, você pode compará-las sem uma conversão de tipo se os campos de ambas as *`structs`* tiverem os mesmos nomes, ordem, e tipos. Você também pode atribuir entre tipos de *`structs`* nomeadas e anônimas se os campos de ambas as *`structs`* tiverem os mesmos nomes, ordem e tipos.

```go
type firstPerson struct {
	name string
	age int
}

f := firstPerson{
	name: "Bob",
	age: 50,
}

var g struct {
	name string
	age int
}

// compiles -- can use = and == between identical named and anonymous structs
g = f
fmt.Println(f == g) // true
```

### Usando composição na *Struct* com incorporação (embedding)

Embora a herança não seja possível com *`structs`* em *Go*, os designers de *Go* incluíram uma alternativa interessante. A alternativa é incorporar tipos em tipos de *`struct`*. Usando a incorporação, você pode adicionar campos a uma *`struct`* a partir de outras *`structs`*. Este recurso de **composição** permite adicionar a uma *`struct`* usando outras *`structs`* como componentes. **Embedding** (incorporação) é diferente de ter um campo do tipo `struct`. Quando você embedded (incorpora), os campos da *`struct`* incorporada são promovidos. Depois de promovido, um campo age como se estivesse definido na *`struct`* de destino.

Para embed (incorporar) uma *`struct`*, você a adiciona como faria com um campo, mas não especifica um nome. Para fazer isso, você adiciona o nome do tipo da *`struct`* a outra *`struct`* sem dar a ela um nome do campo, que se parece com isto:

```
type <name> struct {
	<Type>
}
```

Há duas maneiras válidas de trabalhar com campos incorporados em uma estrutura: `<structValue>.<fieldName>` ou `<structValue>.<type>.<fieldName>`. *Essa capacidade de acessar o tipo por seu nome também significa que os nomes do tipo devem ser exclusivos entre os tipos incorporados e os nomes dos campos raiz*. Ao incorporar tipos de ponteiro, o nome do tipo é o tipo sem a notação de ponteiro, portanto, o nome `*<type>` torna-se `<type>`. O campo ainda é um ponteiro e apenas o nome é diferente.

Quando se trata de promoção, se você tiver qualquer sobreposição com os nomes do campo do seu `struct`, *Go* permite que você incorpore, mas a promoção do campo de sobreposição não acontece. Você ainda pode acessar o campo passando pelo caminho do nome do tipo.

Você não pode usar promoção ao inicializar *`structs`* com tipos incorporados. Para inicializar os dados, você deve usar o nome dos tipos incorporados.

No código abaixo, definiremos algumas *`structs`* e tipos personalizados. Vamos incorporar esses tipos em uma *`struct`*:

```go
package main

import (
	"fmt"
)

type name string

type location struct {
	x int
	y int
}

type size struct {
	width  int
	height int
}

type dot struct {
	name
	location
	size
}

func getDots() []dot {
	var dot1 dot

	dot2 := dot{}
	dot2.name = "A"
	dot2.x = 5
	dot2.y = 6
	dot2.width = 10
	dot2.height = 20

	// When initializing embedded types, you can't use promotion. For name, the result is the same but for location and size, you need to put more work into this:
	dot3 := dot{
		name: "B",
		location: location{
			x: 13,
			y: 27,
		},
		size: size{
			width:  5,
			height: 7,
		},
	}

	dot4 := dot{}
	dot4.name = "C"
	dot4.location.x = 101
	dot4.location.y = 209
	dot4.size.width = 87
	dot4.size.height = 43

	return []dot{dot1, dot2, dot3, dot4}
}

func main() {
	for key, dot := range getDots() {
		fmt.Printf("dot%v: %#v\n", (key + 1), dot)
	}
}
```

Executar o código anterior produz a seguinte saída:

```
dot1: main.dot{name:"", location:main.location{x:0, y:0}, size:main.size{width:0, height:0}}
dot2: main.dot{name:"A", location:main.location{x:5, y:6}, size:main.size{width:10, height:20}}
dot3: main.dot{name:"B", location:main.location{x:13, y:27}, size:main.size{width:5, height:7}}
dot4: main.dot{name:"C", location:main.location{x:101, y:209}, size:main.size{width:87, height:43}}
```

No exemplo de código acima, fomos capazes de definir uma *`struct`* complexa incorporando outros tipos a ela. A incorporação (**embedding**) permite reutilizar *`structs`* comuns, reduzindo o código duplicado, mas ainda fornecendo à *`struct`* uma simples API.

Não veremos muita incorporação (**embedding**) no código *Go* nas aplicações. Ele aparece, mas a complexidade e a exceção significam que os desenvolvedores *Go* preferem ter os outros *`structs`* como campos nomeados.
