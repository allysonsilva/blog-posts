---
id: 9c45b16a-febc-4cff-bfe3-7ce59ab15ddc
title: Escalonando e Implantando Seu Aplicativo
summary: "Um estudo sobre ReplicaSets, Deployments, HPA, DaemonSets e StatefulSets no K8s"
---

## Controladores de *Pods*

O *Kubernetes* oferece várias opções de controladores de *`Pod`* prontos para uso. A opção mais simples é usar um `ReplicaSet`, que mantém um determinado número de instâncias de determinado *`Pod`*. Se uma instância falhar, o `ReplicaSet` ativará uma nova instância para substituí-la.

Em segundo lugar, existem `Deployments`, que controlam um `ReplicaSet`. Os `Deployments` são os controladores mais popular quando se trata de executar um aplicativo no *Kubernetes* e facilitam seu upgrade usando atualização contínua (rolling update) em um `ReplicaSet`.

*Horizontal `Pod` Autoscalers* (HPA) elevam os `Deployments` para o próximo nível, permitindo que os aplicativos sejam escalonados automaticamente para diferentes números de instâncias com base nas métricas de desempenho.

O comportamento real de um controlador, deve ser fácil de prever. Uma visão simplificada do loop de controle padrão se parece com o seguinte diagrama:

![A basic control loop for a Kubernetes controller](/images/articles/Kubernetes/A%20basic%20control%20loop%20for%20a%20Kubernetes%20controller.png?id=fadcdbb2a9f0e7d488d695c8883cfdec "A basic control loop for a Kubernetes controller"))

Na imagem acima, o controlador verifica constantemente o *estado pretendido do cluster* (sete *`Pods`* do aplicativo) em relação ao *estado atual do cluster* (cinco *`Pods`* do aplicativo em execução). Quando o *estado pretendido* não corresponde ao *estado atual*, o controlador agirá por meio da API para corrigir o *estado atual* para corresponder ao *estado pretendido*.

## Usando *`ReplicaSets`*

Um *`ReplicaSet`* é um recurso do *Kubernetes* que garante que seus *`Pods`* estejam sempre em execução. Se o *`Pod`* desaparecer por qualquer motivo, como no caso de um nó ser desligado do cluster ou porque o *`Pod`* foi removido do nó, o *`ReplicaSet`* observará o *`Pod`* ausente e criará um novo *`Pod`* de substituição.

A imagem abaixo mostra o que acontece quando um nó falha e leva dois *`Pods`* com ele. O `Pod A` foi criado via CLI (sem um controlador de alto nível) e, portanto, é um *`Pod`* não gerenciado, enquanto o `Pod B` é gerenciado por um *`ReplicaSet`*. Depois que o nó falha, o *`ReplicaSet`* cria um novo *`Pod`* (`Pod B2`) para substituir o `Pod B` ausente, enquanto o `Pod A` é perdido completamente, nenhum controlador o recriará.

O *`ReplicaSet`* na imagem abaixo gerencia apenas um único *`Pod`*, mas os *`ReplicaSets`*, em geral, destinam-se a criar e gerenciar várias cópias (réplicas) de um modelo de *`Pod`*.

![When a node fails, only pods backed by a ReplicaSet are recreated](/images/articles/Kubernetes/When%20a%20node%20fails,%20only%20pods%20backed%20by%20a%20ReplicaSet%20are%20recreated.png?id=7f543d04e59e9150efc5022e56095f12 "When a node fails, only pods backed by a ReplicaSet are recreated")

Um *`ReplicaSet`* monitora constantemente a lista de *`Pods`* em execução e **garante que o número real de *`Pods`* de um "tipo" sempre corresponda ao número desejado**. Se houver poucos *`Pods`* em execução, ele criará novas réplicas a partir de um modelo do *`Pod`*. Se muitos desses *`Pods`* estiverem em execução, ele removerá as réplicas em excesso.

O controlador *`ReplicaSet`* permitem informar ao *Kubernetes* para manter um determinado número de *`Pods`* para determinado `PodSpec`. O *YAML* de um *`ReplicaSet`* é muito semelhante ao de um *`Pod`*. Na verdade, toda a chave `spec` do *`Pod`* está aninhada no *YAML* do *`ReplicaSet`*, sob a chave `template`.

Existem também algumas outras diferenças importantes, que podem ser observadas no seguinte bloco de código:

```yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
    name: myapp-group
    labels:
        app: myapp
spec:
    replicas: 3
    selector:
        matchLabels:
            app: myapp
    template:
        metadata:
            labels:
                app: myapp
        spec:
            containers:
            - name: myapp-container
              image: nginx
```

No *YAML* acima, além da seção de `template`, que é essencialmente uma definição do *`Pod`* (mesmas configuração da chave `spec` quando a chave `kind` é `Pod`), existe uma chave `selector` e uma chave de `replicas` na especificação do `ReplicaSet`.

### `replicas`

A chave de `replicas` especifica a quantidade total de *`Pods`*, que o `ReplicaSet` garantirá que esteja sempre em execução em um determinado namespace do cluster. Se um *`Pod`* é encerrado ou para de funcionar, o `ReplicaSet` criará um novo *`Pod`* para substituí-lo.

Como um controlador de *ReplicaSet* decide quando um *`Pod`* para de funcionar? Ele analisa o status do *`Pod`*. Se o status atual do *`Pod`* não for `Running` ou `ContainerCreating`, o *ReplicaSet* tentará iniciar um novo *`Pod`*.

O *status do `Pod`* após a criação dos containers é dito pelo *liveness*, *readiness*, ou *startup* probes, que podem ser configurados especificamente para o *`Pod`*. Isso significa que o `ReplicaSet` entrará em ação para criar um novo *`Pod`* de acordo com a configuração de sondagem/health.

### `selector`

A chave de `selector` é importante devido à maneira de como um *ReplicaSet* funciona - é um controlador que é implementado com o `selector` em seu núcleo. O trabalho do *ReplicaSet* é garantir que o número de *`Pods`* em execução correspondentes ao seletor (`spec.selector`) esteja correto.

