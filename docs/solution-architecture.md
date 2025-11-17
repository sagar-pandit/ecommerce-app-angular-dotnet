# Solution Architecture Document (SAD) — E-Commerce Learning Project

**Version:** 1.0

**Prepared for:** Learner (Maya)

**Purpose:**
This Solution Architecture Document captures the full, practical architecture for the learning-focused e-commerce application (Angular + ASP.NET Core + EF Core + SQL Server). It is written as if for a small engineering team, but tuned for a single developer learning journey.

---

## 1. Executive Summary
Build a modular, maintainable e-commerce web application that supports product browsing, cart management, checkout, and order history. The solution will be implemented as an Angular SPA frontend and an ASP.NET Core Web API backend with EF Core and SQL Server for persistence. The architecture emphasizes separation of concerns, testability, and typical production practices while keeping implementation complexity manageable for learning.

---

## 2. Goals & Non-Functional Requirements (NFRs)
### Functional Goals (recap)
- Product listing, search, filters
- Product details
- User auth (register/login)
- Cart and checkout (mock payment)
- Order history
- Admin CRUD for products (post-MVP)

### Non-Functional Requirements
- **Security:** JWT auth, role-based access control for admin APIs, input validation, HTTPS.
- **Maintainability:** Clean architecture with API, Application, Domain, Infrastructure layers.
- **Testability:** Unit & integration tests; sample E2E tests with Playwright
- **Performance:** Pagination, small payloads, caching strategies possible later.
- **Developer Experience:** Docker Compose for local dev, `dotnet watch`, `ng serve`.
- **Scalability (learning-aware):** Stateless API, horizontal-scale ready; DB scaling considered but not implemented.

---

## 3. Architectural Overview
### 3.1 Logical Components
- **Angular SPA (Frontend)** — UI, client-side state, route guards, HTTP interceptors.
- **ASP.NET Core Web API (Backend)** — Controllers (thin), Application services (use-cases), Domain entities, Infrastructure (EF Core, Identity).
- **SQL Server (Persistence)** — Relational store for all domain data.
- **(Optional) Blob Storage** — For product images in production (Azure Blob / AWS S3).
- **(Optional) Cache** — Redis for read caching later.

### 3.2 Deployment Targets (Learning path)
- **Local Dev:** Docker Compose (SQL Server container), local APIs via `dotnet run`, Angular via `ng serve`.
- **Optional Cloud Deploy:** Backend to Azure App Service or Container Apps; Frontend to Azure Static Web Apps or S3 + CloudFront.

---

## 4. Detailed Component Responsibilities
### Frontend (Angular)
- Routing and navigation
- Authentication token management (HttpInterceptor)
- Presentation components: ProductList, ProductDetails, Cart, Checkout, Account, Admin
- Services: ApiService, AuthService, CartService, ProductService, OrderService
- State: simple service-based state; optional NgRx later

### Backend (ASP.NET Core)
- **API Layer (Presentation):** Controllers, Swagger, request/response mapping
- **Application Layer:** Use-cases, DTOs, validation, interfaces for persistence and external services
- **Domain Layer:** Entities, domain rules
- **Infrastructure Layer:** EF DbContext, repository implementations (optional), Identity configuration, email/files stubs, Serilog

### Persistence
- Relational model (see ER Diagram): Users (Identity), Products, Categories, CartItems, Orders, OrderItems, Addresses, Payments
- EF Core Code-First with Migrations

---

## 5. Data Flow and Integration Points
### Key Data Flows
1. **Search / Browsing:** UI -> GET /api/products -> ProductService -> DbContext -> return DTOs
2. **Add to Cart:** UI -> POST /api/cart -> CartService -> DbContext -> upsert CartItem
3. **Checkout:** UI -> POST /api/orders -> OrderService transactionally reads CartItems, creates Order + OrderItems, clears cart, calls PaymentGateway mock
4. **Auth:** UI -> POST /api/auth/login -> IdentityManager -> token issued

