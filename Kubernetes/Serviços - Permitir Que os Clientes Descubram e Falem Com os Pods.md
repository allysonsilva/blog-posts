---
id: 9c456d18-413f-4307-87d7-87bbbb175486
title: Serviços - Permitir Que os Clientes Descubram e Falem Com os Pods
summary: "Um Service no Kubernetes é um recurso que você cria para ter um ponto de entrada único para um grupo de Pods que fornecem o mesmo serviço"
---

Os *`Pods`* precisam encontrar os outros *`Pods`* se quiserem consumir os serviços que fornecem. Ao contrário do mundo não *Kubernetes*, onde um administrador de sistemas configuraria cada aplicativo *client* especificando o *endereço IP* ou *nome do host* exato do servidor que fornece o serviço nos arquivos de configuração do *client*, fazer o mesmo no *Kubernetes* não funcionaria, porque:

- **Os *`Pods`* são efêmeros** - eles podem ir e vir a qualquer momento, seja pelo motivo de reduzir o número de *`Pods`* no nó ou porque um nó do cluster falhou.

- O *Kubernetes* atribui um *endereço IP* a um *`Pod`* depois que o *`Pod`* foi agendado para um nó e antes de ser iniciado - os *clients*, portanto, não podem saber o *endereço IP* do *`Pod`* do servidor antecipadamente.

- O **dimensionamento horizontal** significa que vários *`Pods`* podem fornecer o mesmo serviço - cada um desses *`Pods`* tem seu próprio *endereço IP*. Os *clients* não devem se preocupar com quantos *`Pods`* estão executando o serviço e quais são seus IPs. Eles não deveriam ter que manter uma lista de todos os IPs individuais dos *`Pods`*. Em vez disso, todos esses *`Pods`* devem ser acessíveis por meio de um único *endereço IP*.

Para resolver esses problemas, o *Kubernetes* também fornece outro tipo de recurso - **Services**.

## Introdução

> Um *Service Kubernetes* **é um recurso que você cria para ter um ponto de entrada único e constante para um grupo de `Pods` que fornecem o mesmo serviço**. Cada *Service* tem um endereço IP e uma porta que nunca mudam enquanto o *Service* existe. Os *clients* podem abrir conexões para aquele IP e porta, e essas conexões são então roteadas para um dos *`Pods`* que dão suporte a esse *Service*. Dessa forma, os *clients* de um *Service* não precisam saber a localização dos *`Pods`* individuais, permitindo que esses *`Pods`* sejam movidos pelo cluster a qualquer momento.

## Explicando Com Um Exemplo

Ao criar um *Service* para os *`Pods`* de `front-end` e configurá-lo para ser acessível de fora do cluster, você expõe um único *endereço IP* constante por meio do qual os *clients* externos podem se conectar aos *`Pods`*. Da mesma forma, ao criar um *Service* para o *`Pod`* de `back-end`, você cria um endereço estável para o *`Pod`* de `back-end`. O endereço do *Service* não muda, mesmo que o *endereço IP* do *`Pod`* seja alterado. Além disso, ao criar o *Service*, você também permite que os *`Pods`* de `front-end` encontrem facilmente o *Service* de `back-end` por meio do DNS interno do cluster *Kubernetes*. Todos os componentes do sistema (os dois *Services*, os dois conjuntos de *`Pods`* que dão suporte a esses *Services* e as interdependências entre eles) são mostrados na imagem abaixo.

![Both internal and external clients usually connect to pods through services](/images/articles/Kubernetes/Both%20internal%20and%20external%20clients%20usually%20connect%20to%20pods%20through%20services.png?id=b8f6c4baa8f8ab6213328f44660cbc88 "Both internal and external clients usually connect to pods through services")

## Manipulação

O seguinte *YAML* pode ser usado para criar um *Service*:

```yml
apiVersion: v1
kind: Service
metadata:
  name: app
spec:
  ports:
  - port: 80          # The port this service will be available on
    targetPort: 8080  # The container port the service will forward to
  selector:           # All pods with the foo=bar label will be part of this service
    foo: bar
```

![Label selectors determine which pods belong to the Service](/images/articles/Kubernetes/Label%20selectors%20determine%20which%20pods%20belong%20to%20the%20Service.png?id=6e4a3175879ff55f93ad01a1d8e74625 "Label selectors determine which pods belong to the Service")

