---
id: 9c457e56-9d39-41a3-9dc7-1fae52ca25c3
title: "Pods: Executando Containers no Kubernetes"
summary: "Conhecendo sobre Pods, a unidade básica, mais simples do K8s"
---

> O *`Pod`* é o recurso de construção mais simples do *Kubernetes* e pode ser descrito como a unidade básica do *Deployment*. Assim como definimos um processo como um programa em execução, podemos definir um *`Pod`* como um processo em execução no mundo *Kubernetes*. Os *`Pods`* são a menor unidade de replicação no *Kubernetes*. Um *`Pod`* é basicamente um invólucro em torno dos containers em execução em um nó. Um *`Pod`* pode ter qualquer número de containers em execução, ele especifica um ou mais containers a serem iniciados e executados pelo *Scheduler* do *Kubernetes* em um nó. Usar *`Pods`* em vez de containers individuais tem alguns benefícios. Por exemplo, os containers em um *`Pod`* têm volumes compartilhados, namespaces e cgroups do Linux. Cada *`Pod`* tem um endereço IP exclusivo e as portas são compartilhadas por todos os containers desse *`Pod`*. Isso significa que diferentes containers dentro de um *`Pod`* podem se comunicar uns com os outros usando suas portas correspondentes no `localhost`. Os *`Pods`* têm muitas configurações e extensões possíveis, mas continuam sendo a maneira mais básica de executar aplicativos no *Kubernetes*.

> **Um *Pod* é um ou conjunto de containers e representa o bloco de construção básico no Kubernetes.** Em vez de implantar e gerenciar containers individualmente, você sempre implanta e opera em um conjunto de containers. O principal ponto sobre os *`Pods`* é que, quando um *Pod* contém vários containers, todos eles sempre são executados em um único *nó de trabalho* - nunca se estende por vários *nós de trabalho*, como mostrado na imagem abaixo.

<img src="{{ mix('/images/articles/Kubernetes/All containers of a pod run on the same node. A pod never spans two nodes.png') }}" alt="All containers of a pod run on the same node. A pod never spans two nodes">

## Compreendendo os *Pods*

Da mesma forma que não se deve agrupar vários processos em um único container, é necessário uma construção de nível superior que permita vincular containers e gerenciá-los **como uma única unidade**. Esse é o raciocínio por trás dos *``Pods``*.

Um conjunto de containers permite que você execute processos intimamente relacionados e forneça-os com (quase) o mesmo ambiente, **como se estivessem todos em um único container**, mantendo-os um pouco isolados. Dessa forma, você obtém o melhor dos dois mundos. Você pode aproveitar todos os recursos oferecidos pelos containers e, ao mesmo tempo, dar aos processos a ilusão de estarem juntos.

Uma coisa a ser ressaltada é que, como **os containers em um *`Pod`* são executados no mesmo namespace de rede**, eles compartilham o mesmo endereço IP e portas. Isso significa que os processos que estão sendo executados em containers do mesmo *`Pod`* precisam tomar cuidado para não se vincular aos mesmos números de portas para evitar que haja conflitos de portas. Mas isso só diz respeito a containers no mesmo *`Pod`*. containers de diferentes *`Pods`* nunca podem entrar em conflito de portas, porque as portas de cada *`Pod`* trabalham de forma individual. **Todos os containers em um *`Pod`* também têm a mesma interface de rede de *loopback*, portanto, um container pode se comunicar com outros containers no mesmo *`Pod`* por meio do *`localhost`***.

Para resumir: os *`Pods`* são hosts lógicos e se comportam de maneira muito semelhante a hosts físicos ou VMs no mundo sem containers. Processos em execução no mesmo Pod são como processos em execução na mesma máquina física ou virtual, exceto pelo fato de cada processo ser encapsulado em um container.

### *Pods* - Operações atômicas

**A implantação de um *`Pod`* é uma operação atômica**. Isso significa que um *`Pod`* só é considerado pronto para o serviço quando todos os seus containers estão ativos e em execução. Isso significa que é uma operação tudo ou nada - não existe um *`Pod`* parcialmente implantado que possa atender às solicitações. Isso também significa que todos os containers em um *`Pod`* serão programados no mesmo *nó*.

Um único *`Pod`* só pode ser agendado para um único nó. Isso também é verdadeiro para *`Pods`* de vários containers - todos os containers no mesmo *`Pod`* serão executados no mesmo nó.

*Assim que todos os recursos do `Pod` estiverem prontos, o `Pod` pode começar a atender às solicitações.*

## Organizando os Containers nos *Pods* Corretamente

Você deve pensar nos *`Pods`* como máquinas separadas, onde cada um hospeda apenas um determinado aplicativo. Ao contrário da antiga forma, quando costumávamos colocar todos os tipos de aplicativos no mesmo host, não fazemos isso com os *`Pods`*. Como os *`Pods`* são relativamente leves, você pode ter quantos você precisar com o menor custo possível de sobrecarga. Em vez de colocar tudo em um único *`Pod`*, você deve organizar os aplicativos em vários *`Pods`*, onde cada um contém apenas componentes ou processos altamente relacionados.

O *Kubernetes* não pode dimensionar horizontalmente containers individuais; em vez disso, dimensiona os *`Pods`*. Se o seu *`Pod`* consiste em um container front-end e um container back-end, quando você aumenta o número das instâncias do *`Pod`* para, digamos, dois *`Pods`*, você acaba com dois containers front-end e dois containers back-end.

Normalmente, os componentes do front-end têm requisitos de dimensionamento completamente diferentes dos back-end, portanto, tendemos a escalá-los individualmente. Sem mencionar o fato que os containers de back-end, como os bancos de dados, são geralmente muito mais difíceis de escalar em comparação com os servidores Web front-end (*stateless*). Se você precisar dimensionar um container individualmente, essa é uma clara indicação de que ele precisa ser implantado em um *`Pod`* separado.

Se você precisar dimensionar seu aplicativo, adicione ou remova *`Pods`*. Você não escalona adicionando mais containers a um *`Pod`* existente. Os *`Pod`* de vários containers são apenas para situações em que dois containers diferentes, mas complementares, precisam compartilhar recursos. A imagem abaixo mostra como dimensionar o *front-end* nginx de um aplicativo usando vários *`Pod`* como unidade de dimensionamento.

