---
id: 9c45bde7-b337-4852-86ae-debcac07d949
title: "Services e Ingress – Comunicação Com o Mundo Externo"
summary: "Um estudo sobre aprofundado de Services e Ingress"
---

## Services - Como o Kubernetes Roteia o Tráfego de Rede?

Antes de começar, precisamos nos lembrar de que os IPs dos *`Pods`* não são confiáveis. Quando os *`Pods`* falham, eles são substituídos por novos *`Pods`* com novos IPs. O escalonamento de um *Deployment* apresenta novos *`Pods`* com novos endereços IP. Diminuir um *Deployment* remove os *`Pods`*. Isso cria uma grande quantidade de rotatividade de *IPs* e cria uma situação em que os *IPs* dos *`Pods`* não são confiáveis.

Você também precisa saber três coisas fundamentais sobre os *Services* no *Kubernetes*.

Primeiro, você precisa saber que cada *Service* obtém seu próprio endereço IP, seu próprio nome DNS e sua própria porta.

Segundo, você precisa saber que os *Services* utilizam labels para selecionar dinamicamente os *`Pods`* no cluster para o qual enviarão o tráfego.

### Teoria dos *Services*

A imagem abaixo mostra um simples aplicativo baseado em *`Pod`* implantado por meio de um *Deployment*. Ele mostra um `client` que não tem um *endpoint* de rede confiável para acessar os *`Pods`*. Lembre-se de que é uma má ideia conectar diretamente em um *`Pod`* individual porque esse *`Pod`* pode ser removido a qualquer momento por meio de operações de escalonamento, atualizações, reversões e falhas.

<img src="{{ mix('/images/articles/Kubernetes/Shows a simple Pod-based application deployed via a Kubernetes Deployment.png') }}" alt="Shows a simple Pod-based application deployed via a Kubernetes Deployment">

A imagem abaixo mostra o mesmo aplicativo com um *Service* adicionado. O *Service* está associado aos *`Pods`* e os fornece um IP, DNS e porta estáveis. Ele também atual como balanceador de carga (equilibra a carga de solicitações) entre os *`Pods`*.

<img src="{{ mix('/images/articles/Kubernetes/Shows the same application with a Service added into the mix.png') }}" alt="Shows the same application with a Service added into the mix">

Com um *Service* na frente de um conjunto de *`Pods`*, os *`Pods`* podem aumentar ou diminuir, podem falhar e podem ser atualizados, revertidos ... e enquanto eventos como esses ocorrem, o *Service* na frente deles observa as mudanças e atualizações sua lista de *`Pods`* saudáveis. Mas isso nunca muda o IP, DNS e porta estáveis ​​que o *Service* expõe.

Pense nos *Services* como tendo um *front-end* estático e um *back-end* dinâmico. O *front-end*, que consiste no IP, nome DNS e porta, e nunca muda. O *back-end*, que consiste nos *`Pods`*, pode estar em constante mudança.

### Labels e relacionamento entre *Pods* e *Services*

Os *Services* são fracamente acoplados aos *`Pods`* por meio de `labels` e *seletores de labels*. Essa é a mesma tecnologia que acopla fracamente os *`Pods`* ao *Deployments* e é a chave para a flexibilidade fornecida pelo *Kubernetes*. A imagem abaixo mostra um exemplo em que 3 *`Pods`* são rotulados como `zone=prod` e `version=1`, e o *Service* tem um seletor de label correspondente.

<img src="{{ mix('/images/articles/Kubernetes/Shows an example where 3 Pods are labelled as zone=prod and version=1.png') }}" alt="Shows an example where 3 Pods are labelled as zone=prod and version=1">

Na acima, o *Service* está fornecendo rede estável para todos os três *`Pods`* - você pode enviar solicitações ao *Service* e ele as encaminhará para os *`Pods`*. Ele também fornece balanceamento de carga simples.

Para que um *Service* corresponda a um conjunto de *`Pods`* e, portanto, envie tráfego para eles, **os *`Pods`* devem possuir todos os labels** no *seletor de labels do Service*. No entanto, o *`Pod`* pode ter labels adicionais que não estão listados no *seletor de labels do Service*. Se isso é confuso, os exemplos nas imagens abaixo devem ajudar.

