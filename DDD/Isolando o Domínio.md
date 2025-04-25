---
id: 9a512065-2b18-4471-9b7a-c149bd635382
title: Isolando o Domínio
summary: "A camada do domínio no DDD é onde se encontra as regras do negócio, que resolve e trata dos problemas e soluções do negócio"
---

![DDD Layered Architecture](/images/articles/DDD/DDD-layered-achitecture.png?id=063a253a846dce74686546f485bec4d1)

- Precisamos desacoplar os objetos de domínio de outras funções do sistema para evitarmos confundir os conceitos do domínio com outros conceitos relacionados somente à tecnologia do software ou perder de vista o domínio como um todo na massa formada pelo sistema. [Evans, p. 63]

- A criação de programas que possam controlar tarefas muito complexas exige a separação das coisas, permitindo isolar a concentração em partes diferentes do design. Ao mesmo tempo, as complicadas interações dentro do sistema devem ser mantidas apesar da separação. [Evans, p. 65]

- Existem várias maneiras de se dividir um sistema de software, mas, através da experiência e de convenções, o setor convergiu para as ARQUITETURAS EM CAMADAS e, especificamente, algumas camadas relativamente padronizadas. *O princípio essencial de uma arquitetura em camadas é de que qualquer elemento de uma camada depende somente dos outros elementos da mesma camada ou dos elementos das camadas "abaixo" dela. A comunicação para cima deve passar por algum tipo de mecanismos indireto.* [Evans, p. 65]

O valor das camadas é que cada uma se especializa em um determinado aspecto de um software. Essa especialização permite designs mais coesos de cada aspecto, e facilita muito mais a interpretação desses designs. Obviamente, é fundamental escolher camadas que isolem os aspectos mias importantes do design coeso. Novamente, a experiência e as convenções levaram a uma certa convergência. Embora existam muitas variações, a maioria das arquiteturas bem-sucedias utiliza alguma versão dessas quatro camadas conceituais: [Evans, p. 65]


| **Camada**                                              | **Definição**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
|:---------------------------------------------------:|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
| *Interface com o usuário (ou camada de apresentação)* | Responsável por mostrar informações ao usuário e interpretar os comandos do usuário.                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| *Camada da aplicação*                                 | Define as funções que o software deve executar e direciona os objetos expressivos do domínio para resolver os problemas. Essa camada é mantida estreita. Ela não contém as regras ou conhecimento do negócio, **mas apenas coordena as tarefas e delega trabalhos para conjuntos de objetos e o domínio na camada logo abaixo**. Ela não tem um estado que reflita a situação do negócio, mas pode ter um estado que reflita o andamento de uma tarefa para o usuário ou programa. Essa camada sabe "quando" fazer determinada coisa, mais não se preocupa com o "como". Responsável por orquestrar, organizar e encapsular o comportamento do domínio e controlar o acesso aos dados. |
| *Camada do domínio (ou camada do modelo)*             | Responsável por representar conceitos do negócio, informações sobre a situação do negócio e regras do negócio. Aqui, um estado que reflete a situação do negócio é controlado e usado, embora os detalhes técnicos de sua armazenagem sejam delegados à infraestrutura. **Esta camada é o coração do software do negócio.**                                                                                                                                                                                                |
| *Camada de infraestrutura*                            | Fornece recursos técnicos genéricos que suportam as camadas mais altas: envio de mensagens para o aplicativo, persistência do domínio, desenho de widgets para a IU, e assim por diante. A camada de infraestrutura pode também suportar o padrão de interações entre as quatro camadas através de um framework arquitetural.                                                                                                                                                                                                |

Alguns projetos não fazem uma distinção rígida entre as camadas da interface com o usuário e a do aplicativo. Outros possuem várias camadas de infraestrutura. Mas é a separação fundamental da camada de domínio que possibilita o DESIGN DIRIGIDO POR MODELOS. [Evans, p. 66]

**Logo: Divida um programa complexo em camadas. Desenvolva um design dentro de cada camada que seja coeso e que _dependa somente das camadas abaixo dele_. Siga os padrões de arquitetura usados como padrão para proporcionar um baixo acoplamento com as camadas acima. Concentre todo o código relacionado ao modelo do domínio em uma camada e isole-o do código da interface com o usuário, do aplicativo e da infraestrutura. Os objetos de domínio, livres da responsabilidade de se exibir, de se armazenar, de gerenciar tarefas do aplicativo, e assim por diante, podem se concentrar em expressar o modelo do domínio. Isso permite que um modelo evolua para se tornar rico e limpo o suficiente para captar o conhecimento essencial do negócio e colocá-lo em funcionamento.** [Evans, p. 66]