<img src="{{ mix('/images/articles/Kubernetes/Scaling with Pods.png') }}" alt="Scaling with Pods">

### Entendendo quando usar vários containers em um *Pod*

A principal razão para colocar vários containers em um único *`Pod`* é **quando o aplicativo consiste em um processo principal e um ou mais processos complementares**, conforme mostrado na imagem abaixo.

<img src="{{ mix('/images/articles/Kubernetes/Pods should contain tightly coupled containers, usually a main container and containers that support the main one.png') }}" alt="Pods should contain tightly coupled containers, usually a main container and containers that support the main one">

### Decidindo quando usar vários containers em um *Pod*

Para recapitular como os containers devem ser agrupados em *`Pods`* - ao decidir colocar dois containers em um único *`Pod`* ou em dois *`Pods`* separados, você precisa fazer as seguintes perguntas:

- *Eles precisam ser executados juntos ou podem ser executados em hosts diferentes?*
- *Eles representam um todo ou são componentes independentes?*
- *Eles devem ser escalados juntos ou individualmente?*

Basicamente, você deve sempre preferir executar os containers em *`Pods`* separados, a menos que um motivo específico exija que eles façam parte do mesmo *`Pod`*.

<img src="{{ mix('/images/articles/Kubernetes/A container shouldnt run multiple processes. A pod shouldnt contain multiple containers if they dont need to run on the same machine.png') }}" alt="A container shouldn&#039;t run multiple processes. A pod shouldn&#039;t contain multiple containers if they don&#039;t need to run on the same machine">

## Implantando *Pods*

Lembre-se de que os *`Pods`* são apenas um meio para a execução de um aplicativo. Portanto, sempre que falamos sobre execução ou implantação dos *`Pods`*, estamos falando sobre execução e implantação de aplicativos.

Para implantar um *`Pod`* em um *cluster Kubernetes*, você o define em um arquivo de manifesto e faz uma chamada `POST` desse arquivo de manifesto para o *`API server`*. O plano de controle verifica a configuração do arquivo *YAML*, grava no armazenamento do cluster e o implanta em um nó íntegro com recursos disponíveis. Esse processo é idêntico para *`Pods`* de um único container e *`Pods`* com vários containers.

<img src="{{ mix('/images/articles/Kubernetes/How do we deploy Pods.png') }}" alt="How do we deploy Pods">

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: hello-pod
  labels:
    zone: prod
    version: v1
spec:
  containers:
  - name: hello-ctr
    image: hello-image
    ports:
    - containerPort: 8080
