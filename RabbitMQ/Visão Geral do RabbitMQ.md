---
id: 9a675e41-4a17-4052-9639-2f5c25cefbf2
title: Visão Geral do RabbitMQ
summary: "Visão geral dos conceitos básicos do RabbitMQ"
---

<img src="{{ mix('/images/articles/RabbitMQ/Messages sent from a sender to a receiver.png') }}" alt="Messages sent from a sender to a receiver">

> Mensagens ou ***message queuing*** é um estilo de comunicação entre aplicativos ou componentes que permite uma arquitetura fracamente acoplada.

> **RabbitMQ** é um *open source message broker* que atua como intermediário ou middleman para aplicativos independentes, fornecendo-lhes uma plataforma comum de comunicação. É uma implementação baseada em *Erlang* do *AMQP*, que oferece suporte a recursos avançados, como clustering. Para persistência dos dados, *RabbitMQ* usa **Mnesia**, o banco de dados do *Erlang* em memória ou arquivo persistente. É um software de ***message queuing*** também conhecido como ***message broker***. Simplificando, é um software onde as filas podem ser definidas e os aplicativos podem se conectar e enviar ou receber as mensagens. Portanto, quando seu aplicativo se conecta ao *RabbitMQ*, ele precisa tomar uma decisão: estou enviando ou recebendo? Ou, na conversa do AMQP, sou um ***producer*** ou um ***consumer***?

> O **AMQP** (Advanced Message Queuing Protocol) é um *open standard protocol* que define a semântica de um protocolo para que os sistemas troquem mensagens. O protocolo define um conjunto de regras que devem ser seguidas pelos sistemas que irão se comunicar entre si. Além de definir a interação que ocorre entre um *consumer*, *producer* e um *broker*, também define a representação das mensagens e comandos que estão sendo trocados. **AMQP** é verdadeiramente interoperável, pois especifica o formato de transmissão das mensagens, não deixando nada aberto à interpretação por um vendor ou hosting platform. Por ser de código aberto, a comunidade *AMQP* é prolífica e tem implementações de broker e clientes em uma ampla variedade de linguagens.

Veja a seguir à lista de conceitos básicos do ***AMQP*** e do ***RabbitMQ***, que serão vistos nos próximos tópicos:

- **Producer**: Aplicativo que publica/envia as mensagens.

- **Consumer**: Aplicativo que recupera/recebe as mensagens.

- **Message**: Dados enviados do **producer** ao **consumer** por meio do *RabbitMQ*.

- **Broker / Message Broker**: É um aplicativo de middleware que recebe às mensagens produzidas pelos *producers* e entrega aos *consumers* ou a outro *broker*, ou seja, é um parte do software que recebe mensagens de um aplicativo ou serviço e as entrega a outro aplicativo, serviço ou *broker*.

- **Virtual Host, vhost**: É uma divisão virtual em um *broker* que permite a segregação de *producers*, *consumers* e todas as construções *AMQP* das quais eles dependem, geralmente por razões de segurança (como multitenancy). É uma maneira de separar os aplicativos que estão usando a mesma instância do *RabbitMQ*; por exemplo, separar os ambientes de desenvolvimento e testes, mantendo-os dentro do mesmo broker em vez de configurar vários *brokers*. Usuários, exchanges, queues e assim por diante são isolados por *vhost* específico. Um usuário conectado a um determinado *vhost* não pode acessar nenhum recurso (queue, exchange e assim por diante) de outro *vhost*. Os usuários podem ter diferentes privilégios de acesso a diferentes *vhosts*.

- **Connection**: Uma conexão de rede física (TCP) entre *producer*, *consumer* e um *broker*. A conexão só fecha na desconexão do cliente ou no caso de falha na rede ou no *broker*.

- **Channel**: Uma conexão lógica virtual dentro de uma *connection* entre um *producer*, *consumer* e um *broker*. Vários canais podem ser estabelecidos em uma única conexão. Os canais permitem o isolamento da interação entre determinado cliente e o *broker* para que não interfiram um no outro. Isso acontece sem abrir várias conexões TCP individuais. Quando as mensagens são publicadas ou consumidas, isso é feito em um canal. Muitos canais podem ser estabelecidos em uma única conexão.

- **Exchange**: É o destino inicial de todas as mensagens publicadas e a entidade responsável por aplicar as regras de roteamento para que as mensagens cheguem aos seus destinos (*Queue*). Em outras palavras, a *exchange* garante que a mensagem recebida acabe nas filas corretas. A fila em que a mensagem vai parar depende das regras definidas pelo *exchange type*. *Uma fila precisa ser vinculada a pelo menos uma exchange para receber as mensagens*. As regras de roteamento podem ser: *direct* (point-to-point), *topic* (publish-subscribe) e *fanout* (multicast).

- **Queue**: É o destino final das mensagens prontas para serem consumidas. Uma única mensagem pode atingir várias filas se a regra de roteamento da *exchange* for assim. Uma fila é uma sequência de itens; neste caso, mensagens. A fila existe no *broker*. **Cada mensagem é enviada a apenas um *Consumer* inscrito na fila.**

- **Binding**: É uma link virtual entre uma *exchange* e uma fila dentro do *broker* que permite que as mensagens fluam da *exchange* para a fila. Uma *routing key* pode ser associada a um *binding* em relação à regra de roteamento da *exchange*.

- **Routing Key** - A chave que a *exchange* analisa para decidir como rotear a mensagem para as filas. Pense na *routing key* como o endereço de destino de uma mensagem.

- **Users** - É possível se conectar ao *RabbitMQ* com um determinado nome de usuário e senha, com permissões atribuídas, como leitura, gravação e configuração. Os usuários também podem receber permissões para virtual hosts específicos.

- **Acknowledgments and Confirms** - São indicadores de que as mensagens foram recebidas ou manipuladas. Acknowledgements podem ser usados ​​em ambas as direções; por exemplo, um *consumer* pode indicar ao servidor que recebeu/processou uma mensagem e o servidor pode relatar a mesma coisa ao *producer*.

O diagrama a seguir ilustra uma visão geral de alguns dos conceitos do **AMQP**:

<img src="{{ mix('/images/articles/RabbitMQ/Overview of some of the concepts defined by the AMQP specification.png') }}" alt="Overview of some of the concepts defined by the AMQP specification">

<!-- ### The RabbitMQ broker -->

## Comunicação Por Meio de Mensagens?

Os aplicativos têm uma mesma necessidade em comum - *comunicação por meio de trocas de mensagens*; os sistemas precisam se comunicar e enviar mensagens entre si. Às vezes, eles precisam ter certeza de que a mensagem enviada chegou ao seu destino. Às vezes, eles precisam receber uma resposta imediata, mas não o tempo todo. Em alguns casos, eles podem até precisar receber mais de uma resposta. Com base nessas diferentes necessidades, surgiram diferentes estilos de comunicação entre sistemas.

No estilo de interação *one-way*, em que os sistemas interagem uns com os outros de maneira assíncrona por meio da transmissão de mensagens e, geralmente, por meio de partes de retransmissão conhecidas como *message brokers*. Nesse esquema, comumente referido como sistema de mensagens ou *message queuing*, os sistemas desempenham o papel de ***producers*** ou ***consumers*** das mensagens. Eles podem publicar (*producer*) uma mensagem para um *broker* informando para qual ***consumer*** deve ser entrege. Se uma resposta for necessária, ela eventualmente virá por meio do mesmo mecanismo, mas com papeis invertidos (as funções de ***consumer*** e ***producer*** serão trocadas). Um sistema em modo de *comunicação assíncrona* não espera por uma resposta ou requer uma informação de retorno; ele continua processando, não importa o quê. O exemplo mais comum de tal interação é um e-mail. A questão é que a *comunicação assíncrona* não envolve esperar por uma resposta para continuar o processamento. Na verdade, pode não haver resposta ou pode levar algum tempo para que uma resposta seja enviada. Seja qual for o caso, o sistema não depende de uma resposta para dar continuidade ao processo.

As mensagens fluem em uma direção, do **producer** para o **broker** e, finalmente, para o **consumer**:

<img src="{{ mix('/images/articles/RabbitMQ/Basic components of a one-way interaction with message queuing.png') }}" alt="Basic components of a one-way interaction with message queuing">

Uma *mensagem* pode incluir qualquer informação. O *message broker* armazena as mensagens até que um aplicativo se conecte e recupere/consuma. O aplicativo *consumer* então processa a *mensagem* apropriadamente. Um *message producer* adiciona mensagens a fila sem ter que esperar que sejam processadas imediatamente.

### Fluxo de Mensagens no RabbitMQ

1. O **producer** publica/envia uma mensagem para uma **exchange**. *Ao criar uma **exchange**, seu tipo deve ser especificado*.

