---
id: 9be8d84b-331a-4f54-8679-3ada0485e7cb
title: Introdução a Ponteiros em Go
summary: "Veja os conceitos básico para o uso de ponteiros / referências no Go."
---

## Introdução

> *Um ponteiro é simplesmente uma variável que contém a localização na memória onde um valor está armazenado. Cada variável é armazenada em um ou mais locais da memória, chamados de endereços. Diferentes tipos de variáveis ​​podem ocupar diferentes quantidades de memória.*

> *Um ponteiro é simplesmente uma variável cujo conteúdo é o endereço onde outra variável está armazenada.*

```go
var x int32 = 10
var y bool = true
pointerX := &x
pointerY := &y
var pointerZ *string
```

Embora diferentes tipos de variáveis ​​possam ocupar diferentes lugares na memória, cada ponteiro, não importa para que tipo esteja apontando, é sempre do mesmo tamanho: *um número que contém a localização na memória onde os dados estão armazenados*.

O *zero value* para um ponteiro é `nil`. Vimos `nil` algumas vezes, como o *zero value* para *slices*, *maps* e *interfaces*. Todos esses tipos são implementados com ponteiros. Como já foi abordado, `nil` é um identificador não tipado que representa a falta de um valor para certos tipos. Ao contrário de `NULL` em `C`, `nil` não é outro nome para `0`; você não pode convertê-lo em um número.

A sintaxe de ponteiro em *Go* é parcialmente parecida com `C` e `C++`. Como *Go* tem um *garbage collector*, a maior parte da dor do gerenciamento de memória foi removida. Além disso, alguns dos truques que você pode fazer com ponteiros em `C` e `C++`, incluindo aritmética de ponteiros, não são permitidos no *Go*.

> O caractere `&` é o operador de *`address`*. Ele precede um tipo de valor e retorna o endereço do local da memória de onde o valor está armazenado.

```go
x := "hello"
pointerToX := &x
```

> O caractere `*` é o operador de *`indirection`*. Ele precede uma variável do tipo ponteiro e retorna o valor referênciado. Isso é chamado de *dereferencing*.

```go
x := 10
pointerToX := &x

fmt.Println(pointerToX)     // prints a memory address
fmt.Println(*pointerToX)    // prints 10

z := 5 + *pointerToX
fmt.Println(z) // prints 15
```

Antes de *dereferencing* um ponteiro, você deve certificar-se de que o ponteiro não é `nil`. Seu programa entrará em pânico se você tentar acessar a referência de um ponteiro `nil`.

```go
var x *int
fmt.Println(x == nil)   // prints true
fmt.Println(*x)         // panics
```

> Um tipo de ponteiro é um tipo que representa um ponteiro. É escrito com um `*` antes do nome do tipo. Um tipo de ponteiro pode ser baseado em qualquer tipo.

```go
x := 10
var pointerToX *int
pointerToX = &x
```

*A função interna `new` cria uma variável de ponteiro.* Ele retorna um ponteiro para uma instância de *zero-value* do tipo fornecido.

```go
var x = new(int)
fmt.Println(x == nil)   // prints false
fmt.Println(*x)         // prints 0
```

A função `new` raramente é usada. Para *`structs`*, use `&` antes do literal para criar uma instância de ponteiro. *Você não pode usar `&` antes de um literal primitivo (números, booleanos e strings) ou uma constante porque eles não têm um endereço de memória; eles só existem em tempo de compilação.* Quando você precisar de um ponteiro para um tipo primitivo, declare uma variável e aponte para ela:

```go
x := &Foo{}
var y string
z := &y
```

Não ser capaz de obter o endereço de uma constante às vezes é inconveniente. Se você tiver uma estrutura com um campo de um ponteiro para um tipo primitivo, não poderá atribuir um literal diretamente ao campo:

```go
type person struct {
	FirstName string
	MiddleName *string
	LastName string
}

p := person {
	FirstName: "Jonh",
	MiddleName: "Peterson", // This line won't compile
	LastName: "Silva",
}
```

Compilar este código retorna o erro:

```
cannot use "Peterson" (type string) as type *string in field value
```

Se você tentar colocar um `&` antes de “Peterson”, receberá a mensagem de erro:

```
cannot take the address of "Peterson"
```

