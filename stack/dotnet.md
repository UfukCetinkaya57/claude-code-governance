# .NET / ASP.NET CORE STACK REFERANSI

## Tespit
- `*.csproj` veya `*.sln` dosyasi mevcut -> bu stack aktif

---

## Teknoloji Tablosu

| Katman | Arac |
|--------|------|
| Framework | ASP.NET Core Web API (.NET 8+) |
| ORM | Entity Framework Core |
| Validation | FluentValidation |
| Auth | Microsoft.AspNetCore.Authentication.JwtBearer |
| Password | Isopoh.Cryptography.Argon2 (argon2id, onerilen) veya BCrypt.Net-Next (kabul edilir) |
| Logging | Serilog |
| Cache | IDistributedCache + StackExchange.Redis |
| Test | xUnit + WebApplicationFactory + FluentAssertions |
| Mock | Moq veya NSubstitute |
| Test Data | Bogus / AutoFixture |
| API Docs | .NET 9+: Microsoft.AspNetCore.OpenApi + Scalar. .NET 8: Swashbuckle (deprecated in .NET 9) |
| Mapping | Mapster veya Mapperly (source generator). AutoMapper onerilmez (debug zorlugu, silent failure) |
| Health Check | Microsoft.Extensions.Diagnostics.HealthChecks |

---

## Middleware Pipeline Sirasi

```csharp
app.UseExceptionHandler();       // 1. Global exception handler
app.UseSerilogRequestLogging();  // 2. Request/Response logging
app.UseCors();                   // 3. CORS
app.UseRateLimiter();            // 4. Rate limiting (.NET 7+)
app.UseAuthentication();         // 5. Authentication
app.UseAuthorization();          // 6. Authorization
app.MapControllers();            // 7. Endpoint routing
```

Sira ONEMLIDIR. Degistirirsen davranis degisir.

---

## Dependency Injection Lifetime

| Lifetime | Ne Zaman | Ornek |
|----------|----------|-------|
| Scoped | Request-bazli | DbContext, UnitOfWork, Service'ler |
| Transient | Her cagrildiginda yeni | Stateless utility, validator |
| Singleton | Uygulama omru boyunca | IMemoryCache, HttpClientFactory, config |

**KURALLAR:**
- DbContext ASLA Singleton olmaz
- Singleton servis Scoped servisi inject EDEMEZ
- Extension metot ile grupla: `services.AddAuthServices()`, `services.AddProductServices()`

---

## API Pattern'leri

### Controller-Based (Varsayilan, karmasik endpoint'ler icin)
```csharp
[ApiController]
[Route("api/v1/[controller]")]
[Produces("application/json")]
public class UsersController : ControllerBase
{
    [HttpGet]
    [ProducesResponseType(typeof(ApiResponse<List<UserDto>>), 200)]
    public async Task<ActionResult<ApiResponse<List<UserDto>>>> GetAll(
        [FromQuery] PaginationRequest request,
        CancellationToken cancellationToken)
    { }
}
```

### Minimal API (.NET 8+, basit CRUD endpoint'ler icin)
```csharp
var group = app.MapGroup("api/v1/users").RequireAuthorization();

group.MapGet("/", async ([AsParameters] PaginationRequest req, IUserService service, CancellationToken ct) =>
    Results.Ok(await service.GetAllAsync(req, ct)));

group.MapGet("/{id:guid}", async (Guid id, IUserService service, CancellationToken ct) =>
    await service.GetByIdAsync(id, ct) is { } user
        ? Results.Ok(user) : Results.NotFound());
```

**Secim:** Minimal API basit endpoint'ler icin, Controller karmasik logic + attribute'lar gerektiginde.

**KURALLAR (her iki yaklasimda):**
- CancellationToken her async metotta
- [FromBody], [FromQuery], [FromRoute] acikca belirt
- Response tipleri dokumante edilmis

---

## EF Core Kaliplari

### Entity Configuration (ayri dosyada)
```csharp
public class UserConfiguration : IEntityTypeConfiguration<User>
{
    public void Configure(EntityTypeBuilder<User> builder)
    {
        builder.ToTable("users");
        builder.HasKey(x => x.Id);
        builder.Property(x => x.Email).IsRequired().HasMaxLength(255);
        builder.HasIndex(x => x.Email).IsUnique();
        builder.HasQueryFilter(x => !x.IsDeleted); // Soft delete
    }
}
```

### Zorunlu Kurallar
- `AsNoTracking()` -> read-only sorgularda ZORUNLU
- `Include()` / `ThenInclude()` -> N+1 onlemek icin
- `AsSplitQuery()` -> buyuk join'lerde degerlendir
- `Select()` -> projection, sadece gerekli alanlar
- Raw SQL -> sadece performans zorunlulugunda, yorumla
- Migration `Down()` metodu -> ASLA bos birakilmaz

### Migration
```bash
dotnet ef migrations add {IsimAciklayici}
dotnet ef database update
```

---

## FluentValidation

