---
id: 9953d8dd-522f-4a16-af06-7ba7d88f39a9
title: "Criando Feed de Casos de Usos Usando Notifications com Channel Personalizado de Banco de Dados"
summary: "Crie um feed simples com notifications do Laravel 📌"
---

Quando uma notificação deve ser lançada por conta de um caso de uso específico no fluxo do negócio, avisando ao usuário sobre determinada ação/atividade, então, nesse caso é útil ter uma tabela pra poder "listar" e manipular essa notificações pra serem visíveis posteriormente pelo usuário (**feed de atividades dos casos de usos**), além do mais, o próprio usuário pode também fazer determinada ação que acione uma notificação do lado da aplicação para "avisar"/responder ao gatilho de sua ação.

> Esse _feed de notificações_ nada mais é do que uma lista de atividades dos casos de usos dos fluxos da aplicação.

Para que isso possa acontecer, primeiro é necessário criar a tabela para que os dados da notificação possam ser persistidos para que posteriormente possam ser manipulados, isso é, possam ser recuperados, marcados como lidos, ou até mesmo fazer outra ação com determinada notificação.

## Criando tabela para manipular as notificações

```php
<?php

use App\Enums\UserNotificationType;
use App\Enums\UserNotificationStatus;
use Illuminate\Support\Facades\Schema;
use App\Enums\UserNotificationCausedBy;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

return new class extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('user_notifications', function (Blueprint $table) {
            $table->id();
            $table->unsignedInteger('user_id');
            $table->nullableMorphs('subject', 'subject');
            $table->enum('caused_by', UserNotificationCausedBy::values());
            $table->enum('type', UserNotificationType::values())->index();
            $table->enum('status', UserNotificationStatus::values())->default(UserNotificationStatus::SUCCESS->value);
            $table->string('title')->nullable();
            $table->string('message');

            $table->timestamp('read_at')->index()->nullable();
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('user_notifications');
    }
};
```

A coluna de `subject` contêm o recurso em que está sendo manipulado, por exemplo, quando o usuário envia determinado documento para análise da plataforma, o `subject` nesse caso seria o registro do documento em que está sendo enviado.

A coluna de `caused_by` é quando uma notificação é causada pelo usuário, de acordo com algum caso de uso, ou quando é causada pelo sistema, como se fosse a resposta de determinada ação do usuário.

A coluna de `type` é pra ser possível filtrar e manipular determinado tipo de notificação.

A coluna de `status` é pra saber se a notificação é uma notificação de `success` ou `failed`.

O _enum_ de `UserNotificationCausedBy` tem o seguinte conteúdo:

```php
<?php

namespace App\Enums;

enum UserNotificationCausedBy: string
{
    case USER = 'user';
    case SYSTEM = 'system';

    public static function values(): array
    {
        return array_column(self::cases(), 'value');
    }
}
```

O _enum_ de `UserNotificationType` tem o seguinte conteúdo:

```php
<?php

namespace App\Enums;

enum UserNotificationType: string
{
    case PAYMENT_DONE = 'payment_done';
    case ORDER_IN_PROCESS = 'order_in_process';
    // e more ...

    public static function values(): array
    {
        return array_column(self::cases(), 'value');
    }
}
```

O _enum_ de `UserNotificationStatus` tem o seguinte conteúdo:
 
```php
<?php

namespace App\Enums;

enum UserNotificationStatus: string
{
    case SUCCESS = 'success';
    case FAILED = 'failed';

    public static function values(): array
    {
        return array_column(self::cases(), 'value');
    }
}
```

### Criando Model de `UserNotification`

