---
id: 9a4e558c-0c80-44ad-a427-26f8413f9d35
title: Comunicação e Uso da Linguagem
summary: "Técnicas para uma boa comunicação e uso da linguagem no projeto"
---

Um modelo de domínio pode ser a base de uma linguagem comum para um projeto de software. O modelo é um conjunto de conceitos desenvolvidos nas cabeças das pessoas que participam do projeto, com termos e relações que refletem a visão do domínio. Esses termos e inter-relações fornecem a semântica de uma linguagem moldada conforme o domínio, sendo, ao mesmo tempo, precisa o suficiente para o desenvolvimento técnico. Este é um fio essencial que tece o modelo, transformando-o em uma atividade de desenvolvimento e ligando-o ao código. Essa comunicação baseada no modelo não está limitada a diagramas UML. Para que o modelo possa ser usado com o máximo de eficiência possível, é preciso permear cada meio de comunicação. Isso aumenta a utilidade dos documentos de texto escritos, bem como os diagramas informais e a conversa casual re-enfatizada nos processos Agile. Assim, melhora-se a comunicação através do próprio código e através dos testes feitos para aquele código. [Evans, p. 21]

(...) Para que o modelo possa ser usado com o máximo de eficiência possível, é preciso permear cada meio de comunicação. Isso aumenta a utilidade dos documentos de texto escritos, bem como os diagramas informais e a conversação casual re-enfatizada nos processos Agile. Assim, melhora-se a comunicação através do próprio código e através dos testes feitos para aquele código. [Evans, p. 21]

## Linguagem Ubíqua

Para criar um design flexível e rico em conhecimento é necessário ter uma linguagem versátil compartilhada pela equipe e uma experiência ativa com a linguagem que raramente acontecem em projetos de software. [Evans, p. 22]

Desenvolvedores que não entendem o domínio e nem os termos da linguagem ubíqua criam seus próprios conceitos a respeito do design e formas de descrever o domínio. [Evans, p. 22]

Em meio a essa divisão linguística, os especialistas de um domínio descrevem vagamente o que querem. Os desenvolvedores, lutando para entender um domínio novo para eles, entendem tudo vagamente. [Evans, p. 22]

Em um projeto sem uma linguagem em comum, os desenvolvedores têm que traduzir para os especialistas do domínio. Esses especialistas traduzem entre cada desenvolvedor e também entre outros especialistas do domínio. Até mesmo os desenvolvedores traduzem uns para os outros. A tradução confunde os conceitos do modelo, o que leva a uma refatoração destrutiva do código. Uma comunicação indireta esconde a formação de divergências - diferentes membros da equipe utilizam termos de forma diferente, sem perceber. Isso leva a um software pouco confiável cujas peças não se encaixam perfeitamente. A tentativa de tradução impede a troca mútua de conhecimento e ideias que levam a visões profundas do modelo. [Evans, p. 22]

**Um projeto enfrenta sérios problemas quando sua linguagem é ferida. Os especialistas daquele domínio utilizam seu jargão, enquanto os membros da equipe técnica possuem sua própria linguagem sintonizada para discutir o domínio em termos de design.** [Evans, p. 23]

**A terminologia das discussões do dia a dia fica desligada da tecnologia embutida no código (que, em última instância, é o produto mais importante de um projeto de software). E até mesmo uma única pessoa utiliza uma linguagem diferente na fala e na escrita, de forma que as expressões mais incisivas do domínio geralmente aparecem em uma forma transitória que nunca é captada no código ou até mesmo na escrita.** [Evans, p. 23]

**A tradução enfraquece a comunicação e torna anêmica a assimilação do conhecimento.** [Evans, p. 23]

O custo de toda a tradução, além do risco de entendimento errado, é simplesmente muito alto. Um projeto precisa de uma linguagem em comum que seja mais robusta que o mínimo denominador comum. Através do esforço consciente da equipe, *o modelo do domínio pode proporcionar a espinha dorsal daquela linguagem em comum, ligando ao mesmo tempo a comunicação da equipe com a implementação de software. Essa linguagem pode ser onipresente no trabalho da equipe.* [Evans, p. 23]

O vocabulário dessa LINGUAGEM ONIPRESENTE inclui os nomes das classes e operações de destaque. A LINGUAGEM inclui termos para discutir regras que se tornaram explícitas no modelo. E é suplementada com termos provenientes de princípios de organização de alto nível imposto sobre o modelo (tais como MAPAS DE CONTEXTO e estrutura em larga escala). Finalmente, essa linguagem é enriquecida através dos nomes dos padrões que a equipe comumente aplica ao modelo do domínio. [Evans, p. 23]