A imagem abaixo mostra um exemplo em que o *Service* não corresponde a nenhum dos *`Pods`*. Isso ocorre porque o *Service* está procurando *`Pods`* com dois labels, mas os *`Pods`* possuem apenas um deles. A lógica por trás disso é um booleano **`AND`**.

<img src="{{ mix('/images/articles/Kubernetes/Shows an example where the Service does not match any of the Pods.png') }}" alt="Shows an example where the Service does not match any of the Pods">

A próxima imagem mostra um exemplo que funciona. Funciona porque o *Service* está procurando dois labels e os *`Pods`* no diagrama possuem os dois. Não importa que os *`Pods`* possuam labels adicionais que o *Service* não está procurando. O *Service* está procurando *`Pods`* com dois labels, ele os encontra e ignora o fato de que os *`Pods`* têm labels adicionais - tudo o que é importante é que os *`Pods`* possuam os labels que o *Service* está procurando.

<img src="{{ mix('/images/articles/Kubernetes/Show an example that the service works.png') }}" alt="Show an example that the service works">

Os trechos de código a seguir, do *YAML* de *Service* e YAML de *Deployment*, mostram como seletores e labels são implementados.

```yml
apiVersion: v1
kind: Service
metadata:
  name: svc-test
  labels:
    app: hello-world
spec:
  type: NodePort
  ports:
  - port: 8080
    nodePort: 30001
    targetPort: 8080
    protocol: TCP
  selector:
    app: hello-world # Traffic is routed to Pods with this label
```

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: svc-test
spec:
  replicas: 10
  selector:
    matchLabels:
      app: hello-world
  template:
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
      - name: hello-ctr
        image: image
        ports:
        - containerPort: 8080
