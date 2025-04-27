---
id: 9c456d18-413f-4307-87d7-87bbbb175486
title: Introdução ao Kubernetes
summary: "Introdução a arquitetura, Pods e principais conceitos do K8s"
---

> *Kubernetes (K8s) é um framework de orquestração que implanta e gerencia aplicativos em contêineres. Ele implanta os aplicativos e responde dinamicamente às mudanças.*

> *O Kubernetes permite que você execute seus aplicativos em milhares de nós como se todos esses nós fossem um único e enorme server. Ele abstrai a infra-estrutura subjacente e, ao fazer isso, simplifica o desenvolvimento, a implantação e o gerenciamento para as equipes de desenvolvimento e de operações*.

O ***Kubernetes*** pode:

- *Implantar seu aplicativo para atender às demandas do negócios (**auto-scaling**, **self-healing**, **rolling updates** etc.).*
- *Escalar para cima ou para baixo dinamicamente de acordo com a demanda.*
- *Self-heal quando as coisas quebram.*
- ***Realizar atualizações e reversões contínuas com zero tempo de inatividade.***

E a melhor parte do *Kubernetes*, ele pode fazer tudo isso sem você ter que supervisionar ou se envolver nas decisões. Obviamente, primeiro você precisa configurar as coisas, mas depois de fazer isso, você pode deixar o *Kubernetes* fazer sua mágica.

## O Que é Um Aplicativo de Microservices?

Um aplicativo de **microservices** é um aplicativo de negócio criado a partir de várias *pequenas partes especializadas* que se comunicam e formam um aplicativo com significado de negócio bem definido. Por exemplo, você pode ter um aplicativo de *e-commerce* que compreende todos os seguintes pequenos componentes especializados:

- *Front-end na web*
- *Serviço de catálogo*
- *Carrinho de compras*
- *Serviço de autenticação*
- *Serviço de logging*
- e mais…

Cada um desses serviços individuais é chamado de **microservice**. Normalmente, cada um pode ser codificado e controlado por diferentes equipes, e cada um pode ter sua própria peridiocidade de atualização e pode ser escalonado independentemente de todos os outros. Por exemplo, você pode corrigir e dimensionar o microservice de *catálogo* sem afetar nenhum dos outros componentes do aplicativo.

*Com tudo isso em mente, o Kubernetes **implanta** e **gerencia** (**orquestra**) os aplicativos que são executados como contêineres (**containerized**) e que são construídos de maneiras (**microservices**) que permitem que eles sejam escalonados, self-heal e atualizados em linha com os requisitos do negócio.*

## Sobre Orquestração de Contêineres

### O que é orquestração de contêineres?

A **orquestração de contêineres** é um padrão para a execução de aplicativos na cloud e em data center. Usando **contêineres** - *unidades de aplicativo pré-configuradas com dependências agrupadas* - como base, os desenvolvedores podem executar muitas instâncias de um aplicativo em paralelo.

### Benefícios

A **orquestração de contêineres** oferece alguns benefícios, mas destacaremos os principais. Primeiro, *permite que os desenvolvedores criem aplicativos de alta disponibilidade com facilidade*. Por ter várias instâncias de um aplicativo em execução, um sistema de orquestração de contêineres pode ser configurado para substituir automaticamente quaisquer instâncias com falha por novas.

Isso pode ser estendido para a cloud, tendo essas várias instâncias do aplicativo espalhadas por data centers físicos/zonas de disponibilidade, portanto, *se um data center ficar inativo, outras instâncias do aplicativo permanecerão ativas e evitarão tempo de inatividade, garantindo assim, alta disponibilidade*.

Em segundo lugar, *a orquestração de contêineres permite aplicativos altamente escalonáveis*. Como novas instâncias do aplicativo podem ser criadas e destruídas facilmente(**ephemeral containers**), *a ferramenta de orquestração pode ser escalonada para cima ou para baixo automaticamente para atender à demanda*. Em um ambiente cloud, novas máquinas virtuais (*VMs*) ou máquinas físicas podem ser adicionadas à ferramenta de orquestração para fornecer um pool maior de computação. Esse processo pode ser totalmente automatizado para permitir o dimensionamento tanto no nível micro quanto no nível macro.

### Kubernetes como um cluster

O **Kubernetes** é como qualquer outro *cluster* - *um monte de nós e um plano de controle*. O plano de controle expõe uma API, tem um agendador/planejador para atribuir trabalho aos nós e o estado é registrado em um armazenamento persistente. *Os nós são onde os serviços do aplicativo são executados*.

*Pode ser útil pensar no plano de controle como o cérebro e nos nós como o músculo*. Nessa analogia, o **plano de controle** é o *cérebro* porque implementa todos os recursos importantes, como **auto-scaling** (escalonamento automático) e **atualizações contínuas** (rolling updates) com **zero-downtime** (tempo de inatividade zero). Os nós são o músculo porque eles fazem o trabalho árduo diário da execução do código do aplicativo.

### Como funciona?

Para que a *orquestração* aconteça, você começa com um aplicativo, entregando/deploy ao cluster (**Kubernetes**). O **cluster** é composto por um ou mais *nós mestres* e um grupo de *nós workers*.

Os *nós mestres*, às vezes chamados de head ou *nós principais*, são os responsáveis ​​pelo *cluster*. Isso significa que eles tomam as decisões de agendamento, monitoramento, implementam mudanças, respondem a eventos e muito mais. Por essas razões, *frequentemente nos referimos aos mestres como o* **plano de controle** ou **Control Plane**.

Os *nós workers* são onde os serviços do aplicativo são executados. Cada nó reporta/responde para os mestres e fica observando constantemente novas atribuições de trabalho/mudanças.

Para executar aplicativos em um **cluster Kubernetes**, seguimos este simples padrão:

1. *Escreva o aplicativo como pequenos microservices independentes.*
2. *Empacote cada microservice em seu próprio contêiner.*
3. *Envolva cada contêiner em seu próprio *Pod*.*
4. *Implante Pods no cluster por meio de controladores, como: `Deployments`, `DaemonSets`, `StatefulSets`, `CronJobs`, etc.*