EOF
# pod/hello-pod created
```

Execute o comando `kubectl get pods` para verificar o status:

```bash
kubectl get pods
# NAME            READY   STATUS              RESTARTS    AGE
# hello-pod       0/1     ContainerCreating   0           9s
```

Você pode ver que o container ainda está sendo criado - provavelmente esperando que a imagem seja baixada do Docker Hub.

Você pode adicionar a flag `--watch` ao comando `kubectl get pods` para monitorá-lo e ver quando o status muda para `Running`.

## Anatomia do *Pod*

No nível mais alto, um *`Pod`* é um ambiente para executação de containers. O *`Pod`* em si não executa nada, é apenas um sandbox para hospedar os containers.

O *`Pod`* é apenas uma coleção de recursos do sistema que os containers executados dentro dele irão herdar e compartilhar. Esses recursos do sistema são *namespaces do kernel* e incluem:

- **Network namespace**: *IP address, port range, routing table...*
- **UTS namespace**: *Hostname*
- **IPC namespace**: *Unix domain sockets...*

Isso significa que todos os containers em um *`Pod`* compartilham o nome do host, endereço IP, memória e volumes.

Vejamos como isso afeta a rede.

Se você estiver executando vários containers em um *`Pod`*, todos eles compartilham o mesmo ambiente do *`Pod`*. Isso inclui coisas como o *IPC namespace*, *memória compartilhada*, volumes, rede e muito mais. Por exemplo, isso significa que todos os containers no mesmo *`Pod`* compartilharão o mesmo endereço IP (o IP do *`Pod`*). Isso é mostrado na imagem abaixo.

<img src="{{ mix('/images/articles/Kubernetes/Same Pod environment.png') }}" alt="Same Pod environment">

Se dois containers no mesmo *`Pod`* precisam se comunicar (container a container dentro do *`Pod`*), eles podem usar as portas na interface `localhost` do *`Pod`*, conforme mostrado na imagem abaixo.

<img src="{{ mix('/images/articles/Kubernetes/Same Pod environment - localhost.png') }}" alt="Same Pod environment - localhost">

Os *`Pods`* de vários containers são ideais quando você tem requisitos para containers fortemente acoplados que podem precisar compartilhar memória e armazenamento. No entanto, se você não precisa acoplar seus containers, coloque-os em seus próprios *`Pods`*. Isso mantém as coisas limpas, pois cada *`Pod`* é dedicado a uma única tarefa. Ele também cria muito tráfego de rede não criptografada.

## *Pods* e a Rede Compartilhada

Cada *`Pod`* cria seu próprio `namespace` de rede. Isso inclui: um único endereço IP, um único intervalo de portas *TCP* e *UDP* e uma única tabela de roteamento. Se um *`Pod`* tiver um único container, esse container terá acesso total ao IP, intervalo de portas e tabela de roteamento. Se for um *`Pod`* de vários containers, todos os containers no *`Pod`* compartilharão o IP, o intervalo de portas e a tabela de roteamento.

Os containers em um *`Pod`* compartilham a mesma rede, de modo que cada container tem o mesmo endereço IP - o endereço IP do *`Pod`*. Vários containers podem receber tráfego externo, mas precisam escutar em portas diferentes, e os containers dentro do mesmo *`Pod`* podem se comunicar usando o `localhost`. Cada container tem seu próprio sistema de arquivos, mas pode montar volumes do *`Pod`*, para que os containers possam trocar informações compartilhando as mesmas montagens. A imagem abaixo mostra o layout de um *`Pod`* com dois containers.

<img src="{{ mix('/images/articles/Kubernetes/The Pod is a shared network and storage environment for many containers.png') }}" alt="The Pod is a shared network and storage environment for many containers">

A imagem abaixo mostra dois *`Pods`*, cada um com seu próprio IP. Mesmo que um deles seja um *`Pod`* de vários containers, ele ainda obtém apenas um único IP.

<img src="{{ mix('/images/articles/Kubernetes/Pods and shared networking.png') }}" alt="Pods and shared networking">

Na imagem acima, o acesso externo aos containers no *`Pod 1`* é obtido por meio do endereço IP do *`Pod`* acoplado à porta do container que você deseja alcançar. Por exemplo, `10.0.10.15:80` o levará ao container principal (`Main container`). A comunicação entre containers funciona por meio do `localhost` do *`Pod`* e do número da porta. Por exemplo, o container principal pode alcançar o container de suporte (`Supporting container`) via `localhost:5000`.

Uma última vez. Cada container em um *`Pod`* compartilha todo o `namespace` de rede do *`Pod`* - IP, localhost, intervalo de portas, tabela de roteamento e muito mais.

No entanto, como já dissemos, é mais do que apenas networking. Todos os containers em um *`Pod`* têm acesso aos mesmos volumes, à mesma memória, aos mesmos soquetes *IPC* e muito mais.

Esse modelo de rede torna a comunicação entre os *`Pods`* realmente simples. Cada *`Pod`* no cluster tem seus próprios endereços IP que são totalmente roteáveis. Como cada *`Pod`* obtém seu próprio IP roteável, eles podem se comunicar entre si sem a necessidade de mapeamentos de portas.

<img src="{{ mix('/images/articles/Kubernetes/Inter-Pod communication.png') }}" alt="Inter-Pod communication">

Conforme mencionado anteriormente, a comunicação *intra-pod* - onde dois containers no mesmo *`Pod`* precisam se comunicar - pode acontecer por meio da interface `localhost` do *`Pod`*.

<img src="{{ mix('/images/articles/Kubernetes/Intra-Pod communication.png') }}" alt="Intra-Pod communication">

Se você precisa expor vários containers no mesmo *`Pod`* para o mundo externo, poderá expô-los em portas individuais. *Cada container precisa de sua própria porta, e dois containers no mesmo *`Pod`* não podem usar a mesma porta.*

## *Pod* Lifecycle (Ciclo de Vida do *Pod*)

Os *`Pods`* são criados, executam os aplicativos e são encerrados. Se eles encerram inesperadamente, você não os traz de volta a execução. Em vez disso, o *Kubernetes* inicia um novo *`Pod`* em seu lugar. No entanto, embora o novo *`Pod`* tenha a aparência e a sensação do antigo, não é. É um novo *`Pod`* com um ID e endereço IP totalmente novos.

Isso tem implicações sobre como você deve projetar seus aplicativos. Não os projete de forma que fiquem fortemente acoplados a uma instância específica de um *`Pod`*. Em vez disso, projete-os de forma que, quando os *`Pods`* falham, um totalmente novo (com um novo ID e endereço IP) possa surgir em algum outro lugar do cluster e tomar o seu lugar sem problemas.

O ciclo de vida de um *`Pod`* é mais ou menos assim. Você o define em um arquivo de manifesto *YAML* e faz o `POST` do manifesto para o *`API server`*. Uma vez lá, o conteúdo do manifesto é mantido no armazenamento do cluster como um registro de *intent* (estado desejado) e o *`Pod`* é programado para um *nó íntegro* com recursos suficientes. Depois de agendado para um nó, ele entra no estado **pending** enquanto o *container runtime* no nó baixa imagens e inicia todos os containers. O *`Pod`* permanece no estado **pending** até que todos os seus recursos estejam ativos e prontos. Depois que tudo estiver pronto, o *`Pod`* entra no estado de **running**. Depois de concluir todas as suas tarefas, ele é encerrado e entra no estado **succeeded**.

Quando um *`Pod`* não pode ser iniciado, ele pode permanecer no estado **pending** ou vai para o estado de **failed**. Tudo isso é mostrado na imagem abaixo.

<img src="{{ mix('/images/articles/Kubernetes/Pod lifecycle.png') }}" alt="Pod lifecycle">

Para ver quais *`Pods`* estão em execução no cluster, você pode executar `kubectl get pods` para obter os *`Pods`* no namespace `default` do contexto atual ou `kubectl get pods --all-namespaces` para obter os *`Pods`* em todos os namespaces.

A saída de `kubectl get pods` deve ser como da seguinte forma:

```
NAME        READY   STATUS    RESTARTS   AGE
first-pod   1/1     Running   0          15s
```

Como você pode ver, os *`Pods`* têm um valor `STATUS` que diz em qual estado o *`Pod`* está atualmente.

Os valores para o estado do *`Pod`* são os seguintes:

- **Running**: Esse estado significa que o *`Pod`* foi atribuído a um dos *nós do cluster* e pelo menos um dos containers está em execução ou em processo de inicialização.

- **Succeeded**: Se os containers dentro do *`Pod`* estiverem configurados para executar um comando que pode ser concluído ou encerrado (não um comando de longa duração, como iniciar um servidor da web), o *`Pod`* mostrará o estado de *Succeeded* se esses containers concluíram seus processos no comando. Significa que o *`Pod`* foi executado e todos os containers foram encerrados com sucesso.

- **Pending**: Significa que o *`Pod`* foi enviado ao cluster, mas o controlador ainda não criou todos os containers. Pode ser porque o *`Pod`* está aguardando o download das imagens dos containers ou está aguardando para ser agendado pelo `kube-scheduler` em um dos *nós do cluster*.

- **Unknown**: Significa que o *Kubernetes* não pode dizer em qual estado o *`Pod`* realmente está. Isso geralmente significa que o nó em que o *`Pod`* reside está apresentando algum tipo de erro. Ele pode estar sem espaço em disco, desconectado do restante do cluster ou com algum outro problema. Pode ser também devido à incapacidade do controlador de se conectar ao nó ao qual o *`Pod`* foi atribuído. Pode ser também porque o nó está inacessível por algum motivo - o nó pode ter sido desligado, por exemplo, ou está inacessível pela rede.

- **Failed**: Este estado significa que o *`Pod`* foi executado e pelo menos um dos containers foi encerrado com um código de saída diferente de zero, ou seja, não conseguiu executar seus comandos.

Os *`Pods`* que são implantados por meio de arquivos de manifesto são *singletons* - eles não são gerenciados por um controlador que pode adicionar recursos como auto-scaling e recursos de self-healing. Por esse motivo, quase sempre implantamos *`Pods`* por meio de controladores de nível superior, como *Deployments* e *DaemonSets*, pois eles podem reprogramar os *`Pods`* quando falham.

Nesse tópico, é importante pensar nos *`Pods`* como mortais. Quando eles morrem, eles se vão. Não há como trazê-los de volta dos mortos.

Esse é um dos principais motivos pelos quais você deve projetar seus aplicativos de forma que não armazenem o estado nos *`Pods`*. É também por isso que não devemos confiar nos IPs individuais dos *`Pod`*. *Singleton* *`Pods`* não são confiáveis!

## [Implementação dos *Pods*](https://kubernetes.io/docs/concepts/workloads/pods/)

*Os `Pods` são implementados usando princípios de isolamento do Linux, como cgroups e namespaces*, e geralmente podem ser considerados uma *máquina host lógica*. *Os `Pods` executam um ou mais containers* (que podem ser baseados em Docker, CRI-O ou outros runtimes) e esses containers podem se comunicar entre si da mesma maneira que diferentes processos em uma VM podem se comunicar.

Para que os containers em dois *`Pods`* diferentes se comuniquem, eles precisam acessar o outro *`Pod`* (e o container) por meio de seu IP. **Por padrão, apenas os containers em execução no mesmo *`Pod`* podem usar métodos de comunicação lower-level**.

### Paradigma do *Pod*

No nível mais básico, existem dois tipos de Pods:

- *Pods com um único container*
- *Pods com vários containers*

Geralmente, é uma prática recomendada ter um único container por *`Pod`*. Essa abordagem permite dimensionar as diferentes partes do aplicativo separadamente e mantém as coisas mais simples quando se trata em criar um *`Pod`* que é executado e manipulado sem problemas.

Os *`Pods`* com vários containers, por outro lado, são mais complexos, mas podem ser úteis em várias circunstâncias:

- *Se houver várias partes do aplicativo que são executadas em containers separados, mas estão fortemente acopladas, você pode executá-las dentro do mesmo `Pod` para tornar a comunicação e o acesso ao sistema de arquivos sem interrupções.*
- *Ao implementar o padrão `sidecar`, onde containers de suporte são injetados junto com o container do aplicativo principal para lidar com registros, métricas, rede ou outras funcionalidades avançadas.*

A imagem a seguir mostra uma implementação de *common sidecar*:

<img src="{{ mix('/images/articles/Kubernetes/Common sidebar implementation.png') }}" alt="Common sidebar implementation">

Na imagem acima, temos um único *`Pod`* com dois containers: o container do aplicativo executando um servidor da web e um aplicativo de logging que extrai os logs do *`Pod`* do servidor e encaminha para a infraestrutura de logging. Esse é um uso muito aplicável do *sidecar pattern*.

### Namespaces

> Os *namespaces* é **uma forma de separar logicamente diferentes áreas do cluster**. Um caso de uso comum é ter um namespace por ambiente - um para *dev*, outro para *teste*, outro para *produção* - todos estando dentro do mesmo cluster.

É possível especificar as permissões do usuário por *namespace* - por exemplo, permitindo que um usuário implante novos aplicativos e recursos no namespace *dev*, mas não em *produção*.

Para ver todos os namespaces existente no cluster, execute `kubectl get namespaces` ou `kubectl get ns` e veja o resultado como da seguinte forma:

```
NAME              STATUS   AGE
default           Active   4m49s
kube-node-lease   Active   4m51s
kube-public       Active   4m51s
kube-system       Active   4m51s
```

### [Noções básicas sobre as especificações do *Pod*](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/#Pod)

Para poder configurar um *`Pod`* com êxito, primeiro devemos ser capazes de ler e entender um arquivo de configuração - manifesto *YAML* do *`Pod`*. Aqui está um exemplo do arquivo de configuração de um *`Pod`*:

```yml
apiVersion: v1
kind: Pod
metadata:
    name: first-pod
    namespace: dev
    labels:
        environment: dev
    annotations:
        userId1: 1234as
        userId2: 9876df
