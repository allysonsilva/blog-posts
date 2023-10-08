---
id: 9a5232cf-cde2-48f9-88d4-52216b34126b
title: Um Modelo Expresso em Software
summary: Esse artigo se concentra nos elementos do modelo, colocando em foco os principais conceitos como Entidades e Serviços"
---

- Para que possa haver um comprometimento com a implementação sem que se perca o efeito de um DESIGN DIRIGIDO POR MODELOS, é necessária uma restruturação das coisas básicas. **A ligação entre modelo e implementação tem que ser feita detalhadamente.** [Evans, p. 77]

- As ideias de alta coesão e baixo acoplamento, geralmente vistas como uma medida técnica, podem ser aplicadas aos próprios conceitos. Em um DESIGN DIRIGIDO POR MODELOS, *os MÓDULOS fazem parte do modelo e devem refletir os conceitos do domínio.* [Evans, p. 78]

## ENTIDADES (OU OBJETOS DE REFERÊNCIA)

- **Alguns objetos não são definidos principalmente por seus atributos. Eles representam uma linha de identidade que atravessa o tempo e geralmente representações distintas. Às vezes, um objeto pode ser combinado com outros objetos, embora os atributos possam diferir. Um objeto deve ser distinguido de outros objetos, embora eles possam ter os mesmos atributos. Um erro de identidade pode levar a dados corrompidos.** [Evans, p. 85, 86]

- Um objeto definido princialmente por sua identidade é chamado de ENTIDADE. ENTIDADES têm considerações especiais de modelagem e design. Elas possuem ciclos de vida que podem radicalmente mudar sua forma e conteúdo, mas deve ser mantida uma linha de continuidade. Suas identidades devem ser definidas de forma que possam ser efetivamente rastreadas. Suas definições de classes, responsabilidades, atributos e associações devem girar em torno do que elas são, em vez dos atributos específicos que elas carregam consigo. Até mesmo para ENTIDADES que não se transformam de forma tão radical ou que não tenham ciclos de vida tão complicados, sua colocação na categoria semântica leva a modelos mais lúcidos e implementações mais robustas. [Evans, p. 86]

- Uma ENTIDADE é qualquer coisa que tenha continuidade através de um ciclo de vida e distinções independentes de atributos que sejam importantes para o usuário do aplicativo. [Evans, p. 86]

- **Quando um objeto é distinguido por sua identidade, em vez de seus atributos, faça com que isso seja essencial para sua definição do modelo. Mantenha a definição da classe simples e concentrada na continuidade do ciclo de vida e na identidade. Defina um meio de distinguir cada objeto, independentemente de sua forma ou histórico. O MODELO deve definir o que _significa_ ser a mesma coisa.** [Evans, p. 87]

- (...) a responsabilidade mais básica das ENTIDADES é estabelecer a continuidade de forma que o comportamento possa ser claro e previsível. [Evans, p. 88]

- Além das questões de identidade, as ENTIDADES tendem a cumprir suas responsabilidades coordenando as operações de objetos que elas possuem. [Evans, p. 88]

## OBJETOS DE VALOR

- *Um objeto que representa um aspecto descritivo do domínio sem nenhuma identidade conceitual é chamado de OBJETO DE VALOR. OBJETOS DE VALOR são instanciados para representar elementos do design com os quais só nos preocupamos pelo QUE eles são, e não QUEM ou QUAIS são.* [Evans, p. 93]

- OBJETO DE VALOR é uma classe elementar que contém principalmente (mas não apenas) dados escalares. Portanto, é uma *classe wrapper* que reúne informações relacionadas. **É imutável**. Não tem setters e apenas propriedades somente leitura.

- Dois OBJETOS DE VALOR são iguais quando os valores são os mesmos, igualdade se dá por valor e não por referência na memória.

- Um OBJETO DE VALOR pode ser uma montagem de outros objetos. [Evans, p. 93]

- OBJETOS DE VALOR podem até se referir a entidades. [Evans, p. 94]

- (...) são geralmente transmitidos como parâmetros em mensagens entre objetos. Eles são frequentemente transitórios, criados por uma operação e, em seguida, descartados. [Evans, p. 94]

- **Quando você só se preocupa com os atributos de um elemento do modelo, classifique-o como um OBJETO DE VALOR. Faça com que ele expresse o significado dos atributos que ele transmite e dê a ele uma funcionalidade relacionada. Trate o OBJETO DE VALOR como imutável. Não dê a ele nenhuma identidade e evite as complexidades de design necessárias para manter ENTIDADES.** [Evans, p. 94]

- Os atributos que formam um OBJETO DE VALOR devem formar um todo conceitual. [Evans, p. 94]