## Arquitetura do Kubernetes

<img src="{{ mix('/images/articles/Kubernetes/Kubernetes architecture.png') }}" alt="Kubernetes architecture">

A primeira coisa a entender é com quais componentes o *`API server`* pode interagir. No diagrama anterior, podemos dizer que o *`API server`* pode se comunicar com quase todos os componentes (exceto o *`container runtime`*, que é gerenciado pelo *`kubelet`*) e que também serve para interagir diretamente com os usuários finais (`CLI` ou `API RESTful`). Esse design faz com que o *`API server`* atue como o "coração" do **Kubernetes**. Além disso, o *`API server`* também examina as solicitações de entrada e grava objetos da API em seu armazenamento (`etcd`). Em outras palavras, isso torna o *`API server`* o responsável pelas medidas de controle de segurança, como autenticação, autorização e auditoria.

A segunda coisa a entender é como os diferentes componentes do *Kubernetes* (exceto para o *`API server`*) interagem uns com os outros. Acontece que não há conexão explícita entre eles - o *`controller manager`* não se comunica com o *`scheduler`*, nem o *`kubelet`* se comunica com o *`kube-proxy`*. Eles precisam trabalhar em coordenação uns com os outros para realizar muitas funcionalidades, mas nunca se comunicam diretamente. Em vez disso, eles se comunicam implicitamente por meio do *`API server`*. Mais precisamente, eles se comunicam observando, criando, atualizando ou excluindo objetos da API.

### O plano de controle do Kubernetes (*Master Nodes*)

O *plano de controle* do *Kubernetes* é um pacote/conjunto de aplicativos e serviços executados nos **nós mestres**.

É recomendável executar `3` ou `5` *nós mestres* (números impares) para garantia de alta disponibilidade. Também é considerado uma boa prática não executar os aplicativos do negócio nos mestres. Isso permite que os mestres se concentrem inteiramente no **gerenciamento do cluster**.

Existem vários serviços/componentes altamente especializados que formam o **núcleo** da responsabilidade do **Kubernetes**. Eles são os seguintes:

- **`kube-apiserver`**: É o *servidor da API do Kubernetes*. Lida com as instruções enviadas ao *Kubernetes*.

- **`kube-scheduler`**: Este componente lida com o trabalho de decidir em quais nós colocar as cargas de trabalho (*workloads*), o que pode se tornar bastante complexo.

- **`kube-controller-manager`**: Este componente fornece um *loop de controle* que garante que a *configuração desejada* do cluster e dos aplicativos em execução nele seja implementada.

- **`etcd`**: É um armazenamento de chave-valor distribuído que contém a configuração do cluster.

> **Esses componentes armazenam e gerenciam o *estado* do cluster.**

Geralmente, *todos esses componentes assumem a forma de serviços do sistema executados em cada nó mestre*. Eles podem ser iniciados manualmente se você quiser inicializar seu cluster por `CLI`, utilizando bibliotecas de criação de cluster (*Kops*, *Kubeadm*, ...) ou gerenciados por um serviço de cloud, como *Elastic Kubernetes Service* (*EKS*), isso geralmente será feito automaticamente em uma configuração de produção.

#### `kube-apiserver`

> Toda a comunicação, entre todos os componentes, deve passar pelo *`API server`*. Os componentes internos do *Kubernetes*, assim como os componentes externos do usuário, se comunicam por meio da mesma **API**. É o componente central usado por todos os outros componentes e por clientes, como *`kubectl`*. Ele fornece uma interface *CRUD* para consultar e modificar o estado do cluster por meio de uma *API RESTful*. Ele armazena esse estado no *`etcd`*.

*É o único componente que se comunica com o armazenamento (`etcd`).*

Ele expõe uma **API RESTful**, com endpoints para cada tipo de recurso, junto com uma versão da API que é passada no caminho da request, para a qual se faz um `POST` HTTPS dos arquivos de configuração *YAML*. Esses arquivos *YAML*, que são chamados de manifestos, contêm o estado desejado do aplicativo. Este estado desejado inclui coisas como: qual imagem de contêiner usar, quais portas expor e quantas réplicas de *Pod* executar por exemplo.

Todas as requests para a *`API server`* estão sujeitas as verificações de **autenticação** e **autorização**, que uma vez feitas, a configuração no arquivo *YAML* é validada, persistida (*`etcd`*) e implantada no cluster.

Além de fornecer uma maneira consistente de armazenar objetos no *`etcd`*, ele também executa a validação desses objetos, para que os clientes não possam armazenar objetos configurados incorretamente. Junto com a validação, ele também lida com o *bloqueio otimista*, de forma que as alterações em um objeto nunca sejam substituídas por outros clientes no caso de atualizações simultâneas.

Um dos clientes do *`API server`* é a ferramenta de linha de comando *`kubectl`*. Ao criar um recurso de um arquivo *YAML*, por exemplo, *`kubectl`* envia o conteúdo do arquivo para o *`API server`* por meio de uma request `POST HTTPs`. A imagem abaixo, mostra o que acontece dentro do *`API server`* quando ele recebe a request. Isso é explicado com mais detalhes nos próximos subtópicos.

<img src="{{ mix('/images/articles/Kubernetes/The operation of the API server.png') }}" alt="The operation of the API server">

O diagrama a seguir mostra o ciclo de vida da request da API e o que acontece dentro do *`API server`* quando ele recebe uma request:

<img src="{{ mix('/images/articles/Kubernetes/API server HTTP request flow.png') }}" alt="API server HTTP request flow">

Como você pode ver no diagrama anterior, a request HTTP passa pelos estágios de autenticação, autorização e controle de admissão. Vamos dar uma olhada em cada um deles nos subtópicos a seguir.

**Autenticando o cliente com plugins de Authentication**