As relações do modelo se tornam as regras combinatórias que todas as linguagens possuem. Os significados das palavras e das frases ecoam a semântica do modelo. [Evans, p. 23]

A linguagem baseada no modelo deve ser usada entre os desenvolvedores para descrever não só artefatos do sistema, mas tarefas e funcionalidades. Este mesmo modelo deve suprir a linguagem para os desenvolvedores e especialistas do domínio para que eles possam comunicar-se uns com os outros, e para que os especialistas do domínio possam comunicar entre si sobre os requisitos, o planejamento do desenvolvimento e os recursos. Quanto mais permeável for a ação da linguagem, mais suave será o fluxo do entendimento. [Evans, p. 23]

Ao usarmos extensivamente a linguagem baseada no modelo e não ficarmos satisfeitos até que ela consiga fluir, chegamos a um modelo que é completo e compreensível, formado por elementos simples que combinam entre si para expressar ideias complexas. Logo: [Evans, p. 24, 25]

> **Use o modelo como a espinha dorsal da linguagem. Comprometa equipe a exercitar essa linguagem de forma implacável em todas as comunicações feitas dentro da equipe e no código. Use a mesma linguagem em diagramas, na escrita, e principalmente na fala.**
>
> **Elimine as dificuldades experimentando expressões alternativas, que reflitam modelos alternativos. Em seguida, refatore o código, renomeando classes, métodos e módulos para se adequar ao novo modelo. Resolva as confusões com os termos da conversação, exatamente como chegamos a um acordo sobre o significado das palavras comuns.**
>
> **Reconheça que uma mudança ocorrida na LINGUAGEM ONIPRESENTE é uma mudança no modelo.**
>
> **Os especialistas do domínio devem fazer objeção a termos ou estrutura que sejam estranhos ou inadequados para transmitir a compreensão do domínio; os desenvolvedores devem prestar atenção a ambiguidades ou inconsistências que possam atrapalhar o design.**

Com uma LINGUAGEM ONIPRESENTE, o modelo não é simplesmente um artefato do projeto. Ele se torna parte integral de tudo que os desenvolvedores e especialistas do domínio fazem juntos. A LINGUAGEM transmite o conhecimento de forma dinâmica. As discussões ocorridas na LINGUAGEM dão vida ao significado existente por trás dos diagramas e dos códigos. [Evans, p. 25]

A LINGUAGEM ONIPRESENTE é o principal portador dos aspectos do design que não aparecem no código - estruturas em larga escala que organizam o sistema inteiro, CONTEXTOS DELIMITADOS que definem as relações de diferentes sistemas e modelos, e outros padrões aplicados ao modelo e ao design. [Evans, p. 25]

## Modelando em voz alta

(...) mas, da próxima vez que você estiver presente a uma discussão sobre requisitos ou design, ouça atentamente. Você vai ouvir descrições de recursos no jargão comercial ou em versões do jargão usadas pelo leigo. Você vai ouvir conversas sobre artefatos técnicos e funcionalidades concretas. Sim, você também vai ouvir termos do modelo do domínio; substantivos óbvios na linguagem comum provenientes do jargão comercial normalmente serão codificados como objetos, e, por isso, esses termos tenderão a ser mencionados. [Evans, p. 28]

Uma das melhores maneiras de refinar um modelo é explorar utilizando a fala, tentando, em voz alta, várias construções a partir das possíveis variações do modelo. É fácil ouvir aquilo que não soa bem. [Evans, p. 29]

É importantíssimo que brinquemos com as palavras e frases, exercitando nossas habilidades linguísticas em função do esforço de modelagem, assim como é importantíssimo que engajemos nosso raciocínio visual/espacial esboçando diagramas, assim como empregamos nossa capacidade analítica com uma análise e um design metódicos, e aquela "percepção" misteriosa no código. Essas formas de pensar se complementam uma à outra, e todas são necessárias para encontrar modelos e design que sejam úteis. [Evans, p. 29]

A medida que usamos a LINGUAGEM ONIPRESENTE do modelo do domínio nas discussões - sobretudo discussões em que os desenvolvedores e dos especialistas do domínio esboçam cenários e requisitos - ficamos mais fluentes na linguagem e ensinamentos uns aos outras suas nuances. Naturalmente passamos a compartilhar a linguagem que falamos de uma maneira que nunca acontece com diagramas e documentos. [Evans, p. 29, 30]

