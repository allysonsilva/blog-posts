---
id: 9949c8f2-cc32-4784-8919-3d56dae6454d
title: "Validando Conta do Usuário com Verificação em Duas Etapas"
summary: "Utilize a estratégia de verificação em duas etapas (2-Step Verification) para validar o cadastro de uma nova conta do usuário"
---

Quando um novo usuário é cadastrado no sistema, normalmente é enviado um e-mail para o mesmo tendo em seu conteúdo um "*link de ativação/verificação*" para saber se o e-mail colocado no formulário de cadastro realmente pertence a ele. Isso é útil quando o fluxo de um novo cadastro não deve ser bloqueado quando o mesmo não clicar no link enviado por e-mail, assim, ele poderia se cadastrar e já utilizar a plataforma de imediato com algumas ressalvas acredito, por conta do seu cadastro não estar 100% completo (sem ter o e-mail validado nesse casso).

Existe outro caso, que é quando o sistema não permite seu uso sem antes ter validado o e-mail e/ou senha, quando o uso dos **dados de contato** faz parte do fluxo de "_ativação da conta_", ou seja, a conta do usuário só é considerada _ativa_, quando os dados de contato estiverem sidos validados, caso contrário, existe algum outro status como "pendente de ativação" por exemplo, e ai, não usaria 100% da plataforma por conta dessa "limitação" do próprio usuário em _não validar os dados de contato_. Dessa forma, fica fácil distinguir entre contas/registros que estão íntegros/consistentes, e aqueles que ainda estão faltando alguma coisa pra que sua conta/registro seja considerado "_válida_".

## Resumo

1. O nome da tabela que será utilizada para armazenar os registros dos usuários do sistema vai se chamar `users` 😮, que terá todos os campos necessário para um novo registro do usuário.
    1. Vamos adicionar 3 novos campos na tabela pra poder manipular a validação da verificação dos codes.
    2. O nome da coluna que armazenará o telefone celular é `phone` e a coluna que terá o e-mail é `email`.
