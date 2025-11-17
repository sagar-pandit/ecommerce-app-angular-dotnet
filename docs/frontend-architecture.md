# Frontend Architecture Document — Angular (E-Commerce Learning Project)

This document describes the frontend architecture, project layout, design patterns, development conventions, and recommended implementation details for the Angular SPA portion of your e-commerce learning project.

It is written to reflect real-world developer practices while being optimized for a solo learner.

---

## 1. Goals
- Build a modular, testable Angular application that consumes the ASP.NET Core Web API.
- Keep components small and focused (Single Responsibility).
- Use standalone components (Angular modern pattern) where appropriate.
- Provide predictable state flow using services; optionally add NgRx later.
- Make it easy to onboard and extend.

---

## 2. Tech & Libraries
- Angular (latest stable, Nov 2025)
- TypeScript
- Angular Router
- HttpClient
- Angular Material (or Tailwind/Bootstrap — choose one)
- RxJS for reactive programming
- Optional: NgRx for state management (phase 8)
- Testing: Jasmine/Karma for unit tests; Playwright for e2e

---

## 3. High-Level Architecture

```
Browser
  ↓
Angular App (SPA)
  ├─ Core Module (Auth, Interceptors, Guards)
  ├─ Shared Module (UI components)
  ├─ Feature Modules
  │   ├─ Products Module
  │   ├─ Cart Module
  │   ├─ Checkout Module
  │   └─ Orders Module
  └─ Admin Module (optional)
  ↓
REST API (ASP.NET Core)
```

Key principles:
- **Feature modules**: group related components, services, and pages.
- **Core module**: singletons—auth service, token interceptor, error handler, logger.
- **Shared module**: reusable UI components, pipes, directives.
- **Lazy loading**: feature modules loaded lazily (products list may be eager; admin lazy).

---

## 4. Project Structure (recommended)
```
client/
└── src/
    ├── app/
    │   ├── core/
    │   │   ├── auth/
    │   │   │   ├── auth.service.ts
    │   │   │   ├── auth.guard.ts
    │   │   │   └── token.interceptor.ts
    │   │   └── http/
    │   │       └── api.service.ts
    │   ├── shared/
    │   │   ├── components/
    │   │   ├── pipes/
    │   │   └── models/
    │   ├── features/
    │   │   ├── products/
    │   │   │   ├── product-list/
    │   │   │   ├── product-detail/
    │   │   │   └── products-routing.module.ts
    │   │   ├── cart/
    │   │   ├── checkout/
    │   │   └── orders/
    │   ├── admin/ (optional)
    │   ├── app-routing.module.ts
    │   └── app.component.ts
    ├── assets/
    └── environments/
```

---

## 5. Important Frontend Patterns

### 5.1 Standalone Components
Prefer standalone components for new development to reduce NgModule boilerplate. Use `standalone: true` alongside `imports` for needed modules.

### 5.2 Services for State
- Use Angular services as the single source of truth for transient UI state (cart contents, cached products).
- Keep state logic in services; components only present data.

### 5.3 Reactive Streams (RxJS)
- Use `BehaviorSubject` for state sharing (e.g., CartService exposes `cart$`).
- Use `switchMap`, `debounceTime`, and `distinctUntilChanged` for search inputs.

### 5.4 API Layer
- Centralize HTTP calls in `ApiService` or feature-specific services (e.g., `ProductService`).
- Provide typed response interfaces.
- Use interceptors for token injection and global error handling.

### 5.5 Error Handling
- Global Error Handler for unexpected errors.
- UI-friendly error messages via a Snackbar/Toast service.

### 5.6 Forms
- Use Reactive Forms for complex forms (checkout, product editor).
- Use validators and show inline errors.

### 5.7 Routing
- Use route guards to protect authenticated routes.
- Lazy load feature modules via `loadChildren` for admin and checkout.

---

## 6. Authentication Flow (Client-side)
1. User submits login form → `AuthService.login()` → POST `/api/auth/login`.
2. On success, store JWT in `localStorage` (learning-phase) and `AuthService.user$` emits user.
3. `token.interceptor` attaches `Authorization` header to outgoing requests.
4. Protected routes use `AuthGuard` which checks `AuthService.isAuthenticated()`.

**Note:** For production, prefer refresh tokens and secure cookie storage; localStorage is acceptable for learning.

---

## 7. Cart Implementation (Client-side)
- CartService stores cart in memory and persists to `localStorage` for session recovery.
- On login, sync local cart with server cart (merge strategy).
- CartService exposes `cart$` BehaviorSubject and methods: `addItem()`, `updateItem()`, `removeItem()`, `clear()`.

---

## 8. UI/UX Guidelines
- Minimal, clean layout: top navbar with search bar, categories dropdown, cart icon with count.
- Responsive design: mobile-first components.
- Accessibility basics: semantic HTML, alt tags for images, keyboard navigation for forms.

---

## 9. Testing Strategy (Frontend)
- **Unit tests**: Test components and services with Jasmine/Karma.
- **Integration**: Component tests with TestBed for routing and DI.
- **E2E tests**: Playwright scripts for login, add-to-cart, checkout flows.

Write tests alongside features; aim for meaningful unit test coverage, not 100%.

---

## 10. Build & Deployment Notes
- Use environment files for API base URLs.
- `ng build --configuration production` for production build.
- Serve static assets via Azure Static Web Apps, Netlify, or S3 + CloudFront.
- Optionally dockerize frontend for parity with backend.

---

## 11. Developer Workflow
- Branch per feature: `feature/products-list`.
- Keep small commits and PRs; write descriptive titles.
- Use `ng lint` and `npm test` in pre-commit hooks (husky optional).

---

## 12. Starter checklist for first frontend tasks
- Scaffold Angular app: `ng new client --routing --style=scss`
- Create Core module and AuthService
- Implement ProductService with `GET /api/products`
- Create ProductListComponent to display product cards
- Hook up simple navigation and run `ng serve`

---

## 13. Useful References
- Angular official docs: https://angular.io
- RxJS docs: https://rxjs.dev
- Angular Material: https://material.angular.io