```

Nos códigos de exemplo acima, o *Service* tem um seletor de label (`.spec.selector`) com um único valor `app=hello-world`. Este é o label que o *Service* está procurando ao consultar o cluster em busca de *`Pods`* correspondentes. O *Deployment* especifica um modelo de *`Pod`* com a mesmo label `app=hello-world` (`.spec.template.metadata.labels`). Isso significa que todos os *`Pods`* que ele implantar terão a label `app=hello-world`. São esses dois atributos que acoplam fracamente o *Service* aos *`Pods`* do *Deployment*.

Quando a *Deployment* e o *Service* são implantados, o *Service* seleciona todas as 10 réplicas do *`Pod`* e fornece um endpoint de rede estável e tráfego de load-balance para eles.

### Relação entre *Services* e *Endpoints*

À medida que os *`Pods`* são criados e encerrados (aumento e diminuição, falhas, atualizações contínuas etc.), o *Service* atualiza dinamicamente sua lista de *`Pods`* correspondentes íntegros. Isso é feito por meio de uma combinação do *seletor de labels* e um objeto chamado `Endpoints`.

Cada *Service* criado obtém automaticamente um objeto `Endpoints` associado. Tudo que este objeto `Endpoints` é, é uma lista dinâmica de todos os *`Pods`* íntegros no cluster que correspondem ao seletor de label do *Service*.

Funciona assim ...

O *Kubernetes* avalia constantemente o seletor de label do *Service* em relação à lista atual de *`Pods`* íntegros no cluster. Todos os novos *`Pods`* que correspondem ao seletor são adicionados ao objeto `Endpoints`, e todos os *`Pods`* que desaparecem são removidos. Isso significa que o objeto `Endpoints` está sempre atualizado. Então, quando um *Service* está enviando tráfego para os *`Pods`*, ele consulta seu objeto `Endpoints` para a lista mais recente de *`Pods`* correspondentes íntegros.

Ao enviar tráfego para os *`Pods`* por meio de um *Service*, um aplicativo normalmente consultará o DNS interno do cluster para obter o endereço IP de um *Service*. Em seguida, ele envia o tráfego para esse endereço IP estável e o *Service* o envia para um *`Pod`*. No entanto, um aplicativo nativo do *Kubernetes* (que é uma maneira elegante de dizer um aplicativo que entende o *Kubernetes* e pode consultar a API do *Kubernetes*) pode consultar a API do `Endpoints` diretamente, ignorando a consulta DNS e o uso do IP do *Service*.

### Acessando *Services* de dentro do cluster

O *Kubernetes* oferece suporte a vários tipos de *Service*. O tipo padrão é `ClusterIP`.

O tipo padrão do *Service* no *Kubernetes* é chamado `ClusterIP`. Ele cria um endereço IP em todo o cluster que os *`Pods`* em qualquer nó podem acessar. O endereço IP funciona apenas dentro do cluster, portanto, os *Services* `ClusterIP` são úteis apenas para a comunicação entre *`Pods`*. Isso é exatamente o que você deseja para um sistema distribuído em que alguns componentes são internos e não devem ser acessíveis fora do cluster.

Um Service `ClusterIP` tem um endereço IP e uma porta estável ​​que só podem ser acessados ​​de dentro do cluster. O `ClusterIP` é registrado em relação ao nome do *Service* no DNS interno do cluster. Todos os *`Pods`* no cluster são pré-programados para saber sobre o DNS do cluster, o que significa que todos os *`Pods`* são capazes de resolver os nomes dos *Services*.

### Acessando os *Services* de fora do cluster

O *Kubernetes* tem outro tipo de *Service* denominado `NodePort`. Isso se baseia no `ClusterIP` e permite o acesso de fora do cluster.

Você já sabe que o tipo de *Service* padrão é `ClusterIP` e ele registra um nome DNS, IP virtual e porta com o DNS do cluster. Um tipo diferente de *Service*, chamado de `NodePort`, se baseia nisso, adicionando outra porta que pode ser usada para acessar o *Service* de fora do cluster. Essa porta adicional é chamada de `NodePort`.

O exemplo a seguir representa um *Service* NodePort:

- *name*: `magic-sandbox`
- *clusterIP*: `172.12.5.17`
- *port*: `8080`
- *nodePort*: `30050`

Esse *Service* `magic-sandbox` pode ser acessado de dentro do cluster via `magic-sandbox` na porta `8080` ou `172.12.5.17` na porta `8080`. Ele também pode ser acessado de fora do cluster enviando uma solicitação para o endereço IP de qualquer nó do cluster na porta `30050`.

Na parte inferior da stack estão os nós do cluster que hospedam os *`Pods`*. Você adiciona um *Service* e usa os labels para associá-lo aos *`Pods`*. O objeto *Service* tem um `NodePort` confiável mapeado para cada nó no cluster - o valor de `NodePort` é o mesmo em todos os nós. Isso significa que o tráfego de fora do cluster pode atingir qualquer nó do cluster no `NodePort` e chegar ao aplicativo (*`Pods`*).

A imagem abaixo mostra um *Service* `NodePort` em que `3` *`Pods`* são expostos externamente na porta `30050` em cada nó do cluster. Na *etapa 1*, um *client externo* atinge o Nó 2 na porta `30050`. Na *etapa 2*, ele é redirecionado para o objeto *Service* (isso acontece mesmo que o Nó 2 não esteja executando um *`Pod`* do *Service*). A *etapa 3* mostra que o *Service* tem um objeto `Endpoint` associado com uma lista sempre atualizada de *`Pods`* que correspondem ao seletor de label. A *etapa 4* mostra o *client* sendo direcionado para `Pod1` no `Node1`.

<img src="{{ mix('/images/articles/Kubernetes/Shows a NodePort Service where 3 Pods are exposed externally on port 30050 on every node in the cluster.png') }}" alt="Shows a NodePort Service where 3 Pods are exposed externally on port 30050 on every node in the cluster">

O *Service* poderia facilmente ter direcionado o cliente para `Pod2` ou `Pod3`. Na verdade, as solicitações futuras podem ir para outros *`Pods`* enquanto o *Service* executa o load-balancing básico.

Existem outros tipos de *Services*, como `LoadBalancer` e `ExternalName`.

Os *Services* de `LoadBalancer` se integram aos load-balancers de seu provedor de nuvem, como AWS, Azure, DO, IBM Cloud e GCP. Eles se baseiam nos *Services* `NodePort` (que, por sua vez, se baseiam nos *Services* `ClusterIP`) e permitem que os clientes na Internet acessem seus *`Pods`* por meio de um dos load-balancers da nuvem. Eles são extremamente fáceis de configurar. No entanto, eles só funcionam se você estiver executando o cluster do *Kubernetes* em uma plataforma de nuvem compatível. Por exemplo, você não pode aproveitar um balanceador de carga ELB na AWS se seu cluster Kubernetes estiver em execução no Microsoft Azure.

Os *Services* `ExternalName` roteiam o tráfego para sistemas fora do seu cluster *Kubernetes* (todos os outros tipos de *Service* roteiam o tráfego para os *`Pods`* no seu cluster).

### Cluster DNS

Os nomes DNS no *Kubernetes* são restritos aos *`Pods`* e *Services*. Os nomes DNS do *`Pod`* contêm várias partes estruturadas como subdomínios.

Um nome típico de domínio totalmente qualificado (**FQDN**) para um *`Pod`* em execução no *Kubernetes* se parece com o seguinte:

```
my-hostname.my-subdomain.my-namespace.svc.my-cluster-domain.example
```

Analisando, temos o seguinte:

- `my-cluster-domain.example`: Corresponde ao nome DNS configurado para a API do cluster. Dependendo da ferramenta usada para configurar o cluster e do ambiente em que ele é executado, pode ser um nome de domínio externo ou um nome DNS interno.

- `svc`: É uma seção que vai ter até mesmo em um nome DNS do *`Pod`* - portanto, podemos simplesmente presumir que estará presente. No entanto, como você verá em breve, geralmente você não acessará *`Pods`* ou *Services* por meio de seus *FQDNs*.

- `my-namespace`: Esta seção do nome DNS será qualquer namespace em que o *`Pod`* esteja operando.

- `my-subdomain`: Corresponde ao campo de subdomínio na especificação do *`Pod`*. Este campo é opcional.

- Por fim, `my-hostname`: será definido como o nome do *`Pod`* nos metadados do *`Pod`*.

Juntos, esse *nome DNS* permite que outros recursos no cluster acessem um determinado *`Pod`*. Isso geralmente não é muito útil por si só, especialmente se você estiver usando *Deployments* e *StatefulSets* que geralmente têm vários *`Pods`*. É aqui que entram os *Services*.

Vamos dar uma olhada no *nome DNS do registro A* para um *Service*:

```
my-svc.my-namespace.svc.cluster-domain.example
```

Como você pode ver, é muito semelhante ao DNS do *`Pod`*, com a diferença de que temos apenas um valor à esquerda do namespace - que é o nome do *Service* (novamente, como acontece com os *`Pods`*, ele é gerado com base no nome nos metadados na especificação).

Um resultado de como esses nomes DNS são tratados é que, em um namespace, você pode acessar um *Service* ou *`Pod`* por meio apenas do nome do *Service* (ou *`Pod`*) e do subdomínio.

Por exemplo, usar o nome DNS do *Service* anterior de dentro do namespace `my-namespace`, o *Service* pode ser acessado simplesmente pelo nome DNS `my-svc`. De fora do namespace `my-namespace`, você pode acessar o *Service* por meio de `my-svc.my-namespace`.

Agora que aprendemos como funciona o DNS no cluster, podemos discutir como isso se traduz em *Service proxy*.

### Implementando *Service* de `ClusterIP`

`ClusterIP` é um tipo simples de *Service* **exposto em um IP interno dentro do cluster**. **Este tipo de *Service* não é acessível de fora do cluster**. Vamos dar uma olhada no arquivo *YAML* para o Service:

```yml
apiVersion: v1
kind: Service
metadata:
  name: my-svc    # The Service uses the domain name my-svc
