---
id: 9c45aa90-b90d-4759-a3b3-b171d682ae9d
title: "Controles de Posicionamento do Pod"
summary: "Saiba como posicionar o Pod em determinado Nó de acordo com algumas regras. Saiba mais um pouco de nodeAffinity e podAffinity."
---

> Posicionamento do *`Pod`* significa controlar *para qual nó* um *`Pod`* está programado no *Kubernetes*.

## Identificando Casos de Uso Para Colocação do *Pod*

Os controles de posicionamento do *`Pod`* são ferramentas que o *Kubernetes* fornece para decidir em qual nó agendar um *`Pod`* ou impedir o agendamento do *`Pod`* para determinados nós. Isso pode ser usado em vários padrões diferentes, mas revisaremos alguns dos principais. Para começar, o próprio *Kubernetes* implementa controles de posicionamento do *`Pod`* por padrão - vamos ver como.

### Controles de posicionamento de integridade do nó

O *Kubernetes* usa alguns controles de posicionamento padrão para especificar quais nós não são *unhealthy* de alguma forma. Estes são geralmente definidos usando *taints* e tolerâncias, que revisaremos em detalhes mais adiante.

Algumas *taints* padrão (que discutiremos na próxima seção) que o *Kubernetes* usa são as seguintes:

- *memory-pressure*
- *disk-pressure*
- *unreachable*
- *not-ready*
- *out-of-disk*
- *network-unavailable*
- *unschedulable*
- *uninitialized (only for cloud-provider-created nodes)*

Essas condições podem marcar nós como incapazes de receber novos *`Pods`*, embora haja alguma flexibilidade na forma como essas taints são tratadas pelo *scheduler*, como veremos mais adiante. O objetivo desses controles de posicionamento criados pelo sistema é evitar que nós *unhealthy* recebam cargas de trabalho que podem não funcionar corretamente.

Além dos controles de posicionamento criados pelo sistema para integridade do nó, existem vários casos de uso em que você, como usuário, pode querer implementar um agendamento ajustado, como veremos na próxima seção.

### Aplicativos que exigem diferentes tipos de nós

Em um cluster *Kubernetes* heterogêneo, cada nó não é criado igual. Você pode ter algumas VMs mais poderosas (ou bare metal) e algumas menos - ou ter diferentes conjuntos especializados de nós.

Por exemplo, em um cluster que executa pipelines de ciência de dados, você pode ter nós com recursos de aceleração de GPU para executar algoritmos de deep learning, nós de computação regulares para atender aplicativos, nós com grandes quantidades de memória para fazer inferência com base em modelos concluídos e muito mais.

Usando os controles de posicionamento do *`Pod`*, você pode garantir que as várias partes da sua plataforma sejam executadas no hardware mais adequado para a tarefa em questão.

### Aplicativos que exigem conformidade de dados específicos

Semelhante ao exemplo anterior, onde os requisitos do aplicativo podem ditar a necessidade de diferentes tipos de computação, certas necessidades de conformidade de dados podem exigir tipos específicos de nós.

Por exemplo, provedores de nuvem como AWS e Azure geralmente permitem que você compre VMs com tenancy dedicado - o que significa que nenhum outro aplicativo é executado no hardware e hipervisor subjacentes. Isso é diferente de outras VMs típicas de provedor de nuvem, onde vários clientes podem compartilhar uma única máquina física.

Para certos regulamentos de dados, esse nível de tenancy dedicado é necessário para manter a conformidade. Para atender a essa necessidade, você pode usar os controles de posicionamento do *`Pod`* para garantir que os aplicativos relevantes sejam executados apenas em nós com tenancy dedicado, reduzindo os custos executando o plano de controle em VMs mais típicas sem ele.

### Multi-tenant clusters

Se você estiver executando um cluster com vários tenants (separados por namespaces, por exemplo), poderá usar os controles de posicionamento do *`Pod`* para reservar certos nós ou grupos de nós para um tenant, para separá-los fisicamente ou de outra forma de outros tenants no cluster. Isso é semelhante ao conceito de hardware dedicado na AWS ou Azure.

### Vários domínios de falha

Embora o *Kubernetes* já forneça alta disponibilidade, permitindo que você agende cargas de trabalho executadas em vários nós, também é possível estender esse padrão. Podemos criar nossas próprias estratégias de agendamento do *`Pod`* que levam em conta domínios de falha que se estendem por vários nós. Uma ótima maneira de lidar com isso é através dos recursos do *`Pod`* ou afinidade/antiafinidade do nó, que discutiremos mais adiante.