### Exposição de múltiplas portas no mesmo *Service*

Seu *Service* expõe apenas uma única porta, mas os *Services* também podem oferecer suporte a várias portas. Por exemplo, se seus *`Pods`* escutassem em duas portas - digamos `8080` para HTTP e `8443` para HTTPS - você poderia usar um único *Service* para encaminhar as portas `80` e `443` para as portas `8080` e `8443` do `Pod`. Não é necessário criar dois *Services* diferentes em tais casos. O uso de um único *Service* de múltiplas portas expõe todas as portas do *Service* por meio de um único IP do cluster.

*NOTA: Ao criar um Service com várias portas, você deve especificar um nome para cada porta.*

```yml
spec:
  ports:
    - name: http
      port: 80           # Port 80 is mapped to the pods’ port 8080
      targetPort: 8080
    - name: https
      port: 443          # Port 443 is mapped to pods’ port 8443
      targetPort: 8443
  selector:
    foo: bar            # The label selector always applies to the whole service
```

### Exposição de *Services* a clients externos

![Exposing a service to external clients](/images/articles/Kubernetes/Exposing%20a%20service%20to%20external%20clients.png?id=1fde71da907c2e7a0a1f6a61d984e413 "Exposing a service to external clients")

Existem algumas maneiras de tornar um *Service* acessível externamente:

- Configurando o tipo do *Service* para `NodePort` - Para um *Service* `NodePort`, cada nó do cluster abre uma porta no próprio nó (daí o nome) e redireciona o tráfego recebido nessa porta para o *Service* subjacente. O *Service* não está acessível apenas no IP e na porta do cluster interno, mas também por meio de uma porta dedicada em todos os nós.

- Definir o tipo de *Service* para `LoadBalancer`, uma extensão do tipo `NodePort` - isso torna o *Service* acessível por meio de um **balanceador de carga dedicado**, provisionado a partir da infraestrutura em nuvem na qual o *Kubernetes* está sendo executado. O *balanceador de carga* redireciona o tráfego para a porta do nó em todos os nós. Os *clients* se conectam ao *Service* por meio do *IP do balanceador de carga*.

- Criação de um recurso `Ingress`, um mecanismo diferente para expor vários *Services* por meio de um *único endereço IP* - opera no nível HTTP (camada de rede 7) e pode, portanto, oferecer mais recursos do que os *Services* da camada 4.

### Usando `NodePort`

O primeiro método de expor um conjunto de *`Pods`* a *clients externos* é criar um *Service* e definir seu tipo como `NodePort`. Ao criar um *Service* `NodePort`, você faz com que o *Kubernetes* reserve uma porta em todos os seus nós (o mesmo número de porta é usado em todos eles) e encaminha as conexões de entrada para os *`Pods`* que fazem parte do *Service*.

Isso é semelhante a um *Service* regular (seu tipo real é `ClusterIP`), mas um *Service* `NodePort` pode ser acessado não apenas por meio do *IP do cluster* interno do *Service*, mas também por meio de qualquer IP e porta reservada do nó.

#### Criando um Service de `NodePort`

Agora, você criará um *Service* `NodePort` para ver como pode usá-lo. O código a seguir mostra o *YAML* para o *Service*.

```yml
apiVersion: v1
kind: Service
metadata:
  name: my-service          # The name of a Service is used as the DNS domain name
spec:
  type: NodePort            # Set the service type to NodePort
  ports:
  - port: 80                # This is the port of the service’s internal cluster IP
    targetPort: 8080        # This is the target port of the backing pods
    nodePort: 30123         # The service will be accessible through port 30123 of each of your cluster nodes
  selector:
    app: app-service        # Matches all Pods with an app label set to app-service
```

Você define o tipo como `NodePort` e especifica a porta do nó à qual este *Service* deve ser vinculado em todos os nós do cluster. Especificar a porta não é obrigatório; O *Kubernetes* escolherá uma porta aleatória se você omiti-la.

A imagem abaixo mostra seu *Service* exposto na porta `30123` de ambos os nós do cluster. Uma conexão de entrada para uma dessas portas será redirecionada para um *`Pod`* selecionado aleatoriamente, que pode ou não ser aquele em execução no nó ao qual a conexão está sendo feita.