2. A **exchange** recebe a mensagem e agora é responsável pelo roteamento da mensagem. A **exchange** analisa diferentes atributos e a chave da mensagem, dependendo do tipo de **exchange**.

3. Nesse caso, exite dois **bindings** para duas filas diferentes da **exchange**. A **exchange** roteia a mensagem para a fila correta, dependendo desses atributos.

4. As mensagens permanecem na fila até que o **consumer** as processe.

5. O **consumer** então remove a mensagem da fila depois de manipulada.

<img src="{{ mix('/images/articles/RabbitMQ/Illustration of the message flow in RabbitMQ.png') }}" alt="Illustration of the message flow in RabbitMQ">

### Benefícios do uso de *message queuing* / A loosely coupled architecture

A vantagem da abordagem de *message queuing* é que os sistemas se tornam fracamente acoplados uns aos outros. Eles não precisam saber exatamente onde estão localizados; um mero nome é suficiente para localizá-los. Os sistemas podem, portanto, ser desenvolvidos de maneira independente sem nenhum impacto uns sobre os outros, pois a confiabilidade da entrega da mensagem é confiada ao *broker*.

A comunicação entre vários aplicativos desempenha um papel importante em sistemas distribuídos. Existem muitos exemplos de quando uma fila de mensagens pode ser usada, então vamos destacar alguns recursos e benefícios do *message queuing* em arquiteturas de microsserviço:

- **Desenvolvimento e manutenção mais fáceis**: Dividir uma aplicação em vários serviços permite responsabilidades separadas e dá aos desenvolvedores a liberdade de escrever código para um serviço específico em qualquer linguagem escolhida. Será mais fácil manter o código escrito e fazer alterações no sistema; ao atualizar um único esquema de autenticação, apenas o módulo de autenticação deve ter código adicionado para teste, sem interromper quaisquer outras funções.

- **Isolamento de falha**: Uma falha pode ser isolada em um único módulo e, portanto, não afetará outros serviços. Por exemplo, um aplicativo com um serviço de relatório temporariamente fora do ar não afetará os serviços de autenticação ou pagamento. Como outro exemplo, fazer alterações no serviço de relatórios ainda permite que os clientes realizem transações essenciais, mesmo quando não conseguem visualizar os relatórios.

- **Níveis aprimorados de velocidade e produtividade**: Diferentes desenvolvedores podem trabalhar em diferentes módulos ao mesmo tempo. Além de acelerar o ciclo de desenvolvimento, a fase de teste também é impactada pelo uso de microsserviços e filas de mensagens. Isso ocorre porque cada serviço pode ser testado por conta própria para determinar a prontidão do sistema em geral.

- **Escalabilidade aprimorada**: Os microsserviços também permitem um scale-out sem muito esforço. É possível adicionar mais *consumers* se a fila de mensagens estiver crescendo. Adicionar novos componentes a apenas um serviço é fácil de fazer, sem alterar nenhum outro serviço.

- **Fácil entendimento**: Como cada módulo em uma arquitetura de microsserviço representa uma única funcionalidade, é fácil conhecer os detalhes relevantes para uma tarefa. Por exemplo, a contratação de um consultor para um único serviço não exige que ele entendam todo o sistema.

Resumindo, a arquitetura permite o seguinte:

- Os *producers* ou *consumers* podem ser atualizados um a um, sem que eles impactem uns aos outros.
- Os *producers* ou *consumers* podem falhar sem impactar uns aos outros
- O desempenho de cada lado não afeta o outro lado
- O número de instâncias de *producers* e *consumers* pode aumentar ou diminuir, acomodando/equilibrando a carga de trabalho (workload) com total independência entre os *workers*
- Os *producers* desconhecem a localização e tecnologia dos *consumers* e vice-versa

O diagrama a seguir ilustra o acoplamento fraco entre o *producer* e o *consumer*:

<img src="{{ mix('/images/articles/RabbitMQ/Message queuing enabling a loosely coupled architecture.png') }}" alt="Message queuing enabling a loosely coupled architecture">

## RabbitMQ Workflow

Um ***message broker*** atua como middleman para vários serviços. Um *broker* pode ser usado para reduzir cargas de trabalho. Uma tarefa que geralmente leva muito tempo para ser processada pode ser delegada a um terceiro cuja única função é executá-la.

<img src="{{ mix('/images/articles/RabbitMQ/A sketch of the RabbitMQ workflow.png') }}" alt="A sketch of the RabbitMQ workflow">

Os ***Producers*** criam mensagens e as publicam/enviam para um *broker server* (*RabbitMQ*). Os ***Consumers*** se conectam a um *broker server* e *subscribe* (assinam) para uma fila. Pense em uma fila como uma caixa de correio. Sempre que uma mensagem é publicada em uma determinada caixa de correio, o *RabbitMQ* a envia para um dos *Consumers* que estão *subscribed*. É simples: os *Producers* criam e enviam as mensagens para o *RabbitMQ* e os *Consumers* se conectam também ao RabbitMQ para que possam receber as mensagens.

Antes de consumir ou publicar as mensagens no *Rabbit*, primeiro é necessário se conectar a ele. Ao conectar, estará sendo criado uma conexão TCP entre o aplicativo e o *Rabbit*. Depois que a conexão TCP é aberta (e está autenticado), o aplicativo cria um canal *AMQP*. Este canal é uma conexão virtual dentro da conexão TCP "real" e é sobre o canal que você emite os comandos *AMQP*. Cada canal tem um ID único atribuído a ele (a biblioteca AMQP se encarregará disso). Quando estiver publicando ou recebendo uma mensagem, tudo é feito em um canal.

### Exemplo de um fluxo para criação de PDFs

<img src="{{ mix('/images/articles/RabbitMQ/Application architecture example with RabbitMQ.png') }}" alt="Application architecture example with RabbitMQ">

**1.** O usuário envia uma solicitação de criação de PDF ao aplicativo web.

**2.** O aplicativo web (***producer***) envia/publica a mensagem para o *RabbitMQ* que inclui dados da solicitação, como nome e e-mail.

**3.** Uma ***exchange*** aceita as mensagens de um aplicativo ***producer*** e as roteia/encaminha para as filas corretas de mensagens.

**4.** O processador de PDF (***consumer***) recebe o *job message* da fila e inicia o processamento do PDF.

## The Advanced *Message Queuing* Model

O modelo *AMQP* define logicamente três componentes abstratos no software do *broker* que estabelece o comportamento de roteamento das mensagens:

- ***Exchange*** - O componente do *message broker* que roteia/encaminha as mensagens para filas
- ***Queue*** - Uma estrutura de dados na memória ou no disco que armazena as mensagens
- ***Binding*** - São regras que informam as *exchanges* em qual fila as mensagens devem ser armazenadas

### Exchanges

As mensagens não são publicadas diretamente em uma fila; em vez disso, o *producer* envia uma mensagem para uma **exchange**. O trabalho de uma **exchange** é aceitar mensagens dos aplicativos producer e roteá-las para as filas de mensagens corretas. Ele faz isso com a ajuda dos **bindings** e **routing keys**. Um **binding** vincula a fila a uma **exchange**, enquanto a **routing key** (chave de roteamento) é como um endereço para a mensagem. Isso é o ponto principal que a **exchange** procura ao decidir como rotear a mensagem para as filas.

As **exchanges** são um dos três componentes definidos pelo modelo *AMQP*. São agentes de roteamento de mensagens que vivem em um virtual host (vhost) dentro do *RabbitMQ*. Uma **exchange** recebe mensagens enviadas para o *RabbitMQ* pelo *producer* e determina para qual fila encaminha-lás com base na **routing key**. As **exchanges** definem os comportamentos de roteamento que são aplicados às mensagens. O *RabbitMQ* tem vários tipos de **exchange**, cada um com diferentes comportamentos de roteamento. A imagem abaixo mostra uma visão lógica de um *producer* enviando uma mensagem ao *RabbitMQ*, roteando uma mensagem por meio de uma **exchange**.

<img src="{{ mix('/images/articles/RabbitMQ/When a publisher sends a message into RabbitMQ, it first goes to an exchange.png') }}" alt="When a publisher sends a message into RabbitMQ, it first goes to an exchange">

**Exchanges** podem ser configuradas para incluir propriedades como `durable` e `auto_delete` em sua criação. As *exchanges* com a propriedade `durable` como `true` ​​sobrevivem às reinicializações do servidor e duram até serem excluídas. As *exchanges* com a propriedade `auto_delete` como `true` são excluídas quando o último objeto vinculado é desvinculado da *exchange*, ou seja, quando não tiver mais nenhum *consumer* conectado ao *broker*.

### Queue

Uma fila é responsável por armazenar as mensagens recebidas e pode conter informações de configuração que definem o que ela pode fazer com uma mensagem. Uma fila pode conter mensagens apenas na RAM ou pode persisti-las no disco antes de entregá-las, seguindo o modelo de first-in, first-out (FIFO).