Por enquanto, vamos conceituar um caso em que temos nosso cluster em bare metal com 20 nós por rack físico. Se cada rack tiver sua própria conexão de energia dedicada e backup, ele pode ser pensado como um domínio de falha. Quando as conexões de energia falham, todas as máquinas no rack falham. Assim, podemos querer incentivar o *Kubernetes* a executar duas instâncias ou *`Pods`* em racks/domínios de falha separados. A imagem a seguir mostra como um aplicativo pode ser executado em domínios de falha:

<img src="{{ mix('/images/articles/Kubernetes/Failure domains.png') }}" alt="Failure domains">

Como você pode ver na imagem acima, como os *`Pods`* do aplicativo estão espalhados por vários domínios de falha, não apenas vários nós no mesmo domínio de falha, podemos manter o tempo de atividade mesmo que o Domínio de Falha 1 diminua. O aplicativo `A - Pod 1` e o aplicativo `B - Pod 1` estão no mesmo domínio de falha (vermelho). No entanto, se esse domínio de falha (`Rack 1`) cair, ainda teremos uma réplica de cada aplicativo no `Rack 2`.

Usamos a palavra "incentivar" porque é possível configurar algumas dessas funcionalidades como um requisito difícil ou com base no melhor esforço no agendador Kubernetes.

Esses exemplos devem lhe dar uma compreensão sólida de alguns casos de uso em potencial para controles avançados de posicionamento.

Vamos discutir a implementação real agora.

## Controlando *Pods* com Afinidade do Nó

Existem dois tipos de afinidade:

- *Afinidade do nó*
- *Afinidade entre Pods*

A afinidade de nós é um conceito semelhante aos *seletores de nós*, exceto que permite um conjunto muito mais robusto de características de seleção. Vejamos algum exemplo de *YAML* e depois separamos as várias partes:

```yml
apiVersion: v1
kind: Pod
metadata:
  name: affinity-test
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: cpu_speed
            operator: In
            values:
            - fast
            - medium_fast
  containers:
  - name: speedy-app
    image: speedy-app:latest
```

Como você pode ver, nossa especificação do *`Pod`* tem uma chave de afinidade e especificamos uma configuração `nodeAffinity`. Existem dois tipos possíveis de afinidade de nós:

- `requiredDuringSchedulingIgnoredDuringExecution`
    - O agendador não pode agendar o *`Pod`* a menos que a regra seja atendida. Funciona como `nodeSelector`, mas com uma sintaxe mais expressiva.
- `preferredDuringSchedulingIgnoredDuringExecution`
    - O agendador tenta encontrar um nó que atenda à regra. Se um nó correspondente não estiver disponível, o agendador ainda agendará o pod.

A funcionalidade desses dois tipos é mapeada diretamente para como o `NoSchedule` e o `PreferNoSchedule`, respectivamente.

`NoSchedule`, fornecem efeitos rígidos - o que quer dizer que, como os *seletores de nó*, existem apenas duas possibilidades, ou a tolerância existe no *`Pod`* (como veremos em breve) ou o *`Pod`* não está agendado.

`PreferNoSchedule`, por outro lado, fornece ao *scheduler Kubernetes* alguma margem de manobra. Ele informa ao *scheduler* para tentar encontrar um nó para um *`Pod`* que não tenha uma *untolerated taint*, mas se não existir, vá em frente e programe-o de qualquer maneira. Ele implementa um efeito suave/soft.

### Usando afinidade do nó `requiredDuringSchedulingIgnoredDuringExecution`

Para `requiredDuringSchedulingIgnoredDuringExecution`, o *Kubernetes* nunca agendará um *`Pod`* sem um termo que correspondente a um nó.

Para `preferredDuringSchedulingIgnoredDuringExecution`, ele tentará cumprir o requisito suave, mas se não puder, ainda agendará o *`Pod`*.

A real capacidade de *afinidade do nó* sobre seletores de nós e taints e tolerations vem nas expressões e lógicas reais que você pode implementar quando se trata do *seletor*.

As funcionalidades de afinidades `requiredDuringSchedulingIgnoredDuringExecution` e `preferredDuringSchedulingIgnoredDuringExecution` são bem diferentes, então revisaremos cada uma separadamente.

Para a afinidade `required`, temos a capacidade de especificar `nodeSelectorTerms`, que podem ser um ou mais blocos contendo `matchExpressions`. Para cada bloco de `matchExpressions`, pode haver várias expressões.