spec:
    containers:
    - name: my-first-container
      image: nginx
```

Podemos dividir a configuração de um *`Pod`* em quatro componentes principais:

- **apiVersion**: *Versão da API Kubernetes*. Informa ao *Kubernetes* qual versão da *API* deve ser examinada ao criar e configurar um recurso/workloads.

- **kind**: O tipo do recurso/objeto no Kubernetes que deve ser criado.

- **metadata**: *Metadados* ou informações que identificam exclusivamente o objeto que está sendo criando. *`name`* é o nome do recurso, que é como o recurso será exibido por meio do *`kubectl`* e como será armazenado no *`etcd`*. *`namespace`* corresponde ao namespace em que o recurso deve ser criado. Se nenhum namespace for especificado, o recurso será criado no namespace *`default`* - a menos que exista um argumento *`--namespace`* nos comandos *`apply`*. Os *`labels`* são pares de chave-valor, são informções especiais em comparação com outros metadados porque são usados por padrão nos seletores do Kubernetes para filtrar e selecionar os *workloads*. *`annotations`* assim como labels, podem ser usadas pelos controladores para fornecer configuração adicional e dados específicos dos recursos. *Geralmente, é melhor usar labels para as funcionalidade e seletores específicos para os controladores e usar annotations para adicionar dados - isso é apenas uma convenção.*

- **spec**: Chave que contém a configuração específica do recurso. Nesse caso, como o valor do `kind` é *`Pod`*, as configurações são específicas do *`Pod`*.

#### Especificações de recurso do *Pod*

Os *`Pods`* podem ser configurados para ter quantidades específicas de [**memória**](https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource/) e [**CPU**](https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/) alocada a eles. Isso evita que os aplicativos que consumem muitos recursos afetem o desempenho do cluster e também pode ajudar a evitar vazamentos de memória. Existem dois recursos possíveis que podem ser especificados - `cpu` e `memory`. Para cada um deles, existem dois tipos diferentes de especificações, `Requests` e `Limits`, para um total de quatro chaves de especificação de recursos possíveis.

As solicitações (`requests`) e limites (`limits`) da **CPU** podem ser configurados usando `m`, que corresponde a 1 *milli-CPU*, ou apenas usando um número decimal. Portanto, `200m` é equivalente a `0,2`, o que equivale a *20%* ou um *quinto de uma CPU lógica*. Essa quantidade será a mesma quantidade de computação, independentemente do número de cores. *1 CPU* é igual a um núcleo virtual no AWS ou um núcleo no GCP. Vejamos como esses recursos de solicitações e limites aparecem no código abaixo:

```yml
apiVersion: v1
kind: Pod
metadata:
    name: my-app