*Ensure that messages are not lost by declaring a queue as durable and setting the message delivery mode to persistent.*

### Bindings

Você deseja que os *consumers* busquem mensagens nas suas filas. Agora a pergunta é como uma mensagem chega a uma fila para que um *consumer* possa recuperar? Sempre que você quiser publicar/entregar/salvar mensagens a uma fila, você o faz enviando-a a uma **exchange**. Com base em certas regras, o *RabbitMQ* decidirá para qual fila deve entregar a mensagem. Essas regras são chamadas de **routing keys**. O que acontece é que uma fila está vinculada a uma **exchange** por uma **routing key**. Quando você envia uma mensagem para o *broker*, ele terá uma **routing key** - mesmo uma em branco - onde o *RabbitMQ* tentará corresponder às **routing key** usadas nos **bindings**. Se eles corresponderem, a mensagem será entregue na fila daquele **binding**. Se a mensagem não corresponder a nenhum dos padrões dos **bindings**, ela será descartada. Resumindo: o ***broker*** irá encaminhar as mensagens recebidas pelo *producer* para as filas com base em sua **exchange** e **routing key**.

No RabbitMQ, **binding** ou *binding keys* informam a uma **exchange** para quais filas entregar as mensagens. É um "link" configurado para fazer uma conexão entre uma fila e uma **exchange**.

Ao publicar uma mensagem em uma **exchange**, os aplicativos usam um atributo **routing key**. Pode ser um nome da fila ou uma string que descreva semanticamente a mensagem. Quando uma mensagem é avaliada por uma **exchange** para determinar as filas apropriadas para as quais ela deve ser roteada, a **routing key** da mensagem é avaliada em relação à **binding key**. Em outras palavras, o **binding** é aquilo que liga uma fila a uma **exchange**, e a **routing key** são os critérios avaliados em relação ao **binding**.

<img src="{{ mix('/images/articles/RabbitMQ/A queue is bound to an exchange, providing the information the exchange needs to route a message to it.png') }}" alt="A queue is bound to an exchange, providing the information the exchange needs to route a message to it">

## Tipos de Exchanges

- **Direct** - Uma *direct exchange* entrega mensagens às filas com base em uma *message routing key*. Em uma *direct exchange*, a mensagem é roteada para a fila com a correspondência exata da *binding key* como a *routing key* da mensagem. Por exemplo, se a fila estiver ligada à *exchange* usando a binding key `pdfprocess`, uma mensagem publicada na *exchange* com uma routing key `pdfprocess` será roteada para essa fila.

- **Topic** - A *topic exchange* executa uma correspondência wildcard entre a *routing key* e o padrão de roteamento especificado no binding.

- **Fanout** - Uma *fanout exchange* roteia mensagens para todas as filas que estão vinculadas a ela.

<img src="{{ mix('/images/articles/RabbitMQ/Three different exchanges - direct, topic, and fanout.png') }}" alt="Three different exchanges - direct, topic, and fanout">

### Direct Exchange

<img src="{{ mix('/images/articles/RabbitMQ/The direct exchange route messages to specific queues.jpg') }}" alt="The direct exchange route messages to specific queues">

A *exchange* **direct** é útil quando você vai entregar uma mensagem com um alvo específico ou um conjunto de alvos. Qualquer fila vinculada a uma *exchange* com a mesma **routing key** usada para publicar uma mensagem receberá essa mesma mensagem. O *RabbitMQ* usa igualdade de strings ao verificar o **binding** e não permite nenhum tipo de padrão de matching ao usar uma exchange **direct**.

Uma **direct** *exchange* entrega mensagens às filas com base em uma **routing key**. A *routing key* é um atributo da mensagem adicionado pelo *producer*. Pense na *routing key* como um "endereço" que a **exchange** usa para decidir como rotear a mensagem. Uma mensagem vai para a(s) fila(s) que têm a correspondência exata no **binding key** com a **routing key** da mensagem. O tipo de **direct exchange** é útil para distinguir mensagens publicadas na mesma *exchange* usando um identificador de string simples.

<img src="{{ mix('/images/articles/RabbitMQ/Using a direct exchange, messages published by publisher 1 will be routed to queue 1 and queue 2, whereas messages published by publisher 2 will be routed to queues 2 and 3.png') }}" alt="Using a direct exchange, messages published by publisher 1 will be routed to queue 1 and queue 2, whereas messages published by publisher 2 will be routed to queues 2 and 3">

Conforme ilustrado na imagem acima, várias filas podem ser vinculadas a uma **direct** *exchange* usando a mesma *routing key*. Cada fila vinculada com a mesma *routing key* receberá todas as mensagens publicadas com essa *routing key*.

#### Exemplo

A `Queue A` (`create_pdf_queue`) na imagem abaixo está vinculada a uma **direct exchange** `pdf_events` com a **binding key** (`pdf_create`). Quando uma nova mensagem com a **routing key** `pdf_create` chega na **direct exchange**, a *exchange* encaminha para a fila onde ***binding_key = routing_key***, neste caso para a `Queue A` (`create_pdf_queue`).

<img src="{{ mix('/images/articles/RabbitMQ/A message is directed to the queue where the binding key is an exact match of the message’s routing key.png') }}" alt="A message is directed to the queue where the binding key is an exact match of the message’s routing key">

**Cenário 1**

- Exchange: *`pdf_events`*
- `Queue A`: *`create_pdf_queue`*
- **Binding key** entre a *exchange* *`pdf_events`* e a `Queue A` (*`create_pdf_queue`*): *`pdf_create`*

**Cenário 2**

- Exchange: *`pdf_events`*
- `Queue B`: *`create_log_queue`*
- **Binding key** entre a *exchange* *`pdf_events`* e `Queue B` (*`pdf_log_queue`*): *`pdf_log`*

Uma mensagem com a ***routing key*** **`pdf_log`** é enviada para a exchange **`pdf_events`**. A mensagem é roteada para *`create_log_queue`* porque a ***routing key*** *`pdf_log`* corresponde à binding key *`pdf_log`*.

Nota: se a ***routing key*** da mensagem não corresponder a nenhuma ***binding key***, a mensagem será descartada.

### Topic Exchange

***Topic exchange*** faz o roteamento para uma fila com base em uma *correspondência wildcard* entre a **routing key** e algo chamado ***routing pattern***, que é especificado pela ***queue binding***. As mensagens podem ser roteadas para uma ou várias filas, dependendo dessa *correspondência wildcard*.

Como as ***direct exchange***, as ***topic exchanges*** rotearão mensagens para qualquer fila vinculada com uma ***routing key*** correspondente. Mas, ao usar um formato delimitado por ponto (`.`), as filas podem se vincular a *routing keys* usando um padrão de correspondência baseada em wildcard. Usando os caracteres asterisco (`*`) e cerquilha (`#`), você pode combinar partes específicas da *routing key* ou até mesmo várias partes ao mesmo tempo. Um asterisco (`*`) corresponderá a todos os caracteres até encontrar um delimitado de ponto na *routing key* e o caractere cerquilha (`#`) indica uma correspondência de zero ou mais palavras, incluindo quaisquer pontos subsequentes.

A imabem abaixo mostra uma *routing key* de ***topic exchange*** com três partes que é usado em novas imagens de perfil do usuário. A primeira parte indica que a mensagem deve ser encaminhada para *consumers* que sabem como agir em mensagens relacionadas à imagem. A segunda parte indica que a mensagem contém uma nova imagem e a terceira contém dados adicionais que podem ser usados ​​para rotear a mensagem para filas de *consumers* que são específicos para a funcionalidade relacionada ao perfil.

<img src="{{ mix('/images/articles/RabbitMQ/A topic exchange routing key with three parts.png') }}" alt="A topic exchange routing key with three parts">

Se tivéssemos que nos basear no processo de upload de imagens, criando uma arquitetura baseada em mensagens para gerenciar todas as tarefas relacionadas a imagens no site, as seguintes chaves de roteamento poderiam descrever algumas das mensagens que seriam publicadas.

- `image.new.profile`: Para mensagens contendo uma nova imagem de perfil
- `image.new.gallery`: Para mensagens contendo uma nova imagem da galeria de fotos
- `image.delete.profile`: Para mensagens com metadados para exclusão de uma imagem de perfil
- `image.delete.gallery`: Para mensagens com metadados para exclusão de uma imagem da galeria
- `image.resize`: Para mensagens que solicitam o redimensionamento de uma imagem

No exemplo anterior de *routing keys*, a importância semântica da *routing key* deve se destacar claramente, descrevendo a intenção ou o conteúdo da mensagem. Usando chaves nomeadas semanticamente para mensagens roteadas por meio da ***topic exchange***, uma única mensagem pode ser roteada por subseções da *routing key*, entregando a mensagem para filas de tarefas específicas.

