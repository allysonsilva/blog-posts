---
id: 9be8bd23-c372-4628-8c3c-50854961f2c9
title: Tipos, Métodos, e Interfaces em Go
summary: "Aprenda mais um pouco sobre structs e Interfaces."
---

## Métodos

Como a maioria das linguagens, *Go* oferece suporte a métodos em tipos definidos pelo usuário.

**Os métodos para um tipo são definidos no bloco no nível do pacote**:

```go
type Person struct {
	FirstName string
	LastName string
	Age int
}

func (p Person) String() string {
	return fmt.Sprintf("%s %s, age %d", p.FirstName, p.LastName, p.Age)
}
```

**As declarações dos métodos é semelhante às declarações de funções, com uma adição: a especificação do *receiver***. O *receiver* aparece entre a palavra-chave `func` e o nome do método. Assim como todas as outras declarações de variáveis, o nome do *receiver* aparece antes do tipo. Por convenção, o nome do *receiver* é uma abreviatura do tipo, geralmente sua primeira letra. *Não é idiomático usar `this` ou `self`*.

**Assim como as funções, os nomes dos métodos não podem ser sobrecarregados. Você pode usar os mesmos nomes dos métodos para tipos diferentes, mas não pode usar o mesmo nome do método para dois métodos no mesmo tipo**. Embora essa filosofia pareça limitada quando vinda de linguagens que têm *sobrecarga de método*, não reutilizar nomes faz parte da filosofia de *Go* de deixar claro o que seu código está fazendo.

**Os métodos devem ser declarados no mesmo pacote que seu tipo associado**; *Go* não permite adicionar métodos a tipos que você não controla. Embora você possa definir um método em um arquivo diferente dentro do mesmo pacote da declaração de tipo, é melhor manter sua definição de tipo e seus métodos associados juntos para que seja fácil a implementação.

As invocações de método devem parecer familiares para aqueles que usam métodos em outras linguagens:

```go
p := Person {
	FirstName: "Alyson",
	LastName:"Silva",
	Age: 30,
}
output := p.String()
```

### Métodos também são funções

*Os métodos em Go são tão parecidos com funções que você pode usar um método como um substituto para a função sempre que houver uma variável ou parâmetro de um tipo de função*.

Vamos começar com este simples tipo:

```go
type Adder struct {
	start int
}

func (a Adder) AddTo(val int) int {
	return a.start + val
}
```

Criamos uma instância do tipo da maneira usual e invocamos seu método:

```go
myAdder := Adder {start: 10}
fmt.Println(myAdder.AddTo(5))   // 15
```

Também podemos atribuir o método a uma variável ou passá-lo para um parâmetro do tipo `func(int) int`. Isso é chamado de ***method value***:

```go
f1 := myAdder.AddTo
fmt.Println(f1(10)) // 20
```

Um ***method value*** parece um pouco com uma **closure**, pois pode acessar os valores nos campos da instância a partir da qual foi criada.

Você também pode criar uma função a partir do próprio tipo. Isso é chamado de ***method expression***:

```go
f2 := Adder.AddTo
fmt.Println(f2(myAdder, 15)) // 25
```

No caso de uma ***method expression***, o primeiro parâmetro é o *receiver do método*; nossa assinatura de função é `func(Adder, int) int`.

*Method values e method expressions não são casos com muito uso.*

### Funções vs. Métodos

Visto que você pode usar um método como uma função, você pode se perguntar quando deve declarar uma função e quando deve usar um método.

*O diferenciador é se sua função depende ou não de outros dados. Como já cobrimos várias vezes, o estado no nível do pacote deve ser efetivamente imutável. Sempre que sua lógica depende de valores que são configurados na inicialização ou alterados enquanto seu programa está em execução, esses valores devem ser armazenados em uma estrutura e essa lógica deve ser implementada como um método. Se sua lógica depende apenas dos parâmetros de entrada, então deve ser uma função.*

*Tipos, pacotes, módulos, testes e injeção de dependência são conceitos inter-relacionados.*

### *Receivers* de ponteiro e valor

*Go* usa parâmetros do tipo de ponteiro para indicar que um parâmetro pode ser modificado pela função. As mesmas regras se aplicam aos *receivers* do método também. Eles podem ser *receivers* de ponteiro (o tipo é um ponteiro) ou *receivers* de valor (o tipo é um tipo de valor). As seguintes regras ajudam a determinar quando usar cada tipo de *receiver*:

- Se o seu método modifica o *receiver*, você deve usar um ***receiver de ponteiro***.
- Se o seu método precisa manipular instâncias `nil`, ele deve usar um ***receiver de ponteiro***.
- Se o seu método não modificar o *receiver*, você pode usar um ***receiver de valor***.

Usar ou não um *receiver de valor* para um método que não modifica o *receiver* depende dos outros métodos declarados no tipo. **Quando um tipo tem quaisquer métodos de *receiver* de ponteiro, uma prática comum é ser consistente e usar *receivers* de ponteiro para todos os métodos, mesmo aqueles que não modificam o *receiver***.

Abaixo está um código simples para demonstrar **receivers de ponteiro e valor**. Começaremos com um tipo que possui dois métodos, um usando um *receiver de valor* e o outro com um *receiver de ponteiro*:

```go
package main

import (
	"fmt"
	"time"
)

type Counter struct {
	total       int
	lastUpdated time.Time
}

func (c *Counter) Increment() {
	c.total++
	c.lastUpdated = time.Now()
}

func (c Counter) String() string {
	return fmt.Sprintf("total: %d, last updated: %v", c.total, c.lastUpdated)
}

func main() {
	var c Counter
	fmt.Println(c.String())
	c.Increment()
	fmt.Println(c.String())
}
```

Você deve ver a seguinte saída:

```
total: 0, last updated: 0001-01-01 00:00:00 +0000 UTC
total: 1, last updated: 2009-11-10 23:00:00 +0000 UTC m=+0.000000001
```

Uma coisa que você pode notar é que fomos capazes de chamar o método do *receiver de ponteiro*, embora `c` seja um *tipo de valor*. Quando você usa um *receiver de ponteiro* com uma variável local que é um *tipo de valor*, *Go* o converte automaticamente em um *tipo de ponteiro*. Nesse caso, `c.Increment()` é convertido em `(&c).Increment()`.

No entanto, esteja ciente de que as regras para passar valores para funções ainda se aplicam. Se você passar um *tipo de valor* para uma função e chamar um método de *receiver de ponteiro* no valor passado, você está chamando o método em uma cópia.

```go
package main

import (
	"fmt"
	"time"
)

type Counter struct {
	total       int
	lastUpdated time.Time
}

func (c *Counter) Increment() {
	c.total++
	c.lastUpdated = time.Now()
}

func (c Counter) String() string {
	return fmt.Sprintf("total: %d, last updated: %v", c.total, c.lastUpdated)
}

func doUpdateWrong(c Counter) {
	c.Increment()
	fmt.Println("in doUpdateWrong:", c.String())
}

func doUpdateRight(c *Counter) {
	c.Increment()
	fmt.Println("in doUpdateRight:", c.String())
}

func main() {
	var c Counter
	doUpdateWrong(c)
	fmt.Println("in main:", c.String())
	doUpdateRight(&c)
	fmt.Println("in main:", c.String())
}
```

Ao executar este código, você obterá a saída:

```
in doUpdateWrong: total: 1, last updated: 2009-11-10 23:00:00 +0000 UTC m=+0.000000001
in main: total: 0, last updated: 0001-01-01 00:00:00 +0000 UTC
in doUpdateRight: total: 1, last updated: 2009-11-10 23:00:00 +0000 UTC m=+0.000000001
in main: total: 1, last updated: 2009-11-10 23:00:00 +0000 UTC m=+0.000000001
```

O parâmetro em `doUpdateRight` é do tipo `*Counter`, que é uma **instância de ponteiro**. Como você pode ver, podemos chamar tanto `Increment` quanto `String` nele. *Go* considera que os *métodos de ponteiro* e *receiver de valor* estão no conjunto de métodos para uma **instância de ponteiro**. Para uma **instância de valor**, apenas os *métodos receivers de valor* estão no conjunto de métodos.