spec:
    containers:
    - name: my-app-container
      image: busybox
      resources:
          requests:
              memory: "50Mi"
              cpu: "100m"
          limits:
              memory: "200Mi"
              cpu: "500m"
```

Na especificação do *`Pod`* acima, temos um container executando uma imagem Docker que é especificada com `requests` e `limits` de CPU e memória.

1. `resources.limits.cpu`: *Este é o limite de recursos para uso da CPU.*
2. `resources.limits.memory`: *Este é o limite de recursos para uso da memória.*
3. `resources.requests.cpu`: *Este é o uso mínimo de CPU solicitado para colocar seu aplicativo em funcionamento.*
4. `resources.requests.memory`: *Este é o uso mínimo de memória solicitada para colocar seu aplicativo em funcionamento.*

#### Compreender como as solicitações de recursos afetam o agendamento

Ao especificar as solicitações de recursos, você especifica a quantidade mínima de recursos de que seu *`Pod`* precisa. Essas informações são usadas pelo *Scheduler* ao agendar o *`Pod`* para um nó. Cada nó tem uma certa quantidade de CPU e memória que pode ser usado para alocar nos *`Pods`*. Ao programar um *`Pod`*, o *Scheduler* considerará apenas nós com recursos não alocados suficientes para atender aos requisitos de recursos do *`Pod`*. Se a quantidade de CPU ou memória não alocada for menor do que `requests` do *`Pod`*, o *Kubernetes* não agendará o *`Pod`* para esse nó, porque o nó não pode fornecer a quantidade mínima exigida pelo *`Pod`*.

#### Entendendo como o *Scheduler* determina se um *Pod* pode ser agendado para um nó

O que é importante e um tanto surpreendente é que o *Scheduler* não olha quanto de cada recurso individual está sendo usado no momento exato do agendamento, mas na soma dos recursos solicitados (`requests`) pelos *`Pods`* existentes implantados no nó. Mesmo que os *`Pods`* existentes possam estar usando menos do que o solicitado (`requests`), programar outro *`Pod`* com base no consumo real de recursos quebraria a garantia dada aos *`Pods`* já implantados.

Isso é visualizado na imagem abaixo. Três *`Pods`* são implantados no nó. Juntos, eles solicitaram *80%* da CPU do nó e *60%* da memória. O *`Pod D`*, mostrado na parte inferior direita da imagem, não pode ser agendado no nó porque requer *25%* da CPU, o que é mais do que *20%* da CPU não alocada. O fato de os três *`Pods`* estarem usando atualmente apenas 70% da CPU não faz diferença.

<img src="{{ mix('/images/articles/Kubernetes/The Scheduler only cares about requests, not actual usage.png') }}" alt="The Scheduler only cares about requests, not actual usage">

#### Configurando aplicativos com containers *init*

> Os containers `init` são containers especiais dentro de um *`Pod`*. Eles são iniciados, executados e encerrados antes do início dos containers normais do *`Pod`*.

Os containers `init` podem ser usados ​​para muitos casos de uso diferentes, como inicializar arquivos antes que um aplicativo seja iniciado ou garantir que outros aplicativos ou serviços estejam em execução antes de iniciar um *`Pod`*.

Se vários containers `init` forem especificados, eles serão executados em ordem, em sequência até que todos os containers `init` sejam encerrados, na ordem em que foram gravados na especificação do *`Pod`*. Por esse motivo, os containers `init` devem executar um script que seja concluído para que os containers normais seja iniciados e sigam o ciclo de vida dos mesmos. Se o aplicativo ou script do container `init` continuar em execução, os containers normais no *`Pod`* não serão iniciados.

Cada container `init` precisa ser concluído com sucesso antes que o próximo comece, e todos devem ser concluídos com sucesso antes que os containers do *`Pod`* sejam iniciados. A imagem abaixo mostra a sequência de inicialização de um *`Pod`* com containers `init`.

<img src="{{ mix('/images/articles/Kubernetes/Init containers are useful for startup tasks to prepare the Pod for the app containers.png') }}" alt="Init containers are useful for startup tasks to prepare the Pod for the app containers">

No *`Pod`* a seguir, o container `init` está executando um loop para verificar se `config-service` existe por meio do `nslookup`. Depois de ver que `config-service` está ativo, o script termina, o que aciona o início do container `my-app`:

```yml
apiVersion: v1
kind: Pod
metadata:
    name: my-app
spec:
    containers:
    - name: my-app-container
      image: busybox
      command: ["run"]
    initContainers:
    - name: init-before
      image: busybox
      command: ['sh', '-c', 'until nslookup config-service; do echo config-service not up; sleep 2; done;']
```

**Nota:** Quando um container `init` falha, o *Kubernetes* reinicia automaticamente o *`Pod`*, semelhante à funcionalidade de inicialização normal do *`Pod`*. Essa funcionalidade pode ser alterada alterando a configuração de `restartPolicy`.

A imagem do diagrama abaixo mostra o fluxo típico de inicialização do *`Pod`* no Kubernetes:

<img src="{{ mix('/images/articles/Kubernetes/Init container flowchart.png') }}" alt="Init container flowchart">

Se um *`Pod`* tiver mais de um `initContainers`, eles serão executados sequencialmente. Isso é importante quando os `initContainers` com etapas modulares são executados em ordem. O seguinte *YAML* mostra isso:

```yml
apiVersion: v1
kind: Pod
metadata:
    name: my-app
spec:
    containers:
    - name: my-app-container
      image: busybox
      command: ["run"]
    initContainers:
    - name: init-step-1
      image: step1-image
      command: ['start-command']
    - name: init-step-2
      image: step2-image
      command: ['start-command']