Na imagem abaixo, a fila para o worker RPC de detecção facial está vinculada a *routing key* `image.new.profile`, comportando-se como se estivesse vinculada a uma *direct exchange*, recebendo apenas novas solicitações da imagem do perfil. A fila para o *consumer* de hashing da imagem está vinculada a `image.new.#` e receberá novas imagens independentemente da origem. Um *consumer* que mantém um diretório de imagens profile pode consumir de uma fila vinculada a `#.profile` e receber todas as mensagens que terminam em `.profile`. As mensagens de exclusão de imagem seriam publicadas em uma fila vinculada a `image.delete.*`, permitindo que um único *consumer* remova todas as imagens enviadas para o site. Por fim, um *consumer* de auditoria vinculado a `image.#` receberia todas as mensagens relacionadas à imagem para que pudesse registrar informações para ajudar na solução de problemas ou na análise comportamental.

<img src="{{ mix('/images/articles/RabbitMQ/Messages are selectively routed to different queues based on the composition of their routing keys.png') }}" alt="Messages are selectively routed to different queues based on the composition of their routing keys">

*Consumers* que aproveitam uma arquitetura como essa podem ser mais fáceis de manter e escalar, em comparação com um aplicativo monolítico que executa as mesmas ações em mensagens entregues a uma única fila. Um aplicativo monolítico aumenta a complexidade operacional e de código.

Usar uma *topic exchange* com *routing keys* com *namespaced* é uma boa escolha para preparar seus aplicativos para o futuro. Mesmo que a correspondência de padrões no roteamento seja um exagero para suas necessidades no início, uma *topic exchange* (usada com as bindings de filas corretas) pode emular o comportamento das *direct* e *fanout* exchanges. Para emular o comportamento da *direct exchange*, vincule as filas com a *routing key* completa em vez de usar padrões de correspondência. O comportamento de fanout exchange é ainda mais fácil de emular, pois as filas vinculadas a `#` como a *routing key*, receberão todas as mensagens publicadas em uma *topic exchange*. Com essa flexibilidade, é fácil ver por que a *topic exchange* pode ser uma ferramenta poderosa em sua arquitetura baseada em mensagens.

**Exemplo**

A *routing key* deve ser uma lista de palavras, delimitada por um ponto (`.`). Os exemplos podem ser *agreements.us* ou *agreements.eu.stockholm*, que neste caso identifica acordos que são estabelecidos para uma empresa com escritórios em diferentes locais. Os *routing patterns* podem conter um asterisco (`*`) para corresponder a uma palavra em uma posição específica da *routing key* (por exemplo, um *routing pattern* de "`agreements.*.*.b.*`" corresponde apenas a *routing keys* onde a primeira palavra é "`agreements`" e a quarta palavra é "`b`"). Um caractere de cerquilha (`#`) indica uma correspondência de zero ou mais palavras (por exemplo, um *routing pattern* de "`agreements.eu.berlin.#`" corresponde a qualquer *routing key* que comece com "`agreements.eu.berlin`").

<img src="{{ mix('/images/articles/RabbitMQ/Messages are routed to one or many queues based on a match between a message routing key and the routing patterns.png') }}" alt="Messages are routed to one or many queues based on a match between a message routing key and the routing patterns">

Os *consumers* indicam em quais tópicos estão interessados. O *consumer* configura um *binding* com um determinado *routing pattern* para a *exchange*. Todas as mensagens com uma *routing key* que correspondem ao *routing pattern* são roteadas para a fila e permanecem lá até que um *consumer* manipule a mensagem.

**Cenário 1**

A imagem acima mostra um exemplo em que o `consumer A` está interessado em todos os acordos em Berlin.

- *Exchange*: *`agreements`*
- `Queue A`: *`berlin_agreements`*
- **Routing pattern** entre a exchange (*`agreements`*) e `Queue A` (*`berlin_agreements`*): *`agreements.eu.berlin.#`*
- Exemplo de *routing key* da mensagem que corresponde: *`agreements.eu.berlin`* e *`agreements.eu.berlin.headstore`*

**Cenário 2**

O `consumer B` está interessado em todos os acordos.

- *Exchange*: *`agreements`*
- `Queue B`: *`all_agreements`*
- **Routing pattern** entre a exchange (*`agreements`*) e `Queue B` (*`all_agreements`*): *`agreements.#`*
- Exemplo de *routing key* da mensagem que corresponde: *`agreements.eu.berlin`* e *`agreements.us`*

**Cenário 3**

O `consumer C` está interessado em todos os acordos para lojas europeias.

- *Exchange*: *`agreements`*
- `Queue C`: *`store_agreements`*
- **Routing pattern** entre a exchange (*`agreements`*) e `Queue C` (*`store_agreements`*): *`agreements.eu.*.store`*
- Exemplo de *routing key* da mensagem que corresponde: *`agreements.eu.berlin.store`* e *`agreements.eu.stockholm.store`*

As mensagens são roteadas para a fila *`berlin_agreements`* por causa do *routing pattern* de *`agreements.eu.berlin.#`* que corresponde a qualquer *routing keys* que comece com *`agreements.eu.berlin`*. A mensagem também é roteada para a fila *`all_agreements`*, uma vez que a *routing keys* (*`agreements.eu.berlin`*) também corresponde ao routing pattern de *`agreements.#`*.

### Fanout Exchange

Todas as mensagens publicadas por meio de uma ***fanout exchange*** são entregues a todas as filas que estão vinculadas na ***fanout exchange***. Isso oferece vantagens de desempenho significativas porque o *RabbitMQ* não precisa avaliar as *routing keys* ao entregar mensagens, mas a falta de seletividade significa que todos os aplicativos que consomem de filas vinculadas a uma ***fanout exchange*** devem ser capazes de consumir mensagens entregues por meio dela.

<img src="{{ mix('/images/articles/RabbitMQ/Adding another consumer that receives the same message as the RPC consumer by using a fanout exchange.png') }}" alt="Adding another consumer that receives the same message as the RPC consumer by using a fanout exchange">

***Fanout exchanges*** roteiam uma mensagem recebida para todas as filas que estão vinculadas a ela, independentemente das *routing keys* ou *pattern matching* como nas exchanges *direct* e *topic*. As chaves fornecidas são ignoradas.

***Fanout exchanges*** podem ser úteis quando a mesma mensagem precisa ser enviada a uma ou mais filas com *consumers* que podem processar a mesma mensagem de diferentes maneiras, como em sistemas distribuídos projetados para transmitir vários estados e atualizações de configuração.

A próxima imagem mostra um exemplo em que uma mensagem recebida pela *exchange* é roteada para todas as três filas vinculadas à *exchange*.

<img src="{{ mix('/images/articles/RabbitMQ/Fanout Exchange - The received message is routed to all queues that are bound to the exchange.png') }}" alt="Fanout Exchange - The received message is routed to all queues that are bound to the exchange">

**Cenário 1**

- *Exchange*: *`sport_news`*
- `Queue A`: *`Mobile client queue A`*
- Binding: Binding entre a exchange (*`sport_news`*) e a `Queue A` (*`Mobile client queue A`*)

Uma mensagem é enviada para a *exchange* *`sport_news`*. A mensagem é roteada para todas as filas (*`Queue A`*, *`Queue B`*, *`Queue C`*) porque todas as filas estão vinculadas à *exchange*. As *routing keys* fornecidas são ignoradas.

### Resumo dos tipos de *Exchanges*

| **Name** | **Plugin** | **Description**                                                                                 |
|----------|------------|-------------------------------------------------------------------------------------------------|
| `Direct` | No         | *Routes to bound queues based upon the value of a **routing key**. Performs **equality matches** only.* |
| `Fanout` | No         | *Routes to **all bound queues** regardless of the routing key presented with a message.*            |
| `Topic`  | No         | *Routes to all bound queues using routing key **pattern matching** and string equality.*            |

## Propriedades da Mensagem

### Usando as propriedades corretamente

As propriedades da mensagem são um conjunto predefinido de valores especificados pela estrutura de dados `Basic.Properties`. Algumas propriedades, como *`delivery-mode`*, têm significados bem definidos na especificação *AMQP*, enquanto outras, como *`type`*, não têm uma especificação exata.

Em alguns casos, o *RabbitMQ* usa as propriedades para implementar comportamentos específicos em relação à mensagem. Um exemplo disso é a propriedade *`delivery-mode`* mencionada anteriormente. O valor de *`delivery-mode`* dirá ao *RabbitMQ* se deverá manter a mensagem na memória quando a mesma é colocada em uma fila ou se deve primeiro armazenar a mensagem no disco.

<img src="{{ mix('/images/articles/RabbitMQ/Basic.Properties - Message Properties.png') }}" alt="Basic.Properties - Message Properties">