## Uma equipe, uma linguagem

No começo, quando os usuários estão discutindo futuros recursos do sistema que ainda não foram modelados, não há nenhum modelo para que eles possam usar. Mas, assim que eles começarem a trabalhar com essas novas ideias com os desenvolvedores, inicia-se o processo de busca por um modelo compartilhado. É possível que o começo seja estranho e incompleto, mas ele será gradualmente refinado. À medida que a nova linguagem evolui, os especialistas de domínio devem se esforçar ao máximo para adotá-la e ajustar qualquer documento antigo que continue sendo importante. [Evans, p. 30, 31]

Quando os especialistas do domínio utilizam essa LINGUAGEM em discussões com desenvolvedores entre si, eles rapidamente descobrem áreas onde o modelo é inadequado para suas necessidades ou simplesmente parece estar errado. Os especialistas (com a ajuda dos desenvolvedores) também encontrarão área onde a precisão da linguagem baseada no modelo expõe contradições ou a ausência de clareza em seu raciocínio. [Evans, p. 31]

Os desenvolvedores e especialistas do domínio podem informalmente testar o modelo caminhando pelos cenários, usando os objetos do modelo passo a passo. Praticamente todas as discussões são uma oportunidade para que os desenvolvedores e os especialistas possam juntos brincar com o modelo, aprofundando o entendimento de cada um e refinando os conceitos durando o processo. [Evans, p. 31]

Os especialistas podem utilizar a linguagem do modelo escrevendo casos de uso e podem trabalhar ainda mais diretamente com o modelo especificando testes de aceitação. [Evans, p. 31]

O modelo de domínio é normalmente derivado do próprio jargão dos especialistas do domínio, mas passa por uma "limpeza" para ter definições mais rígidas e restritas. [Evans, p. 31]