*NOTA: não escreva métodos `getter` e `setter` para estruturas Go, a menos que você precise deles para atender a uma interface. Go o incentiva a acessar diretamente um campo. Métodos para lógica de negócios. As exceções são quando você precisa atualizar vários campos como uma única operação ou quando a atualização não é uma atribuição direta de um novo valor. O método `Increment` definido anteriormente demonstra essas duas propriedades.*

### Codifique seus métodos para instâncias `nil`

Com as *instâncias de ponteiro*, o que pode fazer você se perguntar o que acontece quando você chama um método em uma instância `nil`. Na maioria das linguagens, isso produz algum tipo de erro.

*Go* faz algo um pouco diferente. Na verdade, ele tenta invocar o método. Se for um método com um *receiver de valor*, o programa entrará em pânico, pois não há valor sendo apontado pelo ponteiro. Se for um método com um *receiver de ponteiro*, ele pode funcionar se o método for escrito para lidar com a possibilidade de uma instância `nil`.

Em alguns casos, esperar um *receiver* `nil` torna o código mais simples. Aqui está uma implementação de uma árvore binária que aproveita os valores `nil` para o *receiver*:

```go
package main

import (
	"fmt"
)

type IntTree struct {
	val         int
	left, right *IntTree
}

func (it *IntTree) Insert(val int) *IntTree {
	if it == nil {
		return &IntTree{val: val}
	}
	if val < it.val {
		it.left = it.left.Insert(val)
	} else if val > it.val {
		it.right = it.right.Insert(val)
	}
	return it
}

func (it *IntTree) Contains(val int) bool {
	switch {
	case it == nil:
		return false
	case val < it.val:
		return it.left.Contains(val)
	case val > it.val:
		return it.right.Contains(val)
	default:
		return true
	}
}

func main() {
	var it *IntTree
	it = it.Insert(5)
	it = it.Insert(3)
	it = it.Insert(10)
	it = it.Insert(2)
	fmt.Println(it.Contains(2))  // true
	fmt.Println(it.Contains(12)) // false
}
```

É muito inteligente que *Go* permita que você chame um método em um *receiver* `nil`, e há situações em que isso é útil, como nosso exemplo de nó de árvore. No entanto, na maioria das vezes não é muito útil. Os *receivers de ponteiro* funcionam melhor com os parâmetros da função de ponteiro; é uma cópia do ponteiro passado para o método. Assim como nenhum parâmetro passado para funções, se você alterar a cópia do ponteiro, você não alterou o original. Isso significa que você não pode escrever um método de *receptor de ponteiro* que trate como `nil` e torne o ponteiro original não `nil`. Se o seu método tiver um *receptor de ponteiro* e não funcionar para um receptor `nil`, verifique se há `nil` e retorne um erro.

### Declarações de tipo não são herança

Além de declarar tipos e literais de estrutura, você também pode declarar um tipo definido pelo usuário com base em outro tipo definido pelo usuário:

```go
type HighScore Score
type Employee Person
```

Existem muitos conceitos que podem ser considerados "orientados a objetos", mas um se destaca: herança. *É aqui que o estado e os métodos de um tipo pai são declarados como disponíveis em um tipo filho e os valores do tipo filho podem ser substituídos pelo tipo pai*.

*Declarar um tipo baseado em outro tipo parece um pouco com herança, mas não é. Os dois tipos têm o mesmo tipo subjacente, mas isso não é tudo. Não há hierarquia entre esses tipos. Em linguagens com herança, uma instância filho pode ser usada em qualquer lugar que a instância pai possa ser usado. A instância filho também possui todos os métodos e estruturas de dados da instância pai*. Esse não é o caso em *Go*. Você não pode atribuir uma instância do tipo `HighScore` a uma variável do tipo `Score` ou vice-versa sem uma conversão de tipo, nem pode atribuir qualquer um deles a uma variável do tipo `int` sem uma conversão de tipo. Além disso, quaisquer métodos definidos no `Score` não são definidos no `HighScore`:

```go
// assigning untyped constants is valid
var i int = 300
var s Score = 100
var hs HighScore = 200
hs = s // compilation error!
s = i // compilation error!
s = Score(i) // ok
hs = HighScore(s) // ok
```

*Para tipos definidos pelo usuário cujos tipos subjacentes são tipos internos, o tipo declarado pelo usuário pode ser usado com os operadores para esses tipos. Como vimos acima, eles também podem receber literais e constantes compatíveis com o tipo subjacente.*

### Tipos são documentações

Embora seja bem entendido que você deve declarar um tipo de estrutura para conter um conjunto de dados relacionados, é menos claro quando você deve declarar um tipo definido pelo usuário com base em outros tipos integrados ou um tipo definido pelo usuário que é baseado em outro tipo definido. A resposta curta é que **os tipos são documentação**. **Eles tornam o código mais claro, fornecendo um nome para um conceito e descrevendo o tipo de dados que é esperado**. *É mais claro para alguém que lê seu código quando um método tem um parâmetro do tipo `Percentage` do que do tipo `int`, e é mais difícil para ele ser chamado com um valor inválido*.

*A mesma lógica se aplica ao declarar um tipo definido pelo usuário com base em outro tipo. Quando você tem os mesmos dados subjacentes, mas diferentes conjuntos de operações para executar, faça dois tipos. Declarar um como baseado no outro evita algumas repetições e deixa claro que os dois tipos estão relacionados.*

## Use Embedding para Composição

O conselho da engenharia de software "**Favorece a composição de objetos em vez da herança de classes**" remonta pelo menos ao livro de 1994 *Design Patterns de Gamma, Helm, Johnson e Vlissides*, mais conhecido como livro da *Gang of Four*. **Embora *Go* não tenha herança, ele incentiva a reutilização de código por meio de composição e promoção**:

```go
type Employee struct {
	Name string
	ID string
}
```

```go
func (e Employee) Description() string {
	return fmt.Sprintf("%s (%s)", e.Name, e.ID)
}
```

```go
type Manager struct {
	Employee
	Reports []Employee
}
```

```go
func (m Manager) FindNewEmployees() []Employee {
	// do business logic
}
```

Observe que `Manager` contém um campo do tipo `Employee`, mas nenhum nome é atribuído a esse campo. Isso torna `Employee` um campo **embedded** (*incorporado*). Quaisquer campos ou métodos declarados em um campo incorporado são **promoted** (*promovidos*) para a estrutura que os contém e podem ser chamados diretamente nele. Isso torna o seguinte código válido:

```go
m := Manager {
	Employee: Employee {
		Name: "Jonh",
		ID: "12345",
	},
	Reports: []Employee{},
}

fmt.Println(m.ID) // 12345
fmt.Println(m.Description()) // Jonh (12345)
```

*DICA: Você pode incorporar qualquer tipo em uma estrutura, não apenas outra estrutura. Isso promove os métodos do tipo incorporado à estrutura que o contém.*

Se a estrutura contida tiver campos ou métodos com o mesmo nome de um campo incorporado, você precisará usar o tipo do campo incorporado para se referir aos campos ou métodos ofuscados. Se você tiver tipos definidos assim:

```go
type Inner struct {
	X int
}

type Outer struct {
	Inner
	X int
}
```

Você só pode acessar o `X` no `Inner` especificando o `Inner` explicitamente:

```go
o := Outer {
	Inner: Inner {
		X: 10,
	},
	X: 20,
}

fmt.Println(o.X)        // 20
fmt.Println(o.Inner.X)  // 10
```

## Embedding Não é Herança

O suporte para *incorporação* é raro em linguagens de programação. Muitos desenvolvedores que estão familiarizados com herança (que está disponível em muitas linguagens) tentam entender a incorporação tratando-a como herança. Você não pode atribuir uma variável do tipo `Manager` a uma variável do tipo `Employee`. Se você deseja acessar o campo `Employee` no `Manager`, deve acessar explicitamente.

