---
id: 9a527a34-ba02-4fc9-bbcf-5859b1b4dfc8
title: Mantendo a Integridade do Modelo
summary: "Um modelo bem-sucedido, seja ele pequeno ou grande, deve ser logicamente consistente do início ao fim, sem definições contraditórias ou sobrepostas"
---

![A navigation map for model integrity patterns](/images/articles/a-navigation-map-for-model-integrity-patterns.png?id=b0b3dd675ab8d0eab4d6a9e8fb4ea622)

(...) o requisito mais fundamental de um modelo é que ele seja internamente consistente; que seus termos sempre tenham o mesmo significado e que ele não contenha nenhuma regra contraditória. A consistência interna de um modelo, de forma que cada termo não gere dúvidas e nenhuma regra seja contraditória, é chamada de *unificação*. Um modelo não tem significado a não ser que seja logicamente consistente. [Evans, p. 318]

Este capitulo estabelece as técnicas para reconhecer, comunicar e escolher os limites de um modelo e suas relações uns com os outros. Tudo começa com o mapeamento do atual terreno do projeto. Um CONTEXTO DELIMITADO define a faixa de aplicabilidade de cada modelo enquanto um MAPA DO CONTEXTO oferece uma visão global dos contextos do projeto e das relações entre eles. [Evans, p. 319]

## CONTEXTO DELIMITADO

Um CONTEXTO DELIMITADO é um limite conceitual em que um modelo de domínio é aplicável. A razão desse limite é destacar que todo uso de um dado termo, frase, sentença de domínio - a Linguagem Ubíqua - dentro do limite tem um significado contextual específico. Qualquer uso do termo fora desse limite poderia significar, e provavelmente significa, algo diferente. Ele fornece um contexto para a linguagem ubíqua que é falada pela equipe e expressa no modelo de software cuidadosamente projetado.

Um CONTEXTO DELIMITADO é um limite explicito dentro do qual um modelo de domínio existe. O modelo de domínio expressa uma linguagem ubíqua como um modelo de software. *O limite é criado porque cada um dos conceitos dentro do modelo, com suas propriedades e operações, tem um significado especial.*

![Cells can exist because their membranes define what is in and out and determine what can pass](/images/articles/cells-can-exist-because-their-membranes-define-what-is-in-and-out-and-determine-what-can-pass.png?id=13d231ca4721e1710f5f76dcfa002e34)

Um modelo se aplica a um contexto. O contexto pode ser uma determinada parte do código, ou o trabalho de uma determinada equipe. (...) O contexto do modelo é qualquer conjunto de condições que deve se aplicar para que se possa dizer que os termos de um modelo possuem um significado específico. [Evans, p. 322]

Para começar a resolver problemas de vários modelos, precisamos definir explicitamente o escopo de um determinado modelo como uma parte delimitada de um sistema de software dentro do qual um único modelo vai se aplicar e será mantido tão unificado quanto possível. Essa definição deve ser conciliada com a organização da equipe. [Evans, p. 322]

**Logo: Defina explicitamente o contexto dentro do qual um modelo se aplica. Estabeleça limites explicitamente em termos da organização da equipe, o uso dentro de partes específicas do aplicativo e as manifestações físicas, tais como as bases dos códigos e os esquemas dos bancos de dados. Mantenha o modelo estritamente consistente dentro desses limites, mas não se deixa distrair ou confundir-se com questões externas.** [Evans, p. 322]

Um CONTEXTO DELIMITADO delimita a aplicabilidade de um determinado modelo de forma que os membros da equipe tenham um entendimento claro e compartilhado do que deve ser consistente e como isso está relacionado a outros CONTEXTOS. Dentro desse CONTEXTO, trabalhe para manter o modelo logicamente unificado, mas não se preocupe com a aplicabilidade fora desses limites. Em outros CONTEXTOS, outros modelos se aplicam, com diferenças em terminologia, em conceitos e regras, e em dialetos da LINGUAGEM ONIPRESENTE. [Evans, p. 323]

Os limites são lugares especiais. As relações entre um CONTEXTO DELIMITADO e seus vizinhos requerem cuidado e atenção. O MAPA DO CONTEXTO marca o território, fornecendo o cenário geral dos CONTEXTOS e suas ligações, enquanto vários padrões definem a natureza das várias relações entre os CONTEXTOS. E um processo de INTEGRAÇÃO CONTÍNUA preserva a unidade do modelo dentro de um CONTEXTO DELIMITADO. [Evans, p. 324]

## MAPA DE CONTEXTO

- A maneira mais fácil de expressar um MAPA DE CONTEXTO é desenhar um diagrama simples que mostra os mapeamentos entre dois ou mais CONTEXTOS DELIMITADOS existentes.

- Além de lhe dar um inventário dos sistemas com os quais o contexto / parte deve interagir, um MAPA DE CONTEXTO serve como um catalisador para a comunicação inter-equipe.

![A simple context map that lists translations such as false cognates and duplicate concepts between two theoretical models in their bounded contexts](/images/articles/a-simple-context-map-that-lists-translations-such-as-false-cognates-and-duplicate-concepts-between-two-theoretical-models-in-their-bounded-contexts.jpg?id=b9f4b6ae0cd941add572a188f195eeaa)

Um MAPA DE CONTEXTO não é uma Arquitetura Corporativa ou diagrama da topologia do sistema. As informações são transmitidas em relação aos modelos de interação e padrões DDD organizacionais. Contudo, Mapas de Contexto podem ser usados em investigações arquitetônicas de alto nível, fornecendo pontos de vista da empresa de outro modo não disponível. Eles podem detectar deficiências arquitetônicas como gargalos de interação. Como eles exibem uma dinâmica organizacional, Mapas de Contexto podem até mesmo ajudar a identificar as difíceis questões de governança que poderiam bloquear o progresso, e outros desafios da equipe e gerência que são mais difíceis de revelar utilizando outros métodos.

