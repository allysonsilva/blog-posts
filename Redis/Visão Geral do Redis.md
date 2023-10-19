---
id: 9a66e0cf-02e5-4596-90b0-3df0bb6f6928
title: Visão Geral do Redis
summary: "Visão geral dos conceitos básicos do Redis."
---

| **Structure type**  | **What it contains**                                                           | **Structure read/write ability**                                                                                           |
|---------------------|--------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------|
| `STRING`            | *Strings, integers, or floating point values*                                  | Operate on the whole string, parts, increment/decrement the integers and floats.                                           |
| `LIST`              | *Linked list of strings*                                                       | Push or pop items from both ends, trim based on offsets, read individual or multiple items, find or remove items by value. |
| `SET`               | *Unordered collection of unique strings*                                       | Add, fetch, or remove individual items, check membership, intersect, union, difference, fetch random items.                |
| `HASH`              | *Unordered hash table of keys to values*                                       | Add, fetch, or remove individual items, fetch the whole hash.                                                              |
| `ZSET (sorted set)` | *Ordered mapping of string members to floating-point scores, ordered by score* | Add, fetch, or remove individual values, fetch items based on score ranges or member value.                                |

## [STRINGs](https://redis.io/docs/data-types/strings/)

No _Redis_, `STRING`s são usadas ​​para armazenar três tipos de valores:

- ***Byte string values***
- ***Integer values***
- ***Floating-point values***

> *Strings* são os tipos de dados mais versáteis no Redis porque têm muitos comandos e múltiplos propósitos. Uma *String* pode se comportar como um valor inteiro, float, string de texto ou bitmap com base em seu valor e nos comandos usados. Ele pode armazenar qualquer tipo de dados: texto (XML, JSON, HTML ou raw text), inteiros, flutuantes ou dados binários (vídeos, imagens ou arquivos de áudio). Um valor *String* não pode exceder 512 MB de texto ou dados binários.

![An example of a STRING, world, stored under a key, hello](/images/articles/Redis/an-example-of-a-STRING-world-stored-under-a-key-hello.svg?id=38fde0aac922e7009494f50915fe4f16 "An example of a STRING, world, stored under a key, hello")

Os tipos de string são os tipos de dados básicos no *Redis*. Embora na terminologia do Redis, string pode ser considerada uma matriz de bytes que pode conter strings, inteiros, imagens, arquivos e objetos serializáveis. Essas matrizes de bytes são *binary safe* por natureza e o tamanho máximo que podem conter é *512 MB*.

### Comandos usados na manipulação de strings