```go
package main

import (
	"fmt"
)

type Employee struct {
	Name string
	ID   string
}

func (e Employee) Description() string {
	return fmt.Sprintf("%s (%s)", e.Name, e.ID)
}

type Manager struct {
	Employee
	Reports []Employee
}

func (m Manager) FindNewEmployees() []Employee {
	// do business logic
	return nil
}

func main() {
	m := Manager {
		Employee: Employee {
			Name: "Jonh",
			ID:   "12345",
		},
		Reports: []Employee{},
	}
	var eFail Employee = m        // compilation error!
	var eOK Employee = m.Employee // ok!
}
```

Você obterá o erro:

```
cannot use m (type Manager) as type Employee in assignment
```

Além disso, não há *dispatch* dinâmico para tipos concretos em *Go*. Os métodos no campo incorporado não têm ideia de que estão incorporados. Se você tiver um método em um campo incorporado que chama outro método no campo incorporado, e a estrutura contida tiver um método com o mesmo nome, o método no campo incorporado não chamará o método na estrutura contida. Esse comportamento é demonstrado no código a seguir:

```go
package main

import "fmt"

type Inner struct {
	A int
}

func (i Inner) IntPrinter(val int) string {
	return fmt.Sprintf("Inner: %d", val)
}

func (i Inner) Double() string {
	result := i.A * 2
	return i.IntPrinter(result)
}

type Outer struct {
	Inner
	S string
}

func (o Outer) IntPrinter(val int) string {
	return fmt.Sprintf("Outer: %d", val)
}

func main() {
	o := Outer{
		Inner: Inner{
			A: 10,
		},
		S: "Hello",
	}
	fmt.Println(o.Double())
}
```

Executar este código produz a saída:

```
Inner: 20
```

*Embora incorporar um tipo concreto dentro de outro não permita que você trate o tipo externo como o tipo interno, os métodos em um campo incorporado contam para o conjunto de métodos da estrutura contida. Isso significa que eles podem fazer com que a estrutura contida implemente uma interface.*

## *Interfaces*

> Embora o modelo de simultaneidade do *Go* receba todo o destaque, a verdadeira estrela do design do *Go* são suas **interfaces implícitas**, *o único tipo abstrato em Go*.

**Uma *interface* é um conjunto de métodos que descreve o comportamento do tipo de dado. As interfaces definem o(s) comportamento(s) do tipo que deve ser satisfeito para implementar essa interface**. Um comportamento descreve o que esse tipo pode fazer. Quase tudo é sobre *comportamento*. Por exemplo, um gato pode miar, andar e pular. Todos esses são comportamentos de um gato. Um carro pode ligar, parar, virar e acelerar. Todos esses são comportamentos de um carro. Da mesma forma, **os comportamentos dos tipos são chamados de *métodos***.

Esses comportamentos são chamados de *methods sets*. Um comportamento é definido por um conjunto de métodos. Um *method set* é um grupo de métodos. Esses *method sets* incluem o nome do método, quaisquer parâmetros de entrada e quaisquer tipos de retorno.

![Graphic representation of interface elements](/images/articles/Go/Graphic%20representation%20of%20interface%20elements.png?id=4eae142e46ff891dee07952e8b8b94b6 "Graphic representation of interface elements")

Quando falamos sobre *comportamentos*, observe que não discutimos os *detalhes de implementação*. Os *detalhes de implementação* são omitidos ao definir uma interface. **É importante entender que nenhuma implementação é especificada ou imposta na declaração de uma interface**. Cada tipo que criamos que implementa uma interface pode ter seus próprios *detalhes de implementação*. Uma interface que possui um método chamado `Greeting()` pode ser implementada de diferentes maneiras por vários tipos. Um tipo de estrutura de pessoa pode implementar `Greeting()` de uma maneira diferente de um tipo da estrutura de animal. **As interfaces se concentram nos comportamentos que o tipo deve exibir/ter**. Não é função da interface fornecer método/como implementar. Esse é o trabalho do tipo que está implementando a interface. **Os tipos, geralmente uma estrutura, contêm os *detalhes de implementação* dos conjuntos de métodos**. Agora que temos um entendimento básico de uma interface, no próximo tópico, veremos como definir uma interface.

### Definindo uma *Interface*

Definir uma interface envolve as seguintes etapas:

![Defining an interface](/images/articles/Go/Defining%20an%20interface.png?id=5142ee12a6c61d4c93ba2f4415fe4a7e "Defining an interface")

Aqui está um exemplo de declaração de uma interface:

```go
type Speaker interface {
	Speak() string
}
```

Vejamos cada parte desta declaração:

- Comece com a palavra-chave `type`, seguida do nome e, em seguida, a palavra-chave `interface`.
- Estamos definindo um tipo de interface chamado `Speaker{}`. É idiomático em *Go* nomear a interface com um sufixo `er`, se for uma interface com um único método, é comum nomear a interface dessa forma.
- Em seguida, você define o conjunto de métodos. Definir um tipo de interface especifica os métodos que pertencem a ela. Nesta interface, estamos declarando um tipo de interface que possui um método chamado `Speak()` e retorna uma `string`.
- O *method set* da interface `Speaker{}` é `Speak()`.

A seguir, está uma interface que é usada com frequência no *Go*:

```go
// https://golang.org/pkg/io/#Reader
type Reader interface {
	Read(p []byte) (n int, err error)
}
```

Vejamos as partes do código acima:

- O nome da interface é `Reader{}`.
- O método definido é `Read()`.
- A assinatura do método `Read()` é `(p []byte) (n int, err error)`.

**As interfaces podem ter mais de um método como seus *method set***. Vejamos uma interface usada no pacote *Go*:

```go
// https://golang.org/pkg/os/#FileInfo
type FileInfo interface {
	Name() string // base name of the file
	Size() int64 // length in bytes for regular files; system-dependent for others
	Mode() FileMode // file mode bits
	ModTime() time.Time // modification time
	IsDir() bool // abbreviation for Mode().IsDir()
	Sys() interface{} // underlying data source (can return nil)
}
```

Como você pode ver, `FileInfo{}` tem vários métodos.

*Em resumo, as interfaces são tipos que declaram method sets (conjuntos de métodos). Semelhante a outras linguagens que utilizam interfaces, eles não implementam os conjuntos de métodos. Os detalhes de implementação não fazem parte da definição de uma interface.* No próximo tópico, veremos o que o *Go* requer para que você seja capaz de implementar a *interface*.

### Implementando uma *Interface*

*As interfaces em outras linguagens de programação requer que o código concreto implementem a interface explicitamente*. **Implementação explícita significa que a linguagem de programação afirma direta e claramente que este objeto está usando esta interface**. Por exemplo, isso está em Java:

```java
class Dog implements Pet
```

A classe `Dog` será implementada pela interface `Pet`. O código afirma explicitamente que a classe `Dog` implementará `Pet`.

***Em Go, as interfaces são implementadas implicitamente. Isso significa que um tipo implementará a interface tendo todos os métodos e suas assinaturas da interface***. Abaixo está um exemplo:

```go
package main

import (
	"fmt"
)

type Speaker interface {
	Speak() string
}

type cat struct{}

func main() {
	c := cat{}
	fmt.Println(c.Speak())
	c.Greeting()
}

func (c cat) Speak() string {
	return "Purr Meow"
}

func (c cat) Greeting() {
	fmt.Println("Meow,Meow!!!!mmmeeeeoooowwww")
}
```

Vamos dividir esse código em partes:

```go
type Speaker interface {
	Speak() string
}
```

Estamos definindo uma interface `Speaker{}`. Ele tem um método que descreve o comportamento de `Speak()`. O método retorna uma string. Para que um tipo que implemente a interface `Speaker{}`, ele deve ter o método listado na *declaração da interface*. Em seguida, criamos um tipo de estrutura vazio chamado `cat`:

```go
type cat struct { }

func (c cat) Speak() string {
	return "Purr Meow"
}
```

O tipo `cat` tem um método `Speak()` que retorna uma *string*. Isso satisfaz a interface `Speaker{}`. Agora é responsabilidade do tipo `cat` fornecer os detalhes de implementação para o método `Speak()`.

