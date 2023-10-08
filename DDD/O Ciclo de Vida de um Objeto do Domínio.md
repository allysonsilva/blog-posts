
---
id: 9a524952-11f8-4b24-8b8b-29d72ff36b7d
title: O Ciclo de Vida de um Objeto do Domínio
summary: "Todos os objetos possuem um ciclo de vida e o controle desses objetos apresenta desafios que podem facilmente atrapalhar uma tentativa de DESIGN DIRIGIDOS POR MODELOS"
---

Os desafios se dividem em duas categorias:

1. Manter a integridade durante todo o ciclo de vida.
2. Impedir que o modelo se deixe levar pela complexidade do gerenciamento do ciclo de vida.

Este artigo trata dessas questões através de três padrões. Primeiro, os AGREGADOS restringem o próprio modelo definindo uma propriedade e limites claros, evitando um emaranhado caótico de objetos. Esse padrão é fundamental para se manter a integridade em todas as fases do ciclo de vida. Em seguida o foco passa a ser o começo do ciclo de vida, usando FÁBRICAS para criar e reconstituir objetos complexos e AGREGADOS, mantendo sua estrutura interna encapsulada. Por último, os REPOSITÓRIOS tratam do meio e do fim do ciclo de vida, fornecendo meios para achar e recuperar objetos persistentes, encapsulando, ao mesmo tempo, a imensa infraestrutura envolvida.

## AGREGADOS

Um AGREGADO é composto por uma única ENTIDADE ou um conjunto de ENTIDADES e OBJETO DE VALOR que devem permanecer consistentes transacionalmente ao longo de toda a vida do agregado. [Livro vermelho]

Um AGREGADO é um grupo de objetos associados que tratamos como sendo uma unidade para fins de alterações de dados. Cada agregado possui uma raiz e um limite. O limite define o que está dentro do agregado. A raiz é uma ENTIDADE única e específica contida no agregado. A raiz é o único membro do agregado à qual objetos externos têm permissão de fazer referências, embora os objetos dentro daquele limite possam fazer referências uns aos outros. ENTIDADES, que não sejam uma raiz, possuem uma identidade local, mais essa identidade precisa ser distinguível somente dentro do AGREGADO, pois nenhum objeto externo jamais pode vê-la fora do contexto da raiz ENTIDADE.

Agora, para traduzir esse agregado conceitual para a implementação, precisamos de um conjunto de regras que se aplique a todas as transações.

- A ENTIDADE raiz possui uma identidade global e é, em última instância, responsável por verificar as invariantes.

- ENTIDADES raiz possuem uma identidade global. As ENTIDADES dentro do limite possuem uma identidade local e única somente dentro do AGREGADO.

- Nada fora do limite do agregado pode fazer referência a qualquer coisa dentro, exceto à entidade raiz. A ENTIDADE raiz pode fornecer referencias às entidades internas para outros objetos, mas esses objetos podem usá-las somente de forma transitória, e não podem reter a referência. A raiz pode fornecer uma cópia de um OBJETO DE VALOR para outro objeto, e não importa o que acontece com ela, pois ela é apenas um VALOR e já não terá nenhuma associação com o agregado.

- Como corolário à regra anterior, somente raízes AGREGADO podem ser obtidas diretamente através de consultas aos bancos de dados Todos os outros objetos devem ser achados através da travessia de associações.

- Objetos dentro do agregado podem fazer referência a outras raízes de agregados.

- Uma operação de exclusão deve remover tudo dentro do limite do AGREGADO de uma só vez.

- Quando é feita uma alteração em qualquer objeto dentro do limite do AGREGADO, todas as invariantes do AGREGADO inteiro devem ser satisfeitas.

**Agrupe as entidades e os objetos de valor em agregados e defina os limites em torno de cada um. Escolha uma entidade para ser a raiz de cada agregado e controle todo a acesso aos objetos dentro do limite através da raiz. Permite que objetos externos façam referência somente à raiz. Referências transitórias a membros internos podem ser transmitidas para uso dentro de apenas uma única operação. Como a raiz controla o acesso, ela não pode ser surpreendida com alterações na parte interna. Esse arranjo torna prática a execução de todas as invariantes para objetos no agregado e para o agregado como um todo em qualquer alteração de estado.**

## FÁBRICAS

_Quando a criação de um objeto, ou um agregado inteiro, se torna complicada ou revela uma grande parte da estrutura interna, as FÁBRICAS fornecem o encapsulamento._

Montar um objeto composto complexo é uma função que deve ser separada de qualquer outra função que esse objeto tenha que fazer quando estiver acabado.