Camadas isoladas exigem uma manutenção muito mais barata, porque tendem a evoluir em ritmos diferentes e a responder a diversas necessidades. A separação também ajuda na aplicação em um sistema distribuído, permitindo que diferentes camadas sejam colocadas flexivelmente em diferentes servidores ou clientes, para minimizar os custos de comunicação e melhorar o desempenho. [Evans, p. 66, 67]

## Relacionando as camadas

- As camadas têm por objetivo possuir baixo acoplamento, com as dependências do design dispostas em apenas um sentido. As camadas superiores podem utilizar ou manipular elementos das camadas inferiores de forma bastante simples, chamando suas interfaces públicas, fazendo referências a elas (pelo menos temporariamente) e geralmente usando meios convencionais de interação. Mas quando um objeto de um nível inferior precisa comunicar-se para cima (além da resposta a uma consulta direta), precisamos de outro mecanismo, baseado em padrões de arquitetura para relacionar camadas tais como chamadas de retorno ou OBSERVERS. [Evans, p. 68, 69]

- As camadas do aplicativo e do domínio chamam os SERVIÇOS fornecidos pela camada da infraestrutura. Quando o escopo de um SERVIÇO foi bem escolhido e a sua interface foi bem elaborada, a parte que chama pode permanecer livremente acoplada sem ter complicações com o comportamento elaborado que a interface SERVIÇO encapsula. [Evans, p. 69]

## Estruturas arquitetônicas

- As melhores estruturas arquitetônicas resolvem problemas técnicos complexos permitindo, ao mesmo tempo, que o desenvolvedor do domínio se concentre em expressar o modelo. Mas as estruturas podem facilmente entrar no caminho, fazendo considerações demais que restringem as opções de design do domínio ou sobrecarregando demasiadamente a implementação a ponto de desacelerar o desenvolvimento. [Evans, p. 70]

- Ao aplicar uma estrutura, a equipe precisa se concentrar em seu objetivo: construir uma implementação que expresse o modelo do domínio e o utilize para resolver problemas importantes. A equipe deve procurar formas de empregar a estrutura com essa finalidade, mesmo que isso signifique não usar todos os recursos da estrutura. [Evans, p. 70]

- A aplicação cautelosa apenas dos recursos mais valiosos da estrutura reduz o acoplamento da implementação e da estrutura, permitindo maior flexibilidade em decisões posteriores sobre o design. [Evans, p. 70]

## É na camada de domínio que reside o modelo

- O modelo do domínio é um conjunto de conceitos. A "camada do domínio" é a manifestação desse modelo e de todos os elementos do design diretamente relacionados. O design e a implementação da lógica do negócio constituem a camada do domínio. Em um DESIGN DIRIGIDO POR MODELOS, as ideias que o software tem da camada do domínio refletem os conceitos do modelo. É impraticável atingir essa correspondência quando a lógica do domínio se mistura a outros detalhes do programa. **O isolamento da implementação do domínio é um pré-requisito do domain-driven design.** [Evans, p. 71]

- Se uma equipe simples com um projeto simples decidir experimentar um DESIGN DIRIGIDO POR MODELOS utilizando a ARQUITETURA EM CAMADAS, ela enfrentará uma difícil curva de aprendizado. Os membros da equipe terão que dominar novas tecnologias complexas e lutar com o processo de aprendizado da modelagem de objetos (que é um grande desafio). O trabalho de gerenciar a infraestrutura e as camadas faz com que tarefas muito simples demorem mais. Projetos simples trazem consigo prazos curtos e expectativas modestas. Muito antes de a equipe concluir a tarefa designada, e muito menos demonstrar as grandes possibilidades de sua abordagem, o projeto terá sido cancelado. [Evans, p. 72]

- Mesmo que a equipe consiga mais tempo, os membros da equipe provavelmente não conseguirão dominar as técnicas sem a ajuda de um especialista. E, no final, se conseguirem superar esses desafios, terão produzido um sistema simples. Recursos mais poderosos nunca foram solicitados. [Evans, p. 72]

- **O DDD é mais compensador para projetos ambiciosos, e requer grandes habilidades. Nem todos os projetos são ambiciosos. Nem todas as equipes de projeto podem reunir essas habilidades.** [Evans, p. 72]

- **Se a arquitetura isola o código relacionado ao domínio para permitir um design de domínio coeso e de baixo acoplamento do resto do sistema, então essa arquitetura pode provavelmente suportar o domain-driven design.** [Evans, p. 75]

- Outros estilos de desenvolvimento têm seu lugar, mais é preciso aceitar limites variáveis de complexidade e flexibilidade. Uma falha ao desacoplar o design do domínio pode realmente ser desastrosa em determinados ambientes. [Evans, p. 75]

- A melhor parte do isolamento de um domínio é a eliminação de todas as outras coisas do caminho para podermos realmente nos concentrar no design do domínio. [Evans, p. 75]
