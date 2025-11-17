# Backend Architecture — Detailed (.NET) — E-Commerce Learning Project

This document provides a detailed, developer-oriented backend architecture for your learning e-commerce project using **ASP.NET Core** and **EF Core**. It is written to replicate how senior backend engineers design server-side systems in real teams, but tuned for a solo learner.

---

## 1. Goals
- Provide a maintainable, testable backend following Clean Architecture principles.
- Keep controllers thin and push business logic into the Application layer.
- Make EF Core usage explicit and safe (migrations, seeding, transactions).
- Provide clear patterns for authentication, authorization, validation, error handling, and logging.

---

## 2. Project Structure (recommended)
```
server/src/
├── Ecommerce.Api/                   # Presentation: Controllers, Program.cs, Swagger
├── Ecommerce.Application/           # Business logic: DTOs, Services, Interfaces, Validators
├── Ecommerce.Domain/                # Entities, Enums, ValueObjects, Domain Exceptions
└── Ecommerce.Infrastructure/        # EF Core, Repositories, Identity, Migrations

server/tests/
├── Ecommerce.UnitTests/
└── Ecommerce.IntegrationTests/
```

---

## 3. Key Patterns & Decisions
- **Dependency Injection (DI):** Register Application interfaces and Infrastructure implementations in a `DependencyInjection` static class under Infrastructure, called from `Program.cs`.
- **DTOs:** Use request and response DTOs in Application layer to decouple API contracts from domain entities.
- **AutoMapper:** Centralize mapping profiles in Application layer.
- **Validation:** Use FluentValidation for request DTOs.
- **Repository Pattern:** Optional—prefer using EF DbContext directly in Application services for simpler code; create lightweight repositories for abstraction if you want to practice the pattern.
- **Unit of Work:** EF DbContext serves as Unit of Work; ensure SaveChanges is only called by Application service orchestration logic.

---

## 4. Authentication & Identity
- Use **ASP.NET Core Identity** with EF stores (Identity tables in same SQL DB).
- On `POST /api/auth/login` validate user credentials and issue a JWT (signed key in env).
- Seed an Admin user and role during startup for dev/testing.
- Protect endpoints with `[Authorize]` and admin routes with `[Authorize(Roles="Admin")]`.

---

## 5. DbContext & EF Core
- Place `EcommerceDbContext` in Infrastructure/Persistence.
- Configure DbSets: `Products`, `Categories`, `CartItems`, `Orders`, `OrderItems`, etc.
- Use Fluent API configurations (separate `EntityTypeConfiguration` classes in `Configurations/`).
- Use `dotnet ef migrations` pointed at Infrastructure project and `--startup-project` Api when applying migrations.

Example commands:
```
cd server/src
dotnet ef migrations add InitialCreate -p Ecommerce.Infrastructure -s Ecommerce.Api
dotnet ef database update -p Ecommerce.Infrastructure -s Ecommerce.Api
```

---

## 6. Sample Service Layer Design
**ProductService (in Application)**
```csharp
public class ProductService : IProductService
{
    private readonly EcommerceDbContext _db;
    private readonly IMapper _mapper;

    public ProductService(EcommerceDbContext db, IMapper mapper) { ... }

    public async Task<PagedResult<ProductDto>> GetProductsAsync(ProductQuery query) {
        var q = _db.Products.AsNoTracking();
        // apply filters, paging
        var items = await q.ToListAsync();
        return new PagedResult<ProductDto>{ Items = _mapper.Map<List<ProductDto>>(items), ... };
    }
}
```
- Keep transactions in Application services when multiple repos/entities are modified (e.g., Checkout creates Order + OrderItems + clears Cart).

---

## 7. Controllers (Presentation Layer)
- Controllers are thin: validate request -> call Application service -> return DTOs.
- Example `ProductsController` endpoints: `GET /api/products`, `GET /api/products/{id}`, `POST /api/products`, `PUT`, `DELETE`.
- Use `[ApiController]` attribute, model binding, and automatic validation results (FluentValidation integrates with ASP.NET pipeline).