Primeiro, o *`API server`* precisa autenticar o cliente que está enviando a request. Isso é executado por um ou mais plugins de autenticação configurados no *`API server`*. O *`API server`* chama esses plugins, até que um deles determine quem está enviando a request. Ele faz isso inspecionando HTTP request.

Quando uma request HTTP é enviada ao *`API server`*, o *`API server`* precisa autenticar o cliente que está enviando essa request. A request HTTP conterá as informações necessárias para autenticação, como username, ID e grupos aos quais o usuário da request pertence. Esses dados serão usados na próxima etapa, que é a **Authorization**. O método de autenticação será determinado pelo header (como `Authorization`) ou pelo certificado da request. Para lidar com diferentes métodos, o *`API server`* tem plugins de autenticação diferentes, como *ServiceAccount tokens*, que são usados ​​para autenticar *ServiceAccounts*, e pelo menos um outro método para autenticar usuários, como *X.509 client certificates*.

O *`API server`* chamará esses plugins um por um até que um deles autentique a request. Se todos eles falharem, a autenticação falhará. Se a autenticação for bem-sucedida, a fase de autenticação está concluída e a request segue para a próxima fase que é **Authorization**.

**Autorizando o cliente com plugins de Authorization**

Depois que a autenticação é bem-sucedida, os atributos da request HTTP são enviados ao plugin de autorização para determinar se o usuário tem permissão para executar a ação solicitada. Existem vários níveis de privilégios que diferentes usuários podem ter; por exemplo, um determinado usuário pode criar um *pod* no namespace solicitado? O usuário pode excluir um *Deployment*? Esse tipo de decisão é feito na fase de autorização.

Além dos plugins de autenticação, o *`API server`* também está configurado para usar um ou mais plugins de autorização. Seu trabalho é determinar se o usuário autenticado pode executar a ação solicitada no recurso solicitado. Por exemplo, ao criar *pods*, o *`API server`* consulta todos os plugins de autorização, por sua vez, para determinar se o usuário pode criar *pods* no *namespace* solicitado. Assim que um plugin diz que o usuário pode executar a ação, o *`API server`* avança para o próximo estágio.

Alguns comandos no `kubectl` como exemplo:

```bash
kubectl auth can-i get pods --all-namespaces
kubectl auth can-i get pods -n default
kubectl auth can-i delete pods
```

**Validando e/ou modificando o recurso na request com os plugins de controle de admissão (Admission Control)**

Após a request ser autenticada e autorizada, ela vai para os módulos de *controle de admissão*. Esses módulos podem modificar ou rejeitar a request. Se a request estiver apenas tentando executar uma operação de leitura, ela ignorará esse estágio; mas se estiver tentando criar, modificar ou excluir, será enviado para os plugins do *controle de admissão*. O Kubernetes vem com um conjunto de controladores de admissão predefinidos, embora você também possa definir controladores de admissão personalizados.

Esses plugins podem modificar o objeto de entrada, em alguns casos para aplicar padrões configurados pelo sistema ou até mesmo para negar a request. Como os módulos de autorização, se algum módulo do *controle de admissão* rejeitar a request, ela será descartada e não será processada posteriormente. Esses plugins podem modificar o recurso por diferentes motivos. Eles podem inicializar os campos ausentes da especificação do recurso para os valores padrão configurados ou até mesmo substituí-los. Eles podem até mesmo modificar outros recursos relacionados, que não estão na request, e também podem rejeitar uma request por qualquer motivo.

**NOTA:** Quando a request está apenas tentando ler os dados, a request não passa pelo *Controle de Admissão*.

Exemplos de plugins de *controle de admissão* incluem:

- `AlwaysPullImages`: Substitui a `imagePullPolicy` do *pod* por `Always`, forçando a imagem ser baixada sempre que o *pod* é implantado.

- `ServiceAccount`: Aplica a *default service account* aos *pods* que não a especificam explicitamente.

- `NamespaceLifecycle`: Impede a criação de *pods* em namespaces que estão em processo de exclusão. Quando você exclui um namespace, ele vai para o estado de `Terminating`, onde o *Kubernetes* tentará remover todos os recursos antes de excluí-lo. Portanto, não podemos criar nenhum objeto novo neste namespace.

- `NamespaceExists`: Quando um cliente tenta criar um recurso em um namespace que não existe, esse controlador irá rejeitar a solicitação.

- `ResourceQuota`: Garante que os *pods* em um determinado namespace usem apenas a quantidade de CPU e memória que foi alocada para o namespace.

Você encontrará uma lista adicionais de plugins de controle de admissão na [documentação do Kubernetes](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/).

**Validando o recurso e armazenando-o persistentemente**

Depois de permitir que a request passe por todos os plugins de *controle de admissão*, o *`API server`* valida o objeto, armazena-o no *`etcd`* e retorna uma resposta ao cliente.

#### `kube-scheduler`

O *Scheduler Kubernetes* decide onde as instâncias (*pods*) de uma carga de trabalho devem ser executadas. Por padrão, essa decisão é influenciada pelos *requisitos dos recursos* da carga de trabalho e pelo *status do nó*. Você também pode influenciar o scheduler por meio dos controles de posicionamento. Esses controles podem atuar para selecionar apenas os nó com determinadas *labels*, ou outros *pods* que já estão em execução em determinado nó e muitas outras possibilidades.

Em alto nível, o scheduler/planejador espera pelos *pods* recém-criados por meio do mecanismo de observação do *`API server`* e atribuir um nó a cada novo *pod* que ainda não tenha o nó definido. Em background, ele implementa uma lógica complexa que filtra os nós incapazes de executar os pods e, em seguida, classifica os nós que são capazes. O sistema de classificação é complexo, mas o nó com a pontuação de classificação mais alta é selecionado para executar os pods.

Tudo o que o *Scheduler* faz é atualizar a definição do *pod* por meio do *`API server`*. O *`API server`* então notifica o *Kubelet* (novamente, por meio do mecanismo de observação descrito anteriormente) que o pod foi agendado. Assim que o *Kubelet* no nó de destino vê que o pod foi programado para seu nó, ele cria e executa os contêineres do *pod*.