Observe que não houve nenhuma declaração explícita que `cat` implementa a interface `Speaker{}`; ele faz isso apenas atendendo aos *requisitos da interface*.

Também é importante notar que o tipo `cat` possui um método chamado `Greeting()`. O tipo pode ter métodos que não são necessários para satisfazer a interface de `Speaker{}`, ou seja, *métodos adicionais*. No entanto, `cat` *deve ter pelo menos os conjuntos de métodos necessários para poder satisfazer a interface*.

O resultado será o seguinte:

```
Purr Meow
Meow,Meow!!!!mmmeeeeoooowwww
```

**Em *Go*, diz-se que se o tipo que satisfaz a interface então há uma implementação**. Não há nenhuma palavra-chave de `implements` como em outras linguagens; **você não precisa dizer que um tipo implementa a interface**. Em *Go*, **se se o tipo tiver os conjuntos de métodos e assinaturas da interface, ele implicitamente implementa a interface**.

No exemplo a seguir, vamos criar um simples programa que demonstra como *implementar interfaces implicitamente*. Teremos uma estrutura de `person` que implementará implicitamente a interface `Speaker{}`. A estrutura de `person` contêm `name`, `age` e `isMarried` como seus campos. O programa chamará o método `Speak()` da estrutura de `person` e exibirá uma mensagem exibindo o `name`. A estrutura `person` também atenderá aos requisitos da interface `Stringer{}` por ter um método `String()`. A interface `Stringer{}` é uma interface que está na linguagem *Go*. Ele pode ser usado na formatação ao imprimir valores. É assim que vamos usá-lo neste exemplo para formatar a impressão dos campos da estrutura `person`:

```go
package main

import (
	"fmt"
)

// We have created a Speaker{} interface
// Any type that wants to implement our Speaker{} interface must have a Speak() method that returns a string.
type Speaker interface {
	Speak() string
}

type person struct {
	name      string
	age       int
	isMarried bool
}

func main() {
	p := person{name: "Alyson", age: 30, isMarried: true}

	fmt.Println(p.Speak())
	fmt.Println(p)
}

// Create a String() method for person and return a string value. This will satisfy the Stringer{} interface, which will now allow it to be called by the fmt.Println() method:
func (p person) String() string {
	return fmt.Sprintf("%v (%v years old).\nMarried status: %v ", p.name, p.age, p.isMarried)
}

// Create a Speak() method for person that returns a string. The person type has a Speak() method that has the same signature as the Speak() method of the Speaker{} interface. The person type satisfies the Speaker{} interface by having a Speak() method that returns the string. To satisfy interfaces, you must have the same methods and method signatures of the interface:
func (p person) Speak() string {
	return "Hi my name is: " + p.name
}
```

Executando o código você obtem a seguinte saída:

```
Hi my name is: Alyson
Alyson (30 years old).
Married status: true
```

No exemplo acima, vimos como é simples implementar *interfaces implicitamente*.

### As *Interfaces* são *Type-Safe Duck Typing*

Até agora, nada do que foi dito é muito diferente das interfaces em outras linguagens. O que torna as interfaces de *Go* especiais é que **elas são implementadas implicitamente**. *Um tipo concreto não declara que implementa uma interface*. ***Se os métodos definido para um tipo concreto contém todos os métodos definido na interface, o tipo concreto implementa a interface. Isso significa que o tipo concreto pode ser atribuído a uma variável ou campo declarado como sendo do tipo da interface***.

Esse **comportamento implícito** torna as interfaces a coisa mais interessante sobre os tipos em *Go*, porque elas permitem a segurança dos tipos e o desacoplamento, unindo a funcionalidade em linguagens estáticas e dinâmicas.

Basicamente, temos feito o que é chamado de *duck typing*. *Duck typing* é um teste de programação de computador: "*If it looks like a duck, swims like a duck, and quacks like a duck, then it must be a duck.*" *Se um tipo corresponder a uma interface, você pode usar esse tipo em qualquer lugar que essa interface é usada*. *Duck typing* corresponde a um tipo com base em métodos, em vez do tipo esperado:

```go
type Speaker interface {
	Speak() string
}
```

Qualquer coisa que corresponda ao método `Speak()` pode ser uma interface `Speaker{}`. **Ao implementar uma interface, estamos essencialmente em conformidade com essa interface, tendo os conjuntos de métodos necessários**:

```go
package main

import (
	"fmt"
)

type Speaker interface {
	Speak() string
}

type cat struct{}

func main() {
	c := cat{}
	fmt.Println(c.Speak())
}

func (c cat) Speak() string {
	return "Purr Meow"
}
```

`cat` corresponde ao método `Speak()` da interface `Speaker{}`, então um `cat` é um `Speaker{}`:

```go
package main

import (
	"fmt"
)

type Speaker interface {
	Speak() string
}

type cat struct{}

func main() {
	c := cat{}
	chatter(c) // Purr Meow
}

func (c cat) Speak() string {
	return "Purr Meow"
}

func chatter(s Speaker) {
	fmt.Println(s.Speak())
}
```

Vamos examinar esse código em partes:

- No código anterior, declaramos um tipo `cat` e criamos um método chamado `Speak()`. *Isso cumpre os conjuntos de métodos exigidos para a interface de `Speaker{}`*.
- Nós criamos um método chamado `chatter` que usa a interface de `Speaker{}` como um argumento.
- Na função `main()`, podemos passar um tipo `cat` para a função `chatter`, que pode ser avaliado para a interface `Speaker{}`. **Isso satisfaz os conjuntos de métodos necessários para a interface.**

Vamos falar sobre por que as linguagens têm interfaces. *Design Patterns* ensinam aos desenvolvedores a *favorecer a composição em vez da herança*. Outro conselho do livro é "*Programe para uma interface, não uma implementação*". **Isso permite que você dependa do comportamento, não da implementação, permitindo trocar as implementações conforme necessário**. Isso permite que seu código evolua com o tempo, conforme os requisitos mudam inevitavelmente.

Os desenvolvedores *Java* usam um padrão diferente. Eles definem uma interface, criam uma implementação da interface, mas apenas se referem à interface no código do cliente:

```java
public interface Logic
{
	String process(String data);
}

public class LogicImpl implements Logic
{
	public String process(String data)
	{
		// business logic
	}
}

public class Client
{
	private final Logic logic; // this type is the interface, not the implementation

	public Client(Logic logic)
	{
		this.logic = logic;
	}

	public void program()
	{
		// get data from somewhere
		this.logic(data);
	}
}

public static void main(String[] args)
{
	Logic logic = new LogicImpl();
	Client client = new Client(logic);

	client.program();
}
```

*Os desenvolvedores de linguagem dinâmica olham para as interfaces explícitas em Java e não vêem como é possível refatorar seu código ao longo do tempo quando você tem dependências explícitas*. Mudar para uma nova implementação de um provedor diferente significa reescrever seu código para depender de uma nova interface.

Os desenvolvedores *Go* decidiram que os dois grupos estão certos. *Se seu aplicativo vai crescer e mudar com o tempo, você precisa de flexibilidade para mudar a implementação*. No entanto, para que as pessoas entendam o que seu código está fazendo (conforme novas pessoas trabalham no mesmo código ao longo do tempo), você também precisa especificar do que o código depende. É aí que entram as **interfaces implícitas**. O código *Go* é uma mistura dos dois estilos anteriores:

```go
type LogicProvider struct {}

func (lp LogicProvider) Process(data string) string {
	// business logic
	return "end"
}

type Logic interface {
	Process(data string) string
}

type Client struct {
	L Logic
}

func(c Client) Program() {
	// get data from somewhere
	c.L.Process(data)
}

func main() {
	c := Client {
		L: LogicProvider{},
	}
	c.Program()
}
```

No código *Go*, há uma *interface*, mas apenas o chamador (`Client`) sabe sobre ela; não há nada declarado no `LogicProvider` para indicar que ele atende à *interface*. Isso é suficiente para permitir um novo provedor de lógica no futuro, bem como fornecer documentação executável para garantir que qualquer tipo passado para o cliente atenderá às suas necessidade.