As propriedades da mensagem forneceram um ponto de entrada útil para definir os metadados sobre uma mensagem. Esses metadados, por sua vez, permitem ao leitor criar contratos entre *producers* e *consumers*. Resumindo, usando propriedades da mensagem, você pode criar mensagens autoexplicativas.

Neste tópico, veremos cada uma das propriedades básicas descritas na imagem acima:

- Usando a propriedade ***`content-type`*** para permitir que os *consumers* saibam como interpretar o corpo da mensagem.

- Usado ***`content-encoding`*** para indicar que o corpo da mensagem pode ser *compressed* ou codificado de alguma forma.

- Usar ***`message-id`*** e ***`correlation-id`*** para identificar as mensagens e suas respostas de maneira única, rastreando-as por meio de seu workflow.

- Aproveitando a propriedade ***`timestamp`*** para reduzir o tamanho da mensagem e criar uma definição de quando uma mensagem foi criada.

- Expirando as mensagens com a propriedade ***`expiration`***.

- Dizendo ao *RabbitMQ* para gravar suas mensagens da fila em disco ou na memória usando ***`delivery-mode`***.

- Usando ***`app-id`*** e ***`user-id`*** para ajudar a rastrear problemas na publicação das mensagens nos *producers*.

- Usando a propriedade ***`type`*** para definir um contrato entre *producers* e *consumers*.

- Encaminhando mensagens de resposta ao implementar determinado padrão usando a propriedade ***`reply-to`***.

- Usando a propriedade de ***`headers`*** para definições de propriedades personalizadas de chave-valor.

### Contrato de mensagem explícita com *`content-type`*

Quando as mensagens não são autodescritivas sobre seu formato de *payload*, é mais provável que seus aplicativos sejam interrompidos devido ao uso de contratos implícitos, que são inerentemente sujeitos a erros. Ao usar mensagens autoexplicativas, os programadores e aplicativos de *consumers* não precisam adivinhar como desserializar os dados recebidos pelas mensagens ou se a desserialização é realmente necessária.

*A estrutura de dados `Basic.Properties` especifica a propriedade **`content-type`** para transmitir qual o formato dos dados está no corpo da mensagem.*

<img src="{{ mix('/images/articles/RabbitMQ/The content-type property is the first property in Basic.Properties.png') }}" alt="The content-type property is the first property in Basic.Properties">

Especificando o formato de serialização na propriedade **`content-type`**, você pode preparar seus aplicativos de *consumers* para futuras implementações. Quando os *consumers* podem reconhecer automaticamente os formatos de serialização que eles suportam e podem processar mensagens seletivamente, você não precisa se preocupar com o que acontece quando um novo formato de serialização é usado e roteado para as mesmas filas.

Se você estiver usando uma estrutura para seu código de *consumer*, convém torná-lo inteligente sobre como ele lida com as mensagens que recebe. Ao fazer com que a estrutura pré-processe a mensagem antes de entregá-la ao código do *consumer*, os corpos das mensagens podem ser desserializados e carregados automaticamente em estruturas de dados nativas em sua linguagem de programação. Em última análise, isso reduziria a complexidade do seu código no aplicativo de *consumer*.

### Reduzindo o tamanho da mensagem com `*content-encoding*`

As mensagens enviadas por *AMQP* não são compactadas por padrão. Isso pode ser problemático com marcação excessivamente detalhada, com mensagens que usam formatos como JSON ou YAML. Seus *producers* podem compactar as mensagens antes de publicá-las e descompactá-las após o recebimento, da mesma forma como as páginas da web podem ser compactadas no servidor com *gzip* e o navegador pode descompactá-las imediatamente antes da renderização.

*A propriedade **`content-encoding`** indica se codificações especiais foram aplicadas ao corpo da mensagem.*

<img src="{{ mix('/images/articles/RabbitMQ/The content-encoding property indicates whether special encodings have been applied to the message body.png') }}" alt="The content-encoding property indicates whether special encodings have been applied to the message body">

É preferível não alterar o contrato da mensagem que está sendo publicada e consumida, minimizando assim quaisquer efeitos potenciais no código. Mas se o tamanho da mensagem está afetando o desempenho e a estabilidade, usar o cabeçalho de `content-encoding` permitirá que os *consumers* classifiquem as mensagens, garantindo que eles possam decodificar qualquer formato com o qual o corpo da mensagem é enviado.

Não confunda *`content-encoding`* com *`content-type`*. Como na especificação HTTP, o *`content-encoding`* é usado para indicar algum nível de codificação além do *`content-type`*. É um campo modificador frequentemente usado para indicar que o conteúdo do corpo da mensagem foi compactado usando alguma forma de compactação. Alguns clientes *AMQP* definem automaticamente o valor de *`content-encoding`* como UTF-8, mas esse é um comportamento incorreto. A especificação AMQP afirma que a *`content-encoding`* armazenar a codificação de conteúdo **MIME**.

*AMQP* é um protocolo binário. O conteúdo no corpo da mensagem é transferido como está e não é codificado ou transformado no processo de marshaling e remarshaling da mensagem.

Combinada com a propriedade *`content-type`*, a propriedade *`content-encoding`* permite que os aplicativos *consumers* operem em um contrato explícito com os *producers*. Isso permite que o código seja protegido contra erros inesperados causados ​​por alterações no formato da mensagem.

### Referenciando as mensagens com *`message-id`* e *`correlation-id`*

Na especificação *AMQP*, *`message-id`* e *`correlation-id`* são especificados "para uso do aplicativo" e não têm um comportamento definido na especificação. Isso significa que, no que diz respeito às especificações, você pode usá-los para qualquer propósito que desejar. Ambos os campos permitem até 255 bytes de dados codificados em UTF-8 e são armazenados como valores não compactados incorporados na estrutura de dados `Basic.Properties`.

*As propriedades **`message-id`** e **`correlation-id`** podem ser usadas para rastrear mensagens individuais e mensagens de resposta à medida que fluem por seus sistemas.*

<img src="{{ mix('/images/articles/RabbitMQ/The message-id and correlation-id properties can be used to track individual messages and response messages as they flow through your systems.png') }}" alt="The message-id and correlation-id properties can be used to track individual messages and response messages as they flow through your systems">

#### *`message-id`*

Alguns tipos de mensagem, como um evento de login, provavelmente não precisam de um ID da mensagem exclusivo associado a ele, mas é fácil imaginar tipos de mensagens que precisariam, como pedidos de venda ou solicitações do suporte. A propriedade *`message-id`* permite que a mensagem transporte dados que a identifica exclusivamente conforme ela flui pelos vários componentes em um sistema fracamente acoplado.

#### *`correlation-id`*

Embora não haja uma definição formal para *`correlation-id`* na especificação *AMQP*, um uso é indicar que a mensagem é uma resposta a outra mensagem fazendo com que ela carregue o *`message-id`* da mensagem relacionada. Outra opção é usá-lo para transportar um ID de transação ou outros dados semelhantes aos quais a mensagem está se referindo.

### A propriedade *`timestamp`*

Um dos campos mais úteis em `Basic.Properties` é a propriedade *`timestamp`*. Assim como *`message-id`* e *`correlation-id`*, *`timestamp`* é especificado como "para uso do aplicativo". Mesmo que sua mensagem não a use, a propriedade *`timestamp`* é muito útil quando você está tentando diagnosticar qualquer tipo de comportamento inesperado no fluxo de mensagens por meio do *RabbitMQ*. Usando a propriedade **`timestamp`** para indicar quando uma mensagem foi criada, os *consumers* podem medir o desempenho na entrega da mensagem.

<img src="{{ mix('/images/articles/RabbitMQ/The timestamp property can carry an epoch value to specify when the message was created.png') }}" alt="The timestamp property can carry an epoch value to specify when the message was created">

Infelizmente, não há contexto de *time zone* para *`timestamp`*, portanto, é aconselhável usar *UTC* ou outro *time zone* consistente em todas as suas mensagens. Ao padronizar o *time zone* antecipadamente, você evitará quaisquer problemas geográficos que possam resultar de suas mensagens viajando entre *time zones* para o *broker*.

### [Expirando automaticamente as mensagens com *`expiration`*](https://www.rabbitmq.com/ttl.html)

A propriedade **`expiration`** diz ao *RabbitMQ* quando ele deve descartar uma mensagem se ela não tiver sido consumida. Se especificada, a propriedade **`expiration`** instruirá o *RabbitMQ* a descartar uma mensagem se a hora atual for maior do que o valor especificado.

<img src="{{ mix('/images/articles/RabbitMQ/To use the expiration property in RabbitMQ, set the string value to a Unix epoch timestamp designating the maximum value for which the message is still valid.png') }}" alt="To use the expiration property in RabbitMQ, set the string value to a Unix epoch timestamp designating the maximum value for which the message is still valid">