Ao identificar os nós que são capazes de executar os pods, o scheduler executa várias verificações. Qualquer nó incapaz de executar o pod é ignorado e os nós restantes são classificados de acordo com coisas como: o nó já tem as imagens necessária, quantos recursos livres o nó possui, quantas *pods* o nó já está executando. Cada critério vale pontos, e o nó com mais pontos é selecionado para executar os pods.

*Se o scheduler não puder encontrar um nó adequado, o pod não pode ser agendada e fica marcado como pendente.*

*O scheduler não é responsável por executar os pods, apenas escolhe os nós em que os pods será executada.*

<img src="{{ mix('/images/articles/Kubernetes/The Scheduler finds acceptable nodes for a pod and then selects the best node for the pod.png') }}" alt="The Scheduler finds acceptable nodes for a pod and then selects the best node for the pod">

**A seleção de um nó pode ser dividida em duas partes, conforme mostrado na imagem acima:**

- Filtrar a lista de todos os nós para obter uma lista de nós aceitáveis para os quais o *pod* pode ser agendado.

- Priorizar os nós aceitáveis e escolher o melhor. Se vários nós tiverem a pontuação mais alta, o algoritmo de *round-robin* será usado para garantir que os *pods* sejam implantados em todos eles de maneira uniforme.

**Para determinar quais nós são aceitáveis ​​para o *pod*, o *Scheduler* passa cada nó por uma lista de funções configuradas. Eles verificam várias coisas, como:**

- O nó pode atender às solicitações do *pod* de recursos de hardware?

- O nó está ficando sem recursos (está relatando uma condição de memória ou de disco)?

- Se a solicitação do *pod* deve ser agendado para um nó específico (por nome), este é o nó?

- O nó tem um rótulo que corresponda ao seletor de nó na especificação do *pod* (se houver um definido)?

- Se a solicitação do *pod* deve ser vinculada a uma porta de host específica, essa porta já está em uso neste nó ou não?

- Se a solicitação do *pod* determina um tipo de volume, esse volume pode ser montado para este *pod* neste nó ou outro *pod* no nó já está usando o mesmo volume?

- O *pod* tolera as taints do nó?

- O *pod* especifica o nó e/ou regras de afinidade ou antiafinidade? Em caso afirmativo, programar o *pod* para este nó violaria essas regras?

Todas essas verificações devem ser aprovadas para que o nó seja elegível para hospedar o *pod*. Depois de realizar essas verificações em cada nó, o *Scheduler* termina com um subconjunto de nós. Qualquer um desses nós pode executar o *pod*, porque eles têm recursos disponíveis suficientes para o *pod* e estão em conformidade com todos os requisitos que você especificou na definição do *pod*.

#### `kube-controller-manager`

Conforme mencionado anteriormente, o *`API server`* não faz nada, exceto armazenar recursos no *`etcd`* e notificar os *worker nodes* sobre as mudanças. O **Scheduler** atribui apenas um nó ao *pod*, portanto, você precisa de outros componentes para garantir que o estado real do cluster corresponda ao estado desejado, conforme especificado nos recursos implantados por meio do *`API server`*. Este trabalho é feito por controladores rodando dentro do **Controller Manager**. É um controlador de controladores.

O *Kubernetes Controller Manager* é um componente que executa vários controladores. Os controladores executam loops de controle que garantem que o estado real do cluster corresponda ao estado desejado, que está armazenado na configuração (`etcd`). Por padrão, isso inclui o seguinte:

- O *controlador do nó*, que garante que os nós estejam funcionando.

- O controlador de replicação, que garante que cada carga de trabalho seja dimensionada corretamente.

- O controlador de endpoints, que lida com a configuração da comunicação e roteamento para cada carga de trabalho.

- Controladores de conta de serviço e token, que lidam com a criação de tokens de acesso da API e contas padrão.

Cada controlador é executado como um *watch-loop* que está constantemente observando a *`API server`* em busca de alterações - o objetivo é garantir que o estado atual do cluster corresponda ao estado desejado.

A lógica implementada por cada loop de controle é efetivamente esta:

1. *Obtenha o estado desejado* (**desired**)
2. *Observe o estado atual* (**current**)
3. *Determinar a diferença*
4. *Reconciliar as diferenças*

Cada loop de controle também é extremamente especializado e interessado apenas em sua própria responsabilidade no cluster do *Kubernetes*. Nenhuma tentativa é feita para complicar as coisas implementando a conscientização de outras partes do sistema - cada loop de controle é responsável apenas pelo seu próprio negócio. Esta é a chave para o design distribuído do *Kubernetes* e segue a filosofia Unix de construir sistemas complexos a partir de pequenas partes especializadas.

#### `etcd`

*`etcd`* é um banco de dados distribuído de chave-valor que hospeda a configuração do cluster de uma forma altamente disponível. É a única parte com estado do *plano de controle* e armazena de forma persistente toda a configuração e o estado do cluster. Como tal, é um componente vital do cluster - **sem armazenamento de cluster, sem cluster**. Uma réplica do *`etcd`* é executada em cada *nó mestre* e usa o algoritmo de *consenso Raft*, que garante que um *quorum* seja mantido antes de permitir qualquer alteração nas chaves ou valores. O *`etcd`* usa o algoritmo de *consenso RAFT* para conseguir isso, o que garante que, a qualquer momento, o estado de cada nó seja o que a maioria dos nós concorda ser o estado atual ou é um dos estados previamente acordados.

Como é a única fonte da verdade para o *cluster*, você deve executar entre `3` e `5` réplicas do *`etcd`* para alta disponibilidade e deve fornecer maneiras adequadas de recuperação quando as coisas derem errado.

Os clientes que se conectam a diferentes nós de um cluster *`etcd`* verão o estado atual real ou um dos estados do passado (no Kubernetes, o único cliente *`etcd`* é o *`API server`*, mas pode haver várias instâncias).