![An external client connecting to a NodePort service either through Node 1 or 2](/images/articles/Kubernetes/An%20external%20client%20connecting%20to%20a%20NodePort%20service%20either%20through%20Node%201%20or%202.png?id=8cddbed3da8df6207bad7a1b06a480ca "An external client connecting to a NodePort service either through Node 1 or 2")

Uma conexão recebida na porta `30123` do primeiro nó pode ser encaminhada para o *`Pod`* em execução no primeiro nó ou para um dos *`Pods`* em execução no segundo nó.

Usando *JSONPath* para obter os IPs de todos os seus nós:

```bash
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="ExternalIP")].address}'
```

Depois de saber os IPs dos nós, você pode tentar acessar seu serviço por meio deles:

```bash
curl http://130.211.97.55:30123
# You've hit app-wt1or
curl http://130.211.99.206:30123
# You've hit app-vrew1
```

Como você pode ver, os *`Pods`* agora estão acessíveis para toda a internet por meio da porta `30123` em qualquer um dos nós. Não importa para qual nó um *client* envia a solicitação. Mas se você apenas apontar os *clients* para o primeiro nó, quando esse nó falhar, eles não poderão mais acessar o *Service*. É por isso que faz sentido colocar um *balanceador de carga* na frente dos nós para garantir que você esteja dividindo as solicitações por todos os nós íntegros e nunca enviando-as para um nó que esteja offline naquele momento.

Se seu cluster *Kubernetes* for compatível, o *balanceador de carga* pode ser provisionado automaticamente criando um *Service* `LoadBalancer` em vez de `NodePort`. Veremos isso a seguir.

### Expor um *Service* por meio de um balanceador de carga

Os clusters do *Kubernetes* executados em provedores de nuvem geralmente oferecem suporte ao fornecimento automático de um **balanceador de carga** da infraestrutura de nuvem. Tudo que você precisa fazer é definir o tipo do *Service* para `LoadBalancer` em vez de `NodePort`. O *balanceador de carga* terá seu próprio endereço IP exclusivo e publicamente acessível e redirecionará todas as conexões para o seu *Service*. Assim, você pode acessar seu *Service* por meio do **endereço IP do balanceador de carga**.

Se o *Kubernetes* estiver sendo executado em um ambiente que não oferece suporte a *Services* `LoadBalancer`, o *balanceador de carga* não será provisionado, mas o *Service* ainda se comportará como um *Service* `NodePort`. Isso ocorre porque um *Service* `LoadBalancer` é uma extensão de um *Service* `NodePort`.

#### Criando um Service de `LoadBalancer`

Para criar um *Service* com um *balanceador de carga* na frente, crie o *Service* a partir do seguinte manifesto *YAML*, conforme mostrado no código a seguir.

```yml
apiVersion: v1
kind: Service
metadata:
  name: my-loadbalancer
spec:
  type: LoadBalancer        # This type of service obtains a load balancer from the infrastructure hosting the Kubernetes cluster
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: my-app
```

O tipo de *Service* é definido como `LoadBalancer` em vez de `NodePort`. Você não está especificando uma porta de nó específica, embora pudesse (você está permitindo que o *Kubernetes* escolha uma).

#### Usar o *Service* através do balanceador de carga

Depois de criar o *Service*, leva tempo para a infraestrutura criar o *balanceador de carga* e gravar seu *endereço IP* no objeto *Service*. Depois de fazer isso, o endereço IP será listado como o endereço IP externo do seu *Service*:

```bash
kubectl get svc my-loadbalancer
```

Nesse caso, o balanceador de carga está disponível no IP 130.211.53.173, então agora você pode acessar o serviço nesse endereço IP:

```bash
curl http://130.211.53.173
```

Consulte a imagem abaixo para ver como as solicitações HTTP são entregues ao *`Pod`*. Os *clients* externos (curl nesse caso) se conectam à porta `80` do *balanceador de carga* e são roteados para a porta do nó atribuído implicitamente em um dos nós. A partir daí, a conexão é encaminhada para uma das instâncias do *`Pod`*.