```php
<?php

namespace App\Support\Notifications;

use App\Models\User;
use App\Enums\UserNotificationType;
use App\Enums\UserNotificationStatus;
use App\Enums\UserNotificationCausedBy;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Relations\MorphTo;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class UserNotification extends Model
{
    /**
     * The table associated with the model.
     *
     * @var string
     */
    protected $table = 'user_notifications';

    /**
     * The attributes that are mass assignable.
     *
     * @var array<string>
     */
    protected $fillable = [
        'user_id',
        'subject_type',
        'subject_id',
        'caused_by',
        'type',
        'status',
        'title',
        'message',
        'read_at',
    ];

    /**
     * The attributes that should be cast.
     *
     * @var array<string, mixed>
     */
    protected $casts = [
        'caused_by' => UserNotificationCausedBy::class,
        'type' => UserNotificationType::class,
        'status' => UserNotificationStatus::class,
        'read_at' => 'datetime',
    ];

    /**
     * The relationships that should always be loaded.
     *
     * @var array
     */
    protected $with = ['user'];

    /**
     * The accessors to append to the model's array form.
     *
     * @var array
     */
    protected $appends = ['humanize_created_at'];

    public function getHumanizeCreatedAtAttribute(): string
    {
        return $this->created_at->diffForHumans();
    }

    /**
     * Get the notifiable entity that the notification belongs to.
     *
     * @return \Illuminate\Database\Eloquent\Relations\MorphTo
     */
    public function subject(): MorphTo
    {
        return $this->morphTo();
    }

    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class, 'user_id');
    }

    /**
     * Mark the notification as read.
     *
     * @return void
     */
    public function markAsRead(): void
    {
        if (is_null($this->read_at)):
            $this->forceFill(['read_at' => $this->freshTimestamp()])->save();
        endif;
    }

    /**
     * Marks many notifications in the `$ids` parameter as read.
     *
     * @param array<int> $ids
     *
     * @return int
     */
    public function markManyAsRead(array $ids): int
    {
        $updatedRow = $this->distinct()
                           ->whereKey($ids)
                           ->update([
                               'read_at' => $this->freshTimestamp()
                           ]);

        return $updatedRow;
    }

    /**
     * Mark the notification as unread.
     *
     * @return void
     */
    public function markAsUnread(): void
    {
        if (! is_null($this->read_at)):
            $this->forceFill(['read_at' => null])->save();
        endif;
    }

    /**
     * Determine if a notification has been read.
     *
     * @return bool
     */
    public function isRead(): bool
    {
        return $this->read_at !== null;
    }

    /**
     * Determine if a notification has not been read.
     *
     * @return bool
     */
    public function isUnread(): bool
    {
        return $this->read_at === null;
    }

    /**
     * Scope a query to only include read notifications.
     *
     * @param \Illuminate\Database\Eloquent\Builder $query
     *
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function scopeRead(Builder $query)
    {
        return $query->whereNotNull('read_at');
    }

    /**
     * Scope a query to only include unread notifications.
     *
     * @param \Illuminate\Database\Eloquent\Builder $query
     *
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function scopeUnread(Builder $query)
    {
        return $query->whereNull('read_at');
    }
}
```

### Trait para relacionar ao `subject`

Para relacionar o `subject` com as notificações, ou seja, para recuperar determinadas notificações em que determinada Model está nas colunas morph de `subject_type` e `subject_id` utilize a trait na model acima:

```php
<?php

namespace App\Support\Notifications\Traits;

use App\Support\Notifications\UserNotification;

trait UserNotifiable
{
    public function userNotifications()
    {
        return $this->morphMany(UserNotification::class, 'subject')->latest();
    }

    /**
     * Get the user's read notifications.
     *
     * @return \Illuminate\Database\Query\Builder
     */
    public function readUserNotifications()
    {
        return $this->userNotifications()->read();
    }

    /**
     * Get the user's unread notifications.
     *
     * @return \Illuminate\Database\Query\Builder
     */
    public function unreadUserNotifications()
    {
        return $this->userNotifications()->unread();
    }
}
```

## Criando driver/[channel](https://laravel.com/docs/10.x/notifications#custom-channels) de bancos de dados