O algoritmo de consenso *RAFT* requer uma maioria (ou quorum) para que o cluster avance para o próximo estado. Como resultado, se o cluster se dividir em dois grupos, o estado nos dois grupos nunca pode divergir, porque para fazer a transição do estado anterior para o novo, é necessário que mais da metade dos nós participem da mudança de estado. Se um grupo contém a maioria de todos os nós, o outro obviamente não contém. O primeiro grupo pode modificar o estado do cluster, enquanto o outro não. Quando os dois grupos se reconectam, o segundo grupo pode alcançar o estado do primeiro grupo (consulte a imagem abaixo).

![In a split-brain scenario, only the side which still has the majority -quorum- accepts state changes](/images/articles/Kubernetes/In%20a%20split-brain%20scenario,%20only%20the%20side%20which%20still%20has%20the%20majority%20(quorum)%20accepts%20state%20changes.png?id=2173690f2f943b4eddfcb1af5b3948f3 "In a split-brain scenario, only the side which still has the majority (quorum) accepts state changes")

O *`etcd`* prefere consistência em vez de disponibilidade. Isso significa que ele não tolerará uma situação de split-brain e interromperá as atualizações do *cluster* para manter a consistência. No entanto, se o *`etcd`* ficar indisponível, os aplicativos em execução no cluster devem continuar a funcionar, você simplesmente não poderá atualizar nada.

**Por que o número de instâncias de `etcd` deve ser um número ímpar?**

O *`etcd`* geralmente é implantado com um número ímpar de instâncias. Tenho certeza de que você gostaria de saber por quê. Vamos comparar ter duas vs. ter uma instância. Ter duas instâncias requer que ambas as instâncias estejam presentes para obter a maioria. Se um deles falhar, o cluster *`etcd`* não poderá fazer a transição para um novo estado porque não existe maioria. Ter duas instâncias é pior do que ter apenas uma única instância. Por ter dois, a chance de falha de todo o cluster aumentou em 100%, em comparação com a de falha de um cluster de único nó.

O mesmo se aplica ao comparar três e quatro instâncias do *`etcd`*. Com três instâncias, uma instância pode falhar e a maioria (de duas) ainda existe. Com quatro instâncias, você precisa de três nós para uma maioria (dois não são suficientes). Em ambos os clusters de três e quatro instâncias, apenas uma única instância pode falhar. Mas ao executar quatro instâncias, se uma falhar, existe uma possibilidade maior de uma instância adicional das três instâncias restantes falhar (em comparação com um cluster de três nós com um nó com falha e dois nós restantes).

Normalmente, para grandes clusters, um cluster *`etcd`* de cinco ou sete nós é suficiente. Ele pode lidar com uma falha de dois ou três nós, respectivamente, o que é suficiente em quase todas as situações.

#### Resumo do plano de controle

Os **nós mestres** no *Kubernetes* executam todos os serviços do *plano de controle* do cluster. Pense nisso como o cérebro do cluster onde todas as decisões de controle e programação são feitas. Em background, um *nó mestre* é composto de vários pequenos loops de controle e serviços especializados. Isso inclui o *`API server`*, o *`etcd`* (armazenamento do cluster), o *Controller Manager* e o *Scheduler*.

O *`API server`* é o front-end para o plano de controle e todas as instruções e comunicações devem passar por ele. Por padrão, ele expõe um endpoint *RESTful* na porta 443.

A imagem abaixo mostra uma visão de alto nível de um *nó mestre* no *Kubernetes* (plano de controle).

<img src="{{ mix('/images/articles/Kubernetes/Kubernetes Master.png') }}" alt="Kubernetes Master">

#### Verificando o status dos componentes do plano de controle

O *`API server`* expõe um recurso chamado **ComponentStatus**, que mostra o status de funcionamento de cada componente do *Control Plane*. Você pode listar os componentes e seus status com `kubectl`:

```bash
kubectl get --raw='/readyz?verbose'
kubectl get --raw='/livez?verbose'
kubectl get --raw='/healthz?verbose'
```

### Os nós de trabalho do Kubernetes

> Cada **nó de trabalho** do *Kubernetes* contém componentes que permitem que ele se comunique com o **plano de controle** e trabalhe com a rede.

> Primeiro, existe o *`kubelet`*, que garante que os contêineres estejam sendo executados no nó conforme determinado pela configuração do cluster. Em segundo lugar, o *`kube-proxy`* que fornece uma camada de *proxy de rede* para cargas de trabalho executando em cada nó. E, finalmente, o *`container runtime`* que é usado para *executar as cargas de trabalho* em cada nó.

Os nós é onde os workers de um cluster do Kubernetes são executados. Em um alto nível, eles fazem três coisas:

1. Observar a *`API server`* para novas atribuições de trabalho;
2. Executar novas atribuições de trabalho;
3. Reportar de volta ao plano de controle (por meio da *`API server`*);

Como podemos ver na imagem abaixo, eles são um pouco mais simples do que os *nós mestres*.

![Kubernetes Node (formerly Minion)](/images/articles/Kubernetes/Kubernetes%20Node%20(formerly%20Minion).png?id=e03f18590e2188a11611b7e82f4de5cc "Kubernetes Node (formerly Minion)")

**Vejamos os três principais componentes de um nó de trabalho**.

#### *`kubelet`*

O *`kubelet`* é o principal agent do *Kubernetes* e é executado em todos os *nós* do *cluster*. Seu objetivo principal é receber uma lista de *`PodSpecs`* (manifestos de *pods*) e garantir que os contêineres prescritos por eles estejam sendo executados no nó. O *`kubelet`* obtém esses *`PodSpecs`* por meio de alguns mecanismos diferentes, mas a principal maneira é consultar o *`API server`*.

Uma das principais tarefas do *`kubelet`* é observar o *`API server`* em busca de novas atribuições de trabalho (obter os manifestos dos *pods*). Sempre que vê uma, ele executa a tarefa e mantém um canal de relatório de volta ao *plano de controle*.

