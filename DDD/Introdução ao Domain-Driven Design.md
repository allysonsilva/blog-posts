---
id: 9963a259-bc4b-4ffa-958e-b6f1ab3a5c3e
title: "Introdução ao DDD (Domain-Driven Design)"
summary: "Introdução aos principais conceitos do DDD"
---

> **DDD** é uma abordagem de desenvolvimento de software que tenta trazer a linguagem de negócios e o código o mais próximo possível. Trata-se de como você deve pensar em um domínio, a linguagem que é utilizada para falar sobre ele e como você organiza o seu software para que ele reflita seu crescente entendimento sobre o assunto.
>
> DDD não é apenas tecnologia. Em seus princípios mais centrais, DDD envolve discutir, ouvir, compreender, descobrir e agregar valor ao negócio, tudo em um esforço para centralizar o conhecimento.

O livro azul (*Domain-Driven Design: Tackling Complexity in the Heart of Software*) descreve e constrói um vocabulário sobre a arte da modelagem de domínio. Ele é basicamente um **framework** para todos aqueles que querem aprender DDD por um cara (Eric Evans) que passou pelo menos 20 anos modelando, aprendendo e errado sobre a arte de se modelar domínios complexos. O que ele deixa de aprendizado, e ele mesmo fala no livro é: "Uma característica comum aos sucessos obtidos foi um modelo de domínio rico que evoluiu através de iterações de design e passaram a fazer parte do tecido que compunha o projeto".

**O livro então, oferece um framework para que se possam tomar decisões de design e um vocabulário técnico para discutir design de domínios. Ele é uma síntese das melhores práticas amplamente aceitas e as próprias opiniões e experiências de Eric Evans. Equipes de desenvolvimento de software que se deparam com domínios complexos podem se valer desse framework para abordar o DDD de forma sistemática.**

O livro ajuda desenvolvedores intermediários a aprender como aplicar sofisticadas técnicas de modelagem e design em problemas práticos.

São duas premissas do livro:

1. Na maioria dos projetos de software, o principal foco deve ser o domínio e a lógica do domínio.
2. Design de domínios complexos devem se basear em um modelo.

**Para melhor entendimento sobre DDD:**

- O DDD é uma maneira de pensar em um conjunto de prioridades, voltado para a aceleração de projetos de software que têm que trabalhar com domínios complicados. Para atingir esse objetivo, o livro apresenta um conjunto completo de práticas, técnicas. e princípios de design.

- O *DDD* é uma filosofia voltada ao domínio do negócio. O entendimento do domínio do negócio é um processo crescente, que necessita da cooperação e comunicação da equipe como um todo. Ao invés do modelo tradicional, em que a comunicação ocorre em apenas algumas etapas do projeto, temos um fluxo de comunicação contínua e colaborativa, por meio de uma única linguagem.

- O *DDD* é uma abordagem de modelagem de software que segue um conjunto de práticas com objetivo de facilitar a implementação de regras/processos complexos de negócios que tratamos como **domínio**.

- *Design Orientado a Domínio* não é sobre tecnologia. Em vez disso, trata-se de desenvolver conhecimento em torno dos negócios e usar a tecnologia para fornecer valor. É sobre fornecer valor no campo em que você está trabalhando, concentrando-se no modelo. Todos participam do processo de descoberta do domínio, desenvolvedores e especialistas em domínio se unem para criar a base de conhecimento compartilhando a mesma linguagem, a Linguagem Ubíqua.

- *Design Orientado a Domínio* fornece uma estrutura para design estratégico e tático - estratégico para identificar as áreas mais importantes a serem desenvolvidas com base no valor comercial e tático para criar um Modelo de Domínio funcional de blocos de construção e padrões já testados.

## Principais conceitos

**Domínio:** *Uma esfera de conhecimento, influência ou atividade. A área de assunto à qual o usuário aplica um programa é o domínio do software.*

**Modelo:** *Um sistema de abstrações que descreve aspectos selecionados de um domínio e pode ser usado para resolver problemas relacionados a esse domínio.*

**Linguagem Ubíqua/Onipresente:** *Uma linguagem estruturada em torno do modelo de domínio e usada por todos os membros da equipe em um contexto delimitado para conectar todas as atividades da equipe ao software.*

**Contexto:** *A configuração na qual uma palavra ou declaração aparece que determina seu significado. Declarações sobre um modelo só podem ser entendidas em um contexto.*

**Contexto Delimitado:** *Uma descrição de um limite (normalmente um subsistema ou o trabalho de uma equipe específica) dentro do qual um modelo específico é definido e aplicável. É um limite conceitual em torno de um sistema. A linguagem Ubíqua dentro de um limite tem um significado contextual específico. Conceitos fora deste contexto podem ter significados diferentes.*

## O desafio da complexidade

Muitas coisas podem fazer um projeto perder seu rumo: a burocracia, objetivos pouco claros e a falta de recursos, para citar apenas alguns. *Mas é a abordagem dada ao design que, em grande parte, determina até que ponto os softwares podem se tornar complexos*.