---

## 8. Error Handling & Middleware
- Implement a global exception-handling middleware that logs exceptions and returns a consistent error envelope.
- Map domain exceptions to appropriate HTTP status codes (e.g., `NotFoundException` -> 404).
- Add middleware for request logging (Serilog request logging) and health checks.

Error response example:
```json
{ "status": 400, "errors": [{ "field": "email", "message": "Invalid" }] }
```

---

## 9. Mapping & DTOs
- Centralize AutoMapper profiles in `Ecommerce.Application/Mappings`.
- Keep DTOs lean: when returning product list, avoid navigation properties that serialize entire graphs.
- Use projection `Select` queries where possible to avoid extra mapping overhead.

---

## 10. Validation
- Implement FluentValidation validators for each incoming request DTO (e.g., `CreateProductCommandValidator`).
- Validation failures should return 400 with details.

---

## 11. Transactions & Concurrency
- Use EF Core transactions for checkout flows:
  ```csharp
  using var tx = await _db.Database.BeginTransactionAsync();
  try {
    // create order, order items
    await _db.SaveChangesAsync();
    await tx.CommitAsync();
  } catch {
    await tx.RollbackAsync();
    throw;
  }
  ```
- For inventory concurrency, consider using rowversion/timestamp and handle `DbUpdateConcurrencyException`.

---

## 12. Testing Strategy (detailed)
- **Unit Tests:** Mock DbContext (or use in-memory provider) for simple unit tests, but prefer mocking repositories or services with Moq.
- **Integration Tests:** Use `DotNet.Testcontainers` to spin up real SQL Server container, apply migrations, and run tests against it to learn real DB behaviors.
- **E2E Tests:** Use Playwright calling the running API and UI.

Example: Integration test using Testcontainers:
```csharp
var container = new MsSqlBuilder().WithPassword("yourStrong(!)Password").Build();
await container.StartAsync();
// configure DbContext to use container.ConnectionString
```

---

## 13. Logging & Observability
- Use Serilog for structured logging (console + file).
- Add correlation Id middleware for request tracing.
- Expose `/health` endpoint for basic liveness and readiness checks.

---

## 14. Dependency Registration (sample)
- Infrastructure project provides `public static IServiceCollection AddInfrastructure(this IServiceCollection services, IConfiguration config)` which registers DbContext, Identity, Repositories, Serilog, and other infra services.
- Application project provides `public static IServiceCollection AddApplication(this IServiceCollection services)` which registers AutoMapper, validators, and application service interfaces.
- In `Program.cs` call both helper methods in order.

---

## 15. Configuration & Secrets
- Use `appsettings.json` for defaults and `appsettings.Development.json` for dev overrides.
- Store JWT signing keys and sensitive connection strings in environment variables during CI/CD and production.
- For local development, use user secrets or Docker compose with environment variable overrides.

---

## 16. Sample `Program.cs` (minimal)
```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddApplication();
builder.Services.AddInfrastructure(builder.Configuration);

builder.Services.AddControllers();
builder.Services.AddAuthentication(...JWT...);
builder.Services.AddAuthorization();

var app = builder.Build();
app.UseMiddleware<ExceptionHandlingMiddleware>();
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
app.Run();
```

---

## 17. Dev Commands & Shortcuts
- Create solution & projects:
  ```bash
  dotnet new sln -n Ecommerce
  dotnet new webapi -o Ecommerce.Api
  dotnet new classlib -o Ecommerce.Application
  dotnet new classlib -o Ecommerce.Domain
  dotnet new classlib -o Ecommerce.Infrastructure
  dotnet sln add **/*.csproj
  ```
- Run migrations (see section 5)
- Run API in dev: `dotnet watch --project Ecommerce.Api`