Se um *`kubelet`* não puder executar uma tarefa específica, ele se reporta ao *mestre* e permite que o *plano de controle* decida quais ações tomar. Por exemplo, se um *Kubelet* não pode executar uma tarefa, ele não é responsável por encontrar outro nó para executá-lo. Ele simplesmente reporta de volta ao *plano de controle* e o *plano de controle* decide o que fazer.

Em suma, o *Kubelet* é o componente responsável por tudo que está sendo executado em um **nó de trabalho**. Sua tarefa inicial é registrar o nó em que está sendo executado, criando um recurso *Node* no *`API server`*. Em seguida, ele precisa monitorar continuamente o *`API server`* para obter os pods que foram programados para o nó e iniciar os contêineres do *pod*. Ele faz isso informando ao *container runtime* para executar um contêiner a partir de uma imagem específica. O *Kubelet* monitora constantemente os contêineres em execução e relata seu status, eventos e consumo de recursos ao *`API server`*.

O *Kubelet* também é o componente que executa os testes de atividade (*liveness probes*) do contêiner, reiniciando os contêineres quando os testes falham. Por último, ele encerra os contêineres quando seu pod é excluído do *`API server`* e notifica o servidor de que o *pod* foi encerrado.

<img src="{{ mix('/images/articles/Kubernetes/The Kubelet runs pods based on pod specs from the API server and a local file directory.png') }}" alt="The Kubelet runs pods based on pod specs from the API server and a local file directory">

#### *Container runtime*

O *Kubelet* precisa de um *container runtime* para realizar tarefas relacionadas ao contêiner - coisas como baixar as imagens, iniciar e parar os contêineres.

No começo, o *Kubernetes* tinha suporte nativo para alguns *container runtimes*, como *Docker*. Mais recentemente, ele mudou para um modelo de plugin denominado **Container Runtime Interface** (*CRI*). Em um alto nível, o *CRI* mascara o mecanismo interno do *Kubernetes* e expõe uma interface clean e documentada para que os outros *container runtimes* se conectem.

Existem muitos *container runtimes* disponíveis para o *Kubernetes*. Um exemplo popular é *`cri-containerd`*. É um projeto open-source da comunidade. Ele tem muito suporte e é o *container runtime* mais popular usado no *Kubernetes*.

#### *`kube-proxy`*

O *`kube-proxy`* é um proxy de rede executado em todos os *nós* do cluster e é responsável pela rede local do cluster. Seu objetivo principal é fazer o encaminhamento de *TCP*, *UDP* e *SCTP* (por meio de stream ou round-robin) para cargas de trabalho em execução no *nó*. Por exemplo, ele garante que cada nó tenha seu próprio endereço IP exclusivo e implemente regras locais de *IPTABLES* ou *IPVS* para lidar com o roteamento e o balanceamento de carga do tráfego na rede do *Pod*.

### A natureza distribuída dos componentes do Kubernetes

Todos os *componentes* mencionados anteriormente são executados como **processos individuais**.

<img src="{{ mix('/images/articles/Kubernetes/Kubernetes components of the Control Plane and the worker nodes.png') }}" alt="Kubernetes components of the Control Plane and the worker nodes">

Os componentes se comunicam apenas com o *`API server`*. Eles não falam um com o outro diretamente. O *`API server`* é o único componente que se comunica com o *`etcd`*. Nenhum dos outros componentes se comunica diretamente com o *`etcd`*, mas, em vez disso, modificam o estado do *cluster* conversando com o *`API server`*.

As conexões entre o *`API server`* e os outros componentes quase sempre são iniciadas pelos componentes, conforme mostrado na imagem acima. Mas o *`API server`* se conecta ao *Kubelet* quando você usa o *Kubelet* para recuperar os logs ou `kubectl attach` para se conectar a um contêiner em execução.

### Entendendo o funcionamento dos controladores

Agora você sabe sobre todos os **componentes de um cluster** do *Kubernetes*. Agora, para solidificar sua compreensão de como o *Kubernetes* funciona, vamos examinar o que acontece quando um recurso de *pod* é criado. Como você normalmente não cria *pods* diretamente, em vez disso, criará um recurso de *Deployment* e verá tudo o que deve acontecer para que os contêineres do *pod* sejam iniciados.

#### Compreender quais componentes estão envolvidos

Antes mesmo de iniciar todo o processo, os controladores, o *Scheduler* e o *Kubelet* estão observando o *`API server`* em busca de alterações em seus respectivos tipos de recursos. Isso é mostrado na imagem abaixo. Cada um dos componentes representados na imagem desempenhará uma responsabilidade no processo que você está prestes a desencadear. O diagrama não inclui o *`etcd`*, porque está oculto atrás do *`API server`* e você pode pensar no *`API server`* como o local onde os objetos são armazenados.

<img src="{{ mix('/images/articles/Kubernetes/Kubernetes components watching API objects through the API server.png') }}" alt="Kubernetes components watching API objects through the API server">

#### A cadeia de eventos

Imagine que você preparou o arquivo *YAML* contendo o manifesto de *Deployment* e está perto de enviar ao *Cluster Kubernetes* por meio do *`kubectl`*. O *`kubectl`* então, envia o manifesto ao *`API server`*  em uma request *HTTP POST*. O *`API server`* valida a especificação de *Deployment*, armazena-a no *`etcd`* e retorna uma resposta para o *`kubectl`*. Agora, uma cadeia de eventos começa a se desdobrar, conforme imagem abaixo.

<img src="{{ mix('/images/articles/Kubernetes/The chain of events that unfolds when a Deployment resource is posted to the API server.png') }}" alt="The chain of events that unfolds when a Deployment resource is posted to the API server">

**1. O *Deployment Controller* cria o *ReplicaSet***

Todos os *clients* da *`API server`* que observam a lista de *Deployments* por meio do mecanismo de observação da *`API server`* são notificados sobre o recurso de *Deployment* recém-criado imediatamente após sua criação. Um desses *clients* é o *Deployment controller*, que é o componente responsável por lidar com *Deployments*.

