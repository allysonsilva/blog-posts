---
id: 994bc7f9-2e77-4abe-b6b4-b7631b503571
title: "Autenticando Usuário Para Uso no Telescope"
summary: "Veja a estratégia de autenticação utilizada no uso do Laravel Telescope 🕵️‍♂️"
---

## Configurando provider de `TelescopeServiceProvider.php`

No arquivo de `app/Providers/TelescopeServiceProvider.php` adicione o método de `authorization`:

```php
/**
 * Configure the Telescope authorization services.
 *
 * @return void
 */
protected function authorization()
{
    $this->gate();

    if ($this->app->request->is(config('telescope.path') . '*')) {
        Auth::shouldUse('web');

        if (! app()->isLocal()) {
            Route::middlewareGroup('telescope', ['web', 'auth:web', Authorize::class]);
        }
    }

    Telescope::auth(function ($request) {
        return app()->isLocal() ||
               Gate::authorize('viewTelescope', [$request->user()]);
    });
}
```

O que o código acima faz é definir os *middlewares* `['web', 'auth:web', Authorize::class]` quando o ambiente não é `local` `! app()->isLocal()`, assim, esse `if` define que quando o ambiente for diferente do `local`, então o usuário deve estar autenticado por causa do *middleware* de `auth:web`.

Além do mais, a adição dos seguintes middlewares, só será feito quando a URL da request satisfazer o `if` de `$this->app->request->is(config('telescope.path') . '*')`.

Altera também o método de `gate`, para o seguinte código:

```php
/**
 * Register the Telescope gate.
 *
 * This gate determines who can access Telescope in non-local environments.
 *
 * @return void
 */
protected function gate()
{
    Gate::define('viewTelescope', function ($user) {
        return in_array($user->email, config('telescope.emails_allowed'));
    });
}
```

### Adicionando `emails_allowed` na config de `config/telescope.php`

Na linha de `return in_array($user->email, config('telescope.emails_allowed'));` no código acima, tem uma chave chamada de `emails_allowed` na config de `telescope.php`, assim, adicione a chave nessa mesma config:

```php
'emails_allowed'  =>  array_filter(explode(',', env('TELESCOPE_EMAILS_ALLOWED', ''))),
```

E no arquivo `.env` adicione `TELESCOPE_EMAILS_ALLOWED` com os e-mails que serão permitidos acessarem o dashboard do Telescope, separando-os por vírgula.

## Rota para autenticar usuário

No arquivo de rotas `web.php` adicione a seguinte rota:

```php
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;
use Illuminate\Support\Facades\Redirect;

Route::get('/signed-login/{user}', function (Request $request, User $user) {
    // Set one week session lifetime (In minutes)
    config(['session.lifetime' => 1 * (60 * 24 * 7)]);

    auth()->login($user);

    return Redirect::route('logged.me.profile');
})->middleware('signed')->name('signed-login');

Route::middleware('auth')->name('logged.')->group(function () {
    Route::name('me.')->group(function () {
        Route::get('me/profile', fn() => auth()->user())->name('profile');
    });
});
```

Esse rota vai ser utilizada para autenticar determinado usuário por meio do parâmetro `{user}`.

Essa rota é [assinada](https://laravel.com/docs/10.x/urls#signed-urls), ou seja, para acessá-la é necessário ter um `hash` de assinatura.

## Comando para gerar link/URL signed

Para simplificar a geração da URL da rota assinada, crie o seguinte arquivo `app/Console/Commands/MakeUserAuthLink.php`:

```php
<?php

namespace App\Console\Commands;

use App\Models\User;
use Illuminate\Console\Command;
use Illuminate\Support\Facades\URL;
use Illuminate\Contracts\Console\Isolatable;

class MakeUserAuthLink extends Command implements Isolatable
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'app:auth-link {user=1}';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Generate an authentication link for the user.';

    /**
     * Execute the console command.
     *
     * @return int
     */
    public function handle()
    {
        $user = User::findOrFail($this->argument('user'));

        $url = URL::temporarySignedRoute(
            'signed-login', now()->addMinutes(2), ['user' => $user->getKey()]
        );

        $this->info('Access the URL to login: '. $url);

        return Command::SUCCESS;
    }
}
```

Agora para gerar a URL assinada, execute `php artisan app:auth-link <id-do-usuario>`, e o comando terá como resultado o link para acessar a rota acima, para em seguida autenticar o usuário, e ai poderá acessar o Telescope com um usuário autenticado.

## Resumo

Acessar o Telescope com um usuário autenticado é uma forma de garantir a segurança em ambientes de dev/staging, garantindo que apenas determinados usuários com determinados e-mails (`TELESCOPE_EMAILS_ALLOWED`) possam fazer/ver o dashboard do Telescope.