```

No código *YAML* do *`Pod`* acima, o container `init` da etapa 1 precisa ser bem-sucedido antes que o container `init` da etapa 2 seja iniciado e ambos precisam mostrar sucesso antes que o container `my-app-container` seja iniciado.

*Init containers are useful for preparing the Pod environment for app and sidecar containers.*

#### [Apresentando diferentes tipos de sondagens no Kubernetes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

Para saber quando um container (e, portanto, um *`Pod`*) falhou, o *Kubernetes* precisa saber como testar se o container está funcionando. Isso é definindo usando **probes**, que o *Kubernetes* pode executar em um intervalo especificado para determinar se o container está funcionando.

Assim que um *`Pod`* é agendado para um *nó*, o *Kubelet* nesse nó executará seus containers e, a partir de então, os manterá em execução enquanto o *`Pod`* existir. Se o processo principal do container falhar, o *Kubelet* irá reiniciar o container. Se o seu aplicativo tiver um bug que causa uma falha de vez em quando, o Kubernetes irá reiniciá-lo automaticamente, portanto, mesmo sem fazer nada de especial no próprio aplicativo, rodar o aplicativo no *Kubernetes* automaticamente lhe dá a habilidade de manter-se funcionando.

Mas às vezes os apps param de funcionar sem que o processo falhe. Seria ótimo ter uma maneira de um aplicativo sinalizar para o *Kubernetes* que ele não está mais funcionando adequadamente e que o *Kubernetes* deve reiniciá-lo.

Existem três tipos de *probes* que o Kubernetes permite configurar: **Readiness**, **Liveness** e **Startup**.

#### Readiness probes - Sinalizando quando um *Pod* está pronto para aceitar solicitações

*Readiness probes* podem ser usadas para determinar se um container está pronto para executar sua função/responsabilidade, como aceitar tráfego via *HTTP*. Esses testes são úteis nos estágios iniciais de um aplicativo em execução, onde ele ainda pode estar iniciando a configuração, por exemplo, e ainda não estar pronto para aceitar as solicitações.

*Readiness probes* é invocada periodicamente e determina se o *`Pod`* específico deve receber solicitações do *client* ou não. Quando a análise do probe de um container retorna sucesso, está sinalizando que o container está pronto para aceitar solicitações.

O código YAML abaixo mostra um `PodSpec` com uma *Readiness probes*:

```yml
apiVersion: v1
kind: Pod
metadata:
    name: my-app
spec:
    containers:
    - name: my-app-container
      image: busybox
      ports:
          - containerPort: 8080
      readinessProbe:
          exec:
              command:
              - cat
              - /tmp/thisfileshouldexist.txt
          initialDelaySeconds: 5
          periodSeconds: 5
```

*As sondas (probes) são definidas por container, não por `Pod`. O Kubernetes executará todas as sondagens por container e usará isso para determinar a integridade total do `Pod`*.

*`initialDelaySeconds=0` diz que a sondagem começa imediatamente após o container ser iniciado.*

Se você não definir o atraso inicial (`initialDelaySeconds`), o *probe* começará a sondar o container assim que for iniciado, o que geralmente leva à falhas, porque o aplicativo não está pronto para começar a receber solicitações. Se o número de falhas exceder o limite de falha (`failureThreshold`), o container será reiniciado antes mesmo de começar a responder às solicitações de maneira adequada.

##### Entendendo o funcionamento de *Readiness probes*

Quando um container é iniciado, o *Kubernetes* pode ser configurado para aguardar um período de tempo antes de executar a primeira verificação de *readiness*. Depois disso, ele invoca o *readiness* periodicamente e age com base no resultado do *readiness probes*. Se um *`Pod`* relatar que não está pronto, ele é removido do *Service*. Se o *`Pod`* ficar pronto novamente, ele será adicionado ao *Service*.

Ao contrário de *liveness probes*, se um container falhar na verificação do *readiness*, ele não será eliminado ou reiniciado. Esta é uma distinção importante entre os *liveness and readiness probes*. Os *liveness probes* mantêm os *`Pods`* saudáveis ​​eliminando os containers não íntegros e substituindo-os por novos e íntegros, enquanto os *readiness probes* garantem que apenas os *`Pods`* que estão prontos para atender às solicitações estejam realmente recebendo juntamente com o *Service*. Isso é principalmente necessário durante a inicialização do container, mas também é útil depois que o container estiver em execução.

Como você pode ver na imagem abaixo, se o *readiness probes* de um *`Pod`* falhar, o *`Pod`* será removido do objeto `Endpoints`. Os *clients* que se conectam ao *Service* não serão redirecionados ao *`Pod`*. O efeito é o mesmo de quando o *`Pod`* não corresponde ao seletor da label do *Service*.

<img src="{{ mix('/images/articles/Kubernetes/A pod whose readiness probe fails is removed as an endpoint of a service.png') }}" alt="A pod whose readiness probe fails is removed as an endpoint of a service">

##### Entendendo por que os *Readiness probes* são importantes

Imagine que um grupo de *`Pods`* (por exemplo, *`Pods`* executando servidores de aplicativos) depende de um serviço fornecido por outro *`Pod`* (um banco de dados, por exemplo). Se, em qualquer ponto, um dos *`Pods`* de `front-end` tiver problemas de conectividade e não puder mais acessar o banco de dados, pode ser aconselhável que sua *readiness probes* sinalize ao *Kubernetes* que o *`Pod`* não está pronto para atender a nenhuma solicitação naquele momento. Se outras instâncias do *`Pod`* não apresentarem o mesmo tipo de problemas de conectividade, elas poderão atender às solicitações normalmente. Uma *Readiness probes* garante que os *clients* falem apenas com os *`Pods`* íntegros e nunca percebam que há algo de errado com o sistema.

#### Liveness probes

*Liveness probes* podem ser usadas para determinar se um aplicativo falhou por algum motivo (por exemplo, devido a um erro de memória). Para containers de aplicativos que são executados por muito tempo, *liveness probes* podem ser úteis como um método para ajudar o *Kubernetes* a reciclar os *`Pods`* antigos e falhos para novos. Embora as sondagens por si mesmas não façam com que um container seja reiniciado, outros recursos e controladores do *Kubernetes* verificarão o status da sondagem e o usarão para reiniciar os *`Pods`* quando necessário. Abaixo está um `PodSpec` com uma definição de *liveness probes* anexada a ele:

```yml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
    containers:
    - name: my-app-container
      image: busybox
      ports:
          - containerPort: 8080
      livenessProbe:
          exec:
              command:
              - cat
              - /tmp/thisfileshouldexist.txt
          initialDelaySeconds: 5
          failureThreshold: 2         # Allow two probes to fail before taking action.
          periodSeconds: 5
