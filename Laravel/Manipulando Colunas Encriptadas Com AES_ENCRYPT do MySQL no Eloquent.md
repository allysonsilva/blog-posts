---
id: 9a3c1041-cf7a-42e7-b79c-d52b8095c34b
title: "Manipulando Colunas Encriptadas com AES_ENCRYPT do MySQL no Eloquent"
summary: "Estenda as funções nativas de CRUD do Eloquent para manipular colunas encriptadas 🔒"
---

Se seu projeto precisa armazenar os dados de forma encriptados, o **MySQL** tem duas funções que pode fazer isso de forma nativa ao salvar o registro (`INSERT`, `UPDATE` ou ao recuperá-lo `SELECT`):

- [`AES_ENCRYPT`](https://dev.mysql.com/doc/refman/8.0/en/encryption-functions.html#function_aes-encrypt)
- [`AES_DECRYPT`](https://dev.mysql.com/doc/refman/8.0/en/encryption-functions.html#function_aes-decrypt)

As funções acima usam algoritmo [AES](https://pt.wikipedia.org/wiki/Advanced_Encryption_Standard) que por padrão utilizam *128 bits* como comprimento de chave.

## Usando encriptação de forma **raw**

Se você **não quiser reutilizar/automatizar o que será feito a seguir**, quiser toda vida a e vida toda usar de forma **raw** as funções de encriptação, então você pode fazer como da seguinte forma:

```php
$model = Model::create([
	'colA' => DB::raw("AES_ENCRYPT('value1', 'secretkey')"),
	'colB' => DB::raw("AES_ENCRYPT('value1', 'secretkey')"),
]);
```

E para recuperar os dados encriptados, teria que fazer o **decrypt** da coluna como da seguinte forma:

```php
Model::select(
	DB::raw('AES_DECRYPT(`colA`, "secretkey")'),
	DB::raw('AES_DECRYPT(`colB`, "secretkey")'),
);
```

_Pontos negativos em usar dessa forma:_

- Mesmo código de **Encrypt** e **Decrypt** está em vários locais resultando em um trabalho maior se houver algum refactor na forma em que os dados são encriptados.
- Deixa a *query* muito verbosa, ou melhor, o código passar a conhecer demais como a encriptação é feita, quando isso deveria estar na responsabilidade do ORM, pois o objetivo de um ORM é ser/fazer exatamente o contrário, abstrair os conceitos e coisas que um banco de dados faz para ser mais fácil de entender e manter o código.
- Quebra o conceito de **interoperabilidade** entre diferentes bancos de dados, que é um dos conceitos centrais de um ORM, poder trocar para diferentes tipos de bancos de dados, sem ter uma grande refatoração no código, pode até haver, mais mitiga os locais no código para alteração, já que o encapsulamento, coesão das responsabilidades de um ORM garante e dá segurança exatamente dessa alteração.
- Se a aplicação pretende mudar entre diferentes bancos de dados por algum motivo, não daria certo porque as funções de `AES_ENCRYPT` e `AES_DECRYPT` só existem no **MySQL**, assim, não seria possível mudar entre diferentes bancos de dados de forma dinâmica sem ter uma grande refatoração no código.

## Estendendo o Eloquent para usar encriptação na Model

Para usar a encriptação de forma **extensível**, **reutilizável**, **encapsulado** por tipo/driver de banco de dados e **interoperável**, veja o que deve ser feito.

### Como será quando tiver terminado de ler esse artigo?

Será muito simples 😎, tão simples quanto colocar uma propriedade do tipo `array` contendo as colunas que serão manipuladas de forma encriptadas, e deixar toda manipulação/responsabilidade para as novas classes junto com Eloquent, basicamente a *Model* deverá ter a seguinte propriedade com as colunas que devem ser manipuladas de forma encriptadas:

```php
    /**
     * Columns that should be manipulated MySQL native encryption.
     *
     * @var string[]
     */
    protected array $encryptedColumns = [
        'colA',
        'colA',
    ];
```

### Criando o **BaseModel**

Primeira coisa a ser feito é criar uma classe **BaseModel**, para sobrescrever o método `newEloquentBuilder` e ter os métodos e propriedades que esssa estratégia requer.

```php
<?php

namespace App\Support\ORM;

use Illuminate\Database\Eloquent\Model;

class BaseModel extends Model
{
    /**
     * Columns that should be manipulated MySQL native encryption.
     *
     * @var string[]
     */
    protected array $encryptedColumns = [];

    /**
     * Retrieves the columns that will be manipulated in an encrypted way.
     *
     * @return string[]
     */
    public function getEncryptedColumns(): array
    {
	return $this->encryptedColumns;
    }

    /**
     * Create a new Eloquent query builder for the model.
     *
     * @param  \Illuminate\Database\Query\Builder  $query
     *
     * @return \Illuminate\Database\Eloquent\Builder|static
     */
    public function newEloquentBuilder($query): BaseEloquentBuilder
    {
	return new BaseEloquentBuilder($query);
    }

    /**
     * Begin querying the model.
     *
     * @return \App\Support\ORM\BaseEloquentBuilder
     */
    public static function query(): BaseEloquentBuilder
    {
	return parent::query();
    }
}
```

### Criando o **BaseEloquentBuilder** e **BaseQueryBuilder** para usar `AES_ENCRYPT`

Deve ser criado um novo **Eloquent Builder** para que a propriedade `$encryptedColumns` presente na **Model**, possa ser transferida para as classes em que devem ser manipuladas, nesse caso a classe de `BaseQueryBuilder` e também `MySqlGrammarEncrypt`.

#### **BaseEloquentBuilder**

O método  `setModel` no *query builder* é sobrescrito basicamente adicionando a linha de `$this->query->setEncryptedColumns($model->getEncryptedColumns());` fazendo com que o array de colunas que deve ser encriptados presente na **Model** (`$encryptedColumns`), possa ser utilizado na classe de **Query** (`BaseQueryBuilder`) e **Grammar** (`MySqlGrammarEncrypt`).

```php
<?php

namespace App\Support\ORM;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Builder as BaseBuilder;

class BaseEloquentBuilder extends BaseBuilder
{
    /**
     * Create a new Eloquent query builder instance.
     *
     * @param \App\Support\ORM\BaseQueryBuilder $query
     *
     * @return void
     */
    public function __construct(BaseQueryBuilder $query)
    {
        parent::__construct($query);
    }

    /**
     * @inheritDoc
     */
    public function setModel(Model $model): static
    {
        $this->model = $model;

        $this->query->from($model->getTable());

        $this->query->setEncryptedColumns($model->getEncryptedColumns());

        return $this;
    }
}
```

#### **BaseQueryBuilder**

A classe abaixo é usada na manipulação da construção dos *statements* do banco de dados, dentre as instruções, está o `INSERT` e o `UPDATE`.

O array com as colunas que devem ser encriptados na propriedade `$encryptedColumns` presente na **Model**, está presente na classe abaixo, para que cada coluna possa ser transformada, adicionando `AES_ENCRYPT` ao *statement*.

```php
<?php

namespace App\Support\ORM;

use Illuminate\Database\Query\Builder as QueryBuilder;

class BaseQueryBuilder extends QueryBuilder
{
    /**
     * Columns that will be handled encrypted.
     *
     * @var string[]|null
     */
    protected ?array $encryptedColumns = [];

    /**
     * Columns that will be handled encrypted.
     *
     * @param string[] $encryptedColumns
     *
     * @return $this
     */
    public function setEncryptedColumns(array $encryptedColumns): static
    {
        $this->encryptedColumns = $encryptedColumns;

        $this->grammar->setEncryptedColumns($this->getEncryptedColumns());

        return $this;
    }

    /**
     * Columns that will be handled encrypted.
     *
     * @return string[]
     */
    public function getEncryptedColumns(): array
    {
        return ! empty($this->encryptedColumns) ? $this->encryptedColumns : [];
    }

    /**
     * @inheritDoc
     */
    public function insertGetId(array $values, $sequence = null)
    {
        $values = $this->grammar->rawSaveColumns($this->getEncryptedColumns(), $values);

        return parent::insertGetId($values, $sequence);
    }

    /**
     * @inheritDoc
     */
    public function insert(array $values)
    {
        $values = $this->grammar->rawSaveColumns($this->getEncryptedColumns(), $values);

        parent::insert($values);
    }

    /**
     * @inheritDoc
     */
    public function update(array $values)
    {
        $values = $this->grammar->rawSaveColumns($this->getEncryptedColumns(), $values);

        parent::update($values);
    }
}
```

**Os métodos de `insertGetId` e `insert` são usados:**

- Quando chamado o método [`save`](https://laravel.com/docs/eloquent#inserts) do Eloquent em uma nova instância da Model, resultando em um `INSERT` no banco.
- Quando chamado método `::create` do Eloquent também.

**O método de `update` é usado:**

- Chamando o método  [`save`](https://laravel.com/docs/eloquent#updates) em uma Model já preenchida / recuperada do banco de dados.
- Usando [Mass Updates](https://laravel.com/docs/eloquent#mass-updates) com método de `->update` na Model no Eloquent.

### Estendendo a conexão do MySQL para manipular o `SELECT` e usar o `AES_DECRYPT`

Já configuramos o `INSERT` e o `UPDATE` adicionando a função `AES_ENCRYPT` nas colunas que devem ser encriptadas. O que falta fazer agora é fazer o **decrypt** da informação no *statement* de `SELECT` e utilizar o `AES_DECRYPT` para recuperar as informações sem encriptação, de forma legível.

Para fazer isso, precisamos estender a conexão padrão do MySQL para manipular a construção do `SELECT` na classe de *Grammar* do Laravel, com isso, deve ser criado um `MySqlConnectionEncrypt`, como da seguinte forma:

```php
<?php

namespace App\Support\Database\Connections;

use App\Support\ORM\BaseQueryBuilder;
use Illuminate\Database\MySqlConnection;
use App\Support\Database\Query\Grammars\MySqlGrammarEncrypt;

class MySqlConnectionEncrypt extends MySqlConnection
{
    /**
     * @inheritDoc
     */
    protected function getDefaultQueryGrammar()
    {
        return $this->withTablePrefix(new MySqlGrammarEncrypt);
    }

    /**
     * @inheritDoc
     */
    public function query()
    {
        return new BaseQueryBuilder(
            $this, $this->getQueryGrammar(), $this->getPostProcessor()
        );
    }
}
```

Agora é necessário configurar o **service container** do Laravel para que em vez de configurar a conexão padrão do MySQL, deve ser utilizando a classe acima, então, o método `register` do `AppServiceProvider` deve ser como da seguinte forma:

```php
<?php

namespace App\Providers;

use Illuminate\Database\Connection;
use Illuminate\Support\ServiceProvider;
use App\Support\Database\Connections\MySqlConnectionEncrypt;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        Connection::resolverFor('mysql', function ($connection, $database, $prefix, $config) {
            return new MySqlConnectionEncrypt($connection, $database, $prefix, $config);
        });
    }
}
```

#### Criado **MySqlGrammarEncrypt**

Essa classe é a principal, pois vai encapsular a criação das colunas encriptadas nos comandos de `INSERT` e `UPDATE` e também, fazer o *decrypt* na instrução de `SELECT`. É responsável por recuperar as partes de uma **query**. Por exemplo o método `compileSelect` (que está presente na classe abaixo) é responsável por recuperar a instrução `SELECT colA, colB ...` como uma string que será utilizado no PDO do PHP, ou o método `compileWheres`, é responsável por recuperar as instruções de `where` da query.

Foi necessário substituir o método `compileSelect` exatamente para que as colunas encriptadas possam ser transformadas, e recuperadas de forma correta do banco de dados, ou seja, com a função `AES_DECRYPT`.

Além do mais, essa classe encapsulará a **key** (`$AESEncryptKey`) para manipular a encriptação dos dados, e sabe/conhece/responsável por encriptar e decriptar as informações, centralizando, encapsulando em um único local, fazendo com que exista um único local para alteração sendo visível a todos que o utilizam.

```php
<?php

namespace App\Support\Database\Query\Grammars;

use InvalidArgumentException;
use Illuminate\Database\Query\Builder;
use Illuminate\Database\Query\Grammars\MySqlGrammar;
use Illuminate\Database\Query\Expression as RawExpression;

class MySqlGrammarEncrypt extends MySqlGrammar
{
    /**
     * Key used to encrypt and decrypt columns in DB.
     *
     * @var string|null
     */
    protected readonly ?string $AESEncryptKey;

    /**
     * Columns that will be handled encrypted.
     *
     * @var string[]
     */
    protected array $encryptedColumns = [];

    /**
     * Create a new Mysql query Grammar instance.
     *
     * @return void
     */
    public function __construct()
    {
        $this->AESEncryptKey = config('app.aesencrypt_key');

        if (empty($this->AESEncryptKey)) {
            throw new InvalidArgumentException('Set encryption key in .env file, use this alias APP_AESENCRYPT_KEY');
        }
    }
    
    /**
     * Columns that will be handled encrypted.
     *
     * @param string[] $encryptedColumns
     *
     * @return $this
     */
    public function setEncryptedColumns(array $encryptedColumns): static
    {
        $this->encryptedColumns = $encryptedColumns;

        return $this;
    }

    /**
     * @inheritDoc
     */
    public function compileSelect(Builder $query): string
    {
        $this->encryptedColumns = $query->getEncryptedColumns();

        $encryptedColumnsForSelect =  $this->rawSelectEncryptedColumns($this->encryptedColumns);

        if (! empty($encryptedColumnsForSelect)) {
            $query->columns = array_merge(($query->columns ?? []), $encryptedColumnsForSelect);
        }

        return parent::compileSelect($query);
    }

    /**
     * @inheritDoc
     */
    protected function wrapValue($value)
    {
        // If the key to be handled in the MySQL statements is in the `encryptedColumns` array,
        // then that column is encrypted and must be handled correctly
        if (in_array($value, $this->encryptedColumns)) {
            return $this->wrapColumnDecrypt($value);
        }

        return parent::wrapValue($value);
    }

    /**
     * Converting the columns in string format to a `raw` format, to be used as it will be by Laravel.
     *
     * @param string[] $encryptedColumns
     *
     * @return array<\Illuminate\Database\Query\Expression>
     */
    public function rawSelectEncryptedColumns(array $encryptedColumns): array
    {
        return array_map(fn ($column) => new RawExpression($this->wrapColumnDecrypt($column, true)), $encryptedColumns);
    }

    /**
     * Transforming encrypted columns to the correct format to be saved in the database.
     *
     * @param string[] $encryptedColumns
     * @param string[] $values
     *
     * @return array<string|\Illuminate\Database\Query\Expression>
     */
    public function rawSaveColumns(array $encryptedColumns, array $values): array
    {
        $this->encryptedColumns = [];

        return collect($values)->map(function ($value, $key) use ($encryptedColumns) {
            if (is_array($value)) {
                return $this->rawSaveColumns($encryptedColumns, $value);
            }

            if (in_array($key, $encryptedColumns)) {
                return new RawExpression($this->wrapValueEncrypt($value));
            }

            return $value;
        })->toArray();
    }

    /**
     * Decrypts a certain column to treat it correctly.
     *
     * @param string $column
     * @param bool $withAlias
     *
     * @return string
     */
    protected function wrapColumnDecrypt(string $column, bool $withAlias = false): string
    {
        return "AES_DECRYPT(`{$column}`, '{$this->AESEncryptKey}')" . ($withAlias ? " AS '{$column}'" : '');
    }

    /**
     * Wrap a single string in keyword identifiers.
     *
     * @param string $value
     *
     * @return string
     */
    protected function wrapValueEncrypt(string $value): string
    {
        return "AES_ENCRYPT('{$value}', '{$this->AESEncryptKey}')";
    }
}
```

No arquivo de `config/app.php` adicione:

```php
// Key used to encrypt and decrypt data in the database
// using `AES_ENCRYPT()` and `AES_DECRYPT()` functions native to MySQL.
'aesencrypt_key' => env('APP_AESENCRYPT_KEY', 'secretKey'),
```

## Usando na Model

Antes de usar na Model as colunas que devem ser encriptadas, o MySQL recomenda que o tipo da coluna seja [`VARBINARY`](https://dev.mysql.com/doc/refman/8.0/en/binary-varbinary.html "11.3.3 The BINARY and VARBINARY Types") or [`BLOB`](https://dev.mysql.com/doc/refman/8.0/en/blob.html "11.3.4 The BLOB and TEXT Types"), para evitar possíveis problemas com remoção de espaço à direita ou na conversão dos caracteres que alterariam os valores dos dados, como pode ocorrer se for usado o `varchar` não binário.

Para usar essa implementação, basta apenas a Model em questão, estender a classe de `BaseModel` e adicionar a propriedade `protected  array $encryptedColumns` com as colunas que devem ser manipuladas de forma encriptadas.

### Exemplos

Supondo que tenhamos as colunas que devem ser manipuladas de forma encriptadas em um Model qualquer:

```php
/**
 * Columns that should be manipulated MySQL native encryption.
 *
 * @var string[]
 */
protected array $encryptedColumns = [
	'colA',
	'colB',
];
```

#### `SELECT`

Se for feito: `MyModel::whereId(1)->first()` no Eloquent, a query no banco de dados que seria executado é a seguinte:

```sql
SELECT
	*,
	AES_DECRYPT( `colA`, 'secretkey' ) AS 'colA',
	AES_DECRYPT( `colB`, 'secretkey' ) AS 'colB' 
FROM
	`any_table` 
WHERE 
	`id` = 1 
LIMIT 1
```

#### `INSERT`

Se for feito uma criação/inserção de um novo registro no Eloquent:

```php
MyModel::create([
    'colA' => 'Nisi neque consectetur',
    'name' => 'Mussum Ipsum',
    'colB' => 'xyz',
    'description' => 'Mussum Ipsum, cacilds vidis litro abertis.',
]);
```

A seguinte **query** seria executado no MySQL:

```sql
INSERT INTO `any_table` ( `colA`, `name`, `colB`, `description` )
VALUES
	(
		AES_ENCRYPT( 'Nisi neque consectetur', 'secretkey' ),
		'Mussum Ipsum',
		AES_ENCRYPT( 'xyz', 'secretkey' ),
		'Mussum Ipsum, cacilds vidis litro abertis.' 
	)
```

#### `UPDATE`

Qualquer uma dos seguintes códigos para atualizar o modelo, resultará na mesma query no SQL:

```php
# Forma 1
$model = MyModel::whereId(1)->first();
$model->colA = 'ABC';
$model->name = 'XYZ';
$model->colB = 'Mussum';
$model->description = 'Ipsum';
$model->save();

# Forma 2
MyModel::whereId(1)->update([
    'colA' => 'ABC',
    'name' => 'XYZ',
    'colB' => 'Mussum',
    'description' => 'Ipsum',
]);
```

Qualquer uma das forma acima terá a mesma instrução de `UPDATE`:

```sql
UPDATE 
	`any_table` 
SET 
	`colA` = AES_ENCRYPT( 'ABC', 'secretkey' ),
	`name` = 'XYZ',
	`colB` = AES_ENCRYPT( 'Mussum', 'secretkey' ),
	`description` = 'Ipsum' 
WHERE `id` = 1
```

## Resumo

Usar essa implementação para manipular as funções de `AES_ENCRYPT` e `AES_DECRYPT` no **Eloquent** é uma boa forma de automatizar, centralizar e encapsular tudo isso.