*DICA: As interfaces especificam o que os chamadores precisam. O código do cliente define a interface para especificar a funcionalidade necessária.*

Isso não significa que as *interfaces* não podem ser compartilhadas. Já vimos várias *interfaces* na biblioteca padrão que são usadas para entrada e saída. Ter uma interface padrão é poderoso; se você escrever seu código para funcionar com `io.Reader` e `io.Writer`, ele funcionará corretamente se estiver gravando em um arquivo no disco ou em um valor na memória.

Além disso, o uso de *interfaces* padrão incentiva o *decorator pattern*. É comum em *Go* escrever funções de fábrica que pegam uma instância de uma *interface* e retornam outro tipo que implementa a mesma *interface*. Por exemplo, digamos que você tenha uma função com a seguinte definição:

```go
func process(r io.Reader) error
```

Você pode processar dados de um arquivo com o seguinte código:

```go
r, err := os.Open(fileName)
if err != nil {
	return err
}

defer r.Close()
return process(r)
return nil
```

A instância `os.File` retornada por `os.Open` atende à interface `io.Reader` e pode ser usada em qualquer código que leia dados. Se o arquivo estiver compactado com gzip, você pode manipular o `io.Reader` em outro `io.Reader`:

```go
r, err := os.Open(fileName)

if err != nil {
	return err
}

defer r.Close()

gz, err = gzip.NewReader(r)

if err != nil {
	return err
}

defer gz.Close()
return process(gz)
```

Agora, o mesmo código que estava lendo de um arquivo descompactado está lendo de um arquivo compactado.

*DICA: Se houver uma interface na biblioteca padrão que descreva o que seu código precisa, use-a!*

É perfeitamente normal para um tipo que atende a uma interface especificar métodos adicionais que não fazem parte da interface. Um conjunto de código do cliente pode não se importar com esses métodos, mas outros sim. Por exemplo, o tipo `io.File` também atende à interface `io.Writer`. Se o seu código se preocupa apenas com a leitura de um arquivo, use a interface `io.Reader` para se referir à instância do arquivo e ignore os outros métodos.

### Embedding de *Interfaces*

Assim como você pode embutir um tipo em uma estrutura, você também pode embutir uma interface em uma outra interface. Por exemplo, a interface `io.ReadCloser` é construída a partir de um `io.Reader` e um `io.Closer`:

```go
type Reader interface {
	Read(p []byte) (n int, err error)
}

type Closer interface {
	Close() error
}

type ReadCloser interface {
	Reader
	Closer
}
```

### Aceitar *Interfaces*, retornar *Structs*

Frequentemente, você ouvirá desenvolvedores *Go* dizerem que seu código deve "**Aceitar interfaces, retornar estruturas**". Isso significa que a lógica de negócios invocada por suas funções deve ser invocada por meio de *interfaces*, mas a saída de suas funções deve ser um tipo concreto - estruturas. Já vimos por que as funções devem aceitar interfaces: **elas tornam seu código mais flexível e declaram explicitamente qual funcionalidade está sendo usada.**

Se você criar uma API que retorna interfaces, estará perdendo **uma das principais vantagens das interfaces implícitas: o desacoplamento**. Você deseja limitar as interfaces de terceiros das quais seu código de cliente depende, porque seu código agora depende permanentemente do módulo que contém essas interfaces, bem como de quaisquer dependências desse módulo e assim por diante. Isso limita a flexibilidade futura. Para evitar o acoplamento, você teria que escrever outra interface e fazer uma conversão de tipo de uma para a outra. Embora depender de instâncias concretas possa levar a dependências, o uso de uma camada de injeção de dependência em seu aplicativo limita o efeito.

Outro motivo para evitar o retorno de interfaces é o controle de versão. Se um tipo concreto for retornado, novos métodos e campos podem ser adicionados sem quebrar o código existente. O mesmo não acontece para uma interface. Adicionar um novo método a uma interface significa que você precisa atualizar todas as implementações existentes da interface ou seu código quebra. Se você fizer uma alteração de última hora em uma API, deverá incrementar seu número de versão principal.

Os erros são uma exceção a esta regra. As funções e métodos *Go* declaram um parâmetro de retorno do tipo de interface de erro. No caso de erro, é muito provável que diferentes implementações da interface possam ser retornadas, então você precisa usar uma interface para lidar com todas as opções possíveis, já que as interfaces são o único tipo abstrato em *Go*.

### *Interfaces* e `nil`

Ao falar sobre ponteiros, também falamos sobre `nil`, o *zero value* para os tipos de ponteiro. Também usamos `nil` para representar o *zero value* para uma instância de *interface*, mas não é tão simples quanto para tipos concretos.

**Para que uma interface seja considerada `nil`, tanto o tipo quanto o valor devem ser `nil`**. O código a seguir imprime verdadeiro nas duas primeiras linhas e falso na última:

```go
package main

import (
	"fmt"
)

func main() {
	var s *string
	fmt.Println(s == nil) // true
	var i interface{}
	fmt.Println(i == nil) // true
	i = s
	fmt.Println(i == nil) // false
}
```

No runtime do *Go*, as *interfaces* são implementadas como um par de ponteiros, um para o tipo subjacente e outro para o valor subjacente. Enquanto o tipo não for `nil`, a *interface* não será `nil`. (Uma vez que você não pode ter uma variável sem um tipo, se o ponteiro do valor não for `nil`, o ponteiro do tipo não será `nil`.)

**O que `nil` indica para uma *interface* é se você pode ou não invocar métodos nela**. Conforme abordamos anteriormente, você pode invocar métodos em instâncias concretas `nil`, portanto, faz sentido que você possa invocar métodos em uma variável de *interface* que foi atribuída a uma instância concreta `nil`. Se uma *interface* for `nil`, invocar qualquer método nela aciona um erro de pânico. Se uma *interface* não for `nil`, você pode invocar métodos nela. (Mas observe que se o valor for `nil` e os métodos do tipo atribuído não manipularem corretamente o `nil`, você ainda pode causar pânico.)

Como uma instância de *interface* com um tipo não `nil` não é igual a `nil`, não é fácil dizer se o valor associado à *interface* é `nil` quando o tipo não é `nil`.

### Polimorfismo

Polimorfismo é a capacidade parecer-se de várias formas. Por exemplo, um `shape` pode parecer como um `square`, `circle`, `rectangle` ou qualquer outra forma:

![Polymorphism example for shape](/images/articles/Go/Polymorphism%20example%20for%20shape.png?id=c2525d441c9d1728ae9a15b653bb179b "Polymorphism example for shape")

*Go* não tem subclasses / herança como outras linguagens orientadas a objetos, porque *Go não tem classes*. A criação de subclasses na programação orientada a objetos é herdar de uma classe para outra. Ao fazer subclasses, você está herdando os campos e métodos de outra classe. *Go* oferece um comportamento semelhante por meio da **embedding** (*incorporação*) de *`structs`* e usando polimorfismo por meio de *interfaces*.

**Uma das vantagens de usar polimorfismo é que ele permite a reutilização de métodos que foram escritos uma vez e testados. O código é reutilizado por ter uma API que aceita uma interface**; se nosso tipo satisfizer essa *interface*, ele pode ser passado para essa API. Não há necessidade de escrever código adicional para cada tipo; só precisamos garantir que atendemos aos requisitos definidos nos métodos da interface. *A obtenção de polimorfismo por meio do uso de interfaces aumentará a capacidade de reutilização do código*. Se sua API aceita apenas tipos concretos, como `int`, `float` e `bool`, apenas esse tipo concreto pode ser passado. Contudo, se sua API aceita uma interface, o chamador pode adicionar os conjuntos de métodos necessários para satisfazer essa interface, independentemente do tipo subjacente. *Essa capacidade de reutilização é obtida permitindo que suas APIs aceitem interfaces*. *Qualquer tipo que satisfaça a interface pode ser passado para a API*. Vimos esse tipo de comportamento em um exemplo anterior. Este é um bom momento para examinar mais de perto a interface `Speaker{}`.