Existem duas maneiras de contornar esse problema. A primeira é fazer o que mostramos acima, introduzir uma variável para manter o valor constante. A segunda maneira é escrever uma função auxiliar que recebe um tipo booleano, numérico ou de string e retorna um ponteiro para esse tipo:

```go
func stringp(s string) *string {
	return &s
}
```

Com essa função, agora você pode escrever:

```go
p := person {
	FirstName: "Jonh",
	MiddleName: stringp("Peterson"), // This works
	LastName: "Silva",
}
```

Por que isso funciona? Quando passamos uma constante para uma função, a constante é copiada para um parâmetro, que é uma variável. Por ser uma variável, ela possui um endereço na memória. A função então retorna o endereço de memória da variável.

## Ponteiros Indicam Parâmetros Mutáveis

As constantes em *Go* fornecem nomes para expressões literais que podem ser calculadas em tempo de compilação. Não há mecanismo na linguagem para declarar que outros tipos de valores que são imutáveis.

A falta de declarações imutáveis ​​no *Go* pode parecer problemática, mas a capacidade de escolher entre os tipos de parâmetro de valor e ponteiro resolve o problema. Em vez de declarar que algumas variáveis ​​e parâmetros são imutáveis, *os desenvolvedores Go usam ponteiros para indicar que um parâmetro é mutável*.

Como *Go é uma linguagem de chamada por valor, os valores passados ​​para as funções são cópias*. Para tipos que não são ponteiros, como primitivos, *structs* e arrays, isso significa que a função chamada não pode modificar a variável original. Como a função chamada possui uma cópia dos dados originais, a imutabilidade dos dados originais é garantida.

No entanto, se um ponteiro é passado para uma função, a função obtém uma cópia do ponteiro. Isso aponta para os dados originais, o que significa que os dados originais podem ser modificados pela função chamada.

Existem algumas implicações relacionadas a isso.

A primeira implicação é que, quando você passa um ponteiro `nil` para uma função, não pode tornar o valor não `nil`. Você só pode reatribuir o valor se já houver um valor atribuído ao ponteiro. Embora confuso no início, faz sentido. Como a localização da memória foi passada para a função por meio de chamada por valor, não podemos alterar o endereço da memória, assim como não podemos alterar o valor de um parâmetro `int`. Isso é demonstrado da seguinte forma:

```go
func failedUpdate(g *int) {
	x := 10
	g = &x
}

func main() {
	var f *int      // f is nil
	failedUpdate(f)
	fmt.Println(f)  // prints nil
}
```

Começamos com uma variável `f` com seu valor inicial de `nil` na função `main`. Quando chamamos `failedUpdate`, copiamos o valor de `f`, que é `nil`, para o parâmetro denominado `g`. Isso significa que `g` também é definido como `nil`. Em seguida, declaramos uma nova variável `x` em `failedUpdate` com o valor 10. Em seguida, alteramos `g` em `failedUpdate` para apontar para `x`. Isso não altera `f` em `main`, e quando saímos de `failedUpdate` e retornamos a `main`, `f` ainda é `nil`.

A segunda implicação de copiar um ponteiro é que se você quiser que o valor atribuído a um parâmetro de ponteiro ainda esteja lá quando sair da função, você deve desreferenciar o ponteiro e definir o valor. Se você alterar o ponteiro, significa que alterou a cópia, não o original. A desreferenciação coloca o novo valor no local da memória apontado pelo original e pela cópia. Aqui está um pequeno programa que mostra como isso funciona:

```go
func failedUpdate(px *int) {
	x2 := 20
	px = &x2
}

func update(px *int) {
	*px = 20
}

func main() {
	x := 10
	failedUpdate(&x)
	fmt.Println(x) // prints 10
	update(&x)
	fmt.Println(x) // prints 20
}
```

Neste exemplo, começamos com `x` na função `main` com valor de 10. Quando chamamos `failedUpdate`, copiamos o endereço de `x` para o parâmetro `px`. Em seguida, declaramos `x2` em `failedUpdate`, com valor de 20. Em seguida, apontamos `px` em `failedUpdate` para o endereço da memória de `x2`. Quando retornamos a função `main`, o valor de `x` permanece o mesmo. Quando chamamos `update`, copiamos o endereço de `x` em `px` novamente. No entanto, desta vez, alteramos o valor para o qual `px` em `update` aponta, a variável `x` na função `main`. Quando retornamos a `main`, `x` foi alterada.