A reutilização de códigos entre CONTEXTOS DELIMITADOS é um perigo a ser evitado. A integração de funcionalidades e dados deve passar por uma tradução. É possível reduzir possíveis confusões definindo a relação entre os diferentes contextos e criando uma visão global de todos os contextos dos modelos do projeto. [Evans, p. 329]

**Logo: Identifique cada modelo que está em jogo no projeto e defina seu CONTEXTO DELIMITADO. Isso inclui os modelos implícitos de subsistemas não orientados a objetos. Dê um nome a cada CONTEXTO DELIMITADO e faça com que os nomes passem a fazer parte da LINGUAGEM ONIPRESENTE.** [Evans, p. 330]

Dentro de cada CONTEXTO DELIMITADO, você tera um dialeto coerente com a LINGUAGEM ONIPRESENTE. Os próprios nomes dos CONTEXTOS DELIMITADOS entrarão naquela LINGUAGEM de forma que você possa falar sem ambiguidades sobre o modelo de qualquer parte do design, simplesmente fazendo com que o seu CONTEXTO seja claro. [Evans, p. 330]

### Organizando e documentando MAPAS DE CONTEXTO

São apenas dois os pontos importantes aqui:

- Os CONTEXTOS DELIMITADOS devem ter nomes para que você possa falar sobre eles. Esses nomes devem entrar na LINGUAGEM ONIPRESENTE da equipe.

- Todos devem saber onde estão as fronteiras e conseguir reconhecer o CONTEXTO de qualquer código ou qualquer situação.

### Relacionamento entre CONTEXTOS:

- **Parceria:** Quando as equipes em dois contextos são bem-sucedidas ou falham juntas, uma relação de cooperação deve surgir. As equipes instituem um processo para o planejamento coordenado do desenvolvimento e gerenciamento conjunto da integração. As equipes devem cooperar na evolução das suas interfaces para acomodar as necessidades de desenvolvimento de ambos os sistemas. Os recursos interdependentes, devem ser agendados de modo que eles estejam concluídos para o mesmo lançamento.

- **Kernel Compartilhado:** Compartilhar parte do modelo e código associado forma uma interdependência muito íntima, que pode alavancar ou minar o trabalho do projeto. Especifique um limite explicito para parte do subconjunto do modelo do domínio que as equipes concordam em compartilhar. Mantenha o Kernel pequeno. Esse material compartilhado explicito tem status especial e não deve ser alterado sem consulta com a outra equipe.

- **Cliente-Fornecedor:** Quando duas equipes estão em um relacionamento upstream / downstrem, em que a equipe upstream pode ser bem-sucedida de maneira interdependente do destino da equipe downstream, as necessidades da equipe downstream são atendidas de várias maneiras com uma ampla multiplicidade de consequências. Prioridades downstream entram no planejamento upstream. Negocie e especifique um orçamento das tarefas para os requisitos downstream de modo que todos compreendam o comprometimento e o cronograma.

- **Conformista:** Quando duas equipes de desenvolvimento têm um relacionamento upstream / downstream em que a equipe upstream não tem motivação para suprir as necessidades da equipe downstream, esta última fica impotente. Altruísmo pode motivar desenvolvedores upstream a fazer promessas, mais é provável que elas não sejam cumpridas. A equipe downstream elimina a complexidade da tradução entre Contextos Delimitados aderindo servilmente ao modelo da equipe upstream.

- **Camada Anticorrupção:** Camadas de tradução podem ser simples, mesmo elegantes, ao conectar Contextos Delimitados bem projetados com equipes cooperativas. Mas quando o controle ou a comunicação não é adequada para alcançar um relacionamento de kernel compartilhada, de parceiro ou de Cliente-Fornecedor, a tradução torna-se mais complexa. A camada de tradução assume um tom mais defensivo. Como um cliente downstream, crie uma camada isolante para fornecer ao sistema a funcionalidade do sistema upstream em termos de seu próprio modelo de domínio. Essa camada fala com o outro sistema por meio de sua interface existente, exigindo pouca ou nenhuma modificação no outro sistema. Internamente, a camada traduz em uma ou ambas as direções, conforme necessário entre os dois modelos.

## LINGUAGEM UBÍQUA

A Linguagem Ubíqua é uma linguagem compartilhada pela equipe. Ela é compartilhada por desenvolvedores e especialistas em domínio. Na verdade, é compartilhada por todos na equipe de projeto. Independentemente de seu papel na equipe, como você está nela, você usa a Linguagem Ubíqua do projeto.

### Ubíqua, mas não *universal*

- Ubíqua significa "generalizada", ou "encontrada em todos os lugares", como falada entre a equipe e expressa pelo modelo de domínio único que a equipe desenvolve.

- O uso da palavra "ubíqua" não é uma tentativa de descrever algum tipo de linguagem de domínio universal por toda a empresa ou em todo o mundo.

- Há uma linguagem ubíqua por Contexto Delimitado.

- A linguagem só é ubíqua dentro da equipe que trabalha no projeto que se desenvolve em um contexto delimitado isolado.

- Em um único projeto que desenvolve um único contexto delimitado, sempre há um ou mais contextos delimitados adicionais com os quais ele se integra usando Mapas de Contextos. Cada um dos múltiplos contextos delimitados que se integram tem sua própria linguagem ubíqua, embora alguns termos de cada um possam se sobrepor.
