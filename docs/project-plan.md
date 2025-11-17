# High-Level Project Plan — E-Commerce Learning Project

## Phase 1 — Foundation & Setup
**Goals:** Prepare repo, architecture & environments.
- Create GitHub repo + README
- Set up backend Clean Architecture (API, Application, Domain, Infrastructure)
- Create Angular workspace (`ng new`)
- Configure SQL Server (local or Docker)
- Add `.editorconfig`, `.gitignore`, base folder structure
- Write initial architecture notes & API contract

## Phase 2 — Backend Core (Auth + Products)
**Goals:** Build essential API modules.
- Implement Identity + JWT Auth (Register/Login)
- Create Product & Category entities
- Add EF Core Migrations + Seed Data
- Product listing, filtering, pagination APIs
- Product detail API
- Configure AutoMapper
- Enable Swagger

## Phase 3 — Frontend Core (Auth + Products)
**Goals:** Build UI skeleton & main pages.
- Create modules: `auth`, `products`, `shared`
- Login & Register pages
- Product list & details
- JWT HttpInterceptor
- UI library (Material/Bootstrap)
- Navbar, footer, loader components

## Phase 4 — Cart & Checkout
**Backend:**
- Cart entities (Cart, CartItem)
- CartService + CartController
- Checkout service (Order creation)

**Frontend:**
- Cart page (add/remove/update)
- Add-to-cart button
- Checkout page (address, review, confirm)

## Phase 5 — Orders & User Account
**Backend:**
- Order & OrderItem entities
- OrderController (list + details)
- Link orders to logged-in user

**Frontend:**
- Order history
- Order details
- User profile page

## Phase 6 — Admin Panel (Optional)
**Backend:**
- Admin CRUD for Products & Categories
- Admin roles/policies

**Frontend:**
- `/admin` module
- Product list, Add/Edit forms

## Phase 7 — Testing, CI/CD & Documentation
**Backend:**
- xUnit unit tests
- Integration tests

**Frontend:**
- Jasmine/Karma unit tests
- Playwright E2E tests

**CI/CD:**
- GitHub Actions build + test + deploy

**Docs:**
- README with screenshots
- API examples
- Architecture overview

## Phase 8 — Final Enhancements (Optional)
- NgRx state management
- Caching (Memory/Redis)
- Wishlist, address book
- Reviews & ratings
- Responsive UI improvements
- Global error handling

## Estimated Timeline
| Phase | Days |
|-------|------|
| 1. Setup | 2–3 |
| 2. Backend Core | 4–6 |
| 3. Frontend Core | 5–7 |
| 4. Cart & Checkout | 5–7 |
| 5. Orders | 3–5 |
| 6. Admin | 4–6 |
| 7. Testing/CI/CD | 3–5 |
| 8. Enhancements | Ongoing |

**Total realistic time: ~5–6 weeks (part-time).**