![A LINGUAGEM ONIPRESENTE é cultivada na interseção dos jargões](https://comment-it.co.uk/wp-content/uploads/2018/01/ubiquitous-language.png)

**Com uma LINGUAGEM ONIPRESENTE, as conversas entre os desenvolvedores, as discussões entre os especialistas do domínio e as expressões no próprio código são todas baseadas na mesma linguagem, derivada de um modelo de domínio compartilhado.** [Evans, p. 32]

## Documentos e diagramas

Diagramas são meios de comunicação e de explicação e facilitam a captação de ideias. Eles servem melhor a esses propósitos se forem sucintos. Diagramas muito cheios de informação para o modelo de objetos falham ao comunicar ou explicar alguma coisa; eles sobrecarregam o leitor com detalhes e falta-lhes significado. [Evans, p. 33, 34]

*O detalhe fundamental sobre o design é captado no código. Uma implementação bem escrita deve ser transparente, revelando o modelo que existe por trás.* [Evans, p. 34]

O modelo não é o diagrama. A finalidade do diagrama é ajudar a comunicar e explicar o modelo. O código pode servir como repositório dos detalhes do design. [Evans, p. 34]

Diagramas cuidadosamente selecionados e construídos podem servir para concentrar a atenção e auxiliar a navegação se não forem obscurecidos por uma compulsão de representar o modelo ou design por completo. [Evans, p. 34]

## Documentos de design escritos

(...) criar documentos escritos que realmente ajudem a equipe a produzir um bom software é um desafio. [Evans, p. 34]

Uma vez que um documento assuma uma forma persistente, ele geralmente perde sua ligação com o fluxo do projeto. E é deixado para trás pela evolução do código, ou pela evolução da linguagem do projeto. [Evans, p. 34]

### Os documentos devem complementar o código e a fala

Um código que funciona não mente, como poderia acontecer com qualquer outro documento. [Evans, p. 34]

O comportamento de um código que funciona não gera dúvidas. [Evans, p. 35]

O código como um documento do design tem seus limites. Ele pode sobrecarregar o leitor com detalhes. Embora seu comportamento não gere dúvidas, isso não significa que eles sejam tão óbvios assim. E o significado por trás de um comportamento pode ser difícil de transmitir. Em outras palavras, documentar exclusivamente por meio de códigos tem alguns dos mesmos problemas básicos encontrados em usar diagramas UML abrangentes. Obviamente, a comunicação falada em massa dentro da equipe fornece o contexto e as diretrizes dentro do código, mais isso é efêmero e localizado. E os desenvolvedores não são as únicas pessoas que precisam entender o modelo. [Evans, p. 35]

*Um documento não deve tentar fazer o que o código já faz bem.* O código já fornece os detalhes. Ele é uma especificação exata do comportamento do programa Outros documentos precisam esclarecer o significado, fazer entender as estruturas em larga escala e concentrar a atenção nos elementos principais. Documentos podem esclarecer a intenção do design quando a linguagem de programação não suporta uma implementação simples de um conceito. Documentos escritos devem complementar o código e a conversa. [Evans, p. 35]

### Os documentos devem trabalhar para sobreviver e manter-se atualizados

O maior valor de um documento do design é explicar os conceitos do modelo, ajudar a navegar pelos detalhes do código e, talvez, proporcionar a compreensão do estilo de uso pretendido pelo modelo. Dependendo da filosia da equipe, o documento do design pode se tão simples quanto um conjunto de esboços colocados nas paredes ou ser substancial. [Evans, p. 36]

*Um documento deve estar envolvido nas atividades do projeto.* A maneira mais fácil de julgar isso é observar a interação do documento com a LINGUAGEM ONIPRESENTE. Será que o documento está escrito na linguagem que as pessoas falam no projeto (agora)? Será que ele está escrito na linguagem embutida no código? Ouça a LINGUAGEM ONIPRESENTE e como ela está mudando. (...) Se os termos explicados em um documento do design não começarem a aparecer nas conversas e no código, o documento não está cumprindo sua finalidade. (...) Se o documento não está tendo nenhum impacto sobre a LINGUAGEM ONIPRESENTE, há algo de errado. [Evans, p. 36]

A LINGUAGEM ONIPRESENTE permite que outros documentos, tais como as especificações dos requisitos, sejam mais concisas e menos ambíguas. À medida que o modelo de domínio passa a refletir o conhecimento mais relevante relativo ao negócio, os requisitos do aplicativo se transformam em cenários dentro daquele modelo, e a LINGUAGEM ONIPRESENTE pode ser usada para descrever este cenário em termos diretamente se conectam ao DESIGN DIRIGIDO POR MODELOS. Em consequência disso, as especificações podem ser escritas de forma mais simples, porque não têm que transmitir o conhecimento do negócio que está por trás do modelo. [Evans, p. 36]

Ao manter os documentos enxutos e concentrá-los na complementação dos códigos e das conversas, eles podem permanecer conectados ao projeto. Deixe que a LINGUAGEM ONIPRESENTE e sua evolução sejam o seu guia para a escolha dos documentos que vão sobreviver e se tecer em meio às atividades do projeto. [Evans, p. 36]

## Base executável

Códigos bem escritos podem ser bastantes comunicativos, mas a mensagem que eles transmitem não é garantidamente precisa. [Evans, p. 37]

É preciso ser meticuloso para escrever um código que não simplesmente **faça** a coisa certa, mas também **diga** a coisa certa. [Evans, p. 37]

Para ter uma comunicação eficaz, o código deve ser baseado na mesma linguagem utilizada para escrever os requisitos - a mesma linguagem que os desenvolvedores falam entre si e com especialistas do domínio. [Evans, p. 37]

## Modelos explanatórios

(...) um modelo deve trazer consigo a implementação, o design e a comunicação da equipe. [Evans, p. 37]

Os modelos também podem ser valiosos como auxílio educacional para ensinar algo sobre o domínio. Um modelo que conduz o design é uma das visões do domínio, mas ele pode ajudar a aprender como ter outras visões, utilizadas somente como ferramentas educacionais, para comunicar o conhecimento geral do domínio. [Evans, p. 37]

O modelo técnico que conduz o processo de desenvolvimento do software deve ser estritamente reduzido ao mínimo necessário para realizar suas funções. O modelo explanatório pode incluir aspectos do domínio que forneçam um contexto que esclarece o modelo mais restrito. [Evans, p. 37, 38]

Modelos explanatórios oferecem a liberdade de criar estilos muito mais comunicativos feitos de acordo com um determinado tópico. Metáforas visuais utilizadas pelos especialistas do domínio em uma área geralmente apresentam explicações mais claras, educando desenvolvedores e harmonizando especialistas. Modelos explanatórios também apresentam o domínio de uma maneira simplesmente diferente; e explicações múltiplas e diversas ajudam as pessoas a aprender. [Evans, p. 38]