No bloco de código que vimos na seção anterior, temos um único termo de seletor de nós, um bloco `matchExpressions` - que por sua vez tem apenas uma única expressão. Esta expressão procura uma `key`, que, assim como com os *seletores de nós*, representa uma label do nó. Em seguida, tem um `operator`, o que nos dá alguma flexibilidade sobre como queremos identificar uma correspondência. Aqui estão os valores possíveis para o `operator`:

- *`In`*
- *`NotIn`*
- *`Exists`*
- *`DoesNotExist`*
- *`Gt`* (*greater than*)
- *`Lt`* (*less than*)

No nosso caso, estamos usando o operador `In`, que verificará se o valor é um dos vários que especificamos. Finalmente, na seção de `values`, podemos listar um ou mais valores que devem corresponder, com base no operador, antes que a expressão seja verdadeira.

Como você pode ver, isso nos dá granularidade significativamente maior na especificação do nosso seletor. Vejamos nosso exemplo com `cpu_speed` usando um operador diferente:

```yml
apiVersion: v1
kind: Pod
metadata:
  name: affinity-test
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: cpu_speed
            operator: Gt
            values:
            - "5"
  containers:
  - name: speedy-app
    image: speedy-app:latest
```

Como você pode ver, estamos usando um seletor `matchExpressions` muito granular. Essa capacidade de usar correspondência de operador mais avançada agora nos permite garantir que nosso `speedy-app` seja agendado apenas em nós que tenham uma velocidade de clock alta o suficiente (neste caso, 5 GHz). Em vez de classificar nossos nós em grupos amplos, como lento e rápido, podemos ser muito mais granulares em nossas especificações.

Em seguida, vamos dar uma olhada no outro tipo de afinidade do nó `preferredDuringSchedulingIgnoredDuringExecution`.

### Usando afinidade do nó `preferredDuringSchedulingIgnoredDuringExecution`

A sintaxe para isso é um pouco diferente e nos dá ainda mais granularidade para afetar esse requisito suave-soft. Vejamos uma especificação YAML do *`Pod`* que implementa isso:

```yml
apiVersion: v1
kind: Pod
metadata:
  name: slow-app-affinity
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: cpu_speed
            operator: Lt
            values:
            - "3"
  containers:
  - name: slow-app
    image: slow-app:latest
```

Isso parece um pouco diferente da sintaxe de `required`.

Para `preferredDuringSchedulingIgnoredDuringExecution`, temos a capacidade de atribuir um peso a cada entrada, com uma preferência associada, que pode novamente ser um bloco `matchExpressions` com várias expressões internas que usam a mesma sintaxe `key-operator-values`.

O valor do `weight` é a principal diferença aqui. Como `preferredDuringSchedulingIgnoredDuringExecution` é um requisito **soft**, podemos listar algumas preferências diferentes com pesos associados e deixar o *scheduler* tentar o seu melhor para satisfazê-las. A maneira como isso funciona internamente é que o *scheduler* passará por todas as preferências e calculará uma pontuação para o nó com base no peso de cada preferência e se foi satisfeita. Assumindo que todos os requisitos rígidos sejam atendidos, o *scheduler* selecionará o nó com a maior pontuação calculada. No caso anterior, temos uma única preferência com um peso de `1`, mas o peso pode variar de `1` a `100`, então vamos ver uma configuração mais complexa para o nosso caso de uso `speedy-app`:

```yml
apiVersion: v1
kind: Pod
metadata:
  name: speedy-app-prefers-affinity
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 90
        preference:
          matchExpressions:
          - key: cpu_speed
            operator: Gt
            values:
            - “3”
      - weight: 10
        preference:
          matchExpressions:
          - key: memory_speed
            operator: Gt
            values:
            - "4"
  containers:
  - name: speedy-app
    image: speedy-app:latest
```

