# LARAVEL / PHP STACK REFERANSI

## Tespit
- `composer.json` + `laravel/framework` dependency -> bu stack aktif

---

## Teknoloji Tablosu

| Katman | Arac |
|--------|------|
| Framework | Laravel 11+ |
| ORM | Eloquent |
| Validation | Form Request |
| Auth | Laravel Sanctum (SPA/Mobile) veya Passport (OAuth2) |
| Password | Hash facade (argon2id onerilen, bcrypt kabul edilir) |
| Logging | Laravel Log (Monolog) |
| Cache | predis / phpredis |
| Test | PHPUnit veya Pest |
| API Docs | Scramble (dedoc/scramble, onerilen — otomatik) veya L5-Swagger |
| Rate Limit | Laravel built-in (ThrottleRequests middleware) |
| Queue | Laravel Queue (Redis/SQS/Database) |

---

## Middleware Sirasi

`bootstrap/app.php` (Laravel 11+ — Kernel.php kaldirildi):

```
api middleware group:
1. throttle:api          // Rate limiting
2. SubstituteBindings    // Route model binding
3. (custom auth middleware)
4. (custom logging middleware)
```

Global middleware:
- TrustProxies
- HandleCors
- PreventRequestsDuringMaintenance
- ValidatePostSize
- TrimStrings

---

## Proje Yapisi

```
app/
├── Http/
│   ├── Controllers/     # API Controller'lar
│   ├── Requests/        # Form Request (validation)
│   ├── Resources/       # API Resource (DTO/transformer)
│   └── Middleware/       # Custom middleware
├── Models/              # Eloquent model'ler
├── Services/            # Is mantigi (Controller'da olmamali)
├── Repositories/        # Veri erisimi (opsiyonel, Eloquent direkt Service'te de olabilir)
├── Policies/            # Authorization policy'leri
├── Exceptions/          # Custom exception'lar
├── Observers/           # Model observer'lar
└── Providers/           # Service Provider'lar (DI kayitlari)

database/
├── migrations/          # DB migration'lar
├── seeders/             # Seed data
└── factories/           # Test factory'ler

routes/
└── api.php              # API route tanimlari

tests/
├── Unit/                # Unit testler
└── Feature/             # Integration/Feature testler
```

---

## Eloquent Kaliplari

### Model
```php
class User extends Model
{
    use HasFactory, SoftDeletes;

    protected $fillable = ['name', 'email', 'password'];
    protected $hidden = ['password', 'remember_token'];
    // Laravel 11+: method-based casts (property-based deprecated)
    protected function casts(): array
    {
        return ['email_verified_at' => 'datetime'];
    }

    // Iliski
    public function orders(): HasMany
    {
        return $this->hasMany(Order::class);
    }
}
```

### Zorunlu Kurallar
- `$fillable` ZORUNLU (mass assignment korunmasi)
- `$hidden` ile hassas alanlar response'dan gizle
- `$casts` ile tip donusumleri acikca belirt
- `SoftDeletes` trait -> soft delete gerekiyorsa
- Scope kullan: `scopeActive($query)` -> `User::active()->get()`
- `select()` -> sadece gerekli alanlar
- `with()` -> N+1 onleme (eager loading)

### Migration
```bash
php artisan make:migration create_users_table
php artisan migrate
php artisan migrate:rollback
```

`down()` metodu ASLA bos birakilmaz.

---

## Form Request Validation

```php
class CreateUserRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true; // veya policy kontrolu
    }

    public function rules(): array
    {
        return [
            'email' => ['required', 'email', 'max:255', 'unique:users'],
            'password' => [
                'required', 'min:8',
                'regex:/[A-Z]/',  // buyuk harf
                'regex:/[a-z]/',  // kucuk harf
                'regex:/[0-9]/',  // rakam
                'regex:/[^a-zA-Z0-9]/',  // ozel karakter
            ],
        ];
    }

    public function messages(): array
    {
        return [
            'email.required' => 'Email alani zorunludur',
            'password.min' => 'Sifre en az 8 karakter olmali',
        ];
    }
}
```

---

## API Resource (DTO)

```php
class UserResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'createdAt' => $this->created_at->toISOString(),
        ];
        // password, remember_token ASLA donmez
    }
}

// Kullanim
return UserResource::collection($users);
return new UserResource($user);
```

---

## Logging

```php
// config/logging.php -> stack channel
Log::info('User created', ['userId' => $user->id]);
Log::error('Payment failed', ['orderId' => $order->id, 'error' => $e->getMessage()]);
```

Correlation ID middleware:
```php
public function handle(Request $request, Closure $next)
{
    $correlationId = $request->header('X-Correlation-Id', Str::uuid()->toString());
    Log::withContext(['correlationId' => $correlationId]);
    $response = $next($request);
    return $response->header('X-Correlation-Id', $correlationId);
}
```

**YASAK:** `Log::info('Login', ['password' => $password])` -> hassas veri loglanmaz

---

## Authentication (Sanctum)

```php
// API token olusturma
$token = $user->createToken('api-token')->plainTextToken;

// Route korumasi
Route::middleware('auth:sanctum')->group(function () {
    Route::get('/me', [AuthController::class, 'me']);
});

// Token iptal
$request->user()->currentAccessToken()->delete(); // logout
$request->user()->tokens()->delete(); // tum session'lar
```

