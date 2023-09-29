---
id: 9a3e7201-268d-4ba8-aa9a-ca35fea91451
title: "O que é o \"Modelo\" do Domínio?"
summary: "Resumo da parte \"Colocando o Modelo de Domínios em Ação\" do livro azul"
---

Evans começa essa parte do livro falando do mapa utilizado pela China no século XVIII para mostrar o mundo, mostrava ele estando no centro do mapa, e ocupando a maior parte do espaço, e ao redor do centro, estava a representação espalhada dos outros países. Ele explica que aquele mapa da visão do mundo para a China era um **modelo** adequado àquela sociedade, que intencionalmente havia sido fechada.

Assim, o _**modelo** do domínio_, pode ser definido como da seguinte forma:

> _(…) Mapas são modelos, e cada modelo representa algum aspecto da realidade com uma ideia que seja de interesse. **Um modelo é uma simplificação. Ele é uma interpretação da realidade que destaca os aspectos relevantes para resolver o problema que se tem em mãos ignorando os detalhes estranhos.** (EVANS, 2003)_

E o _**domínio**_ pode ser definido como da seguinte forma: 

> **Cada programa está relacionado a alguma atividade ou interesse do seu usuário. Essa área de assunto em que o usuário aplica o programa é o _domínio_ do software.** _(EVANS, 2003)_

(...) Para criar softwares que tenham valor para as atividades desempenhadas pelos usuários, a equipe de desenvolvimento deve trazer consigo um conjunto de conhecimentos relacionados a essas atividades. A quantidade de conhecimento necessário pode ser estarrecedora. O volume e a complexidade de informações podem ser imensos. **Modelos são ferramentas para atacar essa sobrecarga.** _**Um modelo é uma forma de conhecimento seletivamente simplificada e conscientemente estruturada. Um modelo adequado faz com que as informações tenham sentido e se concentra em um problema.** (EVANS, 2003)_

Um modelo de domínio não é um diagrama específico; ele é a ideia que o diagrama pretende transmitir. Ele não é simplesmente o conhecimento existente na cabeça de um especialista em domínios; *ele é uma abstração rigorosamente organizada e seletiva daquele conhecimento*. Um diagrama pode representar e comunicar um modelo, assim como acontece com um código cuidadosamente escrito, assim como uma frase em português. (EVANS, 2003)

A modelagem de domínios não é uma questão de se criar um modelo o mais "realista" possível. Mesmo em um domínio de coisas tangíveis da vida real, nosso modelo é uma criação artificial. Ele também não é simplesmente a construção de um mecanismo de software que fornece os resultados necessários. (EVANS, 2003)

## A utilidade de um modelo no Domain-Driven Design

No DDD (domain-driven design), são três utilidades básicas que determinam a escolha de um modelo.

1.  _**O modelo e o coração do design dão forma um ao outro**_: É essa ligação íntima entre o modelo e a implementação que torna o modelo relevante e garante que a análise a ele dedicada se aplique ao produto final, um programa de software. Essa ligação de modelo e implementação também ajuda durante a manutenção e contínuo desenvolvimento, pois o código pode ser interpretado com base na compreensão do modelo. (EVANS, 2003)

2.  _**O modelo é a espinha dorsal de uma linguagem utilizada por todos os membros da equipe**_: Devido à ligação de um modelo e implementação, os desenvolvedores podem conversar sobre o programa nessa linguagem. Eles podem comunicar-se com os especialistas do domínio sem a necessidade de tradução. E como a linguagem é baseada no modelo, nossa capacidade linguística natural pode ser usada para refinar o próprio modelo. (EVANS, 2003)

3.  _**O modelo é um conhecimento destilado**_: O modelo é a forma aceita pela equipe de estruturar o conhecimento do domínio e distinguir os elementos de maior interesse. Um modelo capta a forma que escolhemos para pensar no domínio à medida que selecionamos termos, interpretamos conceitos e os relacionamentos. A linguagem compartilhada permite que os desenvolvedores e os especialistas do domínio trabalhem efetivamente em conjunto à medida que colocam informações nessa forma. A ligação de modelo e implementação faz com que as experiências obtidas com versões anteriores do software sejam aplicáveis como feedback para o processo de modelagem. (EVANS, 2003)

*O uso de um modelo dessa forma pode sustentar o desenvolvimento do software com grandes funcionalidades que, de outra forma, precisariam de investimentos maciços de desenvolvimento na medida dos acontecimentos.* (EVANS, 2003)

## O coração do software

> _(...) A complexidade existente no coração de um software deve ser enfrentada cara a cara. Qualquer tentativa de se fazer o contrário é arriscar a irrelevância. (EVANS, 2003)_

O coração do software está na sua capacidade de resolver problemas relacionados ao domínio para o seu usuário. Quando o domínio é complexo, esta é uma tarefa difícil, exigindo a concentração de esforços de pessoas talentosas e capacitadas. Os desenvolvedores têm que mergulhar a fundo no domínio para adquirir conhecimentos sobre o negócio. É preciso afiar sua capacidade de modelagem e dominar *design de domínios*. (EVANS, 2003)

(...) os líderes de uma equipe que entendem os fundamentos de um domínio podem colocar o projeto / aplicação de volta em seu rumo quando o desenvolvimento de um *modelo* que reflete um profundo entendimento se perde em meio à confusão. (EVANS, 2003)

O livro mostra que o desenvolvimento de domínios traz oportunidades para cultivar capacidades bastante sofisticadas de design. A confusão da maioria dos domínios de software é, na verdade, um interessante desafio técnico. (EVANS, 2003)

O cultivo de técnicas de design torna um desenvolvedor muito mais valioso, até mesmo em um domínio inicialmente pouco familiar. (EVANS, 2003)