Quando a complexidade foge ao controle, os desenvolvedores já não podem entender o software bem o suficiente para alterá-lo ou expandi-lo com facilidade e segurança. Por outro lado, um bom design pode criar oportunidades para explorar essas características complexas.

Contudo, a complexidade mais significativa de muitos aplicativos não é a técnica. **Ela está no próprio domínio, a atividade ou negócio do usuário.** Quando essa complexidade de domínio não é tratada no design, não fará diferença se a linguagem  ou infra escolhida foi bem concebida. Um design de sucesso deve sistematicamente trabalhar com este aspecto fundamental.

## Design versus processo de desenvolvimento

O livro considera que existe duas práticas em ação no projeto. Essas duas práticas são pré-requisito para que se possa aplicar sua abordagem:

1. *O desenvolvimento é interativo:* Vem sido defendido e praticado há décadas, e é pedra fundamental dos métodos de desenvolvimento Agile.

2. *Desenvolvedores e especialistas em domínio têm uma relação íntima:* O DDD compacta uma grande quantidade de conhecimento em um modelo que reflete uma visão profunda do domínio e um enfoque nos conceitos principais. É uma colaboração entre quem conhece o domínio e quem sabe como construir o software. Como o desenvolvimento é interativo, essa colaboração deve continuar durante toda a vida do projeto.

O desenvolvimento Agile, reconhece a importância das decisões de design, mas resiste bastante ao design "aberto". Em vez disso, ele dedica um esforço admirável a comunicar e melhorar a capacidade que o projeto tem de mudar de rumo rapidamente. Com essa capacidade de reação, os desenvolvedores podem utilizar "aquilo que há de mais simples e funcional" em qualquer etapa de um projeto e, em seguida, fazer contínuas refatorações, realizando várias pequenas melhorias no design, chegando finalmente a um design que satisfaça as verdadeiras necessidades do cliente.

Esse minimalismo tem sido um antídoto de grande necessidade para alguns excessos cometidos pelos entusiastas do design. Alguns projetos têm ficado atolados por documentos incômodos que de pouco valem. Eles têm sofrido de "paralisia analítica", onde os membros das equipes têm tanto medo de um design imperfeito que acabam não progredindo.

Infelizmente, algumas ideias de processos podem ser mal interpretadas. Cada pessoa tem uma definição diferente do que vem a ser "mais simples".  A refatoração contínua é uma série de pequenos re-designs; desenvolvedores que não tenham solidificados os princípios do design vão gerar um banco de códigos difícil de entender ou mudar - o contrário da agilidade. Embora o medo de exigências não antecipadas geralmente leve ao exagero na área de engenharia, a tentativa de se evitar o exagero na engenharia pode se transformar em outro medo: o medo de raciocinar profundamente sobre o design. O processo Agile considera que seja possível melhorar um design através da refatoração, e que isso seja feito com frequência e rapidez.

## Uma equipe dirigida por domínios

Embora um desenvolvedor que entenda sobre DD possa adquirir excelentes técnicas e perspectivas de design, os maiores ganhos acontecem quando uma equipe se une para aplicar uma abordagem de DDD e trazer o modelo do domínio para o centro da discussão do projeto. Ao fazer isso, os membros da equipe compartilham uma linguagem que enriquece sua comunicação e a mantém ligada ao software. Eles conseguem produzir uma implementação lúcida em compasso com o modelo, alavancando o desenvolvimento do aplicativo. Eles compartilham um mapa da relação existente entre o trabalho de design de diversas equipes concentrando sistematicamente sua atenção nas características mais distintas e valiosas para a organização.

O DDD é um desafio técnico difícil que pode ser compensado por grandes oportunidades de abertura, exatamente quando a maioria dos projetos de software começa a se consolidar e a se transformar em um legado.

## Resumo

- A lição que o livro quer passar é que os modelos de domínio bons verdadeiramente evoluem com o tempo, e até mesmo os modeladores mais experientes acreditam que adquirem suas melhores ideias após o lançamento inicial de um sistema. E não que os modelos de domínios são modelados primeiro e depois implementados.
  
- Modelagem e design de domínios são tópicos fundamentais.

- Na modelagem de domínios, não se deve separar conceitos de implementação. Não se pode construir um modelo conceitual útil sem considerar a questão da implementação. Mas o principal motivo pela qual conceitos e implementação pertencem um ao outro é que o maior valor de um modelo de domínio está no fato de ele proporcionar um **ubiquitous language** que serve de união para especialistas e tecnólogo no domínio do negócio.

- O **coração** da *complexidade no desenvolvimento de software* está na **dificuldade** essencial do próprio *domínio-problema*. A chave para controlar essa complexidade é um bom modelo de domínio, um modelo que vá além da visão superficial de um domínio introduzindo uma estrutura subjacente, que oferece aos desenvolvedores de software o máximo de aproveitamento que eles procuram.