- Definir OBJETOS DE VALOR e designá-los como imutáveis se trata de seguir uma regra geral: evitar restrições desnecessárias em um modelo deixa os desenvolvedores livres para realizarem a sincronização puramente técnica do desempenho. Definir explicitamente as restrições essenciais permitem que os desenvolvedores aprimorem o design evitando, aos mesmo tempo, alterar um comportamento significativo. Esses aprimoramentos no design são geralmente mais específicos para a tecnologia em uso em um determinado projeto. [Evans, p. 96, 97]

#### Ao tentar decidir se um conceito é um Valor, você deve determinar se ele possui a maioria destas características:

- Ele mede, quantifica ou descreve uma coisa no domínio.
    - Um objeto de valor, é um conceito que mede, quantifica ou de outra forma descreve uma coisa no domínio.

- Ele pode ser mantido imutável.
    - Após ser criado ele torna-se imutável.
    - Depois que o objeto foi instanciado e inicializado, nenhum de seus métodos públicos ou privados, a partir desse momento em diante, faz com que seu estado mude.

- Ele modela um todo conceitual compondo atributos relacionados como uma unidade integral.
    - Um objeto de valor pode possuir apenas um ou alguns atributos individuais, cada um dos quais está relacionado entre si. Cada atributo contribui com uma parte importante para um todo que o atributo descreve coletivamente. Separado dos outros, cada um dos atributos não consegue fornecer um significado coeso. Somente juntos é que todos os atributos formam a medida ou descrição completa pretendida. Isso é diferente de simplesmente agrupar um conjunto de atributos em um objeto. O próprio agrupamento alcança pouco se o todo não conseguir descrever adequadamente outra coisa no modelo.

- Ele é completamente substituível quando a medição ou descrição muda.

- Ele pode ser comparado com outros usando a igualdade de Valor.
    - Quando uma instância do objeto de valor é comparada com outra instância, é usado um teste de igualdade de objeto. Pode haver por todo o sistema, muitas instâncias de Valor que são iguais, mais que não são os mesmos objetos. A igualdade é determinada comparando os tipos de ambos os objetos e então seus atributos. Se tanto os tipos com seus atributos forem iguais, os Valores serão considerados iguais.

- Ele fornece para os colaboradores comportamento livre de efeitos colaterais.
    - Um método de um objeto pode ser projetado como uma Função Livre de Efeitos Coletarias. Uma função é uma operação de um objeto que produz uma saída, mas sem modificar seu próprio estado. Como nenhuma modificação ocorre ao executar uma operação específica, diz-se que essa operação é livre de efeitos colaterais. Todos os métodos de um Objeto de Valor imutável devem ser Funções Livres de Efeitos Colaterais, porque eles não devem violar a qualidade da imutabilidade.

## SERVIÇOS

- Às vezes, a situação simplesmente não se trata de uma coisa. Em alguns casos, um design mais limpo e mais pragmático inclui operações que não pertencem conceitualmente a nenhum objeto. Em vez de forçar a questão, podemos seguir os limites naturais do espaço do problema e incluir SERVIÇOS explicitamente no modelo. [Evans, p. 99]

- Existem operações de domínio importantes que não podem encontrar um lar natural em uma ENTIDADE ou OBJETO DE VALOR. Algumas dessas operações são intrinsecamente atividades ou ações, e não coisas, mais, como nosso paradigma de modelagem são objetos, tentamos enquadrá-las em objetos de alguma forma. [Evans, p. 99]

- Um SERVIÇO é uma operação oferecida como interface que fica isolada no modelo, sem um estado de encapsulamento, como acontece com as ENTIDADES e OBJETOS DE VALOR. SERVIÇOS são um padrão comum nas estruturas técnicas, mas eles também podem se aplicar à camada de domínio. [Evans, p. 100]

- O nome serviço enfatiza a relação com outros objetos. Ao contrário de ENTIDADES e OBJETOS DE VALOR, ele é definido puramente em termos do que ele pode fazer para um cliente. **Um SERVIÇO tende a ser nomeado de acordo com uma atividade, em vez de uma entidade - ele é um verbo em vez de um substantivo.** [Evans, p. 100]

- Os nomes das operações devem ser provenientes da LINGUAGEM ONIPRESENTE ou serem introduzidos nela. Parâmetros e resultados devem ser objetos do domínio. [Evans, p. 100]

- SERVIÇOS devem ser usados conscientemente e não se deve permitir que ele roube o comportamento das ENTIDADES e dos OBJETOS DE VALOR. Mais quando uma operação é realmente um conceito de domínio importante, uma SERVIÇO forma a parte natural de um DESIGN DIRIGIDO POR MODELOS. [Evans, p. 100]

Um SERVIÇO tem três características:

1. A operação está relacionada a um conceito do domínio que não é uma parte natural de uma ENTIDADE ou de um OBJETO DE VALOR.

2. A interface é definida em termos de outros elementos do modelo do domínio.

3. A operação não possui estado.