spec:
  type: ClusterIP # This Service is available only to other Pods
  selector:
    app: web-application # Traffic is routed to Pods with this label
    environment: staging # AND Traffic is routed to Pods with this label
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 8080
```

Assim como acontece com outros recursos do *Kubernetes*, existe o bloco de `.metadata` com o valor de `name`. Esse valor de `name` é sobre como acessar o *Service* de qualquer lugar no cluster. Por esse motivo, o `ClusterIP` é uma ótima opção para *Services* que só precisam ser acessados ​​por outros *`Pods`* de dentro de um cluster.

A seção de `.metadata` define um nome para o *Service*. Você também pode aplicar labels. Todos as labels adicionados na seção de `.metadata` são usados ​​para identificar o *Service* e não estão relacionados as labels para selecionar os *`Pods`*.

Em seguida, existe a especificação(`spec`), que consiste em três pontos principais:

- Em primeiro lugar, temos o objeto `type`, que corresponde ao tipo do *Service*. Como o tipo padrão é `ClusterIP`, não é necessário especificar um tipo se quiser um *Service* `ClusterIP`.

- Em seguida, temos o objeto `selector`. Ele consiste em pares de chave-valor que devem corresponder aos labels nos metadata dos *`Pods`* em questão. Nesse caso, o *Service* procurará *`Pods`* com `app=web-application` e `environment=staging` para encaminhar o tráfego/requisições.

- Finalmente, tem o bloco de `ports`, onde pode ser mapeado as portas entre os *Service* e os *`Pods`*. Nesse caso, a porta `80` (a porta *HTTP*) no *Service* será mapeada para a porta `8080` nos *`Pod`* do aplicativo. Mais de uma porta pode ser aberta nos *Services*, mas o campo de `.spec.ports.name` é obrigatória ao abrir múltiplas portas.

### Implementando *Service* de `NodePort`

`NodePort` é um tipo de *Service* externo, o que significa que ele realmente pode ser acessado de fora do cluster. Ao criar um *Service* `NodePort`, um *Service* `ClusterIP` com o mesmo nome será automaticamente criado e roteado pelo `NodePort`, então você ainda poderá acessar o *Service* de dentro do cluster. Isso torna o `NodePort` uma boa opção para acesso externo a aplicativos quando um *Service* `LoadBalancer` não é viável ou possível.

**Este tipo de *Service* abre uma porta em cada nó do cluster no qual o *Service* pode ser acessado**. Esta porta estará em um intervalo de possíveis valores, que por padrão, é entre `30000-32767` e será vinculada automaticamente na criação do *Service*.

Esta é a aparência do *YAML* do Service `NodePort`:

```yml
apiVersion: v1
kind: Service
metadata:
  name: my-svc