Password hashing (bkz. `guvenlik/CLAUDE.md` — argon2id onerilen):
```php
// config/hashing.php -> driver'i degistir:
// 'driver' => 'argon2id',  // onerilen (OWASP 2024)
// 'driver' => 'bcrypt',    // kabul edilir (varsayilan)

// argon2id (onerilen):
$hash = Hash::make($password); // config'den argon2id kullanir
// veya acikca:
$hash = Hash::make($password, [
    'memory' => 19456,  // KiB (OWASP minimum)
    'time' => 2,        // iterations
    'threads' => 1,     // parallelism
]);
$isValid = Hash::check($password, $hash);

// bcrypt (kabul edilir):
// config/hashing.php -> 'bcrypt' => ['rounds' => 12]
$hash = Hash::make($password); // bcrypt, 12 rounds
```

### JWT Kullanilacaksa
- Laravel Passport (OAuth2 grant'leri ile JWT)
- veya `php-open-source-saver/jwt-auth` (maintained fork)
- **tymon/jwt-auth KULLANMA** — bakimsiz, PHP 8.2+ uyumluluk sorunlari, guvenlik aciklari

```php
// Passport veya jwt-auth config
// Access token: 1 saat, Refresh token: 14 gun
```

---

## Authorization (Policy)

```php
class OrderPolicy
{
    public function view(User $user, Order $order): bool
    {
        return $user->id === $order->user_id;
    }

    public function delete(User $user, Order $order): bool
    {
        return $user->hasRole('admin');
    }
}

// Controller'da
$this->authorize('view', $order);
// veya
Gate::authorize('delete', $order);
```

---

## Error Handling

```php
// app/Exceptions/Handler.php veya bootstrap/app.php (Laravel 11+)
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->render(function (NotFoundHttpException $e, Request $request) {
        if ($request->is('api/*')) {
            return response()->json([
                'success' => false,
                'error' => ['code' => 'NOT_FOUND', 'message' => 'Kaynak bulunamadi'],
            ], 404);
        }
    });
})
```

---

## Test

### Unit Test (Pest)
```php
test('user service throws not found when user missing', function () {
    $repo = Mockery::mock(UserRepository::class);
    $repo->shouldReceive('findById')->andReturnNull();
    $service = new UserService($repo);

    expect(fn() => $service->getById('fake-id'))->toThrow(ModelNotFoundException::class);
});
```

### Feature Test (HTTP)
```php
test('GET /api/v1/users returns 200', function () {
    $response = $this->getJson('/api/v1/users');

    $response->assertStatus(200)
        ->assertJsonStructure(['success', 'data']);
});
```

### Factory & Seeder
```php
User::factory()->count(10)->create(); // Test icin fake data
```

---

## Rate Limiting

```php
// bootstrap/app.php veya RouteServiceProvider
RateLimiter::for('api', function (Request $request) {
    return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
});

RateLimiter::for('login', function (Request $request) {
    return Limit::perMinute(5)->by($request->ip());
});
```

---

## Service Provider (DI)

```php
class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->bind(UserRepositoryInterface::class, UserRepository::class);
        $this->app->singleton(PaymentGateway::class, StripePaymentGateway::class);
    }
}
```

---

## Artisan Komutlari

```bash
php artisan make:model User -mfsc   # Model + Migration + Seeder + Controller
php artisan make:request CreateUserRequest
php artisan make:resource UserResource
php artisan make:policy UserPolicy --model=User
php artisan make:test UserTest --unit
php artisan serve                     # Dev server
php artisan route:list               # Route listesi
php artisan optimize                 # Production cache
```

---

## Docker

```dockerfile
FROM php:8.3-fpm-alpine AS base
RUN docker-php-ext-install pdo pdo_pgsql

FROM base AS build
WORKDIR /app
COPY composer.* ./
RUN composer install --no-dev --optimize-autoloader
COPY . .

FROM base
WORKDIR /app
COPY --from=build /app .
USER www-data
CMD ["php-fpm"]
```

Nginx reverse proxy ornegi:
```nginx
server {
    listen 80;
    server_name api.example.com;
    root /app/public;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass app:9000;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

---

## Laravel Octane (Performans)

Octane, uygulamayi bellekte tutarak her istekte framework'u yeniden baslatmaz.
10x'e kadar performans artisi saglar.

```bash
composer require laravel/octane
php artisan octane:install  # Swoole, RoadRunner veya FrankenPHP
php artisan octane:start
```

**Dikkat:** Singleton'lar request'ler arasi paylasiliyor — state leak'e dikkat et.
`$this->app->scoped()` kullan, static state'den kacin.

---

## Health Check

```php
// routes/api.php
Route::get('/health/live', fn() => response()->json(['status' => 'ok']));

Route::get('/health/ready', function () {
    try {
        DB::select('SELECT 1');
        // Redis::ping(); // varsa
        return response()->json(['status' => 'ready']);
    } catch (\Throwable $e) {
        return response()->json(['status' => 'not ready'], 503);
    }
});
```

---

## OpenTelemetry

```bash
composer require open-telemetry/sdk open-telemetry/exporter-otlp
composer require open-telemetry/opentelemetry-auto-laravel
```

```php
// .env
OTEL_PHP_AUTOLOAD_ENABLED=true
OTEL_SERVICE_NAME=my-api
OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4318
OTEL_TRACES_EXPORTER=otlp
```

`opentelemetry-auto-laravel` paketi otomatik instrumentation saglar (HTTP, DB, Redis).