**Quando um processo significativo ou uma transformação no domínio não é uma responsabilidade natural de uma ENTIDADE, ou OBJETO DE VALOR, acrescente uma operação no modelo como uma interface autônoma declarada como SERVIÇO. Defina a interface em termos da linguagem do modelo e certifique-se de que o nome da operação faça parte da LINGUAGEM ONIPRESENTE. Faca com que o serviço não tenha um estado.** [Evans, p. 100, 101]

### SERVIÇOS e a camada do domínio isolada

- Esse padrão se concentra naqueles SERVIÇOS com um significado importante no domínio por si só, mas, obviamente, SERVIÇOS não são usados apenas na camada do domínio. Esse padrão cuida para que ele possa distinguir SERVIÇOS que pertencem à camada do domínio daqueles de outras camadas, e para que possa fatorar as responsabilidades para manter essa distinção clara. [Evans, p. 101]

- A maioria dos SERVIÇOS discutida na literatura é puramente técnica e pertence à camada da infraestrutura. SERVIÇOS de domínio e de aplicativos trabalham em conjunto com esses SERVIÇOS da infraestrutura. [Evans, p. 101]

#### Dividindo serviços em camadas

**Aplicativo**:

- Digere as entradas.
- Envia mensagens para o serviço do domínio para complementação.

**Domínio**:

- Interage com os objetos do domínio

## MÓDULOS (TAMBÉM CONHECIDOS COMO PACOTES)

- MÓDULOS são elementos de design antigos e estabelecidos. Existem considerações técnicas, mas a sobrecarga cognitiva é a principal motivação para a modularidade. MÓDULOS dão às pessoas duas visões do modelo: elas podem examinar os detalhes de um MÓDULO sem serem ofuscados pelo todo, ou podem examinar as relações entre os módulos em visões que excluem detalhes anteriores. [Evans, p. 104]

**Todo mundo utiliza MÓDULOS, mas poucas pessoas os tratam como partes do modelo já completamente desenvolvidas. O código é dividido em vários tipos de categorias, desde aspectos da arquitetura técnica até as atribuições de trabalho dos desenvolvedores. Mesmo os desenvolvedores que refatoram muito tendem a se contentar com MÓDULOS concebidos na fase inicial do projeto.**

**É óbvio que deva haver um baixo acoplamento entre MÓDULOS e uma alta coesão entre eles. As explicações de acoplamento e coesões tendem a fazer com que eles pareçam uma medida técnica, a ser jugada mecanicamente com base nas distribuições das associações e das interações. Contudo, não se trata simplesmente de um código sendo dividido em MÓDULOS, mas de conceitos. Existe um limite para quantas coisas uma pessoa pode pensar ao mesmo tempo (daí, um baixo acoplamento). Fragmentos incoerentes de ideias são tão difíceis de entender como uma sopa de ideias não diferenciadas (daí, uma alta coesão).** [Evans, p. 104]

Baixo acoplamento e alta coesão são princípios gerais de design que se aplicam tanto a cada objeto quando a MÓDULOS, mas são principalmente importantes nessa granularidade maior de modelagem e design. Esses termos já existem há algum tempo.

Sempre que dois elementos do modelo são separadas em módulos diferentes, as relações entre eles se tornam menos diretas que antes, o que aumenta a sobrecarga para se entender o seu lugar no design. Um baixo acoplamento entre MÓDULOS minimiza esse custo e possibilita a análise do conteúdo de um MÓDULO com o mínimo de referência aos outros que interagem.

Ao mesmo tempo, os elementos de um bom modelo têm sinergia, e MÓDULOS bem escolhidos juntam elementos do modelo que possuem relações conceituais particularmente ricas. Essa alta coesão de objetos com responsabilidades relacionadas entre si permite que o trabalho de modelagem e design se concentre dentro de um único MÓDULO, uma escola de complexidade que a mente humana pode facilmente controlar. [Evans, p. 104]

- Como todas as outras coisas em DDD, os MÓDULOS são um mecanismo de comunicação. O significado da divisão dos objetos precisa conduzir a escolha dos módulos. Ao juntar algumas classes dentro de um módulo, você está dizendo ao próximo desenvolvedor (que examinar o seu design) que ele deve pensar nelas em conjunto. Se o seu modelo está contando uma história, os MÓDULOS são capítulos. O nome do MÓDULO transmite o seu significado. Esses nomes passa a fazer parte da LINGUAGEM ONIPRESENTE. [Evans, p. 104]

- **Escolha MÓDULOS que contem a história do sistema e contenham um conjunto coeso de conceitos. Procure um baixo acoplamento no sentido de conceitos que possam ser entendidos e analisados independentemente uns dos outros. Refine o modelo até que ele se divida de acordo com os conceitos avançados do domínio e o código correspondente também seja desacoplado. Dê nome aos MÓDULOS que passem a fazer parte da linguagem onipresente. MÓDULOS e seus nomes devem refletir uma visão aprofundada do domínio.** [Evans, p. 104]