![An external client connecting to a LoadBalancer service](/images/articles/Kubernetes/An%20external%20client%20connecting%20to%20a%20LoadBalancer%20service.png?id=be860d9dbf8dba7373c1d4923df529d8 "An external client connecting to a LoadBalancer service")

## Expor *Services* Externamente Por Meio do *Ingress*

### Entendendo por que os *Ingress* são necessários

Um motivo importante é que cada *Service* `LoadBalancer` requer seu próprio *balanceador de carga* com seu próprio endereço IP público, enquanto um `Ingress` requer apenas um, mesmo ao fornecer acesso a dezenas de *Services*. Quando um cliente envia uma solicitação HTTP para o `Ingress`, o host e o caminho na solicitação determinam para qual *Service* a solicitação é encaminhada, conforme mostrado na imagem abaixo.

![Multiple services can be exposed through a single Ingress](/images/articles/Kubernetes/Multiple%20services%20can%20be%20exposed%20through%20a%20single%20Ingress.png?id=9a6727119cdaa52d306c1b429e475cc0 "Multiple services can be exposed through a single Ingress")

### Entendendo como funcionam os *Ingress*

A imagem abaixo mostra como o *client* se conectou a um dos *`Pods`* por meio do controlador `Ingress`. O *client* primeiro executou uma pesquisa DNS de `xyz.example.com` e o servidor DNS (ou o sistema operacional local) retornou o IP do controlador *Ingress*. O *client* então enviou uma solicitação HTTP ao controlador *Ingress* e especificou `xyz.example.com` no cabeçalho do host. A partir desse cabeçalho, o controlador determinou qual *Service* o *client* está tentando acessar, procurou os IPs dos *`Pods`* por meio do objeto `Endpoints` associado ao *Service* e encaminhou a solicitação do *client* para um dos *`Pods`*.

Como você pode ver, o controlador *Ingress* não encaminhou a solicitação ao *Service*. Ele só o usou para selecionar um *`Pod`*. A maioria, senão todos, os controladores funcionam assim.

![Accessing pods through an Ingress](/images/articles/Kubernetes/Accessing%20pods%20through%20an%20Ingress.png?id=f1785496c22b0b82b40035cc7bbfe302 "Accessing pods through an Ingress")

#### Mapeando diferentes *Services* para diferentes caminhos do mesmo host

Se você observar as especificações do *Ingress* de perto, verá que as regras (`.spec.rules`) e os caminhos (`.spec.rules.paths`) são matrizes, portanto, podem conter vários itens. Um *Ingress* pode mapear vários hosts e caminhos para vários *Services*.

Você pode mapear vários caminhos no mesmo host para *Services* diferentes, conforme mostrado no código *YAML* a seguir.

```yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - host: xyz.example.com
    http:
      paths:
      - path: /foo
        backend:                # Requests to xyz.example.com/foo will be routed to the foo service.
          serviceName: foo
          servicePort: 80       # All requests will be sent to port 80 of the foo service
      - path: /bar              # Requests to xyz.example.com/bar will be routed to the bar service
        backend:
          serviceName: bar
          servicePort: 80
```

Nesse caso, as solicitações serão enviadas para dois *Services* diferentes, dependendo do caminho na URL solicitada. Os clientes podem, portanto, alcançar dois *Services* diferentes por meio de um único *endereço IP* (do controlador `Ingress`).

#### Mapeando diferentes *Services* para diferentes hosts

Da mesma forma, você pode usar um `Ingress` para mapear para diferentes *Services* com base no host na solicitação HTTP em vez de (apenas) o caminho, conforme mostrado na próximo código *YAML*.

```yml
spec:
  rules:
  - host: foo.example.com # Requests for foo.example.com will be routed to service foo
    http:
      paths:
        - path: /
          backend:
            serviceName: foo
            servicePort: 80
  - host: bar.example.com # Requests for bar.example.com will be routed to service bar
    http:
      paths:
      - path: /
      backend:
        serviceName: bar
        servicePort: 80
```

As solicitações recebidas pelo controlador serão encaminhadas para o *Service* `foo` ou `bar`, dependendo do cabeçalho do Host na solicitação (a maneira como os hosts virtuais são tratados nos servidores da web). O DNS precisa apontar os nomes de domínio `foo.example.com` e `bar.example.com` para o endereço IP do controlador do *Ingress*.