spec:
  type: NodePort
  selector:
    app: web-application
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 8080
```

Como no YAML acima, a única diferença para o tipo de *Service* `ClusterIP` é o objeto `.spec.type`. No entanto, é importante observar que a porta `80` na seção de `ports` só será usada ao acessar a versão `ClusterIP` criada automaticamente pelo *Service*. De fora do cluster, precisaremos ver qual é a porta gerada para acessar o *Service*:

Para fazer isso, podemos criar o Service com o seguinte comando:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: my-svc
spec:
  type: NodePort
  selector:
    app: web-application
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 8080
EOF
```

E então executar:

```bash
kubectl describe service my-svc
```

O resultado do comando acima é a seguinte:

```
Name:                   my-svc
Namespace:              default
Labels:                 app=web-application
Annotations:            <none>
Selector:               app=web-application
Type:                   NodePort
IP:                     10.32.0.8
Port:                   <unset> 8080/TCP
TargetPort:             8080/TCP
NodePort:               <unset> 31598/TCP
Endpoints:              10.200.1.3:8080,10.200.1.5:8080
Session Affinity:       None
Events:                 <none>
```

A partir da saída acima, vemos a linha `NodePort` para ver que a porta atribuída para o *Service* é `31598`. Portanto, esse Service pode ser acessado em qualquer nó como `[NODE_IP]:[ASSIGNED_PORT]`.

- `Selector` é a lista dos labels que os *`Pods`* devem ter para que o *Service* envie tráfego para eles.
- `IP` é o *ClusterIP* interno permanente do *Service*.
- `Port` é a porta que o *Service* escuta dentro do cluster
- `TargetPort` é a porta em que o aplicativo está escutando.
- `NodePort` é a porta de todo o cluster que pode ser usada para acessá-lo de fora do cluster.
- `Endpoints` é a lista dinâmica dos *IPs* dos *`Pod`* íntegros que atualmente correspondem ao seletor do label do *Service*.

Como alternativa, podemos atribuir manualmente um `NodePort IP` ao *Service*. O *YAML* para um `NodePort` atribuído manualmente é o seguinte:

```yml
apiVersion: v1
kind: Service
metadata:
  name: my-svc
spec:
  type: NodePort
  selector:
      app: web-application
  ports:
    - name: http
      protocol: TCP
      port: 80         # The port on which the Service is available to other Pods
      targetPort: 8080 # The port on which the traffic is sent to on the port
      nodePort: 31233  # The port on which the Service is available externally
```

Como visto no *YAML* acima, é definido um objeto `nodePort`/`.spec.ports[0].nodePort` no intervalo `30000-32767`, neste caso, `31233`. Para ver exatamente como este Service `NodePort` funciona entre os nós, veja no seguinte diagrama:

<img src="{{ mix('/images/articles/Kubernetes/NodePort Service.png') }}" alt="NodePort Service">

Como pode ser visto, embora o *Service* seja acessível em todos os nós do cluster (`nó A`, `nó B` e `nó C`), as solicitações de rede ainda têm balanceamento de carga entre os *`Pods`* em todos os nós (`Pod A`, `Pod B` e `Pod C`). Essa é uma maneira eficaz de garantir que o aplicativo possa ser acessado de qualquer nó. Ao usar *Services* em nuvem, no entanto, você tem uma variedade de ferramentas para distribuir solicitações entre os servidores. O próximo tipo de *Service*, `LoadBalancer`, nos permite usar essas ferramentas no contexto do *Kubernetes*.

### Implementando *Service* de `LoadBalancer`

`LoadBalancer` é um tipo de *Service* especial no *Kubernetes* que provisiona um balanceador de carga com base em onde o cluster está sendo executado. Por exemplo, na *AWS*, o Kubernetes provisionará um *Elastic Load Balancer*.

Ao contrário do `ClusterIP` ou `NodePort`, podemos alterar a funcionalidade de um *Service* `LoadBalancer` de maneiras específicas na cloud. Geralmente, isso é feito usando o bloco de `annotations` no arquivo *YAML* do *Service* - que é apenas um conjunto de chaves e valores. Para ver como isso é feito no *AWS*, vamos revisar as especificações para um *Service* `LoadBalancer`:

```yml
apiVersion: v1
kind: Service
metadata:
  name: my-svc
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-ssl-cert: arn:aws..
spec:
  type: LoadBalancer
  selector:
    app: web-application
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 8080
```

Embora possamos criar um `LoadBalancer` sem quaisquer `annotations`, as `annotations` específicas da *AWS* nos dão a capacidade (como visto no código *YAML* anterior) de especificar qual certificado TLS (por meio de seu ARN no Amazon Certificate Manager) dá para anexar ao balanceador de carga. As `annotations` da AWS também permitem a configuração de logs para o balanceadores de carga e muito mais.

Aqui estão algumas `annotations` importantes com suporte do AWS Cloud Provider:

- `service.beta.kubernetes.io/aws-load-balancer-ssl-cert`
- `service.beta.kubernetes.io/aws-load-balancer-proxyprotocol`
- `service.beta.kubernetes.io/aws-load-balancer-ssl-ports`

Finalmente, com o `LoadBalancer` *Services*, cobrimos os tipos de *Services* que você provavelmente usará mais. No entanto, para casos especiais em que o próprio *Services* é executado fora do *Kubernetes*, podemos usar outro tipo de *Services*: `ExternalName`.

### Implementando *Service* de `ExternalName`

Os *Services* do tipo `ExternalName` podem ser usados ​​para fazer proxy de aplicativos que não estão realmente em execução no cluster, mantendo o *Service* como uma camada de abstração que pode ser atualizada a qualquer momento.

Vamos definir o cenário: você tem um aplicativo de produção legado em execução no Azure que deseja acessar de dentro do seu cluster. Você pode acessar este aplicativo legado em `myoldapp.mydomain.com`. No entanto, sua equipe está trabalhando atualmente na criação de containers deste aplicativo e executando-o no *Kubernetes*, e essa nova versão está trabalhando no ambiente de namespace `dev` em seu cluster.

Em vez de solicitar que seus outros aplicativos conversem com lugares diferentes dependendo do ambiente, você sempre pode apontar para um *Service* chamado `my-svc` em seus namespaces de produção (`prod`) e desenvolvimento (`dev`).