- [**`SET key value [EX seconds|PX milliseconds|EXAT timestamp|PXAT milliseconds-timestamp|KEEPTTL] [NX|XX] [GET]`**](https://redis.io/commands/set): *Armazena uma chave com seu respectivo valor. Caso já exista uma chave definida, seu valor é sobrescrito;*

  **Definindo uma chave com tempo de expiração em segundos - `EX`:**

  ```bash
  redis> DEL mykey
  redis> SET mykey "Hello" EX 10 # OK
  # After 5 Seconds
  redis> TTL mykey               # (integer) 5
  # Wait 10 Seconds
  redis> GET mykey               # (nil)
  redis> TTL mykey               # (integer) -2 (Key is expired or does not exist)
  ```

  **Atualizando apenas o valor da chave e mantendo seu tempo de expiração - `KEEPTTL`:**

  ```bash
  redis> DEL mykey
  redis> SET mykey "Will expire in a minute" EX 60 # OK
  redis> GET mykey                                 # "Will expire in a minute"
  redis> TTL mykey                                 # (integer) 50
  redis> SET mykey "Other value" KEEPTTL           # OK
  redis> TTL mykey                                 # (integer) 30
  redis> GET mykey                                 # "Other value"
  # After 1 minute
  redis> GET mykey                                 # (nil)
  redis> TTL mykey                                 # (integer) -2
  ```

  **Atualizando o valor da chave apenas se a mesma não existir - `NX`:**

  ```bash
  redis> DEL mykey
  redis> SET mykey "Hello"           # OK
  redis> GET mykey                   # "Hello"
  redis> TTL mykey                   # (integer) -1 (Key exists but has no expiration time set)
  redis> SET mykey "Other value" NX  # (nil)
  redis> GET mykey                   # "Hello"
  ```

  **Atualizando o valor da chave quando a mesma existir - `XX`:**

  ```bash
  redis> DEL mykey
  redis> SET mykey "Hello"           # OK
  redis> GET mykey                   # "Hello"
  redis> SET mykey "Other value" XX  # OK
  redis> GET mykey                   # "Other value"
  ```

  **Atualizando o valor da chave e retornando seu valor anterior a atualização - `GET`:**

  ```bash
  redis> DEL mykey
  redis> SET mykey "Hello"            # OK
  redis> GET mykey                    # "Hello"
  redis> SET mykey "Other value" GET  # "Hello"
  redis> GET mykey                    # "Other value"
  ```

- [**`GET key`**](https://redis.io/commands/get): *Retorna o valor correspondente à chave informada;*

- [**`GETDEL key`**](https://redis.io/commands/getdel): *Retorna o valor da chave antes de excluí-la. Este comando é semelhante ao `GET`, exceto pelo fato de que após recuperar o valor da chave, a mesma é excluída;*

  ```bash
  redis> DEL mykey
  redis> SET mykey "Hello"            # OK
  redis> GETDEL mykey                 # "Hello"
  redis> GET mykey                    # (nil)
  ```

- [**`GETEX key [EX seconds|PX milliseconds|EXAT timestamp|PXAT milliseconds-timestamp|PERSIST]`**](https://redis.io/commands/getex): *Recupera o valor da chave e, opcionalmente, você pode definir um tempo de expiração. `GETEX` é semelhante ao `GET`, com opção adicional de expiração da chave;*

  ```bash
  redis> DEL mykey
  redis> SET mykey "Hello"            # OK
  redis> GETEX mykey                  # "Hello"
  redis> TTL mykey                    # (integer) -1
  redis> GETEX mykey EX 60            # "Hello"
  redis> TTL mykey                    # (integer) 50
  redis> GETEX mykey PERSIST          # "Hello"
  redis> TTL mykey                    # (integer) -1
  ```

- [**`APPEND key value`**](https://redis.io/commands/append): *Concatena um novo valor ao valor atual de uma chave e altera o valor atual da chave;*

  ```bash
  redis> EXISTS mykey          # (integer) 0
  redis> APPEND mykey "Hello"  # (integer) 5
  redis> APPEND mykey " World" # (integer) 11
  redis> GET mykey             # "Hello World"
  ```

- [**`MGET key [key...]`**](https://redis.io/commands/mget): *Retorna os valores correspondentes às chaves informadas;*

  ```bash
  redis> SET key1 "Hello"  # OK
  redis> SET key2 "World"  # OK
  redis> MGET key1 key2 nonexisting
          # 1) "Hello"
          # 2) "World"
          # 3) (nil)
  ```

- [**`INCR key`**](https://redis.io/commands/incr): *Incrementa (adiciona 1) ao valor (número inteiro) da chave;*

- [**`INCRBY key increment`**](https://redis.io/commands/incrby): *Incrementa ou decrementa o valor (número inteiro) da chave conforme o valor do incremento;*

- [**`INCRBYFLOAT key increment`**](https://redis.io/commands/incrbyfloat): *Incrementa ou decrementa o valor (número de ponto flutuante) da chave conforme o valor do argumento `increment`;*

- [**`DECR key`**](https://redis.io/commands/decr): *Decrementa (remove 1) ao valor (número inteiro) da chave;*

- [**`SETNX key value`**](https://redis.io/commands/setnx): *Define um valor em relação a chave se a mesma não existir. Nesse caso, é igual a `SET`. Quando a chave já contém um valor, nenhuma operação é executada. `SETNX` é a abreviação de "SET if Not eXists";*

  ```bash
  redis> SETNX mykey "Hello"  # (integer) 1
  redis> SETNX mykey "World"  # (integer) 0
  redis> GET mykey            # "Hello"
  ```

- [**`GETSET key value`**](https://redis.io/commands/getset): *Obtém o valor antigo e define um novo valor. Define um novo valor para um chave já existente e retorna o seu valor anterior (antes de ser redefinido pelo comando);*

  ```bash
  redis> DEL mykey            # (integer) 1
  redis> SET mykey "Hello"    # OK
  redis> GETSET mykey "World" # "Hello"
  redis> GET mykey            # "World"
  ```

- [**`MSET key value [key value ...]`**](https://redis.io/commands/mset): *Armazena um ou mais conjuntos de chave valor. Caso uma chave informada já exista, seu valor será sobrescrito pelo novo;*

  ```bash
  redis> MSET first "First Key value" second "Second Key value" # OK
  redis> MGET first second
          # 1) "First Key value"
          # 2) "Second Key value"
  ```

- [**`MSETNX key value [key value ...]`**](https://redis.io/commands/msetnx): *Define todos os valores correspondentes das chaves se todas as chaves não existirem, ou seja, se houver uma única chave já existe, nenhuma operação será executada, nenhuma chave será definada. `MSETNX` é atômico, então todas as chaves fornecidas são definidas de uma vez. Não é possível para os clientes ver que algumas das chaves foram atualizadas enquanto outras permanecem inalteradas;*

  ```bash
  redis> MSETNX key1 "Hello" key2 "there"  # (integer) 1
  redis> MSETNX key2 "new" key3 "world"    # (integer) 0
  redis> MGET key1 key2 key3
          # 1) "Hello"
          # 2) "there"
          # 3) (nil)
  ```

## [LISTs](https://redis.io/docs/data-types/lists/)

> *LISTs* no Redis armazenam uma sequência ordenada de strings.

![An example of a LIST with three items under the key, list-key. Note that item can be in the list more than once](/images/articles/Redis/an-example-of-a-LIST-with-three-items-under-the-key-list-key-Note-that-item-can-be-in-the-list-more-than-once.svg?id=668a408dcb4904fb60e112535889ee0d "An example of a LIST with three items under the key, list-key. Note that item can be in the list more than once")

As listas no *Redis* são um tipo de estrutura muito flexível, porque podem agir como uma *simples collection*, *stack* ou *fila*. O número máximo de elementos que uma lista pode conter é (*2^32 - 1*), o que significa que pode haver **mais de 4 bilhões de elementos _por lista_**.

A vantagem da lista no Redis de ser implementada como uma lista vinculada em vez de array é porque foi projetada para ter gravações mais rápidas do que leituras.

Os comandos no Redis para as listas normalmente começam com a letra `L`. Isso também pode ser interpretado que todos os comandos serão executados da esquerda ou do topo da lista, e onde os comandos são executados da direita ou do final da lista, eles começam com a letra `R`. Os comandos podem ser categorizados nas seguintes partes:

- **Setters e Getters**:

  - [**`LPUSH key element [element ...]`**](https://redis.io/commands/lpush): *Adiciona um ou mais valores ao **início/topo** (**head - esquerda**) da lista definida pela chave. Se a chave não existir, ela será criada como uma lista vazia antes de realizar a operação push. Quando a chave contém um valor que não é uma lista, um erro é retornado;*

    ```bash
    redis> DEL books
    redis> LPUSH books "Clean Code"         # (integer) 1
    redis> LPUSH books "Clean Architecture" # (integer) 2
    redis> LPUSH books "Refactoring"        # (integer) 3
    redis> LRANGE books 0 -1
            # 1) "Refactoring"
            # 2) "Clean Architecture"
            # 3) "Clean Code"
    ```

  - [**`RPUSH key element [element ...]`**](https://redis.io/commands/rpush): *Adiciona um ou mais valores ao **final** (**tail - direita**) da lista definida pela chave. Se a chave não existir, ela será criada como uma lista vazia antes de realizar a operação push. Quando a chave contém um valor que não é uma lista, um erro é retornado;*

    ```bash
    redis> DEL books
    redis> RPUSH books "Clean Code"         # (integer) 1
    redis> RPUSH books "Clean Architecture" # (integer) 2
    redis> RPUSH books "Refactoring"        # (integer) 3
    redis> LRANGE books 0 -1
            # 1) "Clean Code"
            # 2) "Clean Architecture"
            # 3) "Refactoring"
    ```

  - [**`LPUSHX key element [element ...]`**](https://redis.io/commands/lpushx): *Funciona da mesma forma que o comando `LPUSH`; a única diferença entre os dois é que o comando `LPUSHX` insere um novo item somente se a chave existir e for uma lista. Ao contrário do `LPUSH`, nenhuma operação será realizada quando a chave não existir;*

    ```bash
    redis> DEL books
    redis> LPUSH books "Clean Code"         # (integer) 1
    redis> LPUSH books "Clean Architecture" # (integer) 2
    redis> LPUSHX nonexisting "Hello"       # (integer) 0
    redis> LRANGE books 0 -1
            # 1) "Clean Architecture"
            # 2) "Clean Code"
    redis> LRANGE nonexisting 0 -1          # (empty array)
    ```

  - [**`RPUSHX key element [element ...]`**](https://redis.io/commands/rpushx): *Funciona da mesma forma que o comando `RPUSH`; a única diferença entre os dois é que o comando `RPUSHX` insere um novo item somente se a chave existir e for uma lista. Ao contrário do `RPUSH`, nenhuma operação será realizada quando a chave não existir;*

  - [**`LINSERT key BEFORE|AFTER pivot element`**](https://redis.io/commands/linsert): *Insere um elemento na lista antes ou depois da posição do pivot do valor de referência. Quando a chave não existe, é considerada uma lista vazia e nenhuma operação é realizada;*

    ```bash
    redis> DEL mylist
    redis> RPUSH mylist "Hello"         # (integer) 1
    redis> RPUSH mylist "World"         # (integer) 2
    redis> LINSERT mylist BEFORE "World" "value 3" # (integer) 3
    redis> LRANGE mylist 0 -1
            # 1) "Hello"
            # 2) "value 3"
            # 3) "World"
    redis> LINSERT mylist AFTER "World" "value 4"   # (integer) 4
            # 1) "Hello"
            # 2) "value 3"
            # 3) "World"
            # 4) "value 4"
    ```

  - [**`LSET key index element`**](https://redis.io/commands/lset): *Define o valor de um elemento com base no índice especificado;*

  - [**`LRANGE key start stop`**](https://redis.io/commands/lrange): *Obtém a sub-lista de elementos com base no índice inicial e no índice final. Os offsets `start` e `stop` são índices baseados em zero, com `0` sendo o primeiro elemento da lista (o topo da lista), `1` sendo o próximo elemento e assim por diante. Esses offsets também podem ser números negativos, indicando offsets que começam no final da lista. Por exemplo, `-1` é o último elemento da lista, `-2` é o penúltimo e assim por diante;*

- **Data Clean**:

  - [**`LTRIM key start stop`**](https://redis.io/commands/ltrim): *Excluí os elementos fora do intervalo especificado. Apara a lista deixando apenas os itens definidos entre os índices de `start` e `stop`;*

    ```bash
    redis> DEL mylist
    redis> RPUSH mylist "one"         # (integer) 1
    redis> RPUSH mylist "two"         # (integer) 2
    redis> RPUSH mylist "three"       # (integer) 3
    redis> LTRIM mylist 1 -1          # OK
    redis> LRANGE mylist 0 -1
            # 1) "two"
            # 2) "three"
    ```

  - [**`RPOP key [count]`**](https://redis.io/commands/rpop): *Remove e retorna os últimos (tail) elementos da lista ou (nil) caso a lista esteja vazia. Por padrão, o comando remove um único elemento do final da lista. Quando fornecido com o argumento opcional de `[count]`, removerá a quantidade de elementos definido no argumento `[count]`;*

    ```bash
    redis> DEL mylist
    redis> RPUSH mylist "one" "two" "three" "four" "five" # (integer) 5
    redis> RPOP mylist                                    # "five"
    redis> RPOP mylist 2
            # 1) "four"
            # 2) "three"
    redis> LRANGE mylist 0 -1
            # 1) "one"
            # 2) "two"
    ```

  - [**`LREM key count element`**](https://redis.io/commands/lrem): *Remove a quantidade de elementos de acordo com o argumento `count`, começando no índice espeficicado pelo argumento `element`;*

  - [**`LPOP key [count]`**](https://redis.io/commands/lpop): *Remove e retorna os primeiros (head) elementos da lista ou (nil) caso a lista esteja vazia. Por padrão, o comando remove um único elemento do início da lista. Quando fornecido com o argumento opcional de `[count]`, removerá a quantidade de elementos definido no argumento `[count]`;*

    ```bash
    redis> DEL mylist
    redis> RPUSH mylist "one" "two" "three" "four" "five" # (integer) 5
    redis> LPOP mylist                                    # "one"
    redis> LPOP mylist 2
            # 1) "two"
            # 2) "three"
    redis> LRANGE mylist 0 -1
            # 1) "four"
            # 2) "five"
    ```

- **Utility**:

  - [**`LINDEX key index`**](https://redis.io/commands/lindex): *Retorna o valor de um item da lista de acordo com o índice informado. O índice é baseado em zero, então `0` significa o primeiro elemento, `1` o segundo elemento e assim por diante. Índices negativos podem ser usados para designar elementos começando no final da lista. `-1` significa o último elemento, `-2` significa o penúltimo e assim por diante;*

    ```bash
    redis> DEL mylist
    redis> LPUSH mylist "one" "two" "three" # (integer) 3
    redis> LINDEX mylist 0                  # "three"
    redis> LINDEX mylist 1                  # "two"
    redis> LINDEX mylist 2                  # "one"
    redis> LINDEX mylist 3                  # (nil)
    redis> LINDEX mylist -1                 # "one"
    redis> LINDEX mylist -2                 # "two"
    redis> LINDEX mylist -3                 # "three"
    # LLEN
    redis> LLEN mylist                      # (integer) 3
    ```

  - [**`LLEN key`**](https://redis.io/commands/llen): *Retorna a quantidade de itens armazenados em uma lista. Se a chave não existir, ela será interpretada como uma lista vazia e `0` será retornado;*

- **Advanced**:

  - [**`BLPOP key [key ...] timeout`**](https://redis.io/commands/blpop): *Bloqueia a conexão para remover e retornar o primeiro item de uma das listas informadas como parâmetro durante um tempo máximo definido no parâmetro `timeout`. Caso `timeout` seja definido como `0`, a conexão fica bloqueada até que um item de uma das listas informadas seja removido e retornado pelo comando;*

  - [**`BRPOP key [key ...] timeout`**](https://redis.io/commands/brpop): *Bloqueia a conexão para remover e retornar o último item de uma das listas informadas como parâmetro durante um tempo máximo definido no parâmetro `timeout`. Caso `timeout` seja definido como `0`, a conexão fica bloqueada até que um item de uma das listas informadas seja removido e retornado pelo comando;*

  - [**`RPOPLPUSH source destination`**](https://redis.io/commands/rpoplpush): *Opera em duas listas. Retorna e remove atomicamente o último elemento (tail) da lista do argumento de `source` e o envia para a lista do argumento `destination` na posição de primeiro elemento (head);*

  - [**`BRPOPLPUSH source destination timeout`**](https://redis.io/commands/brpoplpush): *`BRPOPLPUSH` é a variante de bloqueio de `RPOPLPUSH`. Nesse caso, se a lista do argumento `source` estiver vazia, o Redis bloqueará a operação/conexão até que um valor seja adicionado ou até que o tempo limite do argumento `timeout` seja atingido. Um tempo limite de zero pode ser usado para bloquear indefinidamente;*

## [SETs](https://redis.io/docs/data-types/sets/)

> Um **Set** no Redis são estruturas de dados que são uma coleção/conjunto não ordenada de strings distintas - não é possível adicionar elementos repetidos/duplicados a um *Set*. Internamente, um **Set** é implementado como uma tabela hash, razão pela qual algumas operações são otimizadas: adição de membros, remoção e execução de pesquisa em `O(1)`, tempo constante. Um dos aspectos interessantes em termos de desempenho dos *Sets* é que eles mostram um tempo constante para adicionar, remover e verificar a existência de um elemento.

![An example of a SET with three items under the key, set-key](/images/articles/Redis/an-example-of-a-SET-with-three-items-under-the-key-set-key.svg?id=d67375c8d3e9768a78580ab3a981ffdf "An example of a SET with three items under the key, set-key")

O número máximo de elementos que um *Set* pode conter é (*2^32 - 1*), o que significa que pode haver mais de 4 bilhões de elementos por *Set*.

Os comandos no Redis para os **Sets** podem ser categorizados nas seguintes partes:

- **Setters e Getters:**

  - [**`SADD key member [member ...]`**](https://redis.io/commands/sadd): *Adicione os membros especificados ao conjunto definido pela chave. Membros especificados que já são membros deste conjunto são ignorados. Se a chave não existir, um novo conjunto será criado antes de adicionar os membros especificados;*

    ```bash
    redis> DEL myset
    redis> SADD myset "Hello"   # (integer) 1
    redis> SADD myset "World"   # (integer) 1
    redis> SADD myset "World"   # (integer) 0
    redis> SCARD myset          # (integer) 2
    redis> SMEMBERS myset
            # 1) "World"
            # 2) "Hello"
    ```

- **Data Clean:**

  - [**`SPOP key [count]`**](https://redis.io/commands/spop): *Remove e retorna um ou mais membros aleatórios do conjunto definido pela chave;*

  - [**`SREM key member [member ...]`**](https://redis.io/commands/srem): *Remova os membros especificados do conjunto definido pela chave. Membros especificados que não são membros deste conjunto são ignorados. Se a chave não existir, ela será tratada como um conjunto vazio e este comando retornará `0`;*

    ```bash
    redis> DEL myset
    redis> SADD myset "one" "two" "three"   # (integer) 3
    redis> SREM myset "one" "three"         # (integer) 2
    redis> SREM myset "four"                # (integer) 0
    redis> SMEMBERS myset
            # 1) "two"
    ```

- **Utility:**

  - [**`SCARD key`**](https://redis.io/commands/scard): *Retorna a quantidade de itens armazenados em um conjunto;*

  - [**`SDIFF key [key ...]`**](https://redis.io/commands/sdiff): *Retorna os membros do conjunto resultantes da diferença entre o primeiro conjunto e todos os conjuntos sucessivos;*

  - [**`SDIFFSTORE destination key [key ...]`**](https://redis.io/commands/sdiffstore): *Este comando é igual a `SDIFF`, mas em vez de retornar o conjunto resultante, ele é armazenado na chave de conjunto do argumento `destination`. Se a chave `destination` já existir, ela será sobrescrito;*

    ```bash
    redis> DEL key key1 key2
    redis> SADD key1 "a" "b" "c"    # (integer) 3
    redis> SADD key2 "c" "d" "e"    # (integer) 3
    redis> SDIFF key1 key2
            # 1) "a"
            # 2) "b"
    redis> SDIFF key2 key1
            # 1) "d"
            # 2) "e"
    redis> SDIFFSTORE key key1 key2 # (integer) 2
    redis> SMEMBERS key
            # 1) "a"
            # 2) "b"
    ```

  - [**`SINTER key [key ...]`**](https://redis.io/commands/sinter): *Retorna os elementos resultantes entre uma intersecção dos conjuntos informados;*

  - [**`SINTERSTORE destination key [key ...]`**](https://redis.io/commands/sinterstore): *Este comando é igual ao `SINTER`, mas em vez de retornar o conjunto resultante, ele é armazenado na chave de conjunto do argumento `destination`. Se a chave `destination` já existir, ela será sobrescrito;*

    ```bash
    redis> DEL key key1 key2
    redis> SADD key1 "a" "b" "c"      # (integer) 3
    redis> SADD key2 "c" "d" "e"      # (integer) 3
    redis> SINTER key1 key2
            # 1) "c"
    redis> SINTERSTORE key key1 key2  # (integer) 1
    redis> SMEMBERS key
            # 1) "c"
    ```

  - [**`SISMEMBER key member`**](https://redis.io/commands/sismember): *Retorna `1` se o valor informado existe no conjunto informado pela chave, caso o valor não exista o comando retorna o valor `0`;*

    ```bash
    redis> DEL myset
    redis> SADD myset "one"       # (integer) 1
    redis> SISMEMBER myset "one"  # (integer) 1
    redis> SISMEMBER myset "two"  # (integer) 0
    ```

  - [**`SMISMEMBER key member [member ...]`**](https://redis.io/commands/smismember): *Retorna `1` se cada membro do argumento de `member` é um membro/está no conjunto da chave informada. Para cada membro, `1` é retornado se o valor for um membro do conjunto, ou `0` se o elemento não for um membro do conjunto ou se a chave não existir;*

    ```bash
    redis> DEL myset
    redis> SADD myset "a" "b" "c" "d" "e" "f" # (integer) 6
    redis> SMISMEMBER myset "a" "d" "f" "notamember"
            # 1) (integer) 1
            # 2) (integer) 1
            # 3) (integer) 1
            # 4) (integer) 0
    ```

  - [**`SMEMBERS key`**](https://redis.io/commands/smembers): *Retorna todos os elementos de um conjunto definido pela chave informada;*

  - [**`SMOVE source destination member`**](https://redis.io/commands/smove): *Mova o membro do argumento `member` do conjunto do argumento `source` para o conjunto do argumento `destination`;*

  - [**`SUNION key [key ...]`**](https://redis.io/commands/sunion): *Retorna os membros do conjunto resultantes da união de todos os conjuntos fornecidos;*

  - [**`SUNIONSTORE destination key [key ...]`**](https://redis.io/commands/sunionstore): *Este comando é igual a `SUNION`, mas em vez de retornar o conjunto resultante, ele é armazenado na chave de conjunto do argumento `destination`. Se a chave `destination` já existir, ela será sobrescrito;*

    ```bash
    redis> DEL key key1 key2
    redis> SADD key1 "a" "b" "c"      # (integer) 3
    redis> SADD key2 "c" "d" "e"      # (integer) 3
    redis> SUNION key1 key2
            # 1) "a"
            # 2) "b"
            # 3) "d"
            # 4) "c"
            # 5) "e"
    redis> SUNIONSTORE key key1 key2  # (integer) 5
    redis> SMEMBERS key
            # 1) "a"
            # 2) "b"
            # 3) "d"
            # 4) "c"
            # 5) "e"
    ```

## [Sorted Sets](https://redis.io/docs/data-types/sorted-sets/)

> Como os tipos de `HASH`, os `ZSET`s também contêm um tipo de chave e valor. As chaves (chamadas de *members*) são únicas/exclusivas e os valores (chamados de *scores*) são limitados a números de ponto flutuante. `ZSET`s têm a propriedade única no Redis de poder ser acessado por um membro (como um `HASH`), mas os itens também podem ser acessados ​​pela ordem de classificação e pelos valores das pontuações.

![An example of a ZSET with two members:scores under the key zset-key](/images/articles/Redis/an-example-of-a-ZSET-with-two-members-scores-under-the-key-zset-key.svg?id=c6f507c3b6373e95f13d1bc8879c0bd3 "An example of a ZSET with two members:scores under the key zset-key")

Um **Sorted Set** é muito semelhante a um *Set*, mas cada elemento de um **Sorted Set** tem uma pontuação associada. Em outras palavras, um **Sorted Set** é uma coleção de strings não repetidas classificadas por uma pontuação. É possível ter elementos com pontuações repetidas. Nesse caso, os elementos repetidos são ordenados lexicograficamente (em ordem alfabética).

As operações em *Sorted Set* são rápidas, mas não tão rápidas quanto as operações em *Set*, porque as pontuações precisam ser comparadas. Adicionar, remover e atualizar um item em um *Sorted Set* é executado em tempo logarítmico, `O(log(N))`, onde `N` é o número de elementos em um *Sorted Set*. Internamente, os *Sorted Sets* são implementados como duas estruturas de dados separadas:

- Uma *skip list* com uma tabela *hash*. Uma *skip list* é uma estrutura de dados que permite uma pesquisa rápida em uma sequência ordenada de elementos.
- Um `ziplist`, com base nas configurações `zset-max-ziplist-entries` e `zset-max-ziplist-value`.

Os *Sorted Sets* são muito parecidos com os *Sets* do Redis, pois não armazenam valores duplicados, mas a área em que eles diferem dos *Sets* é que os valores são classificados com base em uma pontuação, ou inteiros, valores flutuantes. Esses valores são fornecidos ao definir um valor no Conjunto. O desempenho desses *Sorted Sets* é proporcional ao logaritmo do número de elementos. Os dados são sempre mantidos de forma ordenada. Esse conceito é explicado na imagem abaixo:

![The concept of Sorted Sets](/images/articles/Redis/the-concept-of-sorted-sets.png?id=5994331d8395a470d61fdb414c71cab0 "The concept of Sorted Sets")

Os comandos no Redis para os **Sorted Sets** podem ser categorizados nas seguintes partes:

- **Setters e Getters:**

  - [**`ZADD key [NX|XX] [GT|LT] [CH] [INCR] score member [score member ...]`**](https://redis.io/commands/zadd): *Adiciona todos os membros com seus scores/pontuações de uma determinada chave. Se um membro especificado já for membro do conjunto, a pontuação/score é atualizada e o elemento reinserido para garantir a ordem correta;*

    **Definindo as chaves:**

    ```bash
    redis> DEL myzset
    redis> ZADD myzset 1 "one" 2 "two" 3 "three" # (integer) 3
    redis> ZRANGE myzset 0 -1 WITHSCORES
            # 1) "one"
            # 2) "1"
            # 3) "two"
            # 4) "2"
            # 5) "three"
            # 6) "3"
    ```

    **Atualizando apenas os elementos que já existem - `XX`:**

    ```bash
    redis> ZADD myzset XX 10 "one" 20 "notamember" # (integer) 0
    redis> ZRANGE myzset 0 -1 WITHSCORES
            # 1) "two"
            # 2) "2"
            # 3) "three"
            # 4) "3"
            # 5) "one"
            # 6) "10"
    ```

    **Adicionar novos elementos. Não atualizar elementos existentes - `NX`:**

    ```bash
    redis> ZADD myzset NX 20 "one" 30 "thirty" # (integer) 1
    redis> ZRANGE myzset 0 -1 WITHSCORES
            # 1) "two"
            # 2) "2"
            # 3) "three"
            # 4) "3"
            # 5) "one"
            # 6) "10"
            # 7) "thirty"
            # 8) "30"
    ```

    **Atualizar os elementos se a nova pontuação for *MENOR* que a pontuação atual. Essa flag não impede a adição de novos elementos - `LT`:**

    ```bash
    redis> DEL myzset
    redis> ZADD myzset 1 "one" 2 "two" 3 "three" 4 "for" 5 "five" # (integer) 5
    redis> ZADD myzset LT 2 "three" 3 "for" 30 "thirty"           # (integer) 1
    redis> ZRANGE myzset 0 -1 WITHSCORES
            # 1) "one"
            # 2) "1"
            # 3) "three"
            # 4) "2"
            # 5) "two"
            # 6) "2"
            # 7) "for"
            # 8) "3"
            # 9) "five"
            # 10) "5"
            # 11) "thirty"
            # 12) "30"
    ```

    **Atualizar os elementos se a nova pontuação for *MAIOR* que a pontuação atual. Essa flag não impede a adição de novos elementos - `GT`:**

    ```bash
    redis> DEL myzset
    redis> ZADD myzset 1 "one" 2 "two" 3 "three" 4 "for" 5 "five" # (integer) 5
    redis> ZRANGE myzset 0 -1 WITHSCORES
            # 1) "one"
            # 2) "1"
            # 3) "two"
            # 4) "2"
            # 5) "three"
            # 6) "3"
            # 7) "for"
            # 8) "4"
            # 9) "five"
            # 10) "5"
    redis> ZADD myzset GT 2 "one" 3 "two" 4 "three" # (integer) 0
    redis> ZRANGE myzset 0 -1 WITHSCORES
            # 1) "one"
            # 2) "2"
            # 3) "two"
            # 4) "3"
            # 5) "for"
            # 6) "4"
            # 7) "three"
            # 8) "4"
            # 9) "five"
            # 10) "5"
    ```

  - [**`ZRANGE key min max [BYSCORE|BYLEX] [REV] [LIMIT offset count] [WITHSCORES]`**](https://redis.io/commands/zrange): *Retorna uma quantidade (range) de elementos. Os elementos são ordenados do menor score para o maior, e o range de elementos é definido conforme os valores dos argumentos `min` e `max` (pelo índice/rank, score ou lexicográfia). O argumento opcional `WITHSCORES`, quando informado, faz com que o comando além de retornar os elementos também retorne os scores de cada elemento;*

    `<min>` e `<max>` podem ser `-inf` e `+inf`, denotando os infinitos negativo e positivo, respectivamente. Isso significa que você não precisa saber a pontuação mais alta ou mais baixa no conjunto para obter todos os elementos de ou até uma determinada pontuação.

    Por padrão, os intervalos de pontuação especificados por `<min>` e `<max>` são fechados (inclusivos). É possível especificar um intervalo aberto (exclusivo) prefixando o score com o caractere `(`.

    **Retornando todos os elementos juntamente com seus scores/pontuações - `WITHSCORES`:**

    ```bash
    redis> DEL myzset
    redis> ZADD myzset 1 "one" 2 "two" 3 "three" 4 "for" 5 "five" # (integer) 5
    redis> ZRANGE myzset 0 -1 WITHSCORES
            # 1) "one"
            # 2) "1"
            # 3) "two"
            # 4) "2"
            # 5) "three"
            # 6) "3"
            # 7) "for"
            # 8) "4"
            # 9) "five"
            # 10) "5"
    ```

    **Retornando todos os elementos na ordem reversa - `REV`:**

    ```bash
    redis> DEL myzset
    redis> ZADD myzset 1 "one" 2 "two" 3 "three" 4 "for" 5 "five" # (integer) 5
    redis> ZRANGE myzset 0 -1 REV WITHSCORES
            # 1) "five"
            # 2) "5"
            # 3) "for"
            # 4) "4"
            # 5) "three"
            # 6) "3"
            # 7) "two"
            # 8) "2"
            # 9) "one"
            # 10) "1"
    redis> ZRANGE myzset 0 2 REV WITHSCORES
            # 1) "five"
            # 2) "5"
            # 3) "for"
            # 4) "4"
            # 5) "three"
            # 6) "3"
    ```

    **Recuperar os elementos de acordo com o score - `BYSCORE`:**

    ```bash
    redis> DEL myzset
    redis> ZADD myzset 1 "one" 2 "two" 3 "three" 4 "for" 5 "five" # (integer) 5
    redis> ZRANGE myzset 0 1 BYSCORE WITHSCORES
            # 1) "one"
            # 2) "1"
    redis> ZRANGE myzset 2 4 BYSCORE WITHSCORES
            # 1) "two"
            # 2) "2"
            # 3) "three"
            # 4) "3"
            # 5) "for"
            # 6) "4"
    ```

    **Limitando a quantidade de elementos retornados (funciona apenas junto com a opção `BYSCORE` ou `BYLEX`) - `LIMIT`:**

    ```bash
    redis> DEL myzset
    redis> ZADD myzset 1 "one" 2 "two" 3 "three" 4 "for" 5 "five" # (integer) 5
    redis> ZRANGE myzset 0 5 BYSCORE LIMIT 0 1 WITHSCORES
            # 1) "one"
            # 2) "1"
    redis> ZRANGE myzset 0 5 BYSCORE LIMIT 1 1 WITHSCORES
            # 1) "two"
            # 2) "2"
    redis> ZRANGE myzset 0 5 BYSCORE LIMIT 0 2 WITHSCORES
            # 1) "one"
            # 2) "1"
            # 3) "two"
            # 4) "2"
    redis> ZRANGE myzset 0 5 BYSCORE LIMIT 1 2 WITHSCORES
            # 1) "two"
            # 2) "2"
            # 3) "three"
            # 4) "3"
    ```

    **Usando o caractere `(` (apenas com a opção `BYSCORE`) para excluir (exclusivo) os elementos por meio de seus scores:**

    ```bash
    redis> DEL myzset
    redis> ZADD myzset 1 "one" 2 "two" 3 "three" 4 "for" 5 "five" # (integer) 5
    redis> ZRANGE myzset (1 +inf BYSCORE WITHSCORES
            # 1) "two"
            # 2) "2"
            # 3) "three"
            # 4) "3"
            # 5) "for"
            # 6) "4"
            # 7) "five"
            # 8) "5"
    redis> ZRANGE myzset (2 +inf BYSCORE LIMIT 1 2 WITHSCORES
            # 1) "for"
            # 2) "4"
            # 3) "five"
            # 4) "5"
    ```

  - [**`ZREVRANGE key start stop [WITHSCORES]`**](https://redis.io/commands/zrevrange): *Funciona da mesma forma que o comando `ZRANGE`, porém faz a ordenação de elementos levando em consideração o maior score para o menor (decrescente);*

  - [**`ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]`**](https://redis.io/commands/zrangebyscore): *Retorna todos os elementos de uma determinada chave do intervalo por pontuação fornecida. Tendo o score/pontuação entre os valores dos argumentos `min` e `max` (incluindo elementos com score/pontuação igual a `min` ou `max`). Os elementos são considerados ordenados do menor para a maior pontuação (ordem crescente);*

  - [**`ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]`**](https://redis.io/commands/zrevrangebyscore): *Retorna todos os elementos na chave de sorted set com uma pontuação entre `max` e `min` (incluindo elementos com pontuação igual a `max` ou `min`). Ao contrário da ordem padrão dos sorted sets, para este comando os elementos são considerados ordenados dos maiores para as menores pontuações (decrescente);*

  - [**`ZREVRANK key member`**](https://redis.io/commands/zrevrank): *Retorna a classificação do membro na chave de sorted set, com as pontuações ordenadas de forma decrescente. A classificação (ou índice) é baseada em `0`, o que significa que o membro com a pontuação mais alta tem classificação `0`;*

- **Data Clean:**

  - [**`ZREM key member [member ...]`**](https://redis.io/commands/zrem): *Remove os membros especificados. Membros não existentes são ignorados;*

    ```bash
    redis> DEL myzset
    redis> ZADD myzset 1 "one" 2 "two" 3 "three"  # (integer) 3
    redis> ZREM myzset "two"                      # (integer) 1
    redis> ZREM myzset "five"                     # (integer) 0
    redis> ZRANGE myzset 0 -1 WITHSCORES
            # 1) "one"
            # 2) "1"
            # 3) "three"
            # 4) "3"
    ```

  - [**`ZREMRANGEBYRANK key start stop`**](https://redis.io/commands/zremrangebyrank): *Remove todos os elementos do conjunto com classificação entre os argumentos de `start` e `stop`. Tanto os argumentos de `start` quanto `stop` são índices iniciados em `0`, sendo `0` o elemento com a pontuação mais baixa. Esses índices podem ser números negativos, onde indicam offsets começando no elemento com a pontuação mais alta. Por exemplo: `-1` é o elemento com a pontuação mais alta, `-2` o elemento com a segunda pontuação mais alta e assim por diante;*

    ```bash
    redis> DEL myzset
    redis> ZADD myzset 1 "one" 2 "two" 3 "three"  # (integer) 3
    redis> ZREMRANGEBYRANK myzset 1 2             # (integer) 2
    redis> ZRANGE myzset 0 -1 WITHSCORES
            # 1) "one"
            # 2) "1"
    ```

  - [**`ZREMRANGEBYSCORE key min max`**](https://redis.io/commands/zremrangebyscore): *Remove todos os elementos do conjunto  com uma pontuação/score entre os argumentos de `min` e `max` (inclusive);*

    ```bash
    redis> DEL myzset
    redis> ZADD myzset 1 "one" 2 "two" 3 "three"  # (integer) 3
    redis> ZREMRANGEBYSCORE myzset -inf (2        # (integer) 1
    redis> ZRANGE myzset 0 -1 WITHSCORES
            # 1) "two"
            # 2) "2"
            # 3) "three"
            # 4) "3"
    ```

- **Utility:**

  - [**`ZCARD key`**](https://redis.io/commands/zcard): *Retorna a quantidade de elementos armazenados em um sorted Set;*

  - [**`ZCOUNT key min max`**](https://redis.io/commands/zcount): *Retorna o número de membros em um sorted set que possui o score entre um valor do argumento de `min` e `max`;*

    ```bash
    redis> DEL myzset
    redis> ZADD myzset 1 "one" 2 "two" 3 "three"  # (integer) 3
    redis> ZCOUNT myzset -inf +inf                # (integer) 3
    redis> ZCOUNT myzset 2 +inf                   # (integer) 2
    redis> ZCOUNT myzset (2 +inf                  # (integer) 1
    redis> ZCOUNT myzset -inf (3                  # (integer) 2
    redis> ZCOUNT myzset -inf (2                  # (integer) 1
    redis> ZCOUNT myzset (1 3                     # (integer) 2
    ```

  - [**`ZINCRBY key increment member`**](https://redis.io/commands/zincrby): *Incrementa o score de um membro do sorted set definido pela chave informada como argumento;*

  - [**`ZRANK key member`**](https://redis.io/commands/zrank): *Retorna a classificação/índice dos membros de um Sorted Set, realizando a ordenação do menor score para o maior;*

    ```bash
    redis> DEL myzset
    redis> ZADD myzset 1 "one" 2 "two" 3 "three"  # (integer) 3
    redis> ZRANK myzset "one"                     # (integer) 0
    redis> ZRANK myzset "two"                     # (integer) 1
    redis> ZRANK myzset "three"                   # (integer) 2
    redis> ZRANK myzset "four"                    # (nil)
    ```

  - [**`ZSCORE key member`**](https://redis.io/commands/zscore): *Retorna o score de um membro de um sorted set;*

    ```bash
    redis> DEL myzset
    redis> ZADD myzset 1 "one" 2 "two" 3 "three"  # (integer) 3
    redis> ZSCORE myzset "one"                    # "1"
    redis> ZSCORE myzset "three"                  # "3"
    ```

## [Hashes](https://redis.io/docs/data-types/hashes/)

> Enquanto `LIST` e `SET` no Redis contêm uma sequências de elementos, os `HASH`es no Redis armazenam um mapeamento de chave-valor. Os valores que podem ser armazenados em `HASH` são os mesmos que podem ser armazenados como `STRING`: strings themselves, ou se um valor pode ser interpretado como um número, esse valor pode ser manipulado com comandos de números.

![An example of a HASH with two keys:values under the key hash-key](/images/articles/Redis/an-example-of-a-HASH-with-two-keys-values-under-the-key-hash-key.svg?id=805819274e0ba2c079ab52193e1153ad "An example of a HASH with two keys:values under the key hash-key")

Hashes são ótimas estruturas de dados para armazenar objetos que você pode mapear uma chave para um valor. Eles são otimizados para usar a memória com eficiência e procurar dados muito rapidamente. Em um Hash, tanto o nome do campo quanto o valor são *Strings*. Portanto, um Hash é um mapeamento de uma *String* para uma *String*.

Internamente, um **Hash** pode ser um `ziplist` ou um `hash table`. Um `ziplist` é uma *dually linked list* projetada para economizar memória. Em um `ziplist`, os inteiros são armazenados como inteiros reais em vez de uma sequência de caracteres. Embora um `ziplist` tenha otimizações de memória, as pesquisas não são realizadas em tempo constante. Por outro lado, um `hash table` tem pesquisa de tempo constante, mas não é otimizada para memória.

*Instagram had to back-reference 300 million media IDs to user IDs, and they decided to benchmark a Redis prototype using Strings and Hashes. The String solution used one key per media ID and around 21 GB of memory. The Hash solution used around 5 GB with some configuration tweaks. The details can be found at https://instagram-engineering.com/storing-hundreds-of-millions-of-simple-key-value-pairs-in-redis-1091ae80f74c*

Os **Hash** são armazenados de forma que ocupem menos espaço e cada hash no Redis pode armazenar até *2^32* pares de chave-valor, ou seja, mais de 4 bilhões.

Os comandos no Redis para os **Hashes** podem ser categorizados nas seguintes partes:

- **Setters e Getters:**

  - [**`HSET key field value [field value ...]`**](https://redis.io/commands/hset): *Armazena um hash com o campo e seu respectivo valor. Caso o hash e o campo já existam, o valor é sobrescrito;*

    ```bash
    redis> DEL myhash
    redis> HSET myhash field1 "Hello" field2 "World"    # (integer) 2
    redis> HGET myhash field1                           # "Hello"
    ```

  - [**`HGET key field`**](https://redis.io/commands/hget): *Retorna o valor associado ao argumento `field` no hash;*

    ```bash
    redis> DEL myhash
    redis> HSET myhash field1 "Hello" field2 "World"    # (integer) 2
    redis> HGET myhash field1                           # "Hello"
    redis> HGET myhash field2                           # "World"
    redis> HGET myhash field3                           # (nil)
    ```

  - [**`HGETALL key`**](https://redis.io/commands/hgetall): *Retorna todos os campos e seus respectivos valores associados a um hash;*

    ```bash
    redis> DEL myhash
    redis> HSET myhash field1 "Hello" field2 "World" # (integer) 2
    redis> HGETALL myhash
            # 1) "field1"
            # 2) "Hello"
            # 3) "field2"
            # 4) "World"
    ```

  - [**`HMGET key field [field ...]`**](https://redis.io/commands/hmget): *Retorna os valores de todos os campos informados que são associados a um hash;*

    ```bash
    redis> DEL myhash
    redis> HSET myhash field1 "Hello" field2 "World" # (integer) 2
    redis> HMGET myhash field1 field2 nofield
            # 1) "Hello"
            # 2) "World"
            # 3) (nil)
    ```

  - [**`HVALS key`**](https://redis.io/commands/hvals): *Retorna todos os valores da hash;*

    ```bash
    redis> DEL myhash
    redis> HSET myhash field1 "Hello" field2 "World" # (integer) 2
    redis> HVALS myhash
            # 1) "Hello"
            # 2) "World"
    ```

  - [**`HSETNX key field value`**](https://redis.io/commands/hsetnx): *Define o campo e seu valor, apenas se o campo ainda não existir. Se a chave não existir, uma nova chave contendo um hash será criada. Se o campo já existe, esta operação não tem efeito;*

    ```bash
    redis> DEL myhash
    redis> HSETNX myhash field "Hello" # (integer) 1
    redis> HSETNX myhash field "World" # (integer) 0
    redis> HGET myhash field           # "Hello"
            # 1) "Hello"
            # 2) "World"
    ```

  - [**`HKEYS key`**](https://redis.io/commands/hkeys): *Retorna todos os campo da hash;*

    ```bash
    redis> DEL myhash
    redis> HSET myhash field1 "Hello" field2 "World" # (integer) 2
    redis> HKEYS myhash
            # 1) "field1"
            # 2) "field2"
    ```

- **Data Clean:**

    - [**`HDEL key field [field ...]`**](https://redis.io/commands/hdel): R*emove o(s) campo(s) e seu(s) respectivo(s) valor(es) do hash informado;*

    ```bash
    redis> DEL myhash
    redis> HSET myhash field1 "Hello" field2 "World"    # (integer) 2
    redis> HDEL myhash field1 field2 nofield            # (integer) 2
    ```

- **Utility:**

  - [**`HEXISTS key field`**](https://redis.io/commands/hexists): *Determina se o campo do argumento `field` existe na chave hash do argumento `key`;*

    ```bash
    redis> DEL myhash
    redis> HSET myhash field1 "Hello"   # (integer) 2
    redis> HEXISTS myhash field1        # (integer) 1
    redis> HEXISTS myhash field2        # (integer) 0
    ```

  - [**`HINCRBY key field increment`**](https://redis.io/commands/hincrby): *Incrementa ou decrementa o valor (número inteiro) do campo associado a uma chave conforme o valor do incremento;*

  - [**`HINCRBYFLOAT key field increment`**](https://redis.io/commands/hincrbyfloat): *Incrementa ou decrementa o valor (número de ponto flutuante) do campo associado a uma chave conforme o valor do incremento;*

  - [**`HLEN key`**](https://redis.io/commands/hlen): *Retorna a quantidade de campos que um hash possui;*

## [Bitmaps](https://redis.io/docs/data-types/bitmaps/)

Um ***Bitmap*** não é um tipo de dados real no Redis. Por baixo, um **Bitmap** é uma *String*. Também podemos dizer que um *Bitmap* é um conjunto de operações de bits em uma *String*. No entanto, vamos considerá-los como tipos de dados porque o Redis fornece comandos para manipular *Strings* como *Bitmaps*. Os Bitmaps também são conhecidos como *array de bits* ou *bitsets*.

Um *Bitmap* é uma sequência de bits em que cada bit pode armazenar `0` ou `1`. Você pode pensar em um *Bitmap* como uma array de zeros e uns. A documentação do Redis se refere aos índices de *Bitmap* como *offsets*. O domínio do aplicativo determina o que cada índice de *Bitmap* significa.

Os *Bitmaps* são eficientes em termos de memória, suportam pesquisas rápidas de dados e podem armazenar até *2^32 bits* (mais de 4 bilhões de bits).

Veja este exemplo do Bitmap com três bits ativados e dois desativados:

![A Bitmap with one Set on offsets 0, 3, and 4 and zero Set on offsets 1 and 2](/images/articles/Redis/a-Bitmap-with-one-Set-on-offsets-0-3-and-4-and-zero-Set-on-offsets-1-and-2.png?id=371567afb3df29b69c6378c764420d0e "A Bitmap with one Set on offsets 0, 3, and 4 and zero Set on offsets 1 and 2")

Para ver como um Bitmap pode ser eficiente em termos de memória, vamos comparar um Bitmap a um Set. O cenário de comparação é um aplicativo que precisa armazenar todos os IDs de usuário que visitaram um site em um determinado dia (o offset do Bitmap representa um ID de usuário). Presumimos que nosso aplicativo tenha 5 milhões de usuários no total, mas apenas 2 milhões de usuários visitaram o site naquele dia, e que cada ID de usuário pode ser representado por 4 bytes (32 bits, que é o tamanho de um número inteiro em um computador de 32-bit).

A tabela a seguir compara quanta memória um Bitmap e uma implementação de Set levaria para armazenar 2 milhões de IDs de usuário. Neste exemplo, a chave Redis é a data das visitas:

| **Redis key**       | **Data type** | **Amount of bits per user** | **Stored users** | **Total memory**         |
|---------------------|---------------|-----------------------------|------------------|--------------------------|
| *visits:2015-01-01* | `Bitmap`      | 1 bit                       | 5 million        | 1 * 5000000 bits = 625kB |
| *visits:2015-01-01* | `Set`         | 32 bits                     | 2 million        | 32 * 2000000 bits = 8MB  |

Os Bitmaps são tipos especiais de estrutura de dados com espaço eficiente, usados para armazenar tipos especiais de informações. Os Bitmaps são usados especialmente para trabalho de análise em tempo real. Embora o Bitmap só possa armazenar valores em binário (`1` ou `0`), o fato de que eles consomem menos espaço e o desempenho para obter um valor é `O(1)`, os torna muito atraentes para análises em tempo real:

![Representation of Bitmap](/images/articles/Redis/representation-of-bitmap.png?id=57b7394185e81aca86df1be40e76d290 "Representation of Bitmap")

### Manipulando

Use `SETBIT` para definir o valor do bit em um **Bitmap** no *offset* específico.

[**`SETBIT key offset value`**](https://redis.io/commands/setbit): *Define ou clears o bit no offset. O bit é cleared ou apagado dependendo do valor, que pode ser `0` ou `1`; Define o valor do bit de um offset de uma chave;*

```bash
redis> DEL mykey
redis> SETBIT mykey 7 1 # (integer) 0
redis> GETBIT mykey 7   # (integer) 1
redis> SETBIT mykey 7 0 # (integer) 1
redis> GETBIT mykey 7   # (integer) 1
redis> GETBIT mykey 0   # (integer) 0
redis> GETBIT mykey 100 # (integer) 0
```

[**`GETBIT key offset`**](https://redis.io/commands/getbit): *Recupera o valor do bit armazenado para um determinado offset;*

Para retornar o número de bits definido como `1` na chave de *Bitmap*, use o comando `BITCOUNT`.

```bash
redis> DEL mykey
redis> SETBIT mykey 7 1  # (integer) 0
redis> SETBIT mykey 8 1  # (integer) 0
redis> SETBIT mykey 9 1  # (integer) 0
redis> SETBIT mykey 10 1 # (integer) 0
redis> SETBIT mykey 11 0 # (integer) 0
redis> SETBIT mykey 12 0 # (integer) 0
redis> BITCOUNT mykey    # (integer) 4
```

`BITOP` é o comando para realizar operações bit a bit entre Bitmaps. O comando suporta quatro operações bit a bit: `AND`, `OR`, `XOR` e `NOT`. O resultado será armazenado em uma chave de destino.

```bash
redis> DEL mykey1 mykey2 mykey
redis> SETBIT mykey1 7 1   # (integer) 0
redis> SETBIT mykey1 8 1   # (integer) 0
redis> SETBIT mykey1 9 0   # (integer) 0
redis> SETBIT mykey1 10 0  # (integer) 0
redis> SETBIT mykey2 7 1   # (integer) 0
redis> SETBIT mykey2 8 1   # (integer) 0
redis> SETBIT mykey2 9 1   # (integer) 0
redis> SETBIT mykey2 10 1  # (integer) 0
redis> SETBIT mykey2 11 1  # (integer) 0
redis> BITOP AND mykey mykey1 mykey2 # (integer) 2
redis> BITCOUNT mykey      # (integer) 2
redis> BITOP OR mykey mykey1 mykey2 # (integer) 2
redis> BITCOUNT mykey      # (integer) 5
```

## Persistência

Como o *Redis* armazena dados na memória, todos eles são perdidos quando o servidor é reinicializado. Do ponto de vista da redundância de dados, a replicação pode ser usado como uma maneira de fazer backup.

Para manter os dados seguros, *Redis* fornece mecanismos para persistir dados no disco. Existem duas opções de persistência dos dados: **RDB** (*Redis Database*) e **AOF** (*Append Only File*). A persistência **RDB** executa snapshots *point-in-time* do seu conjunto de dados em intervalos especificados, perfeito para backups e recuperação de desastres. A persistência **AOF** registra todas as operações de gravação recebidas pelo servidor, que serão reproduzidas novamente em sua inicialização, reconstruindo o conjunto de dados original. Os comandos são registrados usando o mesmo formato que o próprio protocolo *Redis*, de maneira *append-only*. O *Redis* é capaz de reescrever o log em background quando fica muito grande.

### Manipulando `RDB`

Para um armazenamento de dados na memória, como o *Redis*, habilitar recursos de persistência é a melhor defesa contra perda de dados. Uma maneira óbvia de implementar a persistência é tirar um snapshot da memória onde os dados são armazenados de tempos em tempos. É basicamente assim que o **RDB** funciona no *Redis*.

#### Principais comandos

Para habilitar a persistência **RDB** para uma instância Redis em execução, chame o comando `CONFIG SET` da seguinte maneira:

```bash
127.0.0.1:6379> CONFIG SET save "900 1"
# OK
127.0.0.1:6379> CONFIG SET save "900 1 300 10 60 10000"
# OK
```

Para desativar o recurso de persistência **RDB** para uma instância Redis em execução, adicione uma string vazia à opção `save`:

```bash
127.0.0.1:6379> CONFIG SET save ""
# OK
```

Para verificar se o recurso **RDB** está ativado, obtenha o parâmetro de `save` no `redis-cli`. Se houver uma string vazia na resposta do comando `CONFIG GET save`, isso indica que o recurso **RDB** está desativado. Caso contrário, a política de trigger do snapshotting será executada:

```bash
127.0.0.1:6379> CONFIG GET save
# 1) "save"
# 2) ""
#
# OR
#
127.0.0.1:6379> CONFIG GET save
# 1) "save"
# 2) "900 1 300 10 60 10000"
```

Para executar um **snapshot RDB** manualmente, chame o comando `SAVE` no `redis-cli`. O `redis-cli` ficará bloqueado por um tempo e, em seguida, o Prompt de Comando retorna o seguinte:

```bash
127.0.0.1:6379> SAVE
# OK
```

Alternativamente, podemos chamar também `BGSAVE` para realizar um dumping **RDB** *non-blocking*:

```bash
127.0.0.1:6379> BGSAVE
# Background saving started
```

Para saber as métricas e o status de persistência, execute `INFO Persistence` no `redis-cli`:

```bash
127.0.0.1:6379> INFO Persistence
# Persistence
loading:0
current_cow_size:0
current_cow_size_age:0
current_fork_perc:0.00
current_save_keys_processed:0
current_save_keys_total:0
rdb_changes_since_last_save:0
rdb_bgsave_in_progress:0
rdb_last_save_time:1634267105
rdb_last_bgsave_status:ok
rdb_last_bgsave_time_sec:0
rdb_current_bgsave_time_sec:-1
rdb_last_cow_size:262144
aof_enabled:0
aof_rewrite_in_progress:0
aof_rewrite_scheduled:0
aof_last_rewrite_time_sec:-1
aof_current_rewrite_time_sec:-1
aof_last_bgrewrite_status:ok
aof_last_write_status:ok
aof_last_cow_size:0
module_fork_in_progress:0
module_fork_last_cow_size:0
```

#### Entendendo como funciona

O **RDB** funciona como uma câmera para a memória de dados do *Redis*. Assim que o gatilho da política `save` puder ser executado, ela tirará uma foto dos dados no *Redis* dumping todos os dados em um arquivo no disco local. Antes de introduzir esse processo em detalhes, primeiro explicaremos o valor da opção `save`, que determina o gatilho da política do **RDB** mencionada na seção anterior.

Para o exemplo anterior, `900 1` significa que, pelo menos `1` chave tenha mudado dentro do período de `900 segundos`, um *snapshot* **RDB** será tirado. Pode haver mais de uma política na opção `save` para controlar a frequência com que o *snapshot* **RDB** é tirado. Isso significa que *dumping* os dados no Redis após `x segundos` se pelo menos `y chaves` tiverem sido alteradas e nenhum processo de dumping estiver em andamento.

Podemos iniciar manualmente um processo de dumping **RDB** emitindo os comandos `SAVE` ou `BGSAVE`. A diferença entre esses dois comandos é que o primeiro inicia uma operação síncrona, o que significa que a thread principal do Redis faz o dumping, enquanto o último executa o dumping em background. Você nunca deve usar `SAVE` em seu ambiente de produção porque ele bloqueará o *Redis Server*. Em vez disso, com o `BGSAVE`, a thread principal continua processando os comandos enquanto um processo filho é criado por meio do `fork()` e, em seguida, salva o arquivo de dumping em um arquivo temporário chamado `temp-<bgsave-pid>.rdb`. Quando o processo de dumping terminar, esse arquivo temporário será renomeado para o nome definido no parâmetro `dbfilename` no diretório especificado pelo parâmetro `dir` na configuração *Redis* para substituir o dumping antigo.

Ao listar os processos com o comando `ps`, você pode ver o processo filho com o nome `redis-rdb-bgsave`, que é bifurcado pelo *Redis Server* para executar o `BGSAVE`. Este processo filho salva todos os dados no momento em que o comando `BGSAVE` é processado. Graças ao mecanismo `Copy-On-Write` (*COW*), não há necessidade de o processo filho usar a mesma quantidade de memória que o *Redis Server*.

Durante o processo de dumping, podemos usar o comando `INFO Persistence` para obter as métricas da persistência contínua. O valor de `rdb_bgsave_in_progress` quando for `1`, indica que o dumping está em andamento. O valor de `rdb_current_bgsave_time_sec` é o tempo que o `BGSAVE` está em andamento.

Depois que o processo termina de substituir o arquivo de dump antigo, ela registra uma mensagem com base na quantidade de dados que o processo usou da seguinte maneira:

```
30:C 15 Oct 2021 18:30:26.310 * RDB: 102 MB of memory used by copy-on-write
1:M 15 Oct 2021 18:30:26.348 * Background saving terminated with success
```

Depois que o `BGSAVE` terminar, o status do último `BGSAVE` será salvo na métrica `rdb_last_bgsave_status`, e a data/hora Unix do último salvamento bem-sucedido é salvo em `rdb_last_save_time`:

```bash
127.0.0.1:6379> INFO Persistence
# Persistence
rdb_last_save_time:1634322626
rdb_last_bgsave_status:ok
```

O dump gerado pelos comandos `SAVE` ou `BGSAVE` pode ser usado como um arquivo de backup de dados. Você pode utilizar o `crontab` para copiar periodicamente o arquivo **RDB** para um diretório ou sistema de arquivos distribuídos, como o Amazon *S3*, para fins de restauração posterior:

```bash
cp /data/dump.rdb /somewhere/safe/dump.$(date +%Y%m%d%H%M).rdb
```

Para restaurar um snapshot **RDB**, você precisa copiar o arquivo para o local especificado pelo parâmetro `dir` (`127.0.0.1:6379> CONFIG GET dir`) na configuração do *Redis* e definir a permissão de leitura/gravação desse arquivo para o usuário que inicia a instância *Redis*. Depois disso, você precisa interromper a instância executando o comando `SHUTDOWN NOSAVE` e renomear o novo arquivo de dump para o nome definido pelo parâmetro `dbfilename` (`127.0.0.1:6379> CONFIG GET dbfilename`). Com uma reinicialização, o conjunto de dados é carregado no *Redis* a partir do arquivo de backup.

### Manipulando `AOF`

No tópico anterior, aprendemos sobre a opção **RDB** para persistência de dados no Redis. No entanto, usar *RDB* para persistência não fornece uma durabilidade muito forte. Como mencionado, um dump de dados *RDB* contém apenas um snapshot de dados *point-in-time* do Redis. Embora o Redis dumps dados no arquivo *RDB* periodicamente, os dados entre os dois dumps *RDB* serão perdidos permanentemente quando um processo Redis travar ou houver uma falha de hardware. **AOF** é um arquivo de log *append-only* que registra os comandos de gravação do *Redis Server*. A durabilidade dos dados no **AOF** é maior do que no **RDB**, porque cada comando de gravação é anexado ao arquivo. Neste tópico, mostraremos como ativar o **AOF** no Redis e introduziremos alguns parâmetros de configuração importantes do *AOF*.

#### Principais comandos

As etapas para manipular o *AOF* são as seguintes:

Para habilitar a persistência **AOF** para uma instância *Redis* em execução, execute o comando `CONFIG SET` da seguinte maneira:

```bash
127.0.0.1:6379> CONFIG SET appendonly yes
# OK
```

Se você quiser ativar o *AOF* permanentemente, adicione `appendonly yes` ao arquivo de configuração do Redis e reinicie o servidor:

```bash
cat redis.conf | grep "^appendonly" appendonly yes
```

Para desativar o recurso de persistência *AOF* para uma instância *Redis* em execução, defina `appendonly` como `no`:

```bash
redis-cli config set appendonly no
# OK
```

Para desativar a persistência *AOF* permanentemente, basta definir `appendonly no` no arquivo de configuração e reiniciar o servidor:

```bash
cat redis.conf | grep "^appendonly" appendonly no
```

Para verificar se o recurso *AOF* está ativado, podemos usar `INFO PERSISTENCE` e procurar os itens *AOF*:

```bash
127.0.0.1:6379> INFO PERSISTENCE
# # Persistence
# loading:0
# ...
# aof_enabled:1
# aof_rewrite_in_progress:0
# aof_rewrite_scheduled:0
# aof_last_rewrite_time_sec:0
# aof_current_rewrite_time_sec:-1
# aof_last_bgrewrite_status:ok
# aof_last_write_status:ok
# aof_last_cow_size:225280
# module_fork_in_progress:0
# module_fork_last_cow_size:0
```

#### Entendendo como funciona

Quando a opção **AOF** estiver ativada, o *Redis* criará o arquivo **AOF**. O nome de arquivo AOF padrão é `appendonly.aof`, que pode ser alterado definindo o parâmetro `appendfilename` na configuração `.conf`. O *Redis* também preencherá o arquivo AOF com os dados atuais na memória.

Sempre que o Redis receber um comando de gravação que resultará em uma mudança real dos dados na memória, ele também adicionará o comando ao arquivo **AOF**. No entanto, se analisarmos mais profundamente o processo de gravação do arquivo, o sistema operacional realmente manterá um buffer no qual os comandos do Redis serão gravados primeiro. Os dados no buffer precisam ser liberados no disco para serem salvos permanentemente. Este processo é feito pela API de chamada de sistema Linux `fsync()`, que é uma chamada de bloqueio até que o disco relate que os dados no buffer foram completamente liberados.

Podemos ajustar a frequência de chamar `fsync()` ao anexar comandos ao arquivo **AOF**. Isso é feito com o parâmetro de configuração Redis `appendfsync`, e há três opções:

- `always`: `fsync()` será chamado para cada comando de gravação. Esta opção garante que apenas um comando será perdido em caso de falha inesperada no servidor ou falha de hardware. No entanto, como `fsync()` é uma chamada de bloqueio, o desempenho do Redis será limitado pelo desempenho de gravação do disco físico. Não é aconselhável definir `appendfsync` como `always` porque o desempenho do Redis será degradado significativamente.

- `everysec`: `fsync()` será chamado pelo Redis uma vez por segundo. Com essa opção, apenas um segundo dos dados será perdido em qualquer evento desastroso inesperado. Muitas vezes, é recomendável que você use essa opção para equilibrar a durabilidade e o desempenho dos dados.

- `no`: `fsync()` nunca será chamado pelo Redis. Esta opção deixa o sistema operacional decidir quando liberar os dados do buffer para o disco. Na maioria dos sistemas Linux, a frequência é a cada 30 segundos.

Quando o *Redis Server* estiver sendo encerrado, `fsync()` será chamado explicitamente para garantir que todos os dados de gravação no buffer sejam liberados para o disco.

O arquivo **AOF** será usado para restaurar os dados quando o *Redis Server* estiver sendo iniciado. O *Redis* apenas reproduz o arquivo lendo os comandos e aplicando-os à memória um por um. Os dados serão reconstruídos depois que todos os comandos forem processados.

Enquanto o Redis continua anexando comandos de gravação ao arquivo **AOF**, o tamanho do arquivo pode crescer significativamente. Isso retardará o processo de repetição quando o *Redis* estiver iniciando. O *Redis* fornece um mecanismo para compactar o arquivo **AOF** com uma reescrita *AOF*. Como você deve ter adivinhado, algumas das chaves Redis foram excluídas ou expiraram, para que possam ser limpas no arquivo *AOF*. Os valores para certas chaves Redis foram atualizados várias vezes e apenas o valor mais recente precisa ser armazenado no arquivo AOF. Esta é a ideia básica da compactação dos dados em uma *reescrita AOF*. Podemos usar o comando `BGREWRITEAOF` para iniciar o processo de reescrita ou permitir que o Redis execute uma reescrita *AOF* automaticamente.

Além de acionar manualmente uma reescrita **AOF** com o comando `BGREWRITEAOF`, também podemos configurar o Redis para executar uma reescrita *AOF* automaticamente. Os dois parâmetros de configuração a seguir são para esta finalidade:

- `auto-aof-rewrite-min-size`:  Uma reescrita **AOF** não será acionada se o tamanho do arquivo *AOF* for menor que esse valor. O valor padrão é `64 MB`.

- `auto-aof-rewrite-percentage`: O *Redis* se lembrará do tamanho do arquivo *AOF* após a última operação de *reescrita AOF*. Se o tamanho atual do arquivo *AOF* tiver aumentado nesse valor de percentual, outra reescrita **AOF** será acionada. Definir esse valor como `0` desativará a Reescrita Automática *AOF*. O valor padrão é `100`.

O comando `INFO PERSISTENCE` fornece muitas informações sobre uma reescrita **AOF**. Por exemplo, `aof_last_bgrewrite_status` indica o status da última operação de *reescrita AOF*, `aof_base_size` é o tamanho do último arquivo *AOF*,

O arquivo **AOF** pode estar corrompido ou truncado no final se o sistema operacional travar. O Redis fornece uma ferramenta, `redis-check-aof`, que pode ser usada para corrigir o arquivo AOF corrompido. Para corrigir um arquivo AOF, basta executar:

```bash
redis-check-aof --fix appendonly.aof
```