Um cliente que assuma a criação do objeto torna-se desnecessariamente complicado e obscurece sua responsabilidade. Ele rompe o encapsulamento dos objetos de domínio e os AGREGADOS que estão sendo criados. Pior ainda, se o cliente faz parte da camada da aplicação, isso significa que as responsabilidades escoaram da camada do domínio, todas juntas. Esse acoplamento rígido do aplicativo com as especificações da implementação elimina a maior parte das vantagens da abstração na camada do domínio e torna cada vez mais caras as contínuas alterações.

_A criação de um objeto pode ser uma importante operação por si só, mas operações complexas de montagem não se encaixam com a responsabilidade dos objetos criados. A combinação dessas responsabilidades pode gerar design deselegantes e difíceis de entender. O fato de se fazer uma construção direta para o cliente confunde o design do cliente, rompe o encapsulamento do objeto montado ou do AGREGADO, e provoca um acoplamento excessivo do cliente com a implementação do objeto criado._

- **A criação de objetos complexos é responsabilidade da camada de domínio. Porem essa responsabilidade não pertence aos objetos que expressam o modelo.**
- Existem alguns casos em que a criação de um objeto e sua montagem correspondem a um marco significativo no domínio, como, por exemplo, "Abrir uma conta bancária". Mas a criação e a montagem do objeto geralmente não tem nenhum significado no domínio; elas são uma necessidade da implementação. Para resolver esse problema, temos que acrescentar ideias ao design do domínio que não sejam entidades, objetos de valor ou serviços. Isso foge ao artigo anterior, e é importante esclarecer os fatos: estamos acrescentando elementos ao design que não correspondem a nada no modelo, mais que são, independentes disso, parte da responsabilidade da camada de domínio.

Assim como a interface de um objeto deve encapsular sua implementação, permitindo assim que um cliente utilize o comportamento do objeto sem saber como ele funciona, uma FÁBRICA encapsula o conhecimento necessário para criar um objeto complexo ou um AGREGADO. Ela fornece uma interface que reflete os objetivos do cliente e uma visão abstrata do objeto criado.

Assim:

**Transfira a responsabilidade pela criação de instâncias de objetos complexos e agregados para um objeto separado, que pode não ter nenhuma responsabilidade no modelo de domínio, mas fazer parte do design do domínio. Ofereça uma interface que encapsule toda a montagem complexa e que não requeira que o cliente faca qualquer referência às classes concretas dos objetos que estão sendo instanciados. Crie AGREGADOS inteiros como sendo um pedaço, executando suas invariantes.**

**O uso correto das fábricas pode ajudar a manter um DESIGN DIRIGIDO POR MODELOS em seu curso.**

As duas exigências básicas para qualquer boa FÁBRICA são:

1. Cada método de criação é atômico e executa todas as invariantes do objeto criado ou do agregado. Uma fábrica só devera conseguir gerar um objeto em um estado consistente. Para uma entidade, isso significa a criação do agregado inteiro, com todas as invariantes satisfeitas, mas, provavelmente, com elementos opcionais que ainda deverão ser acrescentados. Para um objeto de valor imutável, isso significa que todos os atributos são inicializados em seu estado final correto. Se a interface possibilita a solicitação de um objeto que não pode ser criado corretamente, deve ser feita alguma exceção ou invocado algum outro mecanismo que garanta que não seja possível nenhum valor de retorno incorreto.

2. a FÁBRICA deve ser abstraída par ao tipo desejado, em vez de para a(s) classe(s) concreta(s) criada(s).

De modo geral, você cria uma FÁBRICA para construir algo cujos detalhes você deseja ocultar, e coloca a FÁBRICA onde deseja que o controle fique. Essas decisões normalmente giram em torno dos agregados.

Por exemplo, se precisasse acrescentar elementos dentro de um agregado já existente, você poderia criar um MÉTODO FABRICA na raiz do AGREGADO. Assim, você oculta a implementação do interior do AGREGADO de qualquer cliente externo, dando, ao mesmo tempo, à raiz a responsabilidade de garantir a integridade do AGREGADO à medida que os elementos são acrescentados.

Outro exemplo seria colocar um MÉTODO FABRICA em um objeto que esteja intimamente envolvido na geração de outro objeto, embora ele não tenha a posse do produto uma vez que este seja criado.

A FÁBRICA impede que o cliente seja acoplado às classes concretas.

Uma FÁBRICA tem um acoplamento bastante rígido com seu produto e, por isso, uma FÁBRICA só deve estar presa a um objeto que tenha uma relação intima e natural com o produto. Quando existe algo que queremos ocultar - seja a implementação concreta ou a mera complexidade de construção - contudo, parece não haver um hospedeiro natural, é preciso criar um objeto FABRICA dedicado ou um serviço. Uma FÁBRICA isolada normalmente gera um agregado inteiro, entregando uma referência à raiz e garantindo a execução das invariantes do agregado do produto. Se um objeto interno ao agregado precisar de uma FÁBRICA, e a raiz do agregado não for um abrigo razoável para ele, vá em frente e crie uma FÁBRICA isolada.