Para usar a propriedade *`expiration`*, defina uma valor de timestamp designando o valor máximo para o qual a mensagem ainda é válida.

Ao usar a propriedade *`expiration`*, se uma mensagem for publicada no servidor com um timestamp de expiração que já passou, a mensagem não será roteada para nenhuma fila, mas será descartada.

### Equilibrando a velocidade com a segurança usando *`delivery-mode`*

A propriedade **`delivery-mode`** é um campo que indica ao *message broker* que você gostaria de persistir a mensagem no disco antes de ser entregue a qualquer *consumer* em espera. No *RabbitMQ*, persistir uma mensagem significa que ela permanecerá na fila até que seja consumida, mesmo se o servidor *RabbitMQ* for reiniciado. A propriedade **`delivery-mode`** tem dois valores possíveis: **`1`** para uma mensagem não persistente e **`2`** para uma mensagem persistente.

<img src="{{ mix('/images/articles/RabbitMQ/The delivery-mode property instructs RabbitMQ whether it must store the message on disk when placing it in a queue or if it may keep the message only in memory.png') }}" alt="The delivery-mode property instructs RabbitMQ whether it must store the message on disk when placing it in a queue or if it may keep the message only in memory">

A propriedade *`delivery-mode`* instrui ao *RabbitMQ* se ele deve armazenar a mensagem no disco ao colocá-la em uma fila ou se pode manter a mensagem apenas na memória. Se definido como "**`2`**", a propriedade *`delivery-mode`* diz ao *RabbitMQ* para persistir a mensagem no disco.

Conforme ilustrado na imagem abaixo, especificar uma mensagem como não persistente permitirá que o *RabbitMQ* use filas apenas na memória.

<img src="{{ mix('/images/articles/RabbitMQ/Publishing messages to memory-only queues.png') }}" alt="Publishing messages to memory-only queues">

Como o *IO* da memória é inerentemente mais rápido do que o *IO* de disco, especificar o modo de entrega como "**`1`**" entregará suas mensagens com o mínimo de latência possível. Conforme ilustrado na imagem abaixo, ao especificar um modo de entrega de "**`2`**", as mensagens são mantidas na fila e salvas no disco.

<img src="{{ mix('/images/articles/RabbitMQ/Publishing messages to disk-backed queues.png') }}" alt="Publishing messages to disk-backed queues">

### Validando a origem da mensagem com *`app-id`* e *`user-id`*

As propriedades *`app-id`* e *`user-id`* fornecem outro nível de informação sobre uma mensagem e têm muitos usos. Como com outras propriedades que podem ser usadas para especificar um contrato comportamental na mensagem, essas duas propriedades podem transportar informações que os aplicativos *consumers* podem validar antes do processamento.

<img src="{{ mix('/images/articles/RabbitMQ/The user-id and app-id properties are the last of the Basic .Properties values, and they can be used to identify the message source.png') }}" alt="The user-id and app-id properties are the last of the Basic .Properties values, and they can be used to identify the message source">

A propriedade *`app-id`* é um valor de string de formato livre que você pode usar para especificar um identificador do aplicativo de publicação.

#### *`app-id`*

A propriedade *`app-id`* é definida na especificação *AMQP* como uma "string curta", permitindo até 255 caracteres UTF-8. Se seu aplicativo tiver um design centrado em API com controle de versão, você pode usar o *`app-id`* para transmitir a API e a versão que foram usadas para gerar a mensagem. Como um método para fazer cumprir um contrato entre o *producer* e o *consumer*, examinar o *`app-id`* antes do processamento permite que o aplicativo descarte a mensagem se for de uma fonte desconhecida ou sem suporte.

Outro uso possível para *`app-id`* é na coleta de dados estatísticos. Por exemplo, se você estiver usando mensagens para transmitir eventos de login, poderá definir a propriedade *`app-id`* para a plataforma e versão do aplicativo que aciona o evento de login. Em um ambiente onde você pode ter aplicativos baseados na web, desktop e mobile, esta seria uma ótima maneira de fazer cumprir um contrato de forma transparente e extrair dados para manter o controle de logins por plataforma, sem nunca inspecionar o corpo da mensagem. Isso é especialmente útil se você deseja ter *consumers* com um único propósito, permitindo que um *consumer* que coleta estatísticas ouça as mesmas mensagens que seu *consumer* de processamento de login. Ao fornecer a propriedade *`app-id`*, o *consumer* de coleta de estatísticas não teria que desserializar ou decodificar o corpo da mensagem.

#### *`user-id`*

No caso de uso de autenticação de usuário, pode parecer óbvio usar a propriedade *`user-id`* para identificar o usuário que efetuou login, mas na maioria dos casos isso não é aconselhável. O *RabbitMQ* verifica cada mensagem publicada com um valor na propriedade *`user-id`* em relação ao usuário *RabbitMQ* que publica a mensagem e, se os dois valores não corresponderem, a mensagem é rejeitada. Por exemplo, se seu aplicativo estiver se autenticando com *RabbitMQ* como o usuário "*www*" e a propriedade *`user-id`* estiver definida como "*xyz*", a mensagem será rejeitada.

Claro, se o seu aplicativo for algo como uma sala de chat ou serviço de mensagens instantâneas, você pode muito bem querer um usuário no *RabbitMQ* para cada usuário do seu aplicativo, e você realmente gostaria de usar o ID do usuário para identificar o usuário real que está fazendo login no seu aplicativo.

### Ser específico com a propriedade *`type`* da mensagem

A versão `0-9-1` da especificação *AMQP* define a propriedade *`type`* de `Basic.Properties` como o "nome do tipo da mensagem", dizendo que é para uso do aplicativo e não tem comportamento formal.

<img src="{{ mix('/images/articles/RabbitMQ/The type property is a free-form string value that can be used to define the message type.png') }}" alt="The type property is a free-form string value that can be used to define the message type">

A propriedade **`type`** pode ser usada para descrever o conteúdo da mensagem.

A propriedade **`type`** é um valor de string de formato livre que pode ser usado para definir o tipo de mensagem.

### Usando *`reply-to`* para fluxos de trabalho dinâmicos

A propriedade *`reply-to`* pode ser usada para transportar a *routing key* que um *consumer* deve usar ao responder a uma mensagem que implementa um padrão RPC.

**`reply-to`** não tem uma definição formal, mas pode transportar uma *routing key* ou valor do nome de fila que pode ser usado para respostas à mensagem.

<img src="{{ mix('/images/articles/RabbitMQ/The reply-to property has no formal definition but can carry a routing key or queue name value that can be used for replies to the message.png') }}" alt="The reply-to property has no formal definition but can carry a routing key or queue name value that can be used for replies to the message">

### Propriedades personalizadas usando a propriedade *`headers`*

A propriedade **`headers`** é uma tabela de chave-valor que permite chaves e valores arbitrários definidos pelo usuário (imagem abaixo). As chaves podem ser strings *ASCII* ou *Unicode* com comprimento máximo de 255 caracteres. Os valores podem ser qualquer tipo de valor *AMQP* válido.

Ao contrário das outras propriedades, a propriedade **`headers`** permite que você adicione quaisquer dados que desejar à tabela de cabeçalhos.

<img src="{{ mix('/images/articles/RabbitMQ/The headers property allows for arbitrary key:value pairs in the message properties.png') }}" alt="The headers property allows for arbitrary key/value pairs in the message properties">

### Resumo

Usando `Basic.Properties` corretamente, sua arquitetura de mensagens pode criar contratos comportamentais rígidos entre *producers* e *consumers*. Além disso, você poderá preparar suas mensagens para futuros projetos de integração que talvez não tenha considerado em seu aplicativo inicial e nas especificações de mensagem.

A tabela abaixo fornece uma visão geral rápida dessas propriedades. Você pode voltar e fazer referência a ele enquanto descobre o uso apropriado das propriedades em seus aplicativos.

| **Property**        | **Type**       | **For use by** | **Suggested or specified use**                                                                                                                                      |
|---------------------|----------------|----------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `app-id`            | *short-string* | Application    | Useful for defining the application publishing the messages.                                                                                                        |
| `content-encoding`  | *short-string* | Application    | Specify whether your message body is encoded in some special way, such as zlib, deflate, or Base64.                                                                 |
| `content-type`      | *short-string* | Application    | Specify the type of the message body using mime-types.                                                                                                              |
| *`correlation-id`*  | *short-string* | Application    | If the message is in reference to some other message or uniquely identifiable item, the `correlation-id` is a good way to indicate what the message is referencing. |
| `delivery-mode`     | *octet*        | RabbitMQ       | A value of `1` tells RabbitMQ it can keep the message in memory; `2` indicates it should also write it to disk.                                                     |
| `expiration`        | *short-string* | RabbitMQ       | An epoch or Unix timestamp value as a text string that indicates when the message should expire.                                                                    |
| `headers`           | *table*        | Both           | A free-form key/value table that you can use to add additional metadata about your message; RabbitMQ can route based upon this if desired.                          |
| `message-id`        | *short-string* | Application    | A unique identifier such as a *UUID* that your application can use to identify the message.                                                                           |
| `timestamp`         | *timestamp*    | Application    | An epoch or Unix timestamp value that can be used to indicate when the message was created.                                                                         |
| `type`              | *short-string* | Application    | A text string your application can use to describe the message type or payload.                                                                                     |
| `user-id`           | *short-string* | Both           | A free-form string that, if used, *RabbitMQ* will validate against the connected user and drop messages if they don’t match.                                          |