Um *Deployment* é baseado por um ou mais *ReplicaSets*, que criam os *pods* reais. Conforme um novo objeto de *Deployment* é detectado pelo *Deployment controller*, ele cria um *ReplicaSet* para a especificação atual da *Deployment*. Isso envolve a criação de um novo recurso *ReplicaSet* por meio da *`API server`*. O *Deployment controller* não lida com *pods* individuais.

**2. O *ReplicaSet controller* cria os recursos do Pod**

O *ReplicaSet* recém-criado é então recuperado pelo *ReplicaSet controller*, que observa as criações, modificações e exclusões de recursos de *ReplicaSet* no *`API server`*. O controlador leva em consideração a contagem de réplicas e o *seletor de pod* definidos no *ReplicaSet* e verifica se pods existentes suficientes correspondem ao *seletor*.

O controlador então cria os *recursos do Pod* com base no *pod template* no *ReplicaSet* (o *pod template* foi copiado do *Deployment* quando o *Deployment controller* criou o *ReplicaSet*).

**3. O *Scheduler* atribui um nó para os Pods recém-criados**

Esses *Pods* recém-criados agora são armazenados no *`etcd`*, mas cada um deles ainda precisa de uma coisa importante - eles ainda não têm um *nó* associado. Seu atributo `nodeName` não está definido. O *Scheduler* monitora os *Pods* como este e, quando encontra um, escolhe o melhor *nó* para o *Pod* e atribui o *Pod* ao *nó*. A definição do *Pod* agora inclui o nome do nó em que ele deve ser executado.

Tudo até agora está acontecendo no **plano de controle** do *Kubernetes*. Nenhum dos controladores que participaram de todo este processo fez nada tangível, exceto atualizar os recursos através do *`API server`*.

**4. O *Kubelet* executa os containers do Pod**

Os *worker nodes* não fizeram nada até este ponto. Os contêineres do *Pod* ainda não foram iniciados. As imagens dos contêineres do *Pod* ainda não foram baixadas.

Mas com o *Pod* agora agendado para um nó específico, o *Kubelet* nesse nó pode finalmente começar a trabalhar. O *Kubelet*, observando as mudanças dos Pods no *`API server`*, vê um novo *Pod* agendado para seu nó, então ele inspeciona a definição do *Pod* e instrui o Docker, ou qualquer outro *container runtime* que está usando, para iniciar os contêineres do *Pod*. O *container runtime* executa os contêineres.

### Namespaces

> Os namespaces são uma maneira de dividir logicamente um *cluster* em vários *clusters* virtuais para fins de gerenciamento.

> Um namespace no *Kubernetes* é uma construção que permite agrupar recursos em seu *cluster*. Eles são um método de separação com muitos usos possíveis. Por exemplo, você pode ter um namespace em seu *cluster* para cada ambiente - *`dev`*, *`staging`* e *`production`*.

Por padrão, o *Kubernetes* cria o namespace `default`, `kube-system` e o `kube-public`. Os recursos criados sem um namespace especificado serão criados no namespace `default`. `kube-system` contém os serviços do *cluster*, como *`etcd`*, o *scheduler* e qualquer recurso criado pelo próprio *Kubernetes* e não pelos usuários. `kube-public` pode ser lido por todos os usuários por padrão e pode ser usado para recursos públicos.

## Empacotando os Aplicativos Para o Kubernetes

Para que um *aplicativo* seja executado em um *cluster Kubernetes*, é necessário alguns pontos. Dentre eles:

1. **Construido como um contêiner**
2. **Executado em um Pod**
3. **Implantado por meio de um arquivo de manifesto declarativo**

É assim... Você codifica um serviço do *aplicativo* em qualquer linguagem. Você cria uma imagem de contêiner que contêm esse serviço e o armazena em um Container Registry. Nesse ponto, o serviço do *aplicativo* está em contêiner.

Em seguida, você define um *Pod Kubernetes* para executar o *aplicativo* em um contêiner. No nível mais alto em que estamos, um *Pod* é apenas um wrapper que permite que um contêiner seja executado em um *cluster Kubernetes*. Depois de definir o *Pod*, você está pronto para implantá-lo no cluster.

É possível executar um *Pod* independente em um *cluster Kubernetes*. Mas o modelo preferencial é implantar todos os *Pods* por meio de controladores de alto nível. O controlador mais comum é o *Deployment*. Ele oferece escalabilidade, self-healing e rolling updates (atualizações contínuas). Você define *Deployments* em arquivos de manifesto *YAML* que especifica coisas como: qual imagem usar e quantas réplicas implantar.

A imagem abaixo mostra o código do *aplicativo* empacotado como um contêiner, em execução dentro de um *Pod*, gerenciado por um controlador *Deployment*.

<img src="{{ mix('/images/articles/Kubernetes/Shows application code packaged as a container, running inside a Pod, managed by a Deployment controller.png') }}" alt="Shows application code packaged as a container, running inside a Pod, managed by a Deployment controller">

Depois que tudo estiver definido no arquivo *YAML* do *Deployment*, faça uma request `POST` para a *`API server`* como o estado desejado do *aplicativo* e deixe o *Kubernetes* implantá-lo.

## O Modelo Declarativo e o Estado Desejado

O **modelo declarativo** e o conceito de **estado desejado** estão no centro do *Kubernetes*.

No Kubernetes, o **modelo declarativo** funciona assim:

1. *Declarar o estado desejado de um aplicativo (microservice) em um arquivo de manifesto*
2. *Fazer a requisição `POST` para a* *`API server`*
3. *O Kubernetes salva no armazenamento do cluster (`etcd`) como o estado desejado do aplicativo*
4. *Kubernetes implementa o estado desejado no cluster*
5. *O Kubernetes implementa loops de observação para garantir que o estado atual do aplicativo não seja diferente do estado desejado*

Vamos examinar cada etapa com mais detalhes.