### External Integration Points (mocked for learning)
- Payment gateway (mock or sandbox)
- Email service (local pickup file or SMTP dev server)

---

## 6. Security Considerations
- **Authentication:** ASP.NET Identity + JWT issuance. JWT used by Angular with interceptor.
- **Authorization:** Role-based policy for Admin endpoints.
- **Input Validation:** FluentValidation on DTOs.
- **Secure Secrets:** Use environment variables or GitHub Secrets; never commit secrets.
- **HTTPS:** Enforce HTTPS (local dev with dev certificate; cloud enforce TLS).
- **Data Protection:** Use built-in ASP.NET data protection keys or key rotation for tokens if later adding refresh tokens.

---

## 7. Error Handling, Observability & Logging
- **Error Handling:** Global exception middleware to return structured errors.
- **Logging:** Serilog to console/file in dev; structured logs for searching.
- **Health Checks:** Basic health endpoints `/health` exposing app and DB connectivity.
- **Metrics (optional):** Prometheus-friendly metrics later if needed.

---

## 8. Testing Strategy
- **Unit Tests:** xUnit + Moq for Application layer and utilities
- **Integration Tests:** Testcontainers for SQL Server or dedicated test DB; test EF behavior with migrations and repository calls
- **E2E Tests:** Playwright for UI flows (Login, Add to Cart, Checkout)

---

## 9. CI/CD and Release Strategy
### CI (GitHub Actions)
- Run on PRs: `dotnet test`, `npm ci && npm test` (frontend), `dotnet build`, `ng build --prod` (optional)
- Linting: `dotnet format`, `eslint` for Angular

### CD (manual initially)
- Build Docker image for backend and push to registry (GitHub Packages or DockerHub)
- Deploy static frontend to Azure Static Web Apps or S3

### Release Strategy
- Keep `main` as deployable; `develop` for integrating feature branches
- Use PR reviews for major changes

---

## 10. Migration, Backup & Rollback (learning-friendly)
- Use EF Core migrations for schema changes; apply migrations during deployment with caution
- Backups: For local learning, not required. For cloud: use provider snapshot/backups
- Rollback: Keep migrations reversible; if DB breaking change, use new migration that transforms data safely

---

## 11. Scalability & Performance Notes
- **API:** Stateless; scale horizontally behind a load balancer
- **DB:** Start with single instance; for scale use read replicas and caching (Redis)
- **Images:** Offload images to CDN/Blob storage
- **Optimization:** Add indexes on search/filter columns; use server-side pagination and projection DTOs

---

## 12. Cost & Environment Considerations (for cloud later)
- Use free tiers for learning (Azure free credits, Azure Static Web Apps free tier)
- Use small DB tiers for testing; consider serverless or managed containers

---

## 13. Risks and Mitigations
- **Risk:** Complexity of Identity + JWT for first-timers. **Mitigation:** Start with simple JWT token issuance and extend to Identity later.
- **Risk:** EF Core migration hiccups. **Mitigation:** Maintain small migrations and seed scripts; test migration locally using Docker SQL Server.
- **Risk:** Overengineering. **Mitigation:** Keep MVP simple; add features iteratively.

---

## 14. Operational Runbook (Quick Dev Commands)
- Run SQL Server via Docker Compose: `docker compose up -d sqlserver`
- Apply migrations: `dotnet ef database update --project Infrastructure --startup-project Api`
- Run backend: `dotnet watch --project Api`
- Run frontend: `ng serve --project frontend`
- Run unit tests: `dotnet test` and `npm test`

---

## 15. Acceptance Criteria (for learning completion)
- Can browse products and view details in UI
- Can register/login and maintain session via JWT
- Can add items to cart and place an order
- Orders persisted in DB and visible in order history
- Basic admin CRUD for products implemented
- Project builds and runs locally with Docker Compose

---

## 16. Appendix
- ER Diagram (separate doc)
- API Contract Document (separate doc)
- Sequence diagrams (separate doc)
- Component diagram (separate doc)