Além de usar propriedades para mensagens autoexplicativas, essas propriedades podem transportar metadados valiosos sobre sua mensagem que permitirão a você criar mecanismos sofisticados de roteamento e transação, sem ter que poluir o corpo da mensagem com informações contextuais pertencentes à mensagem. Ao avaliar a mensagem para entrega, o *RabbitMQ* aproveitará propriedades específicas, como o modo de entrega e a tabela de header, para garantir que suas mensagens sejam entregues como e onde você especificar.

## Configurando Instâncias do *RabbitMQ*

### Dica: Separe projetos e ambientes usando *vhosts*

Assim como é possível criar diferentes bancos de dados em um servidor *MySQL* para diferentes projetos, *vhost* também torna possível separar os aplicativos em um único *broker*. Isolar usuários, exchanges, filas, etc. para um *vhost* específico ou ambientes separados, por exemplo, produção, para um *vhost* e testes para outro *vhost* dentro do mesmo *broker* em vez de configurar vários brokers. A desvantagem de usar uma única instância é que não há isolamento de recursos entre *vhosts*.

### Vários tenants: virtual hosts e separação

Os *vhosts* são para o *RabbitMQ* o que as máquinas virtuais são para os servidores físicos: eles permitem que você execute dados para vários aplicativos com segurança, fornecendo separação lógica entre as instâncias. Isso é útil para qualquer coisa, desde separar vários clientes no mesmo *RabbitMQ* até evitar colisões de nomes em filas e exchanges. Caso contrário, você pode ter que executar vários *Rabbits* e ter as dores de cabeça de gerenciamento que vêm com isso, você pode executar um *RabbitMQ* e construir ou destruir *vhosts* sob demanda.

Vhosts são tão fundamentais para o conceito de *AMQP* que você deve especificar um ao se conectar. O *RabbitMQ* facilita o início, incluindo um vhost padrão pronto para uso chamado **`/`**. Se você não precisa de vários *vhosts*, use apenas o padrão. Um item interessante do *AMQP* é que ele não especifica se as permissões são por *vhost* ou por todo o *broker*. Isso é deixado para o *broker* e, no caso do *RabbitMQ*, as permissões são por *vhost*.

Quando você cria um usuário no *RabbitMQ*, ele geralmente é atribuído a pelo menos um *vhost* e só poderá acessar filas, exchanges e bindings nesses *vhosts* atribuídos. Além disso, ao projetar sua arquitetura de mensagens, tenha em mente que a separação entre *vhosts* é absoluta. Você não pode vincular uma exchange no *vhost* `banana_tree` a uma fila no *vhost* `oak_tree`. Isso é realmente uma coisa boa, não apenas para segurança, mas também para portabilidade. Mas, uma vez que tem você seu próprio *vhost*, você pode mover tudo com segurança para qualquer outro servidor *RabbitMQ* e começar imediatamente a lidar com a nova carga, sem nenhuma colisão de nomes. Portanto, é altamente recomendável identificar os grupos de funcionalidade comuns em sua infraestrutura e dar a cada um seu próprio *vhost*. Além disso, lembre-se de que quando você cria um *vhost* em um cluster *RabbitMQ*, ele é criado em todo o cluster. Assim como os *vhosts* eliminam a necessidade de executar um servidor *RabbitMQ* para cada camada em sua infraestrutura, eles também evitam que você crie clusters diferentes para cada camada.

Já falamos sobre todos os grandes benefícios dos *vhosts*, mas como você os cria? Para RabbitMQ, eles são criados usando o utilitário `rabbitmqctl`. Para criar um *vhost*, basta executar `rabbitmqctl add_vhost [vhost_name]`, onde `[vhost_name]` é o *vhost* que você deseja criar. Excluir um vhost é similarmente simples: `rabbitmqctl delete_vhost [vhost_name]`. Depois de criar um vhost, você pode se conectar a ele e começar a adicionar suas filas e exchanges. Se você precisa descobrir quais vhosts estão sendo executados em um servidor Rabbit, execute `rabbitmqctl list_vhosts`

### Durabilidade das mensagens

Há um segredo sobre a criação de filas e exchanges no *RabbitMQ*: por padrão, eles não sobrevivem à reinicialização. Isso mesmo; reinicie seu servidor *RabbitMQ* e observe se essas filas e exchanges desaparecerem (junto com as mensagens internas). O motivo é por causa de uma propriedade em cada fila e exchange chamada **`durable`**. O padrão é **`false`** e informa ao *RabbitMQ* se a fila (ou exchange) deve ser recriada após uma falha ou reinicialização do *RabbitMQ*. Defina como ***`true`*** e você não terá que recriar essas filas e exchanges quando o servidor for reiniciado. Você também pode pensar que a configuração **`durable`** para `true` nas exchanges e filas é tudo o que você precisa fazer para que suas mensagens não sejam perdidas em uma reinicialização, mas você está errado e isso não é suficiente por si só.

Uma mensagem que pode sobreviver a um travamento do *broker* é chamada de ***persistente***. Você sinaliza uma mensagem como *persistente* definindo a opção de modo de entrega da mensagem como ***`2`*** antes de publicá-la. Neste ponto, a mensagem é indicada como *persistente*, mas deve ser publicada em uma exchange que é **`durable`** e chega em uma fila **`durable`** para sobreviver a reinicialização do servidor. Então, para que uma mensagem que está fluindo dentro do *RabbitMQ* possa sobreviver a um incidente, a mensagem deve:

- *Ter sua opção de modo de entrega definida como **`2`** (persistente)*
- *Ser publicado em uma exchange **`durable`***
- *Chegue em uma fila **`durable`***

A maneira que o *RabbitMQ* garante que as mensagens persistentes sobrevivam a uma reinicialização é gravando-as no disco dentro de um arquivo de log de persistência. Quando você publica uma mensagem persistente em uma exchange **`durable`**, o *RabbitMQ* não enviará a resposta até que a mensagem seja gravada no arquivo de log. Quando você usa mensagens persistentes, é crucial certificar-se de que todos os três elementos necessários estejam configurados corretamente. Depois de consumir uma mensagem persistente de uma fila durável (e acknowledge), o *RabbitMQ* a sinaliza no log de persistência para que possa removê-la. Se o *RabbitMQ* reiniciar a qualquer momento antes de você consumir uma mensagem persistente, ele irá recriar automaticamente as exchanges e filas (e bindings) e reproduzir todas as mensagens do log de persistência nas filas ou exchanges apropriadas.

Você pode estar pensando que deve usar mensagens persistentes para todas as suas mensagens. Você poderia fazer isso, mas pagaria um preço para garantir que suas mensagens sobrevivam aos reinícios do *RabbitMQ*: desempenho. O ato de gravar mensagens no disco é muito mais lento do que apenas armazená-las na RAM e diminuirá significativamente o número de mensagens por segundo que seu servidor *RabbitMQ* pode processar. Não é incomum ver uma redução de 10 vezes ou mais na taxa de transferência de mensagens ao usar a persistência.

Quando você deve usar mensagens persistentes/duráveis? Primeiro, você precisa analisar (e testar) suas necessidades de desempenho. Você precisa processar *100.000* mensagens por segundo em um único servidor *RabbitMQ*? Se assim for, você provavelmente deve olhar para outras formas de garantir a entrega de mensagens. Se a mensagem persistente atender às suas necessidades de desempenho, é uma excelente maneira de ajudar a garantir a entrega.


## Manipulando as Filas

O que acontece se você tentar declarar uma fila que já existe? Contanto que os parâmetros de declaração correspondam exatamente à fila existente, o *RabbitMQ* não fará nada e retornará com sucesso como se a fila tivesse sido criada (se os parâmetros não corresponderem, a tentativa de declaração falhará). Se você deseja apenas verificar se a fila existe, você pode definir a opção `passive` de `queue.declare` como `true`. Com `passive` definido como `true`, `queue.declare` retornará com êxito se a fila existir e retornará um erro se ela não existir.

É importante perceber que, de acordo com a especificação *AMQP*, as configurações de uma fila são imutáveis. Depois de declarar uma fila, você não pode alterar nenhuma das configurações usadas em sua criação. Para alterar as configurações da fila, você deve excluir a fila e recriá-la.