Os *arquivos de manifesto* são criados em *YAML* e informam ao *Kubernetes* como você deseja que um aplicativo funcione. Isso é chamado de **estado desejado**. Inclui coisas como: qual imagem usar, quantas réplicas executar, quais portas de rede ouvir e como executar as atualizações.

Depois de criar o manifesto, faz-se uma requisição `POST` para a *`API server`*. A maneira mais comum de fazer isso é por meio do CLI **`kubectl`**. Isso envia o manifesto para o plano de controle como um *HTTP POST*, geralmente na porta *443*.

Depois que a solicitação é autenticada e autorizada, o *Kubernetes* inspeciona o manifesto, identifica para qual controlador enviá-lo (por exemplo, o Deployment controller) e salva a configuração no armazenamento do cluster como parte do estado desejado do cluster. Feito isso, o worker é agendado no cluster. Isso inclui o trabalho árduo de baixar imagens, iniciar contêineres, construir as redes e iniciar os processos do aplicativo.

Por fim, o Kubernetes utiliza loops de reconciliação em background que monitoram constantemente o estado do cluster. Se o estado atual do cluster variar do estado desejado, o Kubernetes executará todas as tarefas necessárias para reconciliar o problema.

<img src="{{ mix('/images/articles/Kubernetes/Desired state.png') }}" alt="Desired state">

O *modelo declarativo* não é apenas muito mais simples do que scripts longos com muitos comandos imperativos, mas também permite *self-healing*, o dimensionamento e funciona para o controle de versão e auto-documentação. Ele faz isso informando ao cluster como as coisas devem ser. Se eles pararem de se parecer com isso, o *cluster* perceberá a discrepância e fará todo o trabalho árduo para reconciliar a situação.

Mas a história declarativa não termina aí - as coisas dão errado e mudam. Quando isso acontece, o estado atual do cluster não corresponde mais ao estado desejado. Assim que isso acontece, o Kubernetes entra em ação e tenta trazer os dois de volta à harmonia.

## Como o `kubectl` se Comunica Com o Kubernetes?

O *`API server`* gerencia as comunicações entre o usuário final e o *Kubernetes* e também atua como um gateway de API para o *cluster*. Para conseguir isso, ele implementa uma *API RESTful* sobre os protocolos HTTP e HTTPS para realizar operações CRUD para preencher e modificar objetos da *API Kubernetes*, como *Pods*, serviços e muito mais, com base nas instruções enviadas por um usuário via *`kubectl`*. Essas instruções podem ter vários formatos. Por exemplo, para recuperar informações dos *Pods* em execução no cluster, usaríamos o comando *`kubectl get pods`*, enquanto para criar um novo Pod, usaríamos o comando *`kubectl run`*.

Vamos dar uma olhada o que acontece quando você executa um comando *`kubectl`*. Dê uma olhada na imagem a seguir, que fornece uma visão geral do processo:

<img src="{{ mix('/images/articles/Kubernetes/A representative flowchart for the kubectl utility.png') }}" alt="A representative flowchart for the kubectl utility">

Um comando *`kubectl`* é traduzido em uma chamada de *API*, que é enviada ao *`API server`*. O *`API server`* então autentica e valida as solicitações. Assim que os estágios de autenticação e validação forem bem-sucedidos, o *`API server`* recupera ou atualiza os dados no *`etcd`* e responde com as informações solicitadas.

## Ciclo de Vida do Pod e Componentes do Kubernetes

Como um objeto da API do Kubernetes é gerenciado por diferentes componentes do Kubernetes? Vamos considerar um *Pod* como exemplo. Seu *ciclo de vida* pode ser ilustrado da seguinte forma:

<img src="{{ mix('/images/articles/Kubernetes/The process behind the creation of a pod.png') }}" alt="The process behind the creation of a pod">

Todo este processo pode ser dividido da seguinte forma:

1. Um usuário começa a implantar um aplicativo enviando um manifesto *YAML* de *Deployment* ao *`API server`*. O *`API server`* verifica a solicitação e verifica se ela é válida. Se for, ele persiste o objeto *Deployment* em seu armazenamento *`etcd`*.

2. Até agora, o *Pod* ainda não foi criado. O *controller manager* recebe uma notificação do *`API server`* de que um *Deployment* foi criado.

3. Em seguida, o *controller manager* verifica se o número desejado de réplica dos *Pods* já está em execução.

4. Se não houver *Pods* suficientes em execução, ele criará o número apropriado de *Pods*. A criação dos *Pods* é realizada enviando uma solicitação com a especificação do *Pod* para o *`API server`*. É muito semelhante a como um usuário aplicaria o *Deployment YAML*, mas com a principal diferença é que isso acontece dentro do *controller manager* de maneira programática.

5. Embora os *Pods* tenham sido criados, eles não são nada além de alguns objetos API armazenados no *`etcd`*. Agora, o scheduler recebe uma notificação do *`API server`* informando que novos *Pods* foram criados e nenhum nó foi atribuído para que eles possam ser executados.

6. O scheduler verifica o uso dos recursos, bem como a alocação dos *Pods* existentes e, a seguir, calcula o nó que melhor se adapta a cada novo *Pod*. No final desta etapa, o scheduler envia uma solicitação de atualização ao *`API server`*, definindo a especificação `nodeName` do Pod para o nó escolhido.

7. Até agora, os *Pods* receberam um nó adequado para execução. No entanto, nenhum contêiner físico está em execução. Em outras palavras, o aplicativo ainda não funciona. Cada *`kubelet`* (executado em nós de trabalho diferentes) recebe notificações, indicando que alguns *Pods* devem ser executados. Cada *`kubelet`* verificará se os *Pods* a serem executados foram atribuídos ao nó em que um *`kubelet`* está sendo executado.

8. Depois que o *`kubelet`* determina que um *Pod* deve estar em seu nó, ele chama o *container runtime* subjacente (Docker, containerd ou cri-o, por exemplo) para ativar os contêineres no host. Depois que os contêineres estão ativos, o *`kubelet`* é responsável por relatar seu status de volta ao *`API server`*.