Como vimos nos exemplos anteriores, **cada tipo concreto pode implementar uma ou mais interfaces**. A interface de `Speaker{}` pode ser implementada por tipos de `dog`, `cat` ou `fish` conforme a imagem abaixo:

![The Speaker interface implemented by multiple types](/images/articles/Go/The%20Speaker%20interface%20implemented%20by%20multiple%20types.png?id=7ffb8169c60f8a5246d732970c1586ff "The Speaker interface implemented by multiple types")

**Quando uma função aceita uma interface como parâmetro, qualquer tipo concreto que implemente essa interface pode ser passado como argumento**. Agora, você atingiu o polimorfismo ao ser capaz de passar vários tipos concretos para um método ou função que tem um tipo de interface como parâmetro.

#### Exemplo: Calculando a área de diferentes formas usando *Polimorfismo*

No código a seguir é implementado um programa que calcula a área de um triângulo, retângulo e quadrado. O programa usa uma única função que aceita uma interface `Shape`. *Qualquer tipo que satisfaça a interface `Shape` pode ser passado como um argumento para a função*. A função imprimi a área e o nome do `shape` (forma):

```go
package main

import (
	"fmt"
)

type Shape interface {
	Area() float64
	Name() string
}

type triangle struct {
	base   float64
	height float64
}

type rectangle struct {
	length float64
	width  float64
}

type square struct {
	side float64
}

func main() {
	t := triangle{base: 15.5, height: 20.1}
	r := rectangle{length: 20, width: 10}
	s := square{side: 10}

	printShapeDetails(t, r, s)
}

func printShapeDetails(shapes ...Shape) {
	for _, item := range shapes {
		fmt.Printf("The area of %s is: %.2f\n", item.Name(), item.Area())
	}
}

func (t triangle) Area() float64 {
	return (t.base * t.height) / 2
}

func (t triangle) Name() string {
	return "triangle"
}

func (r rectangle) Area() float64 {
	return r.length * r.width
}

func (r rectangle) Name() string {
	return "rectangle"
}

func (s square) Area() float64 {
	return s.side * s.side
}

func (s square) Name() string {
	return "square"
}
```

![Square, triangle, rectangle area of the Shape type](/images/articles/Go/Square,%20triangle,%20rectangle%20area%20of%20the%20Shape%20type.png?id=6bca60c2c4e353b69be4b6ab96163778 "Square, triangle, rectangle area of the Shape type")

Executando o código deverá ver o seguinte resultado:

```
The area of triangle is: 155.78
The area of rectangle is: 200.00
The area of square is: 100.00
```

Neste exemplo, vimos a flexibilidade e o código reutilizável que as *interfaces* fornecem aos nossos programas. *Quando usamos interfaces como argumentos de entrada para uma API, estamos afirmando que um tipo precisa satisfazer a interface*. Ao usar tipos concretos, exigimos que o argumento da API seja desse tipo. Por exemplo, se uma assinatura de função for `func greeting(msg string)`, sabemos que o argumento transmitido deve ser uma `string`. Os tipos concretos podem ser considerados como tipos que não são abstratos (`float64`, `int`, `string` e assim por diante); entretanto, *as interfaces podem ser consideradas um tipo abstrato porque você está satisfazendo os conjuntos de métodos do tipo da interface. O tipo deve atender aos requisitos de ter os conjuntos de métodos definidos pelo tipo da interface*.

No futuro, se exigirmos que outro tipo seja passado, isso significará que o código *upstream* para nossa API precisará ser alterado, ou se o chamador de nossa API precisar alterar seu tipo de dados, ele pode solicitar que mudemos nossa API para satisfazer o contrato. Se usarmos *interfaces*, isso não será um problema; o chamador de nosso código precisa *satisfazer os conjuntos de métodos da interface*. O chamador pode então alterar o tipo subjacente, desde que esteja em conformidade com os requisitos da interface.

### `interface{} || any` - Interfaces vazia

> Uma *interface* vazia é uma *interface* que não possui conjuntos de métodos e nem comportamentos. Uma *interface* vazia não especifica métodos:

```go
interface{}
```

Às vezes, em uma linguagem com *tipagem estática*, você precisa de uma maneira de dizer que uma variável pode armazenar um valor de qualquer tipo. *Go* usa `interface{}` para representar isso:

```go
var i interface{}
i = 20
i = "hello"
i = struct {
	FirstName string
	LastName string
} {"Jonh", "Silva"}
```

Este é um conceito simples, mas complexo para entender. Como você deve se lembrar, *as interfaces são implementadas implicitamente*; não há palavra-chave de `implements`. Como uma interface vazia não especifica nenhum método, *isso significa que cada tipo em *Go* implementa uma interface vazia automaticamente*. **Todos os tipos satisfazem a interface vazia**.

Você deve observar que `interface{}` não é uma *sintaxe especial*. *Um tipo de interface vazio simplesmente afirma que a variável pode armazenar qualquer valor cujo tipo implemente zero ou mais métodos*. Isso corresponde a todos os tipos em *Go*. Como uma interface vazia não informa nada sobre o valor que representa, não há muito que você possa fazer com ela.

No trecho de código a seguir, demonstraremos como usar a interface vazia. Também veremos como uma função que aceita uma interface vazia permite que qualquer tipo seja passado para essa função:

```go
package main

import (
	"fmt"
)

type Speaker interface {
	Speak() string
}

type cat struct {
	name string
}

func main() {
	c := cat{name: "asdf"}
	i := 99
	b := false
	str := "test"
	catDetails(c)
	emptyDetails(c)
	emptyDetails(i)
	emptyDetails(b)
	emptyDetails(str)
}

func (c cat) Speak() string {
	return "Purr Meow"
}

func emptyDetails(i interface{}) {
	fmt.Printf("(%v, %T)\n", i, i)
}

func catDetails(i Speaker) {
	fmt.Printf("(%v, %T)\n", i, i)
}
```

A saída é a seguinte:

```
({asdf}, main.cat)
({asdf}, main.cat)
(99, int)
(false, bool)
(test, string)
```

Vamos avaliar o código em seções:

```go
func emptyDetails(s interface{}) {
	fmt.Printf("(%v, %T)\n", i, i)
}
```

A função aceita uma interface vazia `interface{}`. *Qualquer tipo pode ser passado para a função, pois todos os tipos implementam a interface `interface{}`*. Ele imprime o valor e o tipo concreto. O especificador `%v` imprime o valor e o especificador `%T` imprime o tipo concreto:

```go
func main() {
	c := cat{ name: "asdf" }
	i := 99
	b := false
	str := "test"
	catDetails(c)
	emptyDetails(c)
	emptyDetails(i)
	emptyDetails(b)
	emptyDetails(str)
}
```

Passamos um tipo `cat`, `int`, `bool` e `string`. A função `emptyDetails()` imprimirá cada um deles:

![The type cat implements an empty interface and the Speaker interface](/images/articles/Go/The%20type%20cat%20implements%20an%20empty%20interface%20and%20the%20Speaker%20interface.png?id=e9f86250a11a6f3ee68060a15151d9c5 "The type cat implements an empty interface and the Speaker interface")

*O tipo `cat` implementa a interface vazia `interface{}` e a interface `Speaker{}` implicitamente*.

Um uso comum da *interface vazia* é como um espaço para dados que são lidos de uma fonte externa, como um arquivo *JSON*:

```go
// one set of braces for the interface{} type,
// the other to instantiate an instance of the map
data := map[string]interface{}{}
contents, err := ioutil.ReadFile("testdata/sample.json")

if err != nil {
	return err
}

defer contents.Close()

json.Unmarshal(contents, &data)
// the contents are now in the data map
```

Essas situações devem ser relativamente raras. Evite usar a `interface{}`. Como vimos, *Go foi desenvolvido como uma linguagem fortemente tipada e as tentativas de contornar isso não são idiomáticas*.