Evite chamar construtores dentro de construtores de outras classes. Os construtores devem ser bastantes simples. Montagens complexas, especialmente de AGREGADOS, exigem FABRICAS. O limite para se optar por usar um pequeno MÉTODO FABRICA não é muito alto.

### Criando o design da interface

Ao criar o design da assinatura do método de uma fábrica, seja ele isolado ou um método fabrica, tenha em mente dois pontos:

- *Cada operação deve ser atômica*: você deve transmitir tudo que for necessário para criar um produto completo em uma única interação com a FÁBRICA. É preciso também decidir o que vai acontecer se a criação falhar, caso alguma invariante não seja satisfeita.
- *A FÁBRICA será acoplada a seus argumentos*: se você não for cuidadoso em sua seleção dos parâmetros de entrada, poderá acabar criando um ninho de rato de dependência. O grau de acoplamento vai depender do que você faz com o argumento. Se for simplesmente ligado ao produto, você terá criado uma dependência modesta. Se você estiver removendo partes do argumento para usar na construção, o acoplamento torna-se mais rígido.

Use o tipo abstrato dos argumentos, e não suas classes concretas. A FÁBRICA é acoplada à classe concreta dos produtos; ela não precisa ser acoplada também aos parâmetros concretos.

Uma FÁBRICA é responsável por garantir que todas as invariantes sejam satisfeitas para o objeto ou agregado por ela criados; A fábrica é um lugar logico para colocar as invariantes, mantendo uma maior simplicidade do produto.

Até agora, a fábrica tem desempenhado seu papel bem no início do ciclo de vida de um objeto.

## REPOSITÓRIOS (REPOSITORY)

O objetivo do DDD é criar softwares melhores concentrando-se em um modelo do domínio, e não na tecnologia.

Um repositório representa todos os objetos de um determinado tipo como sendo um conjunto conceitual (normalmente imitado). Ele age como uma coleção, exceto por ter recursos mais elaborados para consultas. Objetos do tipo adequado são acrescentados e removidos, e o maquinário existente por trás do repositório os insere ou os exclui do banco de dados. Essa definição une um conjunto coeso de responsabilidades para fornecer acesso às raízes dos agregados desde o início do ciclo de vida até o seu final.

Os clientes solicitam objetos a partir do repositório utilizando métodos de consulta que selecionam objetos com base em critérios especificados pelo cliente, normalmente o valor de determinado atributo. O repositório recupera o objeto solicitado, encapsulando o maquinário de consultas ao banco de dados e o mapeamento de metadados.

Um repositório retirar um grande fardo das costas do cliente, que agora pode se comunicar com uma interface simples, reveladora de intenções, e pedir o que precisa em termos do modelo.

Os repositórios tem várias vantagens, incluindo as seguintes:

- Eles fornecem ao cliente um modelo simples para obter objetos persistentes e controlar seu ciclo de vida.
- Eles tomam o design do aplicativo e do domínio e os desacoplam da tecnologia de persistência, de várias estratégias do banco de dados e até de várias fontes de dados.
- Eles comunicam decisões de design sobre acesso a objetos.
- Eles permitem a fácil substituição de uma implementação fictícia para uso em testes (normalmente utilizando uma coleção interna à memória.)

*Deixe o controle das transações a cargo do cliente*. Embora o repositório faca inserções e exclusões no banco de dados, normalmente ele não vai efetivar nada. É tentador efetivar após salvar, por exemplo, mas o cliente presumivelmente possui o contexto para iniciar e efetivar corretamente unidades de trabalho. O controle das transações será mais simples se o repositório mantiver suas mãos fora dele.

De modo geral, não brigue com as suas estruturas. Procure encontrar meios para preservar os fundamentos do DDD e deixe de lado as especificações quando a estrutura for antagonista. Procure encontrar afinidades entre os conceitos do DDD e os conceitos da estrutura. Para isso, considere-se que você não tem nenhuma escolha senão usar a estrutura. Se você tiver a liberdade, escolha estruturas, ou partes de estruturas, que sejam harmoniosas com o estilo de design que você deseja usar.

Uma fábrica cuida do início da vida de um objeto; um repositório ajuda a controlar o meio e o final.

A função de uma fábrica é instanciar um objeto potencialmente complexo a partir dos dados. Se o produto for um objeto novo, o cliente saberá disso e poderá acrescentá-los ao repositório, que vai encapsular a armazenagem do objeto no banco de dados.
