# 🔑 Mastering API Token Generation in Laravel 11 with Sanctum

![Laravel Sanctum Token Generation](https://laravel.com/img/logomark.min.svg)

## 🚀 The Ultimate Artisan Command for Secure API Authentication

In modern Laravel applications, API authentication is often handled
using [Laravel Sanctum](https://laravel.com/docs/11.x/sanctum). While Sanctum provides
excellent functionality out of the box, creating a robust token generation command can **streamline your development
workflow** and **enhance security**. Let's build a powerful Artisan command that handles token generation with
expiration dates and user management. i will not talk about the Laravel Sanctum installation and setup, you can find
that in the [Laravel Sanctum documentation](https://laravel.com/docs/11.x/sanctum).

---

## 💡 Why Build a Custom Token Command?

| Benefit                      | Description                                            |
|------------------------------|--------------------------------------------------------|
| ⏱️ **Time-Saving**           | Generate tokens faster than manual methods             |
| 🔒 **Enhanced Security**     | Built-in expiration handling and token rotation        |
| 🤖 **Automation Ready**      | Integrates with deployment scripts and CI/CD pipelines |
| 📊 **Consistent Process**    | Standardized token generation across your team         |
| 🛠️ **Developer Experience** | Beautiful CLI interface with interactive prompts       |

---

## 🛠️ Implementation Deep Dive

### 1. The Token Generation Action (`app/Actions/GenerateTokenAction.php`)

This action handles the core logic of generating a token for a user. It checks if the user exists, creates a new one if
not, and generates a token with an optional expiration date.

```php
<?php

declare(strict_types=1);

namespace App\Actions;

use Carbon\Carbon;
use App\Models\User;
use Illuminate\Support\Str;
use Illuminate\Support\Facades\Hash;

final class GenerateTokenAction
{
    public function execute(string $username, ?string $expires = null): array
    {
        $user = User::query()
            ->firstOrCreate(
                ['email' => $username],
                [
                    'name' => $username,
                    'password' => Hash::make(Str::random(16)),
                ]
            );

        $user->tokens()->delete();

        $expiresAt = $expires !== null && $expires !== '' && $expires !== '0' ? Carbon::parse($expires) : null;
        $token = $user->createToken(
            name: $username,
            expiresAt: $expiresAt,
        );

        return [
            'user' => $user->name,
            'token' => $token->plainTextToken,
            'expires_at' => $expiresAt,
            'created_at' => $user->created_at,
        ];
    }
}

```

#### Key Features:

- 🔄 Automatic token rotation (deletes old tokens)
- 🔐 Secure password generation for new users
- ⏳ Flexible expiration handling
- 📦 Clean, standardized return format

---

### 2. The Artisan Command (`app/Console/Commands/GenerateApiToken.php`)

Create a new Artisan command using the following command:

```php
php artisan make:command GenerateApiToken
```

After that, replace the contents of the generated file with the following code:

```php
<?php

declare(strict_types=1);

namespace App\Console\Commands;

use App\Actions\GenerateTokenAction;
use Illuminate\Console\Command;
use Illuminate\Contracts\Console\PromptsForMissingInput;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
use function Laravel\Prompts\{clear, confirm, info, progress, text};

final class GenerateToken extends Command implements PromptsForMissingInput
{
    protected $signature = 'generate:token
        {username : User email or username}
        {expires? : Expiration date (e.g. "1 day", "2025-12-31")}';

    protected $description = 'Generate or update user token with expiration';

    public function handle(GenerateTokenAction $action): void
    {
        $expires = $this->argument('expires') === 'yes'
            ? text('Enter expiration (e.g., "1 day", "2025-12-31")')
            : $this->argument('expires');

        $results = progress(
            label: 'Generating token',
            steps: 1,
            callback: function () use ($action, $expires): array {
                sleep(1); // Simulate a delay for the progress bar note: this is just for demonstration

                return $action->execute(
                    username: $this->argument('username'),
                    expires: $expires,
                );
            },
        );

        // Extract the first (and only) result from the progress array
        $result = $results[0];
        $this->line('');
        info('✅ API Token Generated Successfully');
        $this->table(
            ['🆔 User', '🔑 Token', '⏳ Expires At'],
            [
                [
                    $result['user'],
                    $result['token'],
                    $result['expires_at']->toDateTimeString(),
                ],
            ]
        );
    }

    protected function promptForMissingArgumentsUsing(): array
    {
        return [
            'username' => 'Enter the username or email address',
        ];
    }

    protected function afterPromptingForMissingArguments(InputInterface $input, OutputInterface $output): void
    {
        if (confirm('Do you want to set an expiration date for the token?')) {
            $input->setArgument(
                'expires',
                text('Enter expiration (e.g., "1 day", "2025-12-31")')
            );
        }
    }
}

```

#### UX Enhancements:

- 🎨 Color-coded output
- 🔄 Progress indicators
- ❓ Interactive prompts
- 📝 Clear, organized information display

---

### 🏗️ Usage Examples

#### 🔧 Basic Interactive Mode

```bash
php artisan generate:token
```

Prompts for:

- Username/Email address
- Whether to set expiration
- Expiration timeframe if selected

#### ⚡ Direct Command Usage

```bash
# Token without expiration
php artisan generate:token info@laranepal.com

# Token with 30-day expiration
php artisan generate:token info@laranepal.com "30 days"

```

🤖 CI/CD Integration

```bash
# Create deployment token (expires in 1 hour)
php artisan generate:token info@laranepal.com "1 hour" >> deploy_token.txt
```

#### Output

```php

 ✅ API Token Generated Successfully

+---------+-----------------------------------------------------+---------------------+
| 🆔 User | 🔑 Token                                            | ⏳ Expires At       |
+---------+-----------------------------------------------------+---------------------+
| bb      | 50|q2uPCYho0wHnZ2AvPtBia8IB8hp9hRWpfqUD8crA20370e93 | 2025-05-15 07:49:51 |
+---------+-----------------------------------------------------+---------------------+

```

---

### 🚀 Advanced Extensions

you can enhance more features such as given in flow chart.

![flow chart](flow.svg)

---

### 🎯 Conclusion

This implementation provides:

- **Enterprise-grade security** 🔐 in a developer-friendly package
- **Beautiful CLI UX** 🎨 that teams will love using
- **Production-ready features** 🏗️ out of the box
- **Extensible architecture** ⚙️ for future needs

---