Se você se encontrar em uma situação em que precisa armazenar um valor em uma *interface vazia*, pode estar se perguntando como ler o valor novamente. Para fazer isso, precisamos examinar as **type assertion** e **type switch**.

Agora que temos um entendimento básico de *interfaces vazias*, veremos vários casos de uso para elas nos próximos tópicos:

- *Type switching*
- *Type assertion*

### Type Assertions and Type Switches

*Go* não manipula nenhum valor quando você o coloca em uma variável de `interface{}`. O que acontece é que o compilador *Go* impede de usá-lo porque não é capaz de realizar suas verificações de type-safety em tempo de compilação. Usar **type assertion** é a sua instrução para descobrir que deseja manipular o valor. Quando você manipula **type assertion**, *Go* executa as verificações de type-safety que teria feito em tempo de compilação fazendo em runtime (tempo de execução), e essas verificações podem falhar. É então sua responsabilidade lidar com as falhas nas verificações de type-safety. As **type assertions** são recursos que causam erros e pânicos em runtime, o que significa que você deve ser extremamente cuidadoso com eles.

***Type assertion*** fornece acesso ao tipo concreto de uma *interface*. Lembre-se de que a interface `interface{}` pode ter qualquer valor:

```go
package main

import (
	"fmt"
)

func main() {
	var str interface{} = "some string"
	var i interface{} = 42
	var b interface{} = true
	fmt.Println(str)
	fmt.Println(i)
	fmt.Println(b)
}
```

A saída é a seguinte:

```
some string
42
true
```

Em cada instância da declaração da variável, cada variável é declarada como uma *interface vazia*, mas o valor concreto para `str` é uma string, para `i` é um inteiro e para `b` é um booleano.

Quando há um tipo de *interface vazia* `interface{}`, às vezes, é benéfico saber o tipo concreto subjacente. Por exemplo, você pode precisar executar manipulação de dados com base nesse tipo. Se esse tipo for uma string, você executaria a modificação e validação de dados de maneira diferente de como faria se fosse um valor inteiro. Isso também entra em jogo quando você está consumindo dados JSON de um esquema desconhecido. Os valores nesse JSON podem ser conhecidos durante o processo de solicitação. Precisaríamos converter esses dados para mapear em `map[string]interface{}` e realizar várias manipulações de dados. Nós poderíamos realizar um *type conversion* com o pacote `strconv`:

```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	var i interface{} = 42
	fmt.Println(strconv.Atoi(i))
}
```

```
./prog.go:10:27: cannot use i (variable of type interface{}) as string value in argument to strconv.Atoi: need type assertion
```

Portanto, parece que não podemos usar **type conversion** porque os tipos não são compatíveis. Precisamos usar ***type assertion***:

```
v := s.(T)
```

A instrução anterior diz que o valor da interface `s` é do tipo `T` e atribui o valor subjacente a `v`:

![Type assertion flow](/images/articles/Go/Type%20assertion%20flow.png?id=04567a7dadfb26a0c6b66d31e2f484ee "Type assertion flow")

A notação para **type assertion** é `<value>.(<type>)`. **Type assertion** resulta em um valor do tipo que foi solicitado e, opcionalmente, um `bool` se foi bem-sucedido ou não. Isto parece com `<value> := <value>.(<type>)` ou `<value>, <ok> := <value>.(type)`. Se você deixar o valor booleano (`<ok>`) de fora e **type assertion** falhar, *Go* causará pânico.

Considere o seguinte código:

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	var str interface{} = "some string"
	v := str.(string)
	fmt.Println(strings.Title(v))
}
```

Vamos examinar o código anterior:

- O código anterior afirma que `str` é do tipo `string` e o atribui à variável `v`.
- Como `v` é uma `string`, ele será impresso com o título casing.

O resultado é o seguinte:

```
Some String
```

Veja outro exemplo:

```go
package main

import (
	"fmt"
)

type MyInt int

func main() {
	var i interface{}
	var mine MyInt = 20
	i = mine
	i2 := i.(MyInt)
	fmt.Println(i2) // 20
}
```

No código acima, a variável `i2` é do tipo `MyInt`.

Você pode se perguntar o que acontece se uma declaração de tipo estiver errada. Nesse caso, *seu código entra em pânico*.

```go
i2 := i.(string)
fmt.Println(i2)
```

Executar este código produz pânico:

```
panic: interface conversion: interface {} is main.MyInt, not string
```

É bom quando a afirmação corresponde ao tipo esperado. Então, o que acontecerá se `s` não for do tipo `T`? Vamos dar uma olhada:

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	var str interface{} = 49
	v := str.(string)
	fmt.Println(strings.Title(v))
}
```

Vamos examinar o código anterior:

- `str{}` é uma *interface vazia* e o tipo concreto é `int`.
- A ***type assertion*** está verificando se `str` é do tipo `string`, mas neste cenário, não é, então o código entrará em pânico.

O resultado é o seguinte:

```
panic: interface conversion: interface {} is int, not string
```

*Go* é muito cuidadoso com os tipos concreto. Mesmo se dois tipos compartilham um tipo subjacente, uma *type assertion* deve corresponder ao tipo do valor subjacente. O código a seguir entra em pânico.

```go
i2 := i.(int)
fmt.Println(i2 + 1)
// panic: interface conversion: interface {} is main.MyInt, not int
```

Ter um pânico sendo lançado não é algo desejável. No entanto, *Go* tem uma maneira de verificar se `str` é uma *string*:

```go
package main

import (
	"fmt"
)

func main() {
	var str interface{} = "The book club"
	v, isValid := str.(int)
	fmt.Println(v, isValid) // 0 false
}
```

Vamos examinar o código anterior:

- Uma ***type assertion*** retorna dois valores, o valor subjacente e um valor booleano.
- `isValid` é atribuído a um tipo de retorno de `bool`. Se retornar `true`, indica que `str` é do tipo `int`. Isso significa que a **assertion** (afirmação) é verdadeira. Podemos usar o booleano que foi retornado para determinar que ação podemos tomar em `str`.
- Quando a **assertion** falhar, ele retornará `false`. O valor de retorno será o *zero value* que você está tentando declarar. Ele também não entrará em pânico.

Travar não é o comportamento desejado. Evitamos isso usando *comma ok idiom*.

```go
i2, ok := i.(int)

if !ok {
	return fmt.Errorf("unexpected type for %v",i)
}

fmt.Println(i2 + 1)
```

O booleano `ok` é definido como verdadeiro se a conversão do tipo for bem-sucedida.

Mesmo se você estiver absolutamente certo de que sua declaração de tipo é válida, use *comma ok idiom*. Você não sabe como outras pessoas (ou você em seis meses) reutilizarão seu código. Mais cedo ou mais tarde, suas **type assertion** não validadas falharão em runtime.

Quando uma *interface* pode ser de vários tipos possíveis, use ***type switch***.

Haverá momentos em que você não saberá o *tipo concreto da interface vazia*. É quando você usará uma ***type switch***. Uma ***type switch*** pode realizar vários tipos de asserções; é semelhante a uma instrução switch. Possui cláusulas `case` e `default`. A diferença é que as instruções de ***type switch*** *avaliam um tipo em vez de um valor*.

Aqui está uma estrutura de sintaxe básica:

```
switch <value> := <value>.(type) {
	case <type>:
		<statement>
	case <type>, <type>:
		<statement>
	default:
		<statement>
}
```

```go
func doThings(i interface{}) {
	switch j := i.(type) {
		case nil:
			// i is nil, type of M is interface{}
		case int:
			// M is of type int
		case MyInt:
			// M is of type MyInt
		case io.Reader:
			// M is of type io.Reader
		case string:
			// M is a string
		case bool, rune:
			// i is either a bool or rune, so M is of type interface{}
		default:
			// no idea what i is, so M is of type interface{}
	}
}
```

***Type switch*** só executa sua lógica se corresponder ao tipo que você está procurando e define o valor para esse tipo.

Vamos examinar o código anterior:

```
i.(type)
```