Em nossa jornada para garantir que o `speedy-app` seja executado no melhor nó possível, decidimos aqui implementar apenas requisitos suaves/*`soft`*. Se não houver nós rápidos, ainda queremos que nosso aplicativo seja agendado e executado. Para esse fim, especificamos duas preferências - um nó com uma velocidade de `cpu_speed` superior a `3` (3 GHz) e uma velocidade de memória superior a `4` (4 GHz).

Como nosso aplicativo está muito mais ligado à CPU do que à memória, decidimos ponderar nossas preferências adequadamente. Nesse caso, `cpu_speed` carrega um peso de `90`, enquanto `memory_speed` carrega um peso de `10`.

Assim, qualquer nó que satisfaça nosso requisito `cpu_speed` terá uma pontuação calculada muito maior do que uma que satisfaça apenas o requisito `memory_speed` - mas ainda menos do que um que satisfaça ambos. Quando estamos tentando agendar `10` ou `100` novos *`Pods`* para este aplicativo, você pode ver como esse cálculo pode ser valioso.

### Afinidades de vários nós

Quando estamos lidando com afinidades de vários nós, há algumas peças-chave da lógica a ter em mente. Primeiro, mesmo com *uma única afinidade de nó*, se for combinada com um *seletor de nós* na mesma especificação do *`Pod`* (o que é realmente possível), o *seletor de nós* deve ser satisfeito antes que qualquer lógica de afinidade de nós entre em jogo. Isso ocorre porque os *seletores de nós* implementam apenas requisitos rígidos e não há operador lógico `OR` entre os dois. Um operador lógico `OR` verificaria ambos os requisitos e garantiria que pelo menos um deles seja verdadeiro - mas os *seletores de nós* não nos permitem fazer isso.

Em segundo lugar, para afinidade do nó `requiredDuringSchedulingIgnoredDuringExecution`, várias entradas do tipo `matchExpressions` em `nodeSelectorTerms` são tratadas em um operador lógico `OR`. Se um, mas não todos, estiver satisfeito - o *`Pod`* ainda será agendado.

Finalmente, para qualquer `nodeSelectorTerm` com vários valores dentro de `matchExpressions`, tudo deve estar satisfeito - este é um operador lógico `AND`. Vejamos um exemplo *YAML* disso:

```yml
apiVersion: v1
kind: Pod
metadata:
  name: affinity-test
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: cpu_speed
            operator: Gt
            values:
            - "5"
          - key: memory_speed
            operator: Gt
            values:
            - "4"
  containers:
  - name: speedy-app
    image: speedy-app:latest
```

Nesse caso, se um nó tiver uma velocidade de CPU de `5`, mas não atender ao requisito de velocidade de memória (ou vice-versa), o *`Pod`* não será agendado.

Em seguida, analisaremos a afinidade entre *`Pods`* e antiafinidade, que fornecem definições de afinidade entre *`Pods`*, em vez de definir regras para os nós.

## Usando Afinidade e Antiafinidade Entre *Pods*

A afinidade entre *`Pods`* e antiafinidade permitem que você dite como os *`Pods`* devem ser executados com base em quais outros *`Pods`* já existem em um nó. Como o número de *`Pods`* em um cluster é tipicamente muito maior do que o número de nós, e algumas regras de afinidade e antiafinidade do *`Pod`* podem ser um pouco complexas, esse recurso pode colocar uma carga no plano de controle do cluster se você estiver executando muitos *`Pods`* em muitos nós. Por esse motivo, *a documentação do Kubernetes não recomenda o uso desses recursos com um grande número de nós em seu cluster*.

Afinidades e antiafinidades do *`Pod`* funcionam de maneira bastante diferente - vamos olhar para cada uma antes de discutir como eles podem ser combinados.

### Afinidade do *Pod*

Tal como acontece com as *afinidades dos nós*, vamos analisar no *YAML* para discutir as partes de uma especificação de afinidade do *`Pod`*:

```yml
apiVersion: v1
kind: Pod
metadata:
  name: not-hungry-app-affinity
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: hunger
            operator: In
            values:
            - "1"
            - "2"
        topologyKey: rack
  containers:
  - name: not-hungry-app
    image: not-hungry-app:latest
```

Assim como com a afinidade do nó, a afinidade do *`Pod`* nos permite escolher entre dois tipos:

- *`preferredDuringSchedulingIgnoredDuringExecution`*
- *`requiredDuringSchedulingIgnoredDuringExecution`*

Novamente, semelhante à *afinidade do nó*, podemos ter um ou mais seletores - que são chamados `labelSelector`, já que estamos selecionando *`Pods`*, não nós. A funcionalidade de `matchExpressions` é a mesma da *afinidade do nó*, mas a afinidade do *`Pod`* adiciona uma nova chave chamada `topologyKey`.

`topologyKey` é, em essência, um seletor que limita o escopo de onde o *scheduler* deve procurar para ver se outros *`Pods`* do mesmo seletor estão em execução. Isso significa que a afinidade do *`Pod`* não precisa significar apenas outros *`Pods`* do mesmo tipo (seletor) no mesmo nó; pode significar grupos de vários nós.

Vamos voltar ao nosso exemplo de domínio de falha no início. Nesse exemplo, cada rack era seu próprio domínio de falha com vários nós por rack. Para estender esse conceito ao `topologyKey`, poderíamos rotular cada nó em um rack com `rack=1` ou `rack=2`. Então podemos usar o `topologyKey` de `rack`, como temos em nosso *YAML*, para designar que o *scheduler* deve verificar todos os *`Pods`* em execução em nós com a mesma `topologyKey` (o que, neste caso, significa todos os *`Pods`* no `Node 1` e `Node 2` no mesmo rack) para aplicar regras de afinidade ou antiafinidade do *`Pod`*.

Então, somando tudo isso, o que nosso exemplo *YAML* diz ao *scheduler* é o seguinte:

- Este *`Pod`* *DEVE* ser agendado em um nó com a label de `rack`, onde o valor da label de `rack` separa os nós em grupos.

- O *`Pod`* será então agendado em um grupo onde já existe um *`Pod`* rodando com a label `hunger` e um valor de `1` ou `2`.

Essencialmente, estamos dividindo o cluster em topologia de domínios - neste caso, racks - e prescrevendo ao *scheduler* apenas para agendar *`Pods`* semelhantes juntos em nós que compartilham o mesmo domínio de topologia. Este é o oposto do nosso primeiro exemplo de domínio de falha, onde não gostaríamos que os *`Pods`* compartilhassem o mesmo domínio, se possível - mas também há razões pelas quais você pode querer manter como *`Pods`* no mesmo domínio. Por exemplo, em uma configuração multitenant em que os tenants querem tenancy de hardware dedicado sobre um domínio, você pode garantir que todos os *`Pods`* que pertencem a um determinado tenant sejam agendados para exatamente o mesmo domínio da topologia.

Você pode usar `preferredDuringSchedulingIgnoredDuringExecution` da mesma maneira. Antes de chegarmos às antiafinidades, aqui está um exemplo com afinidades do *`Pod`* e o tipo `preferred`:

```yml
apiVersion: v1
kind: Pod
metadata:
  name: not-hungry-app-affinity
spec:
  affinity:
    podAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 50
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: hunger
              operator: Lt
              values:
              - "3"
          topologyKey: rack
  containers:
  - name: not-hungry-app
    image: not-hungry-app:latest
```

Como antes, neste bloco de código, temos nosso `weight` - neste caso, `50` - e nossa correspondência de expressão - neste caso, usando um operador *menor que* (`Lt`). Essa afinidade induzirá o *scheduler* a tentar o seu melhor para agendar o *`Pod`* em um nó onde está ou com outro nó no mesmo rack que tem um *`Pod`* funcionando com um `hunger` menor que `3`. O `weight`-peso é usado pelo *scheduler* para comparar nós - conforme discutido na seção sobre afinidades de nós. Especificamente neste cenário, o peso de `50` não faz diferença porque há apenas uma entrada na lista de afinidades.

As *antiafinidades dos `Pods`* estendem esse paradigma usando os mesmos seletores e topologias - vamos dar uma olhada neles em detalhes.

### Anti-afinidades do *Pod*

As anti-afinidades do *`Pod`* permitem que seja **evitado** que os *`Pods`* sejam executados no mesmo domínio de topologia que os *`Pods`* que correspondem a um seletor. Eles implementam a lógica oposta às *afinidades do `Pod`* Vamos analisar em alguns *YAML* e explicar como isso funciona:

```yml
apiVersion: v1
kind: Pod
metadata:
  name: hungry-app
spec:
  affinity:
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: hunger
              operator: In
              values:
              - "4"
              - "5"
          topologyKey: rack
  containers:
  - name: hungry-app
    image: hungry-app
```

Semelhante à afinidade do *`Pod`*, usamos a chave `affinity` como local para especificar nossa antiafinidade em `podAntiAffinity`. Além disso, assim como na afinidade do *`Pod`*, temos a capacidade de usar `preferredDuringSchedulingIgnoredDuringExecution` ou `requireDuringSchedulingIgnoredDuringExecution`. Nós até usamos a mesma sintaxe para o seletor como com afinidade do *`Pod`*.

A única diferença real na sintaxe é o uso de `podAntiAffinity` sob a chave de `affinity`.

Então, o que esse *YAML* faz? Nesse caso, recomendamos ao *scheduler* (um requisito soft) que ele tente agendar este *`Pod`* em um nó onde ele ou qualquer outro nó com o mesmo valor para o *label* de `rack` não tenha *`Pods`* rodando com valores do label de `hunger` de `4` ou `5`. Estamos dizendo ao *scheduler* para tentar não colocar este *`Pod`* em um domínio com nenhum *`Pod`* com os requisitos especificados.

Esse recurso nos dá uma ótima maneira de separar os *`Pods`* por domínio de falha - podemos especificar cada rack como um domínio e dar-lhe um antiafinidade com um seletor de seu próprio tipo. Isso fará com que o *scheduler* programe clones do *`Pod`* (ou tente, em uma afinidade preferida) para nós que não estão no mesmo domínio de falha, dando ao aplicativo maior disponibilidade em caso de falha de domínio.

Temos até a opção de combinar afinidades e antiafinidades do *`Pod`*. Vamos ver como isso pode funcionar.

### Combinando Afinidade e Antiafinidade

Esta é uma daquelas situações em que você pode realmente colocar carga indevida no seu plano de controle do cluster. Combinar afinidades com anti-afinidades do *`Pod`* pode permitir regras incrivelmente diferenciadas que podem ser passadas para o *scheduler*, que tem a tarefa hercúlea de trabalhar para cumpri-las.

Vejamos alguns *YAML* para obter uma especificação de Deployment que combine esses dois conceitos. Lembre-se, afinidade e antiafinidade são conceitos que são aplicados a *`Pods`* - mas normalmente não especificamos *`Pods`* sem um controlador como uma Deployment ou um ReplicaSet. Portanto, essas regras são aplicadas no nível de especificação do *`Pod`* no *YAML* de Deployment.

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hungry-app-deployment
spec:
  selector:
    matchLabels:
      app: hungry-app
  replicas: 3
  template:
    metadata:
      labels:
        app: hungry-app
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - other-hungry-app
            topologyKey: "rack"
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - hungry-app-cache
            topologyKey: "rack"
      containers:
      - name: hungry-app
        image: hungry-app:latest
```

Neste bloco de código, estamos dizendo ao *scheduler* para manipular os *`Pods`* em nossa Deployment como: o *`Pod`* deve ser agendado em um nó com um label de `rack`, de modo que ele ou qualquer outro nó com um label de `rack` e o mesmo valor tenha um *`Pod`* com `app=hungry-labelcache`.

Em segundo lugar, o *scheduler* deve tentar agendar o *`Pod`*, se possível, para um nó com o label de `rack`, de modo que ele ou qualquer outro nó com o label de `rack` e o mesmo valor não tenha um *`Pod`* com o label `app=other-hungry-app` em execução.

Para resumir isso, queremos que nossos *`Pods`* para `hungry-app` sejam executados na mesma topologia que o cache de `hungry-app-cache`, e não queremos que eles estejam na mesma topologia que `other-hungry-app`, se possível.

Como com grande poder vem grande responsabilidade, e nossas ferramentas para afinidade e antiafinidade do *`Pod`* são partes iguais poderosas e redutoras de desempenho, o *Kubernetes* garante que alguns limites sejam definidos sobre as possíveis maneiras pelas quais você pode usar os dois para evitar comportamentos estranhos ou problemas significativos de desempenho.

### Afinidade e antiafinidade do *Pod* e namespaces

Como afinidades e antiafinidades do *`Pod`* causam mudanças de comportamento com base na localização de outros *`Pods`*, os namespaces são uma peça relevante para decidir quais *`Pods`* contam para ou contra uma afinidade ou antiafinidade.

Por padrão, o *scheduler* olhará apenas para o namespace no qual o *`Pod`* com afinidade ou antiafinidade foi criado. Para todos os nossos exemplos anteriores, não especificamos um namespace, então o namespace padrão será usado.

Se você quiser adicionar um ou mais namespaces nos quais os *`Pods`* afetarão a afinidade ou antiafinidade, você pode fazê-lo usando o seguinte *YAML*:

```yml
apiVersion: v1
kind: Pod
metadata:
  name: hungry-app
spec:
  affinity:
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: hunger
              operator: In
              values:
              - “4”
              - “5”
          topologyKey: rack
          namespaces: ["frontend", "backend", "logging"]
  containers:
  - name: hungry-app
    image: hungry-app
```

Neste bloco de código, o *scheduler* olhará para os namespaces `frontend`, `backend` e `log` ao tentar corresponder ao antiafinidade (como você pode ver na chave `namespaces` no bloco `podAffinityTerm`). Isso nos permite restringir em quais namespaces o *scheduler* opera ao validar suas regras.