## Cuidado ao Usar Ponteiros

Dito isso, você deve ter cuidado ao usar ponteiros no *Go*. Conforme discutido anteriormente, eles tornam mais difícil entender o fluxo dos dados e podem criar trabalho extra para o *garbage collector*. *Em vez de preencher uma estrutura passando um ponteiro para ela em uma função, faça com que a função instancie e retorne a estrutura.*

Não faça isso:

```go
func MakeFoo(f *Foo) error {
	f.Field1 = "val"
	f.Field2 = 20

	return nil
}
```

faça isso:

```go
func MakeFoo() (Foo, error) {
	f := Foo {
		Field1: "val",
		Field2: 20,
	}

	return f, nil
}
```

*A única vez em que você deve usar parâmetros de ponteiro para modificar uma variável é quando a função espera uma interface*. Você vê esse padrão ao trabalhar com JSONs.

```go
f := struct {
	Name string `json:"name"`
	Age int `json:"age"`
}

err := json.Unmarshal([]byte(`{"name": "Jonh", "age": 30}`), &f)
```

A função `Unmarshal` preenche uma variável de *slice* de bytes contendo *JSON*. Ela é declarada contendo um *slice de bytes* e um parâmetro de `interface{}`. O valor passado para o parâmetro da `interface{}` deve ser um ponteiro. Se não for, um erro será retornado.

Como a integração JSON é tão comum, essa API às vezes é tratada como um caso comum por novos desenvolvedores *Go*, em vez da exceção que deveria ser.

*Ao retornar valores de uma função, você deve favorecer os tipos de valor. Use apenas um tipo de ponteiro como tipo de retorno se houver um estado dentro do tipo de dados que precise ser modificado.* Quando manipulamos I/O, vemos isso com buffers para leitura ou gravação de dados. Além disso, existem tipos de dados usados ​​com simultaneidade que sempre devem ser passados ​​como ponteiros.

## Desempenho de Ponteiros

Se uma estrutura for grande o suficiente, haverá melhorias de desempenho com o uso de ponteiros para a estrutura como um parâmetro ou um valor de retorno. O tempo para passar um ponteiro para uma função é constante para todos os tamanhos de dados, cerca de um nanossegundo. Isso faz sentido, pois o tamanho de um ponteiro é o mesmo para todos os tipos de dados. Passar um valor para uma função leva mais tempo à medida que os dados ficam maiores. Demora cerca de um milissegundo quando o valor atinge cerca de 10 megabytes de dados.

O comportamento para retornar um ponteiro versus retornar um valor é mais interessante. Para estruturas com dados menores do que um megabyte, é realmente mais lento para retornar um tipo de ponteiro do que um tipo de valor. Por exemplo, uma estrutura com dados de 100 bytes leva cerca de 10 nanossegundos para ser retornada, mas um ponteiro para essa estrutura leva cerca de 30 nanossegundos. Uma vez que suas estruturas são maiores do que um megabyte, a vantagem de desempenho muda. Demora quase 2 milissegundos para retornar 10 megabytes de dados, mas um pouco mais de meio milissegundo para retornar um ponteiro dele.

Você deve estar ciente de que esses tempos são muito curtos. Para a grande maioria dos casos, a diferença entre usar um ponteiro e um valor não afetará o desempenho do programa. Mas se você estiver passando megabytes de dados entre funções, considere o uso de um ponteiro, mesmo que os dados devam ser imutáveis.

## Zero Value vs `nil`

O outro uso comum de ponteiros em *Go* é indicar a diferença entre uma variável ou campo ao qual foi atribuído *zero value* e uma variável ou campo ao qual não foi atribuído nenhum valor. Se essa distinção for importante em seu programa, use um ponteiro `nil` para representar uma variável não atribuída ou campo de estrutura.

Como os ponteiros também indicam mutabilidade, tome cuidado ao usar esse padrão. Em vez de retornar um ponteiro definido como `nil` de uma função, use *comma ok idiom* e retorne o tipo de valor e um booleano.

Lembre-se, se um ponteiro `nil` é passado para uma função, você não pode definir o valor dentro da função, pois não há nenhum lugar para armazenar o valor. Se um valor não `nil` for passado para o ponteiro, não o modifique a menos que documente o comportamento.

*Use um valor de ponteiro para campos na estrutura que são `nullable`.*