Para [enviar](https://laravel.com/docs/10.x/notifications#sending-notifications) uma notificação e salvar ela na tabela acima, é necessário customizar, criar um canal personalizado no Laravel e usá-lo na notificação em que será disparada.

O nome do canal em que estamos manipulando vai se chamar `UserUseCaseChannel`, já que é sobre um canal de notificações de **casos de uso** do usuário.

O arquivo de `UserUseCaseChannel.php` tem o seguinte conteúdo:

```php
<?php

namespace App\Support\Notifications;

use App\Models\User;

class UserUseCaseChannel
{
    /**
     * Send the given notification.
     *
     * @param \App\Models\User $notifiable
     * @param \App\Support\Notifications\UserUseCaseNotification $notification
     *
     * @return void
     */
    public function send(User $notifiable, UserUseCaseNotification $notification): void
    {
        $notification->toUserUseCase($notifiable);
    }
}
```

O canal acima, usa o método de `toUserUseCase` que basicamente tem toda lógica de salvar os dados da notificação no banco de dados.

Para favorecer o principio do **DRY**, vamos criar uma classe abstrata de `UserUseCaseNotification` que terá o método de `toUserUseCase` além de alguns métodos obrigatórios que as classes de notificações terão de implementar. 

```php
<?php

namespace App\Support\Notifications;

use App\Models\User;
use App\Enums\UserNotificationType;
use App\Enums\UserNotificationStatus;
use App\Enums\UserNotificationCausedBy;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Notifications\Notification;

abstract class UserUseCaseNotification extends Notification
{
    protected readonly UserNotificationCausedBy $causedBy;

    protected ?Model $subject = null;

    public function __construct()
    {
        $this->causedBy = UserNotificationCausedBy::USER;
    }

    abstract public function type(): UserNotificationType;
    abstract public function status(): UserNotificationStatus;

    /**
     * Prepare the subject.
     *
     * @param \Illuminate\Database\Eloquent\Model $subject
     *
     * @return static
     */
    public function withSubject(Model $subject): static
    {
        $this->subject = $subject;

        return $this;
    }

    /**
     * Prepare the notification title.
     *
     * @return string|null
     */
    public function title(): ?string
    {
        return null;
    }

    /**
     * Prepare the notification message.
     *
     * @return string
     */
    public function message(): string
    {
        return 'default';
    }

    /**
     * Get the notification's delivery channels.
     *
     * @param mixed $notifiable
     *
     * @return array|string
     */
    public function via($notifiable): array|string
    {
        return UserUseCaseChannel::class;
    }

    /**
     * Get the "use case / activity" representation of the notification.
     *
     * @param \App\Models\User $notifiable
     *
     * @return \App\Models\UserNotification
     */
    public function toUserUseCase(User $notifiable): UserNotification
    {
        $userNotification = app(UserNotification::class);

        if (! is_null($this->subject)):
            $userNotification->subject()->associate($this->subject);
        endif;

        $userNotification->user_id = auth()->id() ?? $notifiable->getKey();
        $userNotification->caused_by = $this->causedBy;
        $userNotification->type = $this->type();
        $userNotification->status = $this->status();
        $userNotification->title = $this->title();
        $userNotification->message = $this->message();
        $userNotification->read_at = null;

        $userNotification->save();

        return $userNotification;
    }
}
```

A classe acima deverá ser utilizada apenas como uma classe "parent" pois se trata de uma classe abstrata com métodos abstratos. Principalmente os métodos de `type` e `status` que devem ter nas classes de notificações.

## Criando feed de notificação - Inserindo no banco de dados

Para criar uma notificação de "atividade" ou "caso de uso" do negócio, é necessário criar uma classe de Notification que estende a classe abstrata de `UserUseCaseNotification`.

Vamos criar uma classe de notificação pra quando o usuário tem o pagamento feito/processado com sucesso. O nome da classe vai se chamar `PaymentDone` com o seguinte código:

```php
<?php

namespace App\Notifications;

use App\Enums\UserNotificationType;
use App\Enums\UserNotificationStatus;
use App\Support\Notifications\UserUseCaseNotification;

class PaymentDone extends UserUseCaseNotification
{
    public function type(): UserNotificationType
    {
        return UserNotificationType::PAYMENT_DONE;
    }

    public function status(): UserNotificationStatus
    {
        return UserNotificationStatus::SUCCESS;
    }

    /**
     * Prepare the notification title.
     *
     * @return string
     */
    public function title(): string
    {
        return __('messages.payment done.title');
    }

    /**
     * Prepare the notification message.
     *
     * @return string
     */
    public function message(): string
    {
        return __('messages.payment done.message');
    }
}
```

E a notificação acima pode ser lançada como da seguinte forma:

```php
use App\Notifications\InvoicePaid;

$user->notify(new  PaymentDone());
```

Após isso, verá que existe um registro na tabela de `user_notifications`.