Digamos, por exemplo, que tenha um *`Pod`* existente executando o aplicativo, `MyApp`. Este *`Pod`* é rotulado-*label* com uma chave `app` com o valor `my-app`.

Agora, digamos que seja criado um *ReplicaSet* com o mesmo aplicativo, que adicionará três instâncias de *`Pods`* adicionais. A configuração `spec.selector.matchLabels` do *ReplicaSet* corresponde ao rótulo `app=my-app` do *`Pod`* já em execução, e a diretiva de `replicas` tem seu valor de três, com a intenção de executar quatro instâncias no total, uma vez que você já tem uma em execução.

O que acontecerá quando o `ReplicaSet` for iniciado? Descobrirá que o número total de *`Pods`* executando o aplicativo será três, não quatro. Isso ocorre porque um *ReplicaSet* tem a capacidade de adotar/juntar *`Pods`* órfãos (sem ser manipulado por nenhum controlador) e colocá-los sob seu gerenciamento.

Quando o *ReplicaSet* é inicializado, ele vê que já existe um *`Pod`* existente que corresponde à chave `spec.selector`. Dependendo do número de réplicas necessárias, um *ReplicaSet* encerrará *`Pods`* existentes ou iniciará novos *`Pods`* que correspondam a diretiva `selector` para criar o número correto.

### `template`

A seção do objeto de `template` contém as informações do *`Pod`* e oferece suporte aos mesmos campos que os *YAMLs* do *`Pod`*, incluindo a seção de `metadata` e a própria especificação - `spec`. A maioria dos outros controladores segue esse padrão - eles permitem que você defina a especificação do *`Pod`* dentro do *YAML* do controlador no mesmo objeto de `template`.

### Testando o controlador *ReplicaSet*

Execute o *`Pod`* no cluster:

```yml
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: ReplicaSet
metadata:
    name: myapp-group
    labels:
        app: myapp
spec:
    replicas: 3
    selector:
        matchLabels:
            app: myapp
    template:
        metadata:
            labels:
                app: myapp
        spec:
            containers:
            - name: myapp-container
              image: nginx
EOF
# replicaset.apps/myapp-group created
```

Para verificar se o *ReplicaSet* foi criado corretamente, execute `kubectl get pods` para buscar os *`Pods`* no namespace `default`.

Como não especificamos um namespace no *YAML* ou no comando, ele será criado no namespace `default`. O resultado do comando `kubectl get pods` deve ser como da seguinte forma:

```
NAME                READY   STATUS    RESTARTS
myapp-group-9wncv   1/1     Running   0
myapp-group-qjgmb   1/1     Running   0
myapp-group-zq9jz   1/1     Running   0
```

Agora, tente excluir um dos *`Pods`* do *ReplicaSet* usando o seguinte comando:

```bash
kubectl delete pods myapp-group-9wncv
# pod "myapp-group-9wncv" deleted
```