```csharp
public class CreateUserValidator : AbstractValidator<CreateUserRequest>
{
    public CreateUserValidator()
    {
        RuleFor(x => x.Email).NotEmpty().EmailAddress().MaximumLength(255);
        RuleFor(x => x.Password).NotEmpty().MinimumLength(8)
            .Matches("[A-Z]").WithMessage("En az bir buyuk harf")
            .Matches("[a-z]").WithMessage("En az bir kucuk harf")
            .Matches("[0-9]").WithMessage("En az bir rakam")
            .Matches("[^a-zA-Z0-9]").WithMessage("En az bir ozel karakter");
    }
}
```

Pipeline behaviour ile otomatik validation (MediatR kullaniliyorsa).
Yoksa controller'da `ModelState` veya filter ile.

---

## Serilog

```csharp
Log.Logger = new LoggerConfiguration()
    .Enrich.FromLogContext()
    .Enrich.WithCorrelationId()
    .WriteTo.Console()  // Development
    .WriteTo.Seq(url)   // Production (veya Elasticsearch, AppInsights)
    .CreateLogger();
```

Hassas veri maskeleme:
```csharp
.Destructure.ByTransforming<LoginRequest>(r => new { r.Email, Password = "***" })
```

---

## Authentication (JWT)

```csharp
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            // RS256 icin: IssuerSigningKey = new RsaSecurityKey(rsaKey)
        };
    });
```

Password hashing (bkz. `guvenlik/CLAUDE.md` — argon2id onerilen):
```csharp
// Onerilen: Isopoh.Cryptography.Argon2 (aktif bakimli, NuGet)
// NOT: Konscious.Security.Cryptography 2019'dan beri guncellenmemis — KULLANMA
using Isopoh.Cryptography.Argon2;

var config = new Argon2Config
{
    Type = Argon2Type.DataIndependentAddressing, // argon2id
    Password = Encoding.UTF8.GetBytes(password),
    Salt = RandomNumberGenerator.GetBytes(16),
    MemoryCost = 19456, // KiB (OWASP minimum)
    TimeCost = 2,       // iterations
    Lanes = 1           // parallelism
};
var hash = Argon2.Hash(config);
var isValid = Argon2.Verify(hash, password);

// Kabul edilir: BCrypt.Net-Next
var bcryptHash = BCrypt.Net.BCrypt.HashPassword(password, workFactor: 12);
var verified = BCrypt.Net.BCrypt.Verify(password, bcryptHash);
```

---

## Test

### Unit Test (xUnit + Moq)
```csharp
public class UserServiceTests
{
    [Fact]
    public async Task GetById_Should_ReturnNotFound_When_UserDoesNotExist()
    {
        // Arrange
        var repo = new Mock<IUserRepository>();
        repo.Setup(r => r.GetByIdAsync(It.IsAny<Guid>())).ReturnsAsync((User?)null);
        var service = new UserService(repo.Object);

        // Act & Assert
        await Assert.ThrowsAsync<NotFoundException>(() => service.GetByIdAsync(Guid.NewGuid()));
    }
}
```

### Integration Test (WebApplicationFactory)
```csharp
public class UsersApiTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;

    public UsersApiTests(WebApplicationFactory<Program> factory)
    {
        _client = factory.CreateClient();
    }

    [Fact]
    public async Task GetAll_Should_Return200_With_UserList()
    {
        var response = await _client.GetAsync("/api/v1/users");
        response.StatusCode.Should().Be(HttpStatusCode.OK);
    }
}
```

### Test DB
- **Testcontainers** (onerilen — gercek DB container ile test, en guvenilir sonuc)
- SQLite in-memory (hizli, cogu constraint destegi var)
- Respawn (DB test isolation)
- EF Core InMemory provider ONERILMEZ (iliskisel kisitlamalari uygulamaz, Microsoft da karsi)

---

## Health Check

```csharp
services.AddHealthChecks()
    .AddDbContextCheck<AppDbContext>()
    .AddRedis(redisConnectionString)
    .AddCheck("custom", () => HealthCheckResult.Healthy());

app.MapHealthChecks("/health/live", new() { Predicate = _ => false });
app.MapHealthChecks("/health/ready");
```

---

## OpenTelemetry

```csharp
// Program.cs
builder.Services.AddOpenTelemetry()
    .WithTracing(tracing => tracing
        .AddAspNetCoreInstrumentation()
        .AddHttpClientInstrumentation()
        .AddEntityFrameworkCoreInstrumentation()
        .AddOtlpExporter());

// appsettings.json
// "OTEL_EXPORTER_OTLP_ENDPOINT": "http://localhost:4317"
// "OTEL_SERVICE_NAME": "my-api"
```

Paketler: `OpenTelemetry.Extensions.Hosting`, `OpenTelemetry.Instrumentation.AspNetCore`, `OpenTelemetry.Exporter.OpenTelemetryProtocol`

---

## Docker

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY . .
RUN dotnet publish -c Release -o /app

FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app .
USER app
ENTRYPOINT ["dotnet", "MyApp.dll"]
```