```

No código acima, *liveness probes* é especificado da mesma maneira que *readiness probes*, com uma adição - `failureThreshold`.

O valor de `failureThreshold` determinará quantas vezes o *Kubernetes* tentará o *health* antes de tomar alguma ação. Para *liveness probes*, o *Kubernetes* reiniciará o *`Pod`* assim que o valor de `failureThreshold` for superado. Para *readiness probes*, o *Kubernetes* simplesmente marcará o *`Pod`* como `Not Ready`. O valor padrão para este limite é `3`, mas pode ser alterado para qualquer valor maior ou igual a `1`.

#### Startup probes

*Startup probes* é um tipo especial de teste que *será executado apenas uma vez, na inicialização do container*. Abaixo está um `PodSpec` com uma definição de *startup probes* anexada a ele:

```yml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
    containers:
    - name: my-app-container
      image: busybox
      ports:
          - containerPort: 8080
      startupProbe:
          exec:
              command:
              - cat
              - /tmp/thisfileshouldexist.txt
          initialDelaySeconds: 15  # Kubernetes will wait 15 seconds before executing the first probe.
          successThreshold: 2
          periodSeconds: 5
```

*Startup probes* fornecem um benefício maior do que simplesmente estender o tempo entre *liveness* ou *readiness probes* - permite que o *Kubernetes* mantenha um tempo de reação ao resolver problemas que acontecem após a inicialização e (mais importante) para evitar que aplicativos de inicialização lenta reiniciem constantemente. Se o aplicativo levar muitos segundos ou até um minuto ou dois para inicializar, será muito mais fácil implementar um *startup probe*.

`successThreshold` é o contrário de `failureThreshold`. Ele especifica quantos sucessos são necessários antes que um container seja marcado como `Ready`.

#### Configuração do mecanismo Probe

Existem vários mecanismos que pode ser especificado em qualquer um dos três probes: `exec`, `httpGet` e `tcpSocket`.

O método `exec` permite que você especifique um comando que será executado dentro do container. Um comando executado com sucesso resultará em um probe aprovado, enquanto um comando falho resultará em uma falha no probe. Se o código de status for `0`, o probe foi bem-sucedido. Todos os outros códigos são considerados falhos. O método `httpGet` permite especificar uma URL no container que será executada com uma solicitação `HTTP GET`. Se a solicitação HTTP retornar um código de resposta de `2xx` ou `3xx`, isso resultará em um probe bem-sucedido. Qualquer outro código HTTP resultará em falha.

A configuração para `httpGet` é semelhante a esta:

```yml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
    containers:
    - name: my-app-container
      image: busybox
      ports:
          - containerPort: 8080
      livenessProbe:                  # A liveness probe that will perform an HTTP GET
          httpGet:
              path: /healthcheck
              port: 8001
              httpHeaders:
              - name: My-Header
                value: My-Header-Value
          initialDelaySeconds: 10     # Wait 10 seconds before running the first probe.
          periodSeconds: 5            # The probe fires every five seconds.
```

O código *YAML* do *`Pod`* acima define uma *liveness probe* `httpGet`, que informa ao *Kubernetes* para executar solicitações HTTP GET periodicamente no caminho `/healthcheck` na porta `8001` para determinar se o container ainda está íntegro. Essas solicitações começam assim que o container é executado.

Finalmente, o método `tcpSocket` tentará abrir o soquete especificado no container e usará o resultado para determinar o *`Pod`* como sucesso ou falha. A configuração `tcpSocket` é assim:

```yml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
    containers:
    - name: my-app-container
      image: busybox
      ports:
          - containerPort: 8080
      readinessProbe:
          tcpSocket:
              port: 8080
          initialDelaySeconds: 5
          periodSeconds: 10