### Filas temporárias

#### Excluíndo filas automaticamente

o RabbitMQ fornece filas que serão excluídas assim que forem usadas e não forem mais necessárias. Criar uma fila de exclusão automática é tão fácil quanto definir o sinalizador `auto_delete` como `true` na solicitação RPC `queue.declare`.

A fila é excluída automaticamente quando o último *consumer* cancela a assinatura. Se você precisar de uma fila temporária usada apenas por um *consumer*, combine `auto_delete` com a configuração `exclusive`. Quando o *consumer* se desconectar, a fila será removida.

#### Permitindo apenas um único *consumer*

Sem a configuração `exclusive` habilitada em uma fila, o *RabbitMQ* permite um comportamento de *consumers* ilimitados. Ele não define restrições quanto ao número de *consumers* que podem se conectar a uma fila e consumir dela. Na verdade, ele incentiva vários *consumers* ao implementar um comportamento de entrega `round-robin` para todos os *consumers* que podem receber mensagens da fila.

Existem certos cenários, em que você deseja garantir que apenas um único *consumer* consiga consumir as mensagens em uma fila. Habilitar o uso exclusivo de uma fila envolve passar um argumento durante a criação da fila e, com o argumento `auto_delete`, habilitar filas `exclusive` remove automaticamente a fila assim que o *consumer* se desconecta.

Uma fila declarada como `exclusive` só pode ser consumida pela mesma conexão e canal em que foi declarada, ao contrário das filas que são declaradas com `auto_delete`, que pode ter qualquer número de *consumers* de qualquer número de conexões. Uma fila `exclusive` também será excluída automaticamente quando o canal em que a fila foi criada for fechado, o que é semelhante a uma fila com `auto_delete` que será removida quando não houver mais *consumers* inscritos nela. Ao contrário de uma fila `auto_delete`, você pode consumir e cancelar o *consumer* de uma fila `exclusive` quantas vezes quiser, até que o canal seja fechado.

#### Expiração automática

o *RabbitMQ* permite um argumento ao declarar uma fila para excluir a mesma se ela não for usada por algum tempo.

Criar uma fila que expira automaticamente é tão simples quanto declarar uma fila com um argumento *`x-expires`* com o tempo de vida da fila (*TTL*) especificado em milissegundos.

Existem algumas regras sobre filas que se expiram automaticamente:

- A fila só irá expirar se não tiver *consumers*. Se você tiver uma fila com *consumers* conectados, ela só será removida automaticamente quando eles emitirem `Basic.Cancel` ou desconectar.
- Como em qualquer outra fila, as configurações e argumentos declarados com um argumento `x-expires` não podem ser redeclarados novamente ou alterados. Se você pudesse redeclarar a fila, estendendo a expiração pelo valor do argumento `x-expires`, estaria violando uma regra fixa na especificação AMQP de que um cliente não deve tentar redeclarar uma fila com configurações diferentes.
- O *RabbitMQ* não oferece garantias sobre a rapidez com que removerá a fila pós-expiração.

### Filas permanentes

#### Durabilidade da fila

Ao declarar uma fila que deve persistir nas reinicializações do servidor, a flag `durable` deve ser definido como `true`. Frequentemente, a durabilidade da fila é confundida com a persistência da mensagem. As mensagens são armazenadas no disco quando uma mensagem é publicada com a propriedade `delivery-mode` definida como `2`. A flag de `durable`, em contraste, instrui o *RabbitMQ* que você deseja que a fila seja persistida até que uma solicitação *Queue.Delete* seja chamada.

Enquanto os aplicativos de estilo RPC geralmente desejam filas que vêm e vão com os *consumers*, as filas duráveis ​​são muito úteis para fluxos de trabalho de aplicativos em que vários *consumers* se conectam à mesma fila e o roteamento e o fluxo de mensagens não mudam dinamicamente.

#### Auto-expiração de mensagens em uma fila

As configurações de *TTL* por mensagem permitem restrições de tempo máximo de uma mensagem. As filas declaradas com *Dead Letter Exchanges* e um valor *TTL* resultarão em mensagens mortas na fila no momento da expiração.

Em contraste com a propriedade de `expiration` de uma mensagem, que pode variar de mensagem para mensagem, a configuração da fila `x-message-ttl` impõe um tempo máximo para todas as mensagens na fila.

O uso de *TTLs* por mensagem com filas fornece valor inerente para mensagens que podem ter valores diferentes para *consumers* diferentes.

## Acknowledging Messages

Pode definir o parâmetro `auto_ack` como `true` quando se inscrever em uma fila. Quando `auto_ack` é especificado, *RabbitMQ* considerará automaticamente a mensagem reconhecida/confirmada pelo *consumer* assim que o *consumer* a receber. Uma coisa importante a lembrar é que as confirmações de mensagem do *consumer* não têm nada a ver com informar ao *producer* que a mensagem que foi recebida. Em vez disso, os reconhecimentos são uma forma de o *consumer* confirmar ao *RabbitMQ* que o *consumer* recebeu a mensagem corretamente e o *RabbitMQ* pode removê-la da fila com segurança.

Existem duas maneiras possíveis de confirmar a entrega da mensagem - quando um *consumer* recebe a mensagem (uma confirmação automática, `auto_ack`) e quando um *consumer* envia de volta uma confirmação (confirmação explícita / manual). Com a confirmação automática, a mensagem é confirmada assim que sai da fila (e, portanto, é removida da fila). É melhor fazer a confirmação automática quando a alta velocidade das mensagens for necessária, se as conexões forem confiáveis e se as mensagens perdidas não forem um problema. Use a confirmação manual se houver o risco de falha no processamento de uma mensagem e o *broker* precisa entregá-la novamente. O reenvio de mensagens não confirmadas não acontece imediatamente, a menos que a mensagem seja rejeitada ou o canal seja fechado.

Se um *consumer* receber uma mensagem e depois se desconectar do Rabbit (ou cancelar a assinatura da fila) antes de reconhecer, o *RabbitMQ* irá considerar a mensagem não entregue e reenviá-la para o próximo *consumer* inscrito. Se o seu aplicativo travar, você pode ter certeza de que a mensagem será enviada a outro *consumer* para processamento. Por outro lado, se seu aplicativo de *consumer* tiver um bug e esquecer de reconhecer uma mensagem, o Rabbit não enviará mais mensagens ao *consumer*. Isso ocorre porque o Rabbit considera o *consumer* não pronto para receber outra mensagem até que reconheça a última que recebeu. Você pode usar esse comportamento a seu favor. Se o processamento do conteúdo de uma mensagem for particularmente intenso, seu aplicativo pode atrasar o reconhecimento da mensagem até que o processamento seja concluído.

E se você quiser rejeitar especificamente uma mensagem, em vez de reconhecê-la depois de recebê-la? Por exemplo, digamos que, ao processar a mensagem, você encontre um erro incorrigível, mas isso afeta apenas esse *consumer* devido a um problema de hardware (esse é um bom motivo para nunca confirmar uma mensagem até que ela seja processada). Contanto que a mensagem ainda não tenha sido confirmada, você tem duas opções:

**1.** Faça com que seu *consumer* se desconecte do servidor *RabbitMQ*. Isso fará com que o *RabbitMQ* coloque a mensagem na fila automaticamente e a entregue a outro *consumer*. A vantagem desse método é que ele funciona em todas as versões do *RabbitMQ*. A desvantagem é a carga extra colocada no servidor *RabbitMQ* da conexão e desconexão do seu *consumer* (uma carga potencialmente significativa se o seu *consumer* estiver encontrando erros em todas as mensagens).

**2.** Use o comando `basic.reject` do AMQP. `basic.reject` faz exatamente o que parece: permite que seu *consumer* rejeite uma mensagem que o *RabbitMQ* lhe enviou. Se você definir o parâmetro `requeue` do comando rejeitar como `true`, o *RabbitMQ* reenviará a mensagem para o próximo *consumer* inscrito. Configurando `requeue` na fila para `false` fará com que o *RabbitMQ* remova a mensagem da fila imediatamente sem reenviá-la para um novo *consumer*. Mais, por que você desejaria que o parâmetro `requeue` seja definido como `false` em vez de confirmar a mensagem? Para que essas mensagens sejam salvas em uma fila especial chamada [Dead Letter Exchanges](https://www.rabbitmq.com/dlx.html), onde as mensagens rejeitadas com `requeue` `false` serão colocadas. Uma fila *DLX* permite que você inspecione mensagens rejeitadas/não entregues em busca de problemas. Você também pode descartar uma mensagem simplesmente reconhecendo-a (este método de descarte tem a vantagem de funcionar com todas as versões do *RabbitMQ*). Isso é útil se você detectar uma mensagem com formato inválido que você sabe que nenhum de seus *consumers* será capaz de processar.