No desenvolvimento, esse *Service* pode ser um *Service* `ClusterIP` que leva ao seu aplicativo recém-armazenado em containers dos *`Pods`*. O *YAML* a seguir mostra como o *Service* em container em desenvolvimento deve funcionar:

```yml
apiVersion: v1
kind: Service
metadata:
  name: my-svc
  namespace: dev
Spec:
  type: ClusterIP
  selector:
    app: newly-containerized-app
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 8080
```

No namespace `prod`, este *Service* seria, em vez disso, um Service `ExternalName`:

```yml
apiVersion: v1
kind: Service
metadata:
  name: my-svc
  namespace: prod
spec:
  type: ExternalName
  externalName: myoldapp.mydomain.com
```

Como nosso Service `ExternalName` não está realmente encaminhando solicitações para os *`Pods`*, não precisamos de um seletor. Em vez disso, especificamos um `ExternalName`, que é o nome DNS para o qual queremos que o Service direcione.

O diagrama a seguir mostra como um *Service* `ExternalName` pode ser usado neste padrão:

<img src="{{ mix('/images/articles/Kubernetes/ExternalName Service configuration.png') }}" alt="ExternalName Service configuration">

No diagrama anterior, o `EC2 Running Legacy Application` é uma *VM AWS*, externa ao cluster. O `Service B` do tipo `ExternalName` encaminhará as solicitações para a *VM*. Dessa forma, o `Pod C` (ou qualquer outro *`Pod`* no cluster) pode acessar o aplicativo legado externo simplesmente por meio do nome DNS do *Kubernetes* dos *Services* de `ExternalName`.

## Ingress - Recebendo Solicitações Para o Cluster

**o *Ingress* fornece um mecanismo granular para rotear solicitações para o cluster**. O Ingress não substitui os *Services*, mas os aumenta com recursos como roteamento baseado na URL. Por que isso é necessário? Existem muitos motivos, incluindo o custo. Um *Ingress* com 10 rotas para os *Services* ClusterIP é muito mais barato do que criar um novo serviço `LoadBalancer` para cada rota - além de manter as coisas simples e fáceis de entender.

Os *Ingress* não funcionam como os *Services* no Kubernetes. Apenas criar o *Ingress* em si não fará nada. Você precisa de dois componentes adicionais:

- Um *Ingress Controller*: Você pode escolher entre várias implementações, baseadas em ferramentas como Nginx ou HAProxy.
- *Services* `ClusterIP` ou `NodePort` para as rotas pretendidas.

Primeiro, vamos discutir como configurar o **Ingress Controller**.

### Ingress controllers

Geralmente, os clusters não vêm configurados com nenhum *Ingress controllers* pré-existente. Você precisará selecionar e implantar um no cluster. `ingress-nginx` é provavelmente a escolha mais popular, mas existem várias outras - consulte na [documentação do Kubernetes](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) para uma lista completa.

Vamos aprender como implantar um *Ingress controllers* - para os fins de estudo, vamos nos ater ao controlador `Nginx Ingress` criado pela comunidade *Kubernetes*, `ingress-nginx`.

A instalação pode variar de controlador para controlador, mas para `ingress-nginx` há duas partes principais. Primeiro, para implantar o próprio controlador principal, execute o seguinte comando, que pode mudar dependendo do ambiente de destino e da versão mais recente do Ingress Nginx:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/cloud/deploy.yaml
```

Em segundo lugar, podemos precisar configurar o Ingress dependendo do ambiente em que estamos executando. Para um cluster em execução no AWS, podemos configurar o ponto de entrada do Ingress para usar um Elastic Load Balancer que criamos no AWS.

O Nginx ingress controller é um conjunto de Pods que atualizará automaticamente a configuração do Nginx sempre que um novo Ingress resource (um recurso Kubernetes personalizado) for criado. Além do Ingress controller, precisaremos rotear as solicitações para o Ingress controller conhecido como entry point(ponto de entrada).

#### Ingress entry point

A instalação padrão do `ingress-nginx` também criará um único Service que atende as solicitações da camada do Nginx, ponto em que as regras de entrada assumem o controle. Dependendo de como você configura o Ingress, pode ser um `LoadBalancer` ou `NodePort` Service. Em um ambiente de nuvem, você provavelmente usará um Service `LoadBalancer` como entry point para o Ingress do cluster.

#### Ingress rules and YAML

Agora que temos nosso Ingress controller instalado e funcionando, podemos começar a configurar as Ingress rules (regras de entrada).

Vamos começar com um simples exemplo. Temos dois Services, `service-a` e `service-b`, que queremos expor em rotas diferentes por meio do Ingress. Depois que o Ingress controller e todos os Elastic Load Balancers associados forem criados (supondo que estejamos executando no AWS), vamos primeiro criar nossos Services trabalhando com as seguintes etapas:

1. Primeiro, vamos ver como criar o `Service A` em YAML. Vamos chamar o arquivo `service-a.yaml`:

```yml
apiVersion: v1
kind: Service
metadata:
    name: service-a