```

## Organização dos *Pods* Usando Labels

### Listando subconjuntos dos *Pods* por meio de seletores de labels

Anexar *labels* aos recursos para que você possa vê-los ao listá-los não é tão interessante. Mas os labels andam de mãos dadas com os **label selectors**. Os *label selectors* permitem que você selecione um subconjunto de *`Pods`* marcados com determinadas labels e realize alguma operação nesses *`Pods`*. Um *label selector* é um critério, que filtra recursos com base na inclusão de um determinado label com um determinado valor.

Um *label selector* pode selecionar recursos que:

- *Contém (ou não contém) um label com uma determinada chave;*
- *Contém um label com uma determinada chave e valor;*
- *Contém um label com uma determinada chave, mas com um valor diferente do que você especifica;*

Para listar todos os *`Pods`* com seus *labels*:

```bash
kubectl get po --show-labels
```

Em vez de listar todos os *labels*, se estiver interessado apenas em determinados *labels*, você poderá especificá-los com a opção `-L` e exibir cada um deles em sua própria coluna:

```bash
kubectl get po -L label_name_1,label_name_2
```

Para ver todos os *`Pods`* que contêm determinado *label* com determinado valor:

```bash
kubectl get po -l label=value
```

Para listar todos os *`Pods`* que incluem o label `env`, qualquer que seja seu valor:

```bash
kubectl get po -l env
```

E aqueles que não têm o label `env`:

```bash
kubectl get po -l `!env`
```

Da mesma forma, você também pode combinar os *label selectors* dos *`Pods`* como da seguinte forma:

- `creation_method!=manual` para selecionar os *`Pods`* com o label `creation_method` com qualquer valor diferente de `manual`
- `env in (prod, devel) `para selecionar *`Pods`* que tenha o label `env` com um dos valores `prod` ou `devel`
- `env notin (prod, devel)` para selecionar *`Pods`* com o label `env` para qualquer valor diferente de `prod` ou `devel`

### Usando labels e seletores para restringir o agendamento dos *Pods*

Como o *Kubernetes expõe todos os nós no cluster como uma única plataforma de implantação*, não importa para qual nó um *`Pod`* está agendado. Como cada *`Pod`* obtém a quantidade exata de recursos computacionais solicitados (CPU, memória e assim por diante) e sua acessibilidade de outros *`Pods`* não é de forma alguma afetada pelo nó no qual o *`Pod`* está programado, geralmente não deve haver necessidade para você dizer ao *Kubernetes* exatamente onde agendar seus *`Pods`*.

Certos casos existem, no entanto, onde você vai especificar onde um *`Pod`* deve ser agendado. Um bom exemplo é quando sua infraestrutura de hardware não é homogênea. Se parte de seus nós de trabalho tiver unidades de disco rígido sem SSD, enquanto outras têm SSDs, você poderá programar determinados *`Pods`* para um grupo de nós e o restante para o outro. Outro exemplo é quando você precisa programar *`Pods`* que realizam computação intensiva baseada em GPU apenas para nós que fornecem a aceleração de GPU necessária.

Você nunca deve dizer especificamente para qual nó um *`Pod`* deve ser programado, porque isso acoplaria o aplicativo à infraestrutura, enquanto a ideia do *Kubernetes* é ocultar a infraestrutura real dos aplicativos executados nela. Mas se você quiser programar para onde um *`Pod`* deve ser agendado, em vez de especificar exatamente um nó, você deve descrever os requisitos do nó e deixar o *Kubernetes* selecioná-lo com base na correspondência dos requisitos. Isso pode ser feito por meio de *labels do nó* e *label selectors* de nó.

### Usando *labels* para categorizar os nós de trabalho (*worker nodes*)

Os *`Pods`* não são o único tipo de recurso do *Kubernetes* ao qual você pode anexar uma label. Os labels podem ser anexados a qualquer objeto do *Kubernetes*, incluindo nodes(nós). Geralmente, quando a equipe de infra adiciona um novo nó ao cluster, eles categorizam o nó anexando labels especificando o tipo de hardware que o nó fornece ou qualquer outra coisa que possa ser útil ao agendar os *`Pods`*.

Vamos imaginar que um dos nós no seu cluster contenha GPU destinada a ser usada para computação em geral. Você deseja adicionar um label ao nó que mostra esse recurso. Você vai adicionar o label `gpu=true` a um de seus nós.

```bash
kubectl label node node-name gpu=true
```

Agora você pode usar um *label selectors* ao listar os nós, como fez antes com os *`Pods`*. Listar apenas nós que incluam o label `gpu=true`:

```bash
kubectl get nodes -l gpu=true
```

### Agendamento dos *Pods* para nós específicos

Agora imagine que você deseje implantar um novo *`Pod`* que precisa de uma GPU para realizar seu trabalho. Para pedir ao *scheduler* para escolher apenas entre os nós que fornecem uma GPU, você adicionará um **seletor de nó** ao *YAML* do *`Pod`*.

```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-name
spec:
  nodeSelector: # nodeSelector tells Kubernetes to deploy this pod only to nodes containing the `gpu=true` label.
    gpu: "true"
  containers:
  - name: container-name
    image: polinux/stress
```

Você adiciona um campo `nodeSelector` na seção `spec`. Quando você cria o *`Pod`*, o *scheduler* só escolhe entre os nós que contêm o label `gpu=true`.

## Usando Namespaces Para Agrupar Recursos

Vamos voltar aos *labels* por um momento. Vimos como eles organizam os *`Pods`* e outros objetos em grupos. Como cada objeto pode ter vários *labels*, esses grupos de objetos podem se sobrepor. Além disso, ao trabalhar com o cluster (por meio do `kubectl`, por exemplo), se você não especificar explicitamente um *label selector*, sempre verá todos os objetos.

Mas *e quanto às vezes em que você deseja dividir objetos em grupos separados e sem sobreposição?* Você pode querer operar somente dentro de um grupo de cada vez. Por essa e outras razões, o *Kubernetes* também agrupa objetos em **namespaces**. Estes não são os *namespaces* do Linux que são usados ​​para isolar processos uns dos outros. **Namespaces do *Kubernetes* fornecem um escopo para os nomes dos objetos**. Em vez de ter todos os seus recursos em um único **namespace**, é possível dividi-los em vários *namespaces*, o que também permite usar os mesmos nomes de recursos várias vezes (em namespaces diferentes).

### Compreendendo a necessidade de *namespaces*

O uso de vários **namespaces** *permite dividir sistemas complexos com vários componentes em grupos distintos menores*. *Eles também podem ser usados ​​para separar recursos em um ambiente de vários tenants, dividindo recursos em ambientes de produção, desenvolvimento e controle de qualidade ou de qualquer outra maneira que você possa precisar*. **Os nomes de recursos precisam ser exclusivos em um namespace**. Dois namespaces diferentes podem conter recursos com o mesmo nome. Mas, enquanto a maioria dos tipos de recursos é *namespaced*, alguns não são. Um deles é o recurso **Node**, que é global e não está vinculado a um único *namespace*.

*Além de isolar recursos, os namespaces também são usados ​​para permitir que apenas certos usuários acessem recursos específicos e até mesmo para limitar a quantidade de recursos computacionais disponíveis para usuários individuais.*

### Criando um namespace

Um namespace é um recurso do *Kubernetes* como qualquer outro, portanto, você pode criá-lo publicando um arquivo *YAML* no *`API server`* do Kubernetes. Vamos ver como fazer isso agora.

```yml
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Namespace
metadata:
  name: custom-namespace
EOF
# namespace "custom-namespace" created
```

### Gerenciando objetos em outros namespaces

Para criar recursos no namespace que você criou, adicione `namespace: custom-namespace` à seção de `metadata` ou especifique o namespace ao criar o recurso com o comando `kubectl apply`:

```bash
kubectl apply -f pod-name.yaml -n custom-namespace
```

Ao listar, descrever, modificar ou excluir objetos em outros namespaces, você precisa passar o sinalizador `--namespace` (ou `-n`) para `kubectl`. Se você não especificar o namespace, o kubectl executa a ação no namespace `default` configurado no contexto atual do `kubectl`. O namespace do contexto atual e o próprio contexto atual podem ser alterados através dos comandos `kubectl config`.

_Para alternar rapidamente para um namespace diferente, você pode configurar o seguinte alias: `alias kcd='kubectl config set-context $(kubectl config currentcontext) --namespace '`. Você pode alternar entre namespaces usando `kcd some-namespace`._