A sintaxe é semelhante à da **type assertion**, `i.(int)`, exceto que o tipo especificado, `int` em nosso exemplo, é substituído pela palavra-chave `type`. O tipo declarado do tipo `i` é atribuído a `v`; então, ele é comparado a cada uma das declarações de `case`.

```
case S:
```

No ***switch type***, *as instruções avaliam os tipos*. No switch regular, eles avaliam os valores. Aqui, ele é avaliado por um tipo de `S`.

Agora que temos um entendimento fundamental da instrução de ***type switch***, vamos dar uma olhada em um exemplo que usa a sintaxe que acabamos de avaliar:

```go
package main

import (
	"fmt"
)

type cat struct {
	name string
}

func main() {
	c := cat{name: "asdf"}
	i := []interface{}{42, "The book club", true, c}
	typeExample(i)
}

func typeExample(i []interface{}) {
	for _, x := range i {
		switch v := x.(type) {
		case int:
			fmt.Printf("%v is int\n", v)
		case string:
			fmt.Printf("%v is a string\n", v)
		case bool:
			fmt.Printf("a bool %v\n", v)
		default:
			fmt.Printf("Unknown type %T\n", v)
		}
	}
}
```

Vamos agora explorar o código em partes:

```go
func main() {
  c := cat { name: "asdf" }
  i := []interface{}{42, "The book club", true,c}
  typeExample(i)
}
```

Na função `main()`, estamos inicializando uma variável, `i`, para uma *slice* de *interfaces*. Na *slice*, temos os tipos `int`, `string`, `bool` e `cat`:

```go
func typeExample(i []interface{})
```

A função aceita uma slice de interfaces:

```go
for _, x := range i {
	switch v := x.(type) {
	case int:
	  fmt.Printf("%v is int\n", v)
	case string:
	  fmt.Printf("%v is a string\n",v)
	case bool:
	  fmt.Printf("a bool %v\n", v)
	default:
	  fmt.Printf("Unknown type %T\n", v)
	}
  }
```

O loop `for` faz um `ranges` sobre a *slice de interfaces*. O primeiro valor na *slice* é `42`. O `switch case` afirma que o valor da *slice* de `42` é um tipo `int`. A declaração `case int` será avaliada como verdadeira e print `42 is int`. Quando o loop for itera sobre o último valor do tipo `cat`, a instrução *switch* não encontrará esse tipo em suas avaliações de case. Como não há nenhum tipo de `cat` sendo verificado nas instruções case, o `default` executará sua instrução print. Aqui estão os resultados do código sendo executado:

```
42 is int
The book club is a string
a bool true
Unknown type main.cat
```

Usando outro exemplo, vamos atualizar uma função `doubler` para usar um ***type switch*** e expandir suas habilidades para lidar com mais tipos.

No código abaixo, realizaremos alguns ***type assertions*** e garantiremos que todas as verificações de type safety estejam em vigor quando for executado - runtime.

```go
package main

import (
	"errors"
	"fmt"
)

func doubler(v interface{}) (string, error) {
	switch t := v.(type) {
	// For string and bool, since we're only matching on one type, we don't need to do any extra safety checks and can work with the value directly:
	case string:
		return t + t, nil
	case bool:
		if t {
			return "truetrue", nil
		}
		return "falsefalse", nil
	// For the floats, we're matching on more than one type. This means we need to do type assertion to be able to work with the value:
	case float32, float64:
		if f, ok := t.(float64); ok {
			return fmt.Sprint(f * 2), nil
		}
		return fmt.Sprint(t.(float32) * 2), nil
	case int:
		return fmt.Sprint(t * 2), nil
	case int8:
		return fmt.Sprint(t * 2), nil
	case int16:
		return fmt.Sprint(t * 2), nil
	case int32:
		return fmt.Sprint(t * 2), nil
	case int64:
		return fmt.Sprint(t * 2), nil
	case uint:
		return fmt.Sprint(t * 2), nil
	case uint8:
		return fmt.Sprint(t * 2), nil
	case uint16:
		return fmt.Sprint(t * 2), nil
	case uint32:
		return fmt.Sprint(t * 2), nil
	case uint64:
		return fmt.Sprint(t * 2), nil
	default:
		return "", errors.New("unsupported type passed")
	}
}

func main() {
	res, _ := doubler(-5)
	fmt.Println("-5  :", res)
	res, _ = doubler(5)
	fmt.Println("5   :", res)
	res, _ = doubler("yum")
	fmt.Println("yum :", res)
	res, _ = doubler(true)
	fmt.Println("true:", res)
	res, _ = doubler(float32(3.14))
	fmt.Println("3.14:", res)
}
```

Executar o código anterior produz a seguinte saída:

```
-5  : -10
5   : 10
yum : yumyum
true: truetrue
3.14: 6.28
```

O tipo da nova variável depende de qual `case` corresponde. Você pode usar `nil` para um `case` para ver se a interface não tem nenhum tipo associado. Se você listar mais de um tipo em um `case`, a variável é do tipo `interface{}`. Assim como uma instrução `switch`, você pode ter um `case` padrão que corresponda a nenhum tipo especificado. Caso contrário, a nova variável tem o tipo de `case` correspondente.

No exemplo de código acima, usamos ***type switch*** para construir um cenário complexo de ***type assertion***. Usar ***type switch*** ainda nos dá controle total das ***type assertions***, mas também nos permite simplificar a lógica de type-safety quando não precisamos desse nível de controle.

#### Exemplo: Analisando dados da `interface{}` vazia

Neste exemplo, recebemos um mapa. A chave do mapa é uma string e seu valor é uma `interface{}` vazia. O valor do mapa contém diferentes tipos de dados armazenados. Nosso trabalho é determinar o tipo de valor de cada chave. Vamos escrever um programa que irá analisar os dados de `map[string] interface{}`. Entenda que os valores dos dados podem ser de qualquer tipo. Precisamos escrever a lógica para tratar os tipos. Vamos armazenar essas informações em uma `slice` de `structs` que conterá o nome da chave, os dados e o tipo de dado:

```go
package main

import (
	"fmt"
)

type record struct {
	key       string
	valueType string
	data      interface{}
}

type person struct {
	lastName  string
	age       int
	isMarried bool
}

type animal struct {
	name     string
	category string
}

func main() {
	m := make(map[string]interface{})
	a := animal{name: "asdf", category: "cat"}
	p := person{lastName: "xyz", isMarried: false, age: 19}

	m["person"] = p
	m["animal"] = a
	m["age"] = 30
	m["isMarried"] = true
	m["lastName"] = "Silva"

	rs := []record{}
	for k, v := range m {
		r := newRecord(k, v)
		rs = append(rs, r)
	}

	for _, v := range rs {
		fmt.Println("Key: ", v.key)
		fmt.Println("Data: ", v.data)
		fmt.Println("Type: ", v.valueType)
		fmt.Println()
	}
}

func newRecord(key string, i interface{}) record {
	r := record{}
	r.key = key
	switch v := i.(type) {
	case int:
		r.valueType = "int"
		r.data = v
		return r
	case bool:
		r.valueType = "bool"
		r.data = v
		return r
	case string:
		r.valueType = "string"
		r.data = v
		return r
	case person:
		r.valueType = "person"
		r.data = v
		return r
	default:
		r.valueType = "unknown"
		r.data = v
		return r
	}
}
```

A saída é a seguinte:

```
Key:  isMarried
Data:  true
Type:  bool

Key:  lastName
Data:  Silva
Type:  string

Key:  person
Data:  {xyz 19 false}
Type:  person

Key:  animal
Data:  {asdf cat}
Type:  unknown

Key:  age
Data:  30
Type:  int
```

O exemplo demonstrou a capacidade de *Go* de identificar o tipo subjacente de uma *interface vazia*. Como você pode ver nos resultados, o ***type switch*** foi capaz de identificar cada tipo, exceto pelo valor da chave de `animal`. Tem seu tipo marcado como `unknown`. Além disso, foi possível até identificar o tipo da estrutura de `person`, e `data` contêm os valores dos campos da estrutura.