Spec:
    type: ClusterIP
    selector:
        app: application-a
    ports:
        - name: http
          protocol: TCP
          port: 80
          targetPort: 8080
```

2. Você pode criar o `Service A` executando o seguinte comando:

```bash
kubectl apply -f service-a.yaml
```

3. A seguir, vamos criar o `Service B`, no qual o código YAML é muito semelhante:

```yml
apiVersion: v1
kind: Service
metadata:
    name: service-b
Spec:
    type: ClusterIP
    selector:
        app: application-b
    ports:
        - name: http
          protocol: TCP
          port: 80
          targetPort: 8000
```

4. Crie o `Service B` executando o seguinte comando:

```bash
kubectl apply -f service-b.yaml
```

5. Finalmente, podemos criar o Ingress com as regras para cada URL. Aqui está o código YAML para o Ingress que dividirá as solicitações conforme necessário com base nas regras de roteamento baseadas na URL:

```yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
    name: my-first-ingress
    annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /
spec:
    rules:
    - host: my.application.com
      http:
          paths:
          - path: /a
            backend:
                serviceName: service-a
                servicePort: 80
          - path: /b
            backend:
                serviceName: service-b
                servicePort: 80
```

No YAML anterior, o Ingress tem um valor de host único, que corresponderia ao cabeçalho `host` da solicitação/request para o tráfego passando para o Ingress. Então, temos duas URLs, `/a` e `/b`, que levam aos dois Services ClusterIP criados anteriormente. Para colocar essa configuração em um formato gráfico, vamos dar uma olhada no seguinte diagrama:

<img src="{{ mix('/images/articles/Kubernetes/Kubernetes Ingress example.png') }}" alt="Kubernetes Ingress example">

Como visto na imagem acima, as regras simples baseadas nas URLs resultam em solicitações de rede sendo roteadas diretamente para os *Pods* adequados. Isso ocorre porque `nginx-ingress` usa o `selector` do Service para obter uma lista de IPs dos *Pods*, mas não usa diretamente o *Service* para se comunicar com os Pods. Em vez disso, a configuração do Nginx (neste caso) é atualizada automaticamente à medida que novos IPs dos Pods ficam online.

O valor do `host` não é realmente necessário. Se não tiver esse objeto, qualquer tráfego que passar pelo Ingress, independentemente do cabeçalho do host (a menos que corresponda a uma regra diferente que especifica um host), será roteado de acordo com a regra. O seguinte YAML mostra isso:

```yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
    name: my-first-ingress
    annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /
spec:
    rules:
    - http:
        paths:
        - path: /a
          backend:
              serviceName: service-a
              servicePort: 80
        - path: /b
          backend:
              serviceName: service-b
              servicePort: 80
```

Esta definição anterior do Ingress fará o tráfego fluir para as regras de roteamento baseadas na URL, mesmo se não houver um valor `host` no cabeçalho.

Da mesma forma, é possível dividir o tráfego em vários URLs de ramificação separados com base no cabeçalho de host, como este:

```yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
    name: multiple-branches-ingress
spec:
    rules:
    - host: my.application.com
      http:
          paths:
          - backend:
                serviceName: service-a
                servicePort: 80
    - host: my.otherapplication.com
      http:
          paths:
          - backend:
                serviceName: service
                servicePort: 80
```