O *ReplicaSet* sempre tentará manter o número especificado de réplicas/*`Pods`* de acordo com a diretiva `.spec.replicas`.

Executando o comando `kubectl get pods` para ver os *`Pods`* em execução novamente:

```
NAME                READY   STATUS              RESTARTS
myapp-group-4x4lq   1/1     ContainerCreating   0
myapp-group-qjgmb   1/1     Running             0
myapp-group-zq9jz   1/1     Running             0
```

Como no resultado do comando acima, o controlador *ReplicaSet* está iniciando um novo *`Pod`* para manter o número de réplicas em três (valor da diretiva `.spec.replicas`).

Por fim, pode-se excluir o *ReplicaSet* usando o seguinte comando:

```bash
kubectl delete replicasets myapp-group
# replicaset.apps "myapp-group" deleted
```

### Compreendendo os benefícios de usar um controlador de replicação

- *Ele garante que um `Pod` (ou várias réplicas do `Pod`) esteja sempre em execução, iniciando um novo `Pod` quando um existente desaparecer/falar;*
- *Quando um nó do cluster falha, ele cria réplicas de substituição para todos os `Pods` que estavam em execução no nó com falha (aqueles que estavam sob o controle do ReplicaSet);*
- *Permite escalonamento horizontal dos `Pods` - tanto manual quanto automático;*

### Entendendo as três partes de um *ReplicaSet*

Um *ReplicaSet* tem três partes essenciais (também mostradas na imagem abaixo):

- *Um label selector, que determina quais `Pods` estão no escopo do ReplicaSet;*
- *Uma contagem de réplicas, que especifica o número desejado de `Pods` que devem estar em execução;*
- *Um modelo do `Pod`, que é template usado ao criar novas réplicas;*

![The three key parts of a ReplicaSet -pod selector, replica count, and pod template-](/images/articles/Kubernetes/The%20three%20key%20parts%20of%20a%20ReplicaSet%20(pod%20selector,%20replica%20count,%20and%20pod%20template).png?id=ff52d554ed5df313af21c3b3ef619fdc "The three key parts of a ReplicaSet (pod selector, replica count, and pod template")

### Entendendo exatamente o que leva o controlador a criar um novo *Pod*

O controlador está respondendo à exclusão de um *`Pod`* criando um novo *`Pod`* de substituição. Bem, tecnicamente, não está respondendo à exclusão em si, mas ao estado resultante - o número inadequado dos *`Pods`*.

Embora um *ReplicaSet* seja notificado imediatamente sobre a exclusão de um *`Pod`* (o *`API server`* permite que os clientes observem as alterações nos recursos e nas listas de recursos), não é isso que faz com que ele crie um *`Pod`* de substituição. A notificação aciona o controlador para verificar o número real de *`Pods`* e tomar as medidas adequadas.

![If a pod disappears, the ReplicaSet sees too few pods and creates a new replacement pod](/images/articles/Kubernetes/If%20a%20pod%20disappears,%20the%20ReplicaSet%20sees%20too%20few%20pods%20and%20creates%20a%20new%20replacement%20pod.png?id=89298d36b82d757efde38d2db4bc8bb0 "If a pod disappears, the ReplicaSet sees too few pods and creates a new replacement pod")

## *Deployments*: Atualizações de Aplicativos Declarativamente

### Teoria dos *Deployments*

Em alto nível, você começa com o código do aplicativo. Isso é empacotado em um container que é empacotado em um *`Pod`* para que possa ser executado no *Kubernetes*. No entanto, os *`Pods`* não se self-heal, não são escalonados e não permitem facilmente atualizações ou reversões. *Deployments* fazem tudo isso. Como resultado, você quase sempre implantará *`Pods`* por meio de um controlador de *Deployment*.

A imagem abaixo mostra alguns *`Pods`* sendo gerenciados por um controlador de *Deployment*.

![Deployment theory](/images/articles/Kubernetes/Deployment%20theory.png?id=c3af1fe3b9ed278ae50b4a77753ccc9d "Deployment theory")

**É importante saber que um único objeto de *Deployment* pode gerenciar apenas um único modelo de Pod**. Por exemplo, se você tiver um aplicativo com um modelo de *`Pod`* para o front-end e outro modelo de *`Pod`* para um serviço back-end de produtos, você precisará de dois *Deployments*. No entanto, como você viu na imagem acima, uma *Deployment* pode gerenciar várias réplicas do mesmo *`Pod`*. Por exemplo, a imagem acima pode ser uma *Deployment* que atualmente gerencia duas réplicas *`Pods`* do servidor da web.

Uma coisa a ser observada é que, em background, os *Deployments* manipulam *ReplicaSet*. Embora seja uma prática recomendada não interagir diretamente com os *ReplicaSets*, é importante entender a função que eles desempenham.

*Os Deployments usam ReplicaSets para fornecer self-healing (autocorreção) e escalonamento.*

Na imagem abaixo mostra os mesmos *`Pods`* gerenciados pelo mesmo *Deployment*. No entanto, desta vez, adicionamos um objeto *ReplicaSet* ao relacionamento e vemos qual objeto é responsável por qual recurso.

![Deployment theory with ReplicaSet](/images/articles/Kubernetes/Deployment%20theory%20with%20ReplicaSet.png?id=5359806fb37c595a452220ccd04c02c2 "Deployment theory with ReplicaSet")

Em resumo, pense em *Deployment* como gerenciador de *ReplicaSets* e *ReplicaSets* como gerenciador de *`Pods`*. Junte todos eles e você terá uma ótima maneira de implantar e gerenciar aplicativos no *Kubernetes*.

> A principal vantagem de um *Deployment* é que ele permite especificar um procedimento de rollout - ou seja, como uma atualização do aplicativo é implantada nos vários `Pods` do *Deployment*. Isso permite configurar/manipular facilmente os controles para impedir/parar atualizações incorretas.

### Usando *Deployments* para atualizar aplicativos declarativamente

Um *`Deployment`* é um recurso *top-level* destinado a implantar aplicativos e atualizá-los declarativamente, em vez de fazer isso por meio de *`ReplicaSet`*, que são considerados conceitos de nível inferior.

Quando você cria um *`Deployment`*, um recurso *`ReplicaSet`* é criado por baixo. Os *`ReplicaSets`* replicam e gerenciam os *`Pods`*. Ao usar *`Deployments`*, os *`Pods`* reais são criados e gerenciados pelos *`ReplicaSets`* do *`Deployment`*, não pelo *`Deployment`* diretamente (o relacionamento é mostrado na imagem abaixo).

![A Deployment is backed by a ReplicaSet, which supervises the deployments pods](/images/articles/Kubernetes/A%20Deployment%20is%20backed%20by%20a%20ReplicaSet,%20which%20supervises%20the%20deployments%20pods.png?id=b519f69c2fce42df3ff561f0a8402aa3 "A Deployment is backed by a ReplicaSet, which supervises the deployments pods")

Usar um *`Deployment`* em vez de construções de nível inferior torna a atualização de um aplicativo muito mais fácil, porque você está definindo o estado desejado por meio de um único recurso de *`Deployment`* e deixando o *Kubernetes* cuidar do resto.

#### Self-healing e escalabilidade

Os *`Pods`* são ótimos. Eles gerenciam os containers, permitindo a co-localização, compartilhamento de volumes, compartilhamento de memória, rede e muito mais. Mas eles não oferecem nada em termos de autocorreção (*self-healing*) e escalabilidade - se o nó em que um *`Pod`* está sendo executado falhar, o *`Pod`* não será reiniciado em outro nó por exemplo.

É aqui que os *Deployments* brilham ...

Os *Deployments* aumentam os *`Pods`* adicionando itens como autocorreção e escalabilidade. Isso significa:

- *Se um `Pod` gerenciado por um Deployment falhar, ele será substituído - self-healing (autocorreção);*
- *Se um `Pod` gerenciado por um Deployment tiver a carga aumentada, você poderá adicionar facilmente mais do mesmo `Pod` para lidar com a carga - scaling (escalonamento)*

Porém, lembre-se de que, em background, os *Deployments* usam um objeto chamado `ReplicaSet` para realizar a autocorreção e a escalabilidade. No entanto, os *ReplicaSets* operam em background e você sempre deve realizar operações em relação ao *Deployment*.

#### É tudo sobre o estado

É fundamental entender três conceitos que são fundamentais para tudo sobre o *Kubernetes*:

- *Estado desejado*
- *Estado atual (às vezes chamado de estado real ou estado observado)*
- *Modelo declarativo*

O estado desejado é o que você deseja. O estado atual é o que você tem. Se os dois combinarem, todos ficarão felizes.

O modelo declarativo é uma maneira de dizer ao *Kubernetes* qual é o estado desejado, sem precisar entrar em detalhes de como implementá-lo. Você deixa o "como" para o *Kubernetes*.

#### Rolling updates (atualizações contínuas) com *Deployments*

Além de *self-healing* e *escalonamento*, os *Deployments* fornecem **rolling-updates** com **zero-downtime**.

Uma atualização contínua (*rolling update*) funciona assim - quando a especificação do *Deployment* é atualizada com uma nova versão do container do *`Pod`*, o *Deployment* encerra um *`Pod`* por vez, cria um novo *`Pod`* com a nova versão do aplicativo e aguarda o registro/status do novo *`Pod`* como `Ready` conforme determinado pelas verificações de *probes* e, em seguida, vá para o próximo *`Pod`*.

Conforme mencionado anteriormente, os *Deployments* usam *ReplicaSets*. Na verdade, toda vez que você cria um *Deployment*, obtém automaticamente um *ReplicaSet* que gerencia os *`Pods`* do *Deployment*.

As melhores práticas indicam que você não deve gerenciar diretamente os *ReplicaSets*. Você deve executar todas as ações no objeto de *Deployment* e deixar o *Deployment* para gerenciar os *ReplicaSets*.

Funciona assim. Você projeta aplicativos como serviços com um *`Pod`*. Por conveniência - *self-healing*, escalonamento, atualizações contínuas (rolling updates) e muito mais - você envolve `Pods` nos *Deployments*. Isso significa criar um arquivo de configuração *YAML* descrevendo todos os itens a seguir:

- *Quantas réplicas dos `Pods`*
- *Qual imagem usar para os containers do `Pod`*
- *Quais portas de rede usar*
- *Detalhes sobre como realizar rolling updates (atualizações contínuas)*

Você faz o `POST` do arquivo *YAML* no *`API server`* e o *Kubernetes* faz o resto.

Depois que tudo estiver funcionando, o *Kubernetes* configura os *loops de observação* para garantir que o estado atual/observado corresponda ao estado desejado.

Agora, suponha que tenha um bug no aplicativo e precise implantar uma imagem atualizada com a correção. Para fazer isso, você atualiza o mesmo arquivo *YAML* do *Deployment* com a nova versão da imagem e faz o `POST` novamente para o *`API server`*. Isso registra um *novo estado desejado no cluster*, solicitando o mesmo número de *`Pods`*, mas todos executando a nova versão da imagem. Para que isso aconteça, o *Kubernetes* cria um novo *ReplicaSet* para os *`Pods`* com a nova imagem. Agora você tem dois *ReplicaSets* - o original para os *`Pods`* com a versão antiga da imagem e um novo para os *`Pods`* com a versão atualizada. Cada vez que o *Kubernetes* aumenta o número de *`Pods`* no novo *ReplicaSet* (com a nova versão da imagem), ele diminui o número de *`Pods`* no antigo *ReplicaSet* (com a versão antiga da imagem). Resultado disso é que, você obtém uma atualização contínua sem tempo de inatividade (**rolling update with zero downtime**).

A imagem abaixo mostra um *Deployment* que foi atualizada uma vez. O *Deployment* inicial criou o *ReplicaSet* à esquerda e a atualização criou o *ReplicaSet* à direita. Você pode ver que o *ReplicaSet* para o *Deployment* inicial foi encerrado e não tem mais réplicas do *`Pod`*. O *ReplicaSet* associado à atualização está ativo e possui todos os *`Pods`*.

![Shows a Deployment that has been updated once](/images/articles/Kubernetes/Shows%20a%20Deployment%20that%20has%20been%20updated%20once.png?id=524a31424ae899dfaa9ffe4d43f758bf "Shows a Deployment that has been updated once")

É importante entender que o *ReplicaSet* antigo ainda tem sua configuração completa, incluindo a versão mais antiga da imagem que ele usava. Isso será importante no próximo tópico.

#### Rollbacks

Como vimos na imagem acima, *ReplicaSets* mais antigos são encerrados e não gerenciam nenhum *`Pod`*. No entanto, eles ainda existem com sua configuração completa. Isso os torna uma ótima opção para reverter para as versões anteriores.

O processo de reversão é essencialmente o oposto de uma atualização contínua - envolva um dos *ReplicaSets* antigos e encerre o atual. Simples.

A imagem abaixo mostra o mesmo aplicativo revertido para a revisão inicial.

![Shows the same app rolled back to the initial revision](/images/articles/Kubernetes/Shows%20the%20same%20app%20rolled%20back%20to%20the%20initial%20revision.png?id=bba9d18efd6e146d25746c991f8e8411 "Shows the same app rolled back to the initial revision")

Mas isso não é o fim. Há inteligência que nos permite dizer coisas como "espere X segundos após cada *`Pod`* executar antes de prosseguir para o próximo *`Pod`*". Há também testes de inicialização, testes de prontidão e testes de atividade que podem verificar a integridade e o status dos *`Pods`*. Em suma, os *Deployments* são excelentes para executar rolling updates (atualizações contínuas) e *versioned rollbacks*.

### Criação de *Deployments*

Criar um *`Deployment`* não é tão diferente de criar um *ReplicaSet*. Uma *`Deployment`* também é composta por um *label selector*, uma contagem de réplicas desejada e um modelo do *`Pod`*. Além disso, ele também contém um campo, que especifica uma estratégia de implantação que define como uma atualização deve ser realizada quando o recurso de *`Deployment`* é modificado.

Veja o código abaixo como exemplo para criação de uma nova implantação:

```yml
cat <<EOF | kubectl apply --record -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
    labels:
      app: myapp
spec:
  replicas: 3
  template:
    metadata:
      name: myapp
      labels:
        app: myapp
    spec:
      containers:
      - image: nginx
        name: myapp-container
  selector:
    matchLabels:
      app: myapp
EOF
```

### Especificação de um *Deployment*

Veja a especificação de um *Deployment*:

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    app: myapp
spec:
  replicas: 3
  minReadySeconds: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp-container
        image: nginx
```

O *YAML* acima isso é muito semelhante à especificação de um *ReplicaSet*. A diferença é uma nova chave na especificação: `strategy`.

Bem no topo, você especifica a versão da API a ser usada. Supondo que você esteja usando uma versão atualizada do *Kubernetes*, os objetos do *Deployment* estão no grupo da API `apps/v1`.

A seção `.metadata` é onde damos um nome e definimos labels ao *Deployment*.

A seção `.spec` é onde a maior parte da ação acontece. Qualquer coisa diretamente abaixo de `.spec` está relacionada ao *`Pod`*. Qualquer coisa aninhada abaixo de `.spec.template` está relacionada ao modelo-*template* do *`Pod`* que o *Deployment* gerenciará. Neste exemplo, o modelo do *`Pod`* define um único container.

`.spec.replicas` informa ao *Kubernetes* como as réplicas dos *`Pods`* devem ser implantadas.

`.spec.selector` é uma lista de labels que os *`Pods`* devem ter para que o *Deployment* possa gerenciar.

`.spec.minReadySeconds` é definido como `10`, dizendo ao *Kubernetes* para aguardar `10` segundos entre cada *`Pod`* sendo atualizado. Isso é útil para controlar a taxa de ocorrência da atualizações - esperas mais longas oferecem a chance de detectar problemas e evitar situações em que você atualiza todos os *`Pods`* para uma configuração com defeito.

`.spec.strategy` informa ao *Kubernetes* como realizar as atualizações nos *`Pods`* gerenciados pelo *Deployment*.

*Usando a configuração de `strategy`, podemos dizer ao Deployment como atualizar o aplicativo, seja por meio de um `RollingUpdate` ou `Recreate`.*

### Exibindo o status do rollout do *Deployment*

Você pode usar os comandos usuais `kubectl get deployments` e *kubectl describe deployments* para ver os detalhes da implantação, mas deixe-me indicar um comando adicional, feito especificamente para verificar o status de uma implantação:

Você pode acompanhar o andamento do *`Deployment`* com o status do lançamento:

```bash
kubectl rollout status deployment/myapp-deployment
# deployment myapp-deployment successfully rolled out
```

De acordo o resultado do comando acima, o *`Deployment`* foi implementado com sucesso, então você deve ver as três réplicas do *`Pod`* funcionando corretamente. Vamos ver:

```bash
kubectl get pods
```

A saída do comando anterior é semelhante:

```
NAME                                READY     STATUS    RESTARTS   AGE
myapp-deployment-21724f6785-7ci7o   1/1       Running   0          18s
myapp-deployment-21724f6785-kzszj   1/1       Running   0          18s
myapp-deployment-21724f6785-qqcnn   1/1       Running   0          18s
```

#### Exibindo histórico do rollout do *Deployment*

Reverter um *`Deployment`* é possível porque as implantações mantêm um histórico de revisão. O histórico é armazenado nos *ReplicaSets* subjacentes. Quando um *`Deployment`* é concluído, o *ReplicaSet* antigo não é excluído e isso permite reverter para qualquer revisão, não apenas a anterior. O histórico de revisão pode ser exibido com o comando `kubectl rollout history`:

```bash
kubectl rollout history deployment/myapp-deployment
```

A saída é semelhante a esta:

```
deployments "myapp-deployment"
REVISION    CHANGE-CAUSE
1           kubectl apply --filename=myapp-deployment.yaml
2           kubectl set image deployment/myapp-deployment nginx=nginx:1.16.1
3           kubectl set image deployment/myapp-deployment nginx=nginx:1.161
```

Lembra da opção de linha de comando `--record` que você usou ao criar o *`Deployment`*? Sem ele, a coluna `CHANGE-CAUSE` no histórico de revisão ficaria vazia, tornando muito mais difícil descobrir o que está por trás de cada revisão.

### Entendendo as estratégias de *Deployments* disponíveis

O modo como o novo estado deve ser alcançado é regido pela estratégia do *Deployment* configurada no próprio *Deployment*. A estratégia padrão é realizar uma atualização contínua (a estratégia é chamada `RollingUpdate`). Uma outra alternativa é a estratégia `Recreate`, que exclui todos os *`Pods`* antigos de uma única vez e, em seguida, cria novos, semelhante a modificar o modelo do *`Pod`* de um *ReplicaSet* e, em seguida, excluir todos os *`Pods`*.

A estratégia `Recreate` faz com que todos os *`Pods`* antigos sejam excluídos antes que os novos sejam criados. Use essa estratégia quando seu aplicativo não suportar a execução de várias versões em paralelo e exigir que a versão antiga seja interrompida completamente antes que a nova seja iniciada. Essa estratégia envolve um curto período de tempo quando seu aplicativo fica completamente indisponível.

`Recreate` é um método de implantação muito básico: todos os *`Pods`* do *Deployment* serão excluídos ao mesmo tempo e novos *`Pods`* serão criados com a nova versão. `Recreate` não tem muito controle diante de um *Deployment* incorreto - se os novos *`Pods`* não forem iniciados por algum motivo, o aplicativo não funcionará.

Com `RollingUpdate`, por outro lado, os implantações/atualizações são mais lentas, mas muito mais controladas. Em primeiro lugar, o novo aplicativo/nova versão será lançado aos poucos, *`Pod`* por *`Pod`*. Pode-se especificar valores para `maxSurge` e `maxUnavailable` para ajustar a estratégia.

A estratégia `RollingUpdate`, por outro lado, remove *`Pods`* antigos um a um, enquanto adiciona novos ao mesmo tempo, mantendo o aplicativo disponível durante todo o processo e garantindo que não haja queda na capacidade de lidar com as solicitações. Esta é a estratégia padrão. Os limites superior e inferior para o número de *`Pods`* acima ou abaixo da contagem de réplicas desejada são configuráveis. *Você deve usar essa estratégia apenas quando seu aplicativo puder executar a versão antiga e a nova ao mesmo tempo.*

Os parâmetros `maxSurge` e `maxUnavailable` permitem acelerar ou desacelerar o processo de `RollingUpdate`. `maxUnavailable` permite ajustar o número máximo de *`Pods`* indisponíveis durante o processo de *rollout*. Isso pode ser uma porcentagem ou um número fixo (nunca tenha mais de "x" *`Pods`* abaixo do estado desejado). `maxSurge` permite ajustar o número máximo de *`Pods`* sobre o número de réplicas do *Deployment* que pode ser criado a qualquer momento ou nunca tenha mais de "x" *`Pods`* acima do estado desejado. Da mesma forma que `maxUnavailable`, `maxSurge` pode ser uma porcentagem ou um número fixo.

### Estratégia de `RollingUpdate`

A imagem a seguir mostra uma atualização usando a estratégia de `RollingUpdate`:

![RollingUpdate process for a Deployment](/images/articles/Kubernetes/RollingUpdate%20process%20for%20a%20Deployment.png?id=956a60b05f308b802fdaeb2f4f8362db "RollingUpdate process for a Deployment")

Na imagem acima, o procedimento `RollingUpdate` segue várias etapas principais. O *Deployment* tenta atualizar os *`Pods`*, um por um. Somente depois que um *`Pod`* é atualizado com sucesso, a atualização prossegue para o próximo *`Pod`*.

As atualizações contínuas são o padrão porque protegem contra o tempo de inatividade, mas mesmo assim, o comportamento padrão é bastante agressivo. Para uma versão de produção, você vai querer ajustar algumas configurações que definem a velocidade da versão e como ela é monitorada. Como parte da especificação da atualização contínua, você pode adicionar opções que controlam a rapidez com que o novo *ReplicaSet* é dimensionado e a velocidade com que o antigo *ReplicaSet* é reduzido, usando os dois valores a seguir:

O novo *ReplicaSet* é ampliado até que a contagem dos *`Pods`* seja a contagem de réplicas desejada mais o valor de `maxSurge` e, em seguida, o *Deployment* aguarda a remoção dos *`Pods`* antigos. O *ReplicaSet* antigo é reduzido para a contagem desejada menos a contagem de `maxUnavailable`, então o *Deployment* espera que os novos *`Pods`* atinjam o estado de `Ready`. Você não pode definir os dois valores como zero porque isso significa que nada mudará.

![Deployment updates in progress, using different rollout options](/images/articles/Kubernetes/Deployment%20updates%20in%20progress,%20using%20different%20rollout%20options.png?id=67486e84376ae112301c1d63a9049c93 "Deployment updates in progress, using different rollout options")

Você pode controlar o ritmo da implantação com os dois campos a seguir na especificação de implantação:

- `minReadySeconds`: Adiciona um atraso onde a implantação espera para garantir que os novos *`Pods`* estejam estáveis. Ele especifica o número de segundos que o *`Pod`* deve estar ativo sem nenhum container travando antes de ser considerado bem-sucedido. O padrão é zero, por isso novos *`Pods`* são criados rapidamente durante os lançamentos.

- `progressDeadlineSeconds`: Especifica por quanto tempo uma atualização de implantação pode ser executada antes de ser considerada como falha de progresso. O padrão é `600` segundos, portanto, se uma atualização não for concluída em 5 minutos, ela será sinalizada como falha.

### Apresentando as propriedades `maxSurge` e `maxUnavailable` da estratégia de `rollingUpdate`

**`maxUnavailable`** é usado para reduzir o *ReplicaSet* antigo. Ele define quantos *`Pods`* podem estar indisponíveis durante a atualização, em relação à contagem de *`Pods`* desejadas. Em uma implantação de `10` *`Pods`*, definir isso para `3` significa que três *`Pods`* serão encerrados imediatamente.

**`maxSurge`** é usado para aumentar a escala do novo *ReplicaSet*. Ele define quantos *`Pods`* extras podem existir, além da contagem de réplicas desejada. Em uma implantação de `10`, definir como `4` criará quatro novos *`Pods`*.

Duas propriedades afetam quantos *`Pods`* são substituídos durante a atualização contínua de um *Deployment*. Eles são `maxSurge` e `maxUnavailable` e podem ser definidos como parte de `rollingUpdate` do atributo da estratégia do *Deployment*, conforme mostrado no *YAML* a seguir.

```yml
spec:
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
    type: RollingUpdate
```

**Propriedades para configurar a taxa de rolling update (atualização contínua)**

**`maxSurge`**: Determina quantas instâncias do *`Pod`* você permite que existam acima da contagem de réplicas desejada configurada no *Deployment*. O padrão é `25%`, então pode haver no máximo `25%` mais instâncias do *`Pod`* do que a contagem desejada. Se a contagem de réplicas desejada for definida como quatro, nunca haverá mais de cinco instâncias do *`Pod`* em execução ao mesmo tempo durante uma atualização. Ao converter uma porcentagem em um número absoluto, o número é arredondado para cima. Em vez de uma porcentagem, o valor também pode ser um valor absoluto (por exemplo, um ou dois *`Pods`* adicionais podem ser permitidos).

**`maxUnavailable`**: Determina quantas instâncias do *`Pod`* podem estar indisponíveis em relação à contagem de réplicas desejada durante a atualização. O padrão também é `25%`, portanto, o número de instâncias de *`Pod`* disponíveis nunca deve cair abaixo de `75%` da contagem de réplicas desejada. Ao converter uma porcentagem em um número absoluto, o número é arredondado para baixo. Se a contagem de réplicas desejada for definida como quatro e a porcentagem for `25%`, apenas um *`Pod`* pode estar indisponível. Sempre haverá pelo menos três instâncias de *`Pod`* disponíveis para atender a solicitações durante todo o lançamento. Tal como acontece com `maxSurge`, você também pode especificar um valor absoluto em vez de uma porcentagem.

Como a contagem de réplicas desejada no caso era três e essas propriedades são padronizadas em `25%`, `maxSurge` permitiu que o número de todos os *`Pods`* chegasse a quatro, e `maxUnavailable` não permitia ter *`Pods`* indisponíveis (em outras palavras, três *`Pods`* tinham que estar disponíveis em todas as vezes). Isso é mostrado na imagem abaixo.

![Rolling update of a Deployment with three replicas and default maxSurge and maxUnavailable](/images/articles/Kubernetes/Rolling%20update%20of%20a%20Deployment%20with%20three%20replicas%20and%20default%20maxSurge%20and%20maxUnavailable.png?id=2ca15d749129add7c35e103287f3f48c "Rolling update of a Deployment with three replicas and default maxSurge and maxUnavailable")

**Quando `maxSurge` for `1` e `maxUnavailable` também for `1`:**

![Rolling update of a Deployment with the maxSurge=1 and maxUnavailable=1](/images/articles/Kubernetes/Rolling%20update%20of%20a%20Deployment%20with%20the%20maxSurge=1%20and%20maxUnavailable=1.png?id=0e8488cea9bd413cffff9ec5b83618a8 "Rolling update of a Deployment with the maxSurge=1 and maxUnavailable=1")

Nesse caso, uma réplica pode estar indisponível; portanto, se a contagem de réplicas desejada for três, apenas duas delas precisam estar disponíveis. É por isso que o processo de distribuição exclui imediatamente um *`Pod`* e cria dois novos. Isso garante que dois *`Pods`* estejam disponíveis e que o número máximo de *`Pods`* não seja excedido (o máximo é quatro neste caso - três mais um de `maxSurge`). Assim que os dois novos *`Pods`* estiverem disponíveis, os dois *`Pods`* antigos restantes serão excluídos.

Isso é um pouco difícil de entender, especialmente porque a propriedade `maxUnavailable` leva você a acreditar que esse é o número máximo permitido de *`Pods`* indisponíveis. Se você observar a imagem anterior, verá dois *`Pods`* indisponíveis na segunda coluna, embora `maxUnavailable` esteja definido como `1`.

É importante ter em mente que `maxUnavailable` é relativo à contagem de réplicas desejada. Se a contagem de réplicas for definida como três e `maxUnavailable` for definido como um, isso significa que o processo de atualização deve sempre manter pelo menos dois (3 menos 1) *`Pods`* disponíveis, enquanto o número de *`Pods`* indisponíveis não pode exceder um.

## Horizontal Pod Autoscaler (HPA)

`Deployments` e `ReplicaSets` permitem especificar um número total de réplicas/*`Pods`* que devem estar disponíveis em um determinado momento. No entanto, nenhuma dessas estruturas/controladores permite o dimensionamento automático - elas devem ser dimensionadas manualmente.

**Horizontal Pod Autoscalers** (`HPA`) fornecem essa funcionalidade por existirem como um controlador de nível superior que pode alterar a contagem de réplicas de um ***Deployment*** ou ***ReplicaSet*** com base em métricas, como *CPU* e *uso de memória*.

Por padrão, um *HPA* pode fazer o escalonamento automático com base na utilização da *CPU*, mas usando métricas personalizadas, essa funcionalidade pode ser estendida.

O arquivo *YAML* para um *HPA* se parece com isso:

```yml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  maxReplicas: 5
  minReplicas: 2
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp-deployment
  targetCPUUtilizationPercentage: 70
```

Na especificação acima, tem `.spec.scaleTargetRef`, que especifica o que deve ser escalonado automaticamente pelo *HPA* e os parâmetros de ajuste.

A definição de `.spec.scaleTargetRef` pode ser em `Deployment` ou `ReplicaSet`. Nesse caso, é definido o *HPA* para dimensionar o *Deployment*, `myapp-deployment`.

Para os parâmetros de ajuste, o *YAML* está usando o escalonamento padrão baseado na *utilização da CPU*, portanto, pode-se usar `.spec.targetCPUUtilizationPercentage` para definir a utilização pretendida/desejada da CPU de cada *`Pod`* executando o aplicativo. Se o **uso médio da CPU de todos os `Pods` do *Deployment* aumentar mais que 70%**, o *HPA* aumentará a especificação do *Deployment* e, se ficar abaixo por muito tempo, reduzirá a quantidade de replicas do *Deployment*.

Um evento de dimensionamento típico se parece como o seguinte:

1. *O uso médio da CPU de um Deployment excede 70% nas três réplicas/Pods.*
2. *O loop de controle HPA percebe esse aumento na utilização da CPU.*
3. *O HPA edita `.spec.replicas` do Deployment com uma nova contagem de réplicas. Essa contagem é calculada baseada na utilização da CPU, com a intenção de deixar o uso da CPU por nó em estado estável abaixo de 70%.*
4. *O controlador Deployment gera uma nova réplica.*
5. *Este processo se repete para aumentar ou diminuir as réplicas no Deployment.*

Em resumo, o HPA monitora a utilização da CPU e da memória e inicia um evento de dimensionamento quando os limites são excedidos.

## Introdução `DaemonSets` (Executando exatamente um `Pod` em cada nó)

> `DaemonSets` são semelhantes aos *ReplicaSets*, exceto que o número de réplicas é fixado em uma réplica por nó. Isso significa que cada nó do cluster manterá uma única réplica do aplicativo ativa a qualquer momento.

Isso acaba se parecendo a seguinte diagrama para um *DaemonSet*:

![DaemonSet spread across three nodes](/images/articles/Kubernetes/DaemonSet%20spread%20across%20three%20nodes.png?id=f4592411d617a8a2e0f2696259929a00 "DaemonSet spread across three nodes")

De acordo com a imagem acima, cada nó (representado por uma caixa) contém um *`Pod`* do aplicativo, controlado pelo *DaemonSet*.

*ReplicaSets* são usados ​​para executar um número específico de *`Pods`* implantados no cluster *Kubernetes*. Mas existem certos casos em que você deseja que um *`Pod`* seja executado em cada nó do cluster (e cada nó precisa executar exatamente uma instância do *`Pod`*, conforme mostrado na imagem abaixo).

Esses casos incluem *`Pods`* relacionados à infraestrutura que executam operações no nível do sistema. Por exemplo, você deseja executar um coletor de log e um monitor de recursos em cada nó. Outro bom exemplo é o próprio processo `kube-proxy` do *Kubernetes*, que precisa ser executado em todos os nós para fazer os serviços funcionarem.

![DaemonSets run only a single pod replica on each node, whereas ReplicaSets scatter them around the whole cluster randomly](/images/articles/Kubernetes/DaemonSets%20run%20only%20a%20single%20pod%20replica%20on%20each%20node,%20whereas%20ReplicaSets%20scatter%20them%20around%20the%20whole%20cluster%20randomly.png?id=c4d3661cedcb2de74072a10a47eef448 "DaemonSets run only a single pod replica on each node, whereas ReplicaSets scatter them around the whole cluster randomly")

Isso torna o *DaemonSet* excelente para executar aplicativos que coletam métricas no nível do nó ou fornecem processos de rede por nó. Uma especificação do *DaemonSet* se parece com isto:

```yml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-collector
spec:
  selector:
    matchLabels:
      name: log-collector
  template:
    metadata:
      labels:
        name: log-collector
    spec:
      containers:
      - name: fluentd
        image: fluentd
```

No código *YAML* acima, é muito semelhante à especificação típica do *ReplicaSet*, exceto que não foi especificado o número de réplicas. Isso ocorre porque um *DaemonSet* tentará executar um *`Pod`* em cada nó do cluster.

Se quiser especificar um subconjunto de nós para executar o aplicativo, pode-se fazer isso usando um seletor de nó, conforme mostrado no *YAML* abaixo:

```yml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-collector
spec:
  selector:
    matchLabels:
      name: log-collector
  template:
    metadata:
      labels:
        name: log-collector
    spec:
      nodeSelector:
        type: bigger-node
      containers:
      - name: fluentd
        image: fluentd
```

Este *YAML* restringirá o *DaemonSet* a nós que correspondem ao seletor `type=bigger-node` em seus *labels*.

### Explicando DaemonSet com um exemplo

Vamos imaginar um *daemon* chamado `ssd-monitor` que precisa ser executado em todos os nós que contêm uma unidade de estado sólida (*SSD*). Você criará um *DaemonSet* que executa em todos os nós marcados como tendo um *SSD*. Os administradores do cluster adicionaram o label `disk=ssd` a todos esses nós, então você criará o *DaemonSet* com um *seletor de nó* que seleciona apenas nós com essa label, conforme mostrado na imagem abaixo.

![Using a DaemonSet with a node selector to deploy system pods only on certain nodes](/images/articles/Kubernetes/Using%20a%20DaemonSet%20with%20a%20node%20selector%20to%20deploy%20system%20pods%20only%20on%20certain%20nodes.png?id=8df77e333ec3ae287b3b0ab3ea5d7ece "Using a DaemonSet with a node selector to deploy system pods only on certain nodes")

Você está definindo um *DaemonSet* que executará um *`Pod`* com um único container. Uma instância desse *`Pod`* será criada para cada nó com o *label* `disk=ssd`.

```yml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ssd-monitor
spec:
  selector:
    matchLabels:
      app: ssd-monitor
  template:
    metadata:
      labels:
        app: ssd-monitor
    spec:
      nodeSelector:
        disk: ssd
      containers:
      - name: main
        image: your-image
```

## Introdução `StatefulSets`

`StatefulSets` são muito semelhantes a `ReplicaSets` e `Deployments`, mas com uma diferença fundamental que os torna melhores para cargas de trabalho com estado. *StatefulSets* mantêm a ordem e a identidade de cada *`Pod`*, mesmo se os *`Pods`* forem reagendados em novos nós.

Por exemplo, em um *StatefulSet* de 3 réplicas, sempre haverá `Pod 1`, `Pod 2` e `Pod 3`, e esses *`Pods`* manterão sua identidade no *Kubernetes* e no armazenamento, independentemente se acontecer qualquer reagendamento.

Vamos dar uma olhada em uma configuração simples de *StatefulSet*:

```yml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: stateful
spec:
  selector:
    matchLabels:
      app: stateful-app
  replicas: 5
  template:
    metadata:
      labels:
        app: stateful-app
    spec:
      containers:
      - name: app
        image: nginx
```

Este *YAML* criará um *StatefulSet* com cinco réplicas do aplicativo.

Veja como o *StatefulSet* mantém a identidade do *`Pod`* de maneira diferente de um *Deployment* ou *ReplicaSet*. Vamos buscar todos os *`Pods`* usando o comando:

```bash
kubectl get pods
```

A saída deve ser semelhante a esta:

```
NAME         READY   STATUS    RESTARTS       AGE
stateful-app-0   1/1     Running   0          23s
stateful-app-1   1/1     Running   0          21s
stateful-app-2   1/1     Running   0          19s
stateful-app-3   1/1     Running   0          17s
stateful-app-4   1/1     Pending   0          15s
```

No resultado acima, existe cinco *`Pods`* *StatefulSet*, cada um com um indicador numérico de sua identidade. Esta propriedade é extremamente útil para aplicativos com estado, como um cluster de banco de dados. No caso de executar um cluster de banco de dados no *Kubernetes*, a identidade dos *`Pods`* mestre versus réplica é importante, e podemos usar identidades *StatefulSet* para gerenciar isso facilmente.

Outro ponto interessante é que você pode ver que o *`Pod`* final (`stateful-app-4`) ainda está iniciando e que a tempo-`AGE` do *`Pod`* aumenta conforme a identidade numérica diminui. Isso ocorre porque os *`Pods`* *StatefulSet* são criados um de cada vez, em ordem.

*StatefulSets são valiosos em conjunto com o armazenamento Kubernetes persistente para executar aplicativos com estado.*