2. No fluxo abaixo, tanto o **e-mail**, quanto o **telefone celular**, devem ser satisfeitos, isso é, validados, para que a conta do usuário possa ser considerada como "_ativa_".
3. O fluxo de validação e verificação se inicial após o **signup** do usuário. Após ele se registrar no sistema, vai ser disparado inicialmente o code para o seu telefone, na tela seguinte, vai pedir o code recém enviado, inserindo e estando válido, então vai ser enviado o próximo code por e-mail, e na tela seguinte inserindo e estando correto o code, então o usuário seria considerado como _validado/íntegro/verificado_.
4. O serviço utilizado nesse artigo pra envio do SMS é o `Twilio`.
5. Tanto o envio do code por SMS quanto por e-mail será feito utilizando [`Jobs`](https://laravel.com/docs/10.x/queues#creating-jobs).

## Primeiros Passos

### Adicionando as novas colunas na tabela de `users`

A migration a seguir deve ser executada para que:

1. Os codes para validação e verificação sejam salvos na coluna de `verification_codes`.
2. Quando o code do e-mail fosse validado com sucesso, então, a coluna de `email_verified_at` teria a data em que isso aconteceu.
3. Quando o code do telefone via SMS fosse validado com sucesso, então, a coluna de `phone_verified_at` teria a data em que isso aconteceu.

```php
<?php

use Illuminate\Support\Facades\Schema;
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
        Schema::table('users', function (Blueprint $table) {
            $table->json('verification_codes')->after('password')->nullable();
            $table->timestamp('email_verified_at')->after('email')->nullable();
            $table->timestamp('phone_verified_at')->after('phone')->nullable();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::table('users', function (Blueprint $table) {
            $table->dropColumn('verification_codes');
            $table->dropColumn('email_verified_at');
            $table->dropColumn('phone_verified_at');
        });
    }
};
```

### Modificando a model de `User` para manipular as novas colunas

A model de `User` tem um dos principais métodos que é o `saveVerificationCode`, responsável por criar e salvar os códigos de verificação na coluna de `verification_codes`. Além do mais, métodos que recuperem os códigos `smsCode` e `emailCode`, e também o conceito de "ativação de conta do usuário" `activateAccount`, e saber se a conta do usuário é considerada como "verificada" `accountHasVerified`.

```php
<?php

namespace App\Models;

use App\Enums\UserStatus;
use App\Notifications\UserVerifyEmail;
use Illuminate\Notifications\Notifiable;
use App\Enums\UserAccountTypeVerification;
use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable implements MustVerifyEmail
{
    use Notifiable;

    /**
     * The attributes that are mass assignable.
     *
     * @var array<int, string>
     */
    protected $fillable = [
        'email',
        'password',
        'phone',
        'verification_codes',
        'email_verified_at',
        'phone_verified_at',
    ];

    /**
     * The attributes that should be cast.
     *
     * @var array
     */
    protected $casts = [
        'verification_codes' => 'object',
        'email_verified_at' => 'datetime',
        'phone_verified_at' => 'datetime',
    ];

	/**
     * Retrieve common user data.
     *
     * @return array
     */
    public function commonUserData(): array
    {
        return [
            'id' => $this->getKey(),
            'name' => $this->name,
            'email' => $this->email,
            'status' => $this->status,
            'has_phone_verified' => $this->hasVerifiedPhone(),
            'has_email_verified' => $this->hasVerifiedEmail(),
        ];
    }

    /**
     * Send the email verification notification.
     *
     * @return void
     */
    public function sendEmailVerificationNotification(): void
    {
        $this->notify(new UserVerifyEmail);
    }

    /**
     * Create the verification code structure to be saved in the database.
     *
     * @param \App\Enums\UserAccountTypeVerification $type
     * @param array|string|null $value
     *
     * @return int
     */
    public function saveVerificationCode(UserAccountTypeVerification $type, array|string|null $value = null): int
    {
        $code = random_int(100000, 999999);

        $data = [
            UserAccountTypeVerification::SMS->value => null,
            UserAccountTypeVerification::EMAIL->value => null,
        ];

        $data[$type->value] = [
            'code' => $code,
            'value' => $value,
            'created_at' => now(),
        ];

        $this->fill([
            'verification_codes' => $data,
        ])->save();

        return $code;
    }

    /**
     * Determine if the user has verified their email address.
     *
     * Checks if the Email code has been validated.
     *
     * @return bool
     */
    public function hasVerifiedEmail(): bool
    {
        return ! is_null($this->email_verified_at);
    }

    /**
     * Mark the given user's email as verified.
     *
     * @return bool
     */
    public function markEmailAsVerified(): bool
    {
        return $this->forceFill([
            'email_verified_at' => $this->freshTimestamp(),
        ])->save();
    }

    /**
     * Determine whether the user has verified their phone number.
     *
     * Checks if the SMS code has been validated.
     *
     * @return bool
     */
    public function hasVerifiedPhone(): bool
    {
        return ! is_null($this->phone_verified_at);
    }

    /**
     * Mark the given user's phone number as verified.
     *
     * @return bool
     */
    public function markPhoneAsVerified(): bool
    {
        return $this->forceFill([
            'phone_verified_at' => $this->freshTimestamp(),
        ])->save();
    }

    /**
     * Retrieves the user's SMS code.
     *
     * @return int|null
     */
    public function smsCode(): ?int
    {
        return intval($this->verification_codes?->sms?->code) ?: null;
    }

    /**
     * Retrieves the `value` column data from `sms`.
     *
     * @return mixed
     */
    public function phoneData(): mixed
    {
        return $this->verification_codes?->sms?->value;
    }

    /**
     * Retrieves the user's Email code.
     *
     * @return int|null
     */
    public function emailCode(): ?int
    {
        return intval($this->verification_codes?->email?->code) ?: null;
    }

    /**
     * Retrieves the `value` column data from `email`.
     *
     * @return mixed
     */
    public function emailData(): mixed
    {
        return $this->verification_codes?->email?->value;
    }

    /**
     * Reset the `verification codes` field.
     *
     * @return void
     */
    public function resetVerificationCodes(): void
    {
        if (empty($this->emailCode()) && empty($this->smsCode())) {
            $this->fill(['verification_codes' => null])->save();
        }
    }

    /**
     * Activate the user account.
     *
     * @return bool
     */
    public function activateAccount(): bool
    {
        return $this->forceFill([
            'status' => UserStatus::ENABLE,
            'verification_codes' => null,
        ])->save();
    }

    /**
     * Checks if the user account has been validated in all possible steps.
     *
     * @return bool
     */
    public function accountHasVerified(): bool
    {
        return  $this->status === UserStatus::ENABLE &&
                $this->verification_codes === null;
    }
}
```

O enum de `UserStatus` tem o seguinte código:

```php
<?php

namespace App\Enums;

enum UserStatus: string
{
    case ENABLE = 'enable';
    case DISABLE = 'disable';
}
```

O enum de `UserAccountTypeVerification` tem o seguinte código:

```php
<?php

namespace App\Enums;

enum UserAccountTypeVerification: string
{
    case SMS = 'sms';
    case EMAIL = 'email';
}
```

### Configurando o Twilio

Para enviar o code com Twilio, é necessário instalar o pacote oficial para o PHP, que nesse caso é o [`twilio/sdk`](https://github.com/twilio/twilio-php). Após instalá-lo, crie um arquivo chamado `twilio.php` na pasta de `config`, tendo o seguinte conteúdo:

```php
<?php

return [
    'number' => env('TWILIO_NUMBER'),
    'account_sid' => env('TWILIO_ACCOUNT_SID'),
    'auth_token' => env('TWILIO_AUTH_TOKEN'),
];
```

As credenciais acima são recuperadas através do painel no [dashboard](https://www.twilio.com/console) na conta do Twilio.

Para usar a classe de `Twilio\Rest\Client`, precisamos configurá-la no _service container no Laravel_, para favorecer o código com a [inversão de dependência](https://github.com/allysonsilva/estudos/blob/master/Arquitetura/SOLID.md#d-dependency-inversion-principle-dip), deixando o contêiner do Laravel resolvendo a criação desse objeto, assim, temos todos as vantagens do [princípio da inversão de dependência](https://github.com/allysonsilva/estudos/blob/master/Arquitetura/SOLID.md#d-dependency-inversion-principle-dip), além do mais, quando esse código for testado, e precisar simular, e saber se realmente o método está sendo chamada (testes de spy) ou mockar determinado método do Client pra ter determinado comportamento esperado pelo código, será beeemmmm mais fácil, pois novamente, estamos deixando a responsabilidade da instância desse objeto para o _Container do Laravel_, e não tendo um alto acoplamento usando `new` diretamente, que nesse caso, não seria possível simular/mockar `new` nos testes.

Na classe de `AppServiceProvider` no método de `register` adicione o seguinte código:

```php
use Twilio\Rest\Client;
use App\Support\APIs\SMSClient;
use App\Support\APIs\TwilioClient;

$this->app->bind(Client::class, fn () => new Client(config('twilio.account_sid'), config('twilio.auth_token')));

$this->app->bind(SMSClient::class, TwilioClient::class);
```

Agora vamos encapsular toda as chamadas ao serviço do Twilio em uma classe com nomes de métodos legíveis ao nosso código, fazendo um contrato entre essa nova classe e o código da aplicação, assim, quando o Client do Twilio mudar, não precisamos refatorar a aplicação em vários locais (se tiver usando diretamente o Client do Twilio ao invés da estratégia abaixo), precisa ser feito em apenas um único local/método, pois toda responsabilidade não fica mais disperso na aplicação, e sim em uma classe encapsulada/responsável por ser uma espécie de "proxy" basicamente entre o Client do Twilio e o código da aplicação. Então, a classe é `TwilioClient` que terá apenas o método `send`, que saberá como enviar o SMS para o usuário. Além do mais, se no futuro o projeto decidir mudar do Twilio para alguma outra ferramenta, é só novamente configurar no _Container do Laravel_ mudando a classe concreta na resolução de `$this->app->bind(SMSClient::class, TwilioClient::class);`.

```php
<?php

namespace App\Support\APIs;

use Twilio\Rest\Client;

class TwilioClient implements SMSClient
{
    private readonly string $twilioNumber;

    private const BR_DDI = '+55';

    public function __construct(private Client $twilio)
    {
        $this->twilioNumber = config('twilio.number');
    }

    public function send(string $toPhone, string $message): void
    {
        $this->twilio->messages->create(static::BR_DDI . $toPhone, [
            'from' => $this->twilioNumber,
            'body' => $message,
        ]);
    }
}
```

A interface de `SMSClient` tem apenas o contrato do método de `send`:

```php
<?php

namespace App\Support\APIs;

interface SMSClient
{
    public function send(string $toPhone, string $message): void;
}
```

### Organizando as rotas

As seguintes rotas, servem para exemplificar o fluxo que acontecerá, que é:

1. O usuário se cadastra na rota de `signup`, e nesse mesmo endpoint/código, após um novo registro ser inserido na tabela de `users`, então, é disparado o primeiro Job (`SendUserVerificationCodeBySMS`), que enviará o code por SMS para o telefone celular do usuário inserido no formulário de cadastro.
2. Após ter recebido o code via SMS no telefone, é então validado na rota de `account.verify` no endpoint de `POST /account/verify`.
	1. Após o code estar válido, então, será executado o segundo Job que nesse caso é pra enviar o code via e-mail (`SendUserVerificationCodeByEmail`).
3. O usuário recebendo o code no seu e-mail que colocou no formulário de cadastro, novamente deve ser validado em `POST /account/verify`.
4. O code do e-mail estando válido, então, o callback pode ser o que o negócio determina. Pode ser alterado o status do usuário, pode ser feito várias coisas, isso quem vai decidir é o negócio.
5. Se por acaso o usuário fechou a aplicação ou precisar reenviar o code por SMS ou e-mail, então, pode ser usando o endpoint de `POST /account/verify/resend`.

```php
Route::controller(RegisterController::class)->group(function () {
    Route::post('signup', 'signup')->middleware('guest')->name('signup');

    Route::middleware('auth')->name('logged.')->group(function () {
        Route::post('account/verify', 'accountVerify')->name('account.verify');
        Route::post('account/verify/resend', 'accountVerifyResend')->name('account.verify.resend');
    });
});
```

## Criando Job para enviar code por SMS usando Twilio

Após o usuário se cadastrar, é enviado um code por SMS para o seu telefone celular colocado no formulário de cadastro, então, no mesmo fluxo/código/método em que um novo usuário é cadastrado, deve ser feito o envio do code por SMS ou por E-mail, no nosso caso, primeiro é validado o telefone celular, e após isso, recebendo o code via SMS e validando-o, então, é enviado o code para seu e-mail.

O primeiro Job que será disparado para notificar o usuário via SMS pra que o mesmo posso estar validando seu telefone celular é o `SendUserVerificationCodeBySMS`, com o seguinte código:

```php
<?php

namespace App\Jobs;

use App\Models\User;
use Illuminate\Bus\Queueable;
use App\Support\APIs\TwilioClient;
use Illuminate\Queue\SerializesModels;
use Illuminate\Queue\InteractsWithQueue;
use App\Enums\UserAccountTypeVerification;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;

class SendUserVerificationCodeBySMS implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    /**
     * Create a new job instance.
     *
     * @param \App\Models\User $user
     * @param string|null $phone
     */
    public function __construct(public User $user, public ?string $phone = null)
    {
        //
    }

    /**
     * Execute the job.
     *
     * @param \App\Support\APIs\TwilioClient $twilioClient
     *
     * @return void
     */
    public function handle(TwilioClient $twilioClient): void
    {
        $phone = ($this->phone ?? $this->user->phone);

        $code = $this->user->saveVerificationCode(UserAccountTypeVerification::SMS, [
            'phone' => $phone,
        ]);

        $message = "$code is your verification code";

        $twilioClient->send($phone, $message);
    }
}
```

O job acima pode ser executado em um endpoint por exemplo de `POST /signup` em um controller de `RegisterController`, no método de `signup`, como no código abaixo:

```php
public function signup(AuthSignupRequest $request): Response
{
    DB::beginTransaction();

    try {
        $user  = $this->userService->signup($request->safe());

        SendUserVerificationCodeBySMS::dispatch($user);

        DB::commit();
    } catch (Exception $e) {
        DB::rollBack();

        throw $e;
    }

    return response()->json(['user' => $user->commonUserData()]);
}
``` 

## Criando Job para enviar code via e-mail

O Job a seguir será executado no segundo passo, quando o usuário já tiver validado o telefone celular, e precisará validar seu e-mail:

```php
<?php

namespace App\Jobs;

use App\Models\User;
use Illuminate\Bus\Queueable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Queue\InteractsWithQueue;
use App\Enums\UserAccountTypeVerification;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;

class SendUserVerificationCodeByEmail implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    /**
     * Create a new job instance.
     *
     * @param \App\Models\User $user
     * @param string|null $email
     *
     * @return void
     */
    public function __construct(public User $user, public ?string $email = null)
    {
    }

    /**
     * Execute the job.
     *
     * @return void
     */
    public function handle(): void
    {
        $this->user->email = $this->email ?? $this->user->email;
        $this->user->saveVerificationCode(UserAccountTypeVerification::EMAIL, $this->user->email);

        $this->user->sendEmailVerificationNotification();
    }
}
```

O método de `sendEmailVerificationNotification` na model de `User` é para enviar uma notificação via e-mail, que nesse caso é a `UserVerifyEmail` com o seguinte código:

```php
<?php

namespace App\Notifications;

use Illuminate\Notifications\Notification;
use Illuminate\Notifications\Messages\MailMessage;

class UserVerifyEmail extends Notification
{
    /**
     * Get the notification's channels.
     *
     * @param  mixed  $notifiable
     *
     * @return array
     */
    public function via($notifiable): array
    {
        return ['mail'];
    }

    /**
     * Build the mail representation of the notification.
     *
     * @param mixed $notifiable
     *
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable): MailMessage
    {
        $code = $notifiable->emailCode();

        return (new MailMessage)
            ->subject('Your verification code')
            ->greeting('See your verification code')
            ->line('Please open the app on your mobile and validate your email by entering the code below.')
            ->line('Use the following verification code: ' . $code);
    }
}
```

## Validando o code

Para validar o code tanto por SMS quanto por e-mail é o mesmo endpoint, mesmo body, o que muda é o tipo de validação, ou nesse caso, o valor do chave de `type` no body da requisição.

Mais primeiro, deve ser criado o `FormRequest` que valide corretamente o `POST` da request de validação do code, nesse caso, o arquivo de `AuthAccountVerifyRequest`.

```php
<?php

namespace App\Http\Requests;

use Illuminate\Validation\Rules\Enum;
use App\Enums\UserAccountTypeVerification;
use Illuminate\Foundation\Http\FormRequest;

class AuthAccountVerifyRequest extends FormRequest
{
    /**
     * Get the validation rules that apply to the request.
     *
     * @return array<string, mixed>
     */
    public function rules(): array
    {
        return [
            'type' => ['required', new Enum(UserAccountTypeVerification::class)],
            'code' => ['required', 'integer'],
        ];
    }

    /**
     * Retrieves account verification type.
     *
     * @return \App\Enums\UserAccountTypeVerification
     */
    public function type(): UserAccountTypeVerification
    {
        return UserAccountTypeVerification::from($this->input('type'));
    }
}
```

Agora, vamos criar uma classe chamada `UserService` pra ser um serviço responsável por manipular tudo aquilo que for relacionado ao usuário, contendo inicialmente os métodos de `accountVerify`, `verifySMSCode` e `verifyEmailCode`, com o seguinte código:

```php
<?php

namespace App\Services;

use App\Models\User;
use App\Enums\UserAccountTypeVerification;
use App\Jobs\SendUserVerificationCodeByEmail;
use App\Services\Exceptions\InvalidSMSCodeException;
use App\Services\Exceptions\InvalidEmailCodeException;
use App\Services\Exceptions\AccountHasAlreadyVerifiedException;

class UserService
{
    public function __construct(User $model)
    {
    }

    /**
     * Process to validate customer account verification.
     *
     * @param \App\Models\User $user
     * @param int $code
     * @param \App\Enums\UserAccountTypeVerification $type
     *
     * @return void
     */
    public function accountVerify(User $user, int $code, UserAccountTypeVerification $type): void
    {
        if ($user->accountHasVerified()) {
            throw new AccountHasAlreadyVerifiedException;
        }

        if ($this->verifySMSCode($user, $code, $type)) {
            SendUserVerificationCodeByEmail::dispatch($user);

            return;
        }

        $this->verifyEmailCode($user, $code, $type);

		// Aqui pode ser feito alguma ação de negócio quando ambos os steps (SMS/Telefone e E-mail)
		// forem validados com sucesso, nesse caso, o code está "ativando" a conta do usuário que 
		// outrora era considerada como "pendente de ativação"
        $user->activateAccount();
    }

    /**
     * Checks the user code sent via SMS.
     *
     * @param \App\Models\User $user
     * @param int $code
     * @param \App\Enums\UserAccountTypeVerification $type
     *
     * @return bool|null
     */
    public function verifySMSCode(User $user, int $code, UserAccountTypeVerification $type): ?bool
    {
        if ($type === UserAccountTypeVerification::SMS) {
            if ($user->smsCode() !== $code) throw new InvalidSMSCodeException;

            if (! $user->hasVerifiedPhone()) $user->markPhoneAsVerified();

            return true;
        }

        return null;
    }

    /**
     * Check the user code sent via Email.
     *
     * @param \App\Models\User $user
     * @param int $code
     * @param \App\Enums\UserAccountTypeVerification $type
     *
     * @return bool|null
     */
    public function verifyEmailCode(User $user, int $code, UserAccountTypeVerification $type): ?bool
    {
        if ($type === UserAccountTypeVerification::EMAIL) {
            if ($user->emailCode() !== $code) throw new InvalidEmailCodeException;

            if (! $user->hasVerifiedEmail()) $user->markEmailAsVerified();

            return true;
        }

        return null;
    }
}
```

No controller de `RegisterController`, adicione o método de `accountVerify`, com o seguinte código:

```php
<?php

namespace App\Http\Controllers\Auth;

use App\Services\UserService;
use Illuminate\Http\JsonResponse;
use Illuminate\Foundation\Bus\DispatchesJobs;
use App\Http\Requests\AuthAccountVerifyRequest;
use Illuminate\Foundation\Validation\ValidatesRequests;
use Illuminate\Foundation\Auth\Access\AuthorizesRequests;
use Illuminate\Routing\Controller as LaravelBaseController;

class RegisterController extends LaravelBaseController
{
    use AuthorizesRequests, DispatchesJobs, ValidatesRequests;

    /**
     * Create a new Controller instance.
     *
     * @param \App\Services\UserService $userService
     */
    public function __construct(private UserService $userService)
    {
    }

    /**
     * Process to validate customer account verification.
     *
     * @param \App\Http\Requests\AuthAccountVerifyRequest $request
     *
     * @return \Illuminate\Http\JsonResponse
     *
     * @throws \App\Services\Exceptions\InvalidSMSCodeException
     * @throws \App\Services\Exceptions\InvalidEmailCodeException
     * @throws \App\Services\Exceptions\AccountHasAlreadyVerifiedException
     */
    public function accountVerify(AuthAccountVerifyRequest $request): JsonResponse
    {
        $this->userService->accountVerify(auth()->user(), $request->code, $request->type());

        return response()->json(status: JsonResponse::HTTP_NO_CONTENT);
    }
}
```

Pronto, agora é só enviar a requisição com o *payload* de:

```json
{
    "type": "sms or email",
    "code": 123456
}
```

Onde o `type` pode ser `sms` ou `email`.

## Reenviado o code

Se o usuário demorou muito pra validar o code, ou existe alguma coisa de "skyp" naquele momento da aplicação pra que ele possa fazer isso depois, então, o código abaixo é exatamente pra isso, pra enviar o code via SMS ou e-mail em qualquer momento (de acordo com as regras do negócio).

Primeiro, adicione o método de `accountVerifyResend` na controller `RegisterController`:

```php
/**
 * Process to resend customer account verification code.
 *
 * @param \App\Http\Requests\AuthAccountVerifyResendRequest $request
 *
 * @return \Illuminate\Http\JsonResponse
 */
public function accountVerifyResend(AuthAccountVerifyResendRequest $request): JsonResponse
{
    $user = auth()->user();

    $this->userService->accountVerifyResend($user, $request->type());

	return response()->json(status: JsonResponse::HTTP_NO_CONTENT);
}
```

Crie um `FormRequest` chamado de `AuthAccountVerifyResendRequest` tendo o seguinte código:

```php
<?php

namespace App\Http\Requests;

use Illuminate\Validation\Rules\Enum;
use App\Enums\UserAccountTypeVerification;
use App\Support\Http\Requests\BaseRequest;

class AuthAccountVerifyResendRequest extends BaseRequest
{
    /**
     * Get the validation rules that apply to the request.
     *
     * @return array<string, mixed>
     */
    public function rules(): array
    {
        return [
            'type' => ['required', new Enum(UserAccountTypeVerification::class)],
        ];
    }

    /**
     * Retrieves account verification type.
     *
     * @return \App\Enums\UserAccountTypeVerification
     */
    public function type(): UserAccountTypeVerification
    {
        return UserAccountTypeVerification::from($this->input('type'));
    }
}
```

Em seguida, adicione o método de `accountVerifyResend` no serviço de `UserService`:

```php
/**
 * Process to resend verification account to customer.
 *
 * @param \App\Models\User $user
 * @param \App\Enums\UserAccountTypeVerification $type
 *
 * @return void
 */
public function accountVerifyResend(User $user, UserAccountTypeVerification $type): void
{
    if ($type === UserAccountTypeVerification::SMS) {
        SendUserVerificationCodeBySMS::dispatch($user);
    } elseif ($type === UserAccountTypeVerification::EMAIL) {
        SendUserVerificationCodeByEmail::dispatch($user);
    }
}
```

Agora é só enviar a request com o seguinte payload:

```json
{
	"type":  "sms or email"
}
```

## Bônus: "Esqueci minha senha" - enviando code via SMS ou e-mail

Para que o usuário posso resetar sua senha (*esqueci minha senha*), podemos enviar o code para seu telefone celular via SMS, ou para seu e-mail, para que o seu registro no banco de dados possa ser recuperado e manipulado corretamente. Pra isso, é necessário primeiro configurar as rotas necessárias para tais ações:

```php
Route::controller(ForgotController::class)->middleware('guest')->group(function () {
    Route::post('forgot-password/send-code', 'sendCode')->name('forgot-password.send-code');
    Route::post('forgot-password/validate-code', 'validateCode')->name('forgot-password.validate-code');
    Route::post('reset-password', 'resetPassword')->name('forgot-password.password-update');
});
```

O controller de `ForgotController` é responsável por tudo que envolve o "recuperar senha", e tem o seguinte código:

```php
<?php

namespace App\Http\Controllers\Auth;

use App\Services\UserService;
use Illuminate\Http\JsonResponse;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Facades\Password;
use Illuminate\Auth\Events\PasswordReset;
use Illuminate\Validation\ValidationException;
use App\Http\Requests\AuthResetPasswordRequest;
use App\Http\Requests\AuthForgotPasswordRequest;
use Illuminate\Routing\Controller as LaravelBaseController;
use App\Http\Requests\AuthForgotPasswordValidateCodeRequest;

class ForgotController extends LaravelBaseController
{
    /**
     * Create a new Controller instance.
     *
     * @param \App\Services\UserService $userService
     */
    public function __construct(private UserService $userService)
    {
    }

    /**
     * Send a reset code to the given user.
     *
     * @param \App\Http\Requests\AuthForgotPasswordRequest $request
     *
     * @return \Illuminate\Http\JsonResponse
     */
    public function sendCode(AuthForgotPasswordRequest $request): JsonResponse
    {
        $user = $this->userService->forgotPasswordSendCode($request->type(), $request->input('value'));
        $token = null;

        $callbackResetLink = (function ($user, $newToken) use (&$token) {
            $token = $newToken;

            // $user->sendPasswordResetNotification($token);
        });

        // We will send the password reset link to this user. Once we have attempted
        // to send the link, we will examine the response then see the message we
        // need to show to the user. Finally, we'll send out a proper response.
        $status = Password::broker()->sendResetLink(['email' => $user->getEmailForPasswordReset()], $callbackResetLink);

        if ($status === Password::RESET_LINK_SENT) {
			return response()->json(['user' => ['email' => $user->getEmailForPasswordReset()], 'token' => $token]]);
		}

        throw ValidationException::withMessages(['email' => __($status)]);
    }

    /**
     * Validates the code previously registered.
     *
     * @param \App\Http\Requests\AuthForgotPasswordValidateCodeRequest $request
     *
     * @return \Illuminate\Http\JsonResponse
     */
    public function validateCode(AuthForgotPasswordValidateCodeRequest $request): JsonResponse
    {
        $this->userService->forgotPasswordValidateCode($request->type(), $request->code, $request->email);
        
        return response()->json(status: JsonResponse::HTTP_NO_CONTENT);
    }

    /**
     * Reset the user's password.
     *
     * @param \App\Http\Requests\AuthResetPasswordRequest $request
     *
     * @return \Illuminate\Http\JsonResponse
     */
    public function resetPassword(AuthResetPasswordRequest $request): JsonResponse
    {
        $token = '';

        // Here we will attempt to reset the user's password. If it is successful we
        // will update the password on an actual user model and persist it to the
        // database. Otherwise we will parse the error and return the response.
        $status = Password::broker()->reset(
            $request->only('email', 'password', 'password_confirmation', 'token'),
            function ($user, $password) use (&$token) {
                $user->forceFill(['password' => Hash::make($password)])->save();

                event(new PasswordReset($user));

                auth()->login($user);
            }
        );

        if ($status === Password::PASSWORD_RESET) {
            return  redirect()->intended('dashboard');
        }

        throw ValidationException::withMessages(['email' => [__($status)]]);
    }
}
```

Adicione o seguinte código a Model de `User`:

```php
/**
 * Get the e-mail address where password reset links are sent.
 *
 * @return string
 */
public function getEmailForPasswordReset(): string
{
    return $this->email;
}
```

Os `FormRequest` de `AuthResetPasswordRequest`, `AuthForgotPasswordRequest` e `AuthForgotPasswordValidateCodeRequest` tem os seguintes códigos:

*AuthResetPasswordRequest.php*
```php
<?php

namespace App\Http\Requests;

use App\Models\User;
use Illuminate\Validation\Rules\Password;
use Illuminate\Foundation\Http\FormRequest;

class AuthResetPasswordRequest extends FormRequest
{
    /**
     * Get the validation rules that apply to the request.
     *
     * @return array<string, mixed>
     */
    public function rules()
    {
        return [
            'token' => 'required',
            'email' => [
                'required',
                'email',
                'exists:' . User::class
            ],
            'password' => [
                'required',
                Password::min(8)->letters()->numbers(),
                'confirmed'
            ],
        ];
    }
}
```

*AuthForgotPasswordRequest.php*
```php
<?php

namespace App\Http\Requests;

use App\Models\User;
use App\Enums\UserStatus;
use Illuminate\Validation\Rule;
use Illuminate\Validation\Rules\Enum;
use App\Enums\UserAccountTypeVerification;
use Illuminate\Foundation\Http\FormRequest;

class AuthForgotPasswordRequest extends FormRequest
{
    private array $valueRules = ['required'];

    /**
     * Prepare the data for validation.
     *
     * @return void
     */
    protected function prepareForValidation(): void
    {
        if (! $this->filled('type')):
            return;
        endif;

        if ($this->type() === UserAccountTypeVerification::SMS) {
            $this->valueRules = array_merge($this->valueRules, [Rule::exists(User::class, 'phone')]);
        } elseif ($this->type() === UserAccountTypeVerification::EMAIL) {
            $this->valueRules = array_merge($this->valueRules, [Rule::exists(User::class, 'email')]);
        }
    }

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array<string, mixed>
     */
    public function rules()
    {
        return [
            'type' => ['required', new Enum(UserAccountTypeVerification::class)],
            'value' => $this->valueRules,
        ];
    }

    /**
     * Retrieves account verification type.
     *
     * @return \App\Enums\UserAccountTypeVerification
     */
    public function type(): UserAccountTypeVerification
    {
        return UserAccountTypeVerification::from($this->input('type'));
    }
}
```

*AuthForgotPasswordValidateCodeRequest.php*
```php
<?php

namespace App\Http\Requests;

use App\Models\User;
use Illuminate\Validation\Rule;
use Illuminate\Validation\Rules\Enum;
use App\Enums\UserAccountTypeVerification;
use Illuminate\Foundation\Http\FormRequest;

class AuthForgotPasswordValidateCodeRequest extends FormRequest
{
    /**
     * Get the validation rules that apply to the request.
     *
     * @return array<string, mixed>
     */
    public function rules()
    {
        return [
            'type' => ['required', new Enum(UserAccountTypeVerification::class)],
            'code' => ['required', 'integer', 'regex:/^\d{6}$/'],
            'email' => ['required', Rule::exists(User::class)],
        ];
    }

    /**
     * Retrieves account verification type.
     *
     * @return \App\Enums\UserAccountTypeVerification
     */
    public function type(): UserAccountTypeVerification
    {
        return UserAccountTypeVerification::from($this->input('type'));
    }
}
```

Adicione o seguinte código ao serviço de `UserService.php`:

```php
/**
 * Send password reset code to user's phone or email.
 *
 * @param \App\Enums\UserAccountTypeVerification $type
 * @param string $value
 *
 * @return \App\Models\User
 */
public function forgotPasswordSendCode(UserAccountTypeVerification $type, string $value): User
{
    if ($type === UserAccountTypeVerification::SMS) {
        /** @var \App\Models\User */
        $user = $this->model->getByPhone($type, $phone);
    } else {
        /** @var \App\Models\User */
        $user = $this->model->getByEmail($type, $email);
    }

    $this->accountVerifyResend($user, $type);

    return $user;
}

/**
 * Checks and validates code confirmation.
 *
 * @param \App\Enums\UserAccountTypeVerification $type
 * @param int $code
 * @param string $email
 *
 * @throws \App\Services\Exceptions\InvalidEmailCodeException
 * @throws \App\Services\Exceptions\InvalidSMSCodeException
 *
 * @return void
 */
public function forgotPasswordValidateCode(UserAccountTypeVerification $type, int $code, string $email): void
{
    /** @var \App\Models\User */
    $user = $this->repository->whereEmail($email)->firstOrFail();

    $this->verifySMSCode($user, $code, $type);
    $this->verifyEmailCode($user, $code, $type);

    $user->fill(['verification_codes' => null])->save();
}
```

Dessa forma, será capaz de recuperar/resetar a senha usando codes de verificação para atualizar o registro do usuário dono do telefone celular ou e-mail, já que esses dois itens são exclusívos/únicos por pessoal.
