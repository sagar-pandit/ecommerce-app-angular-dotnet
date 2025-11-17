# API Contract Document — E-Commerce Learning Project

Human-readable, Swagger-style API contract for the learning e-commerce backend.

> Base URL (dev): `https://localhost:5001/api`

---

## Authentication
All protected endpoints require a Bearer JWT token in the `Authorization` header:
```
Authorization: Bearer <token>
```

### POST /api/auth/register
Register a new user.
- **Request**
```json
{
  "name": "string",
  "email": "user@example.com",
  "password": "P@ssw0rd"
}
```
- **Response**
  - `201 Created` (success)
  - `400 Bad Request` (validation error)

### POST /api/auth/login
Authenticate and receive JWT.
- **Request**
```json
{
  "email": "user@example.com",
  "password": "P@ssw0rd"
}
```
- **Response**
  - `200 OK`
```json
{
  "token": "eyJ...",
  "expiresIn": 3600,
  "user": {
    "id": 123,
    "name": "User Name",
    "email": "user@example.com"
  }
}
```
  - `401 Unauthorized` (invalid credentials)

---

## Products

### GET /api/products
Get paged list of products. Public.
- **Query parameters**
  - `search` (string, optional)
  - `categoryId` (int, optional)
  - `page` (int, optional, default=1)
  - `pageSize` (int, optional, default=20)
- **Response** `200 OK`
```json
{
  "items": [ {ProductDto} ],
  "page": 1,
  "pageSize": 20,
  "totalItems": 123,
  "totalPages": 7
}
```

`ProductDto` simplified:
```json
{
  "id": 1,
  "name": "Product name",
  "slug": "product-name",
  "description": "...",
  "price": 199.99,
  "imageUrl": "https://...",
  "categoryId": 5
}
```

### GET /api/products/{id}
Get product details by id. Public.
- **Response** `200 OK` → `ProductDto` with additional fields like `stockQty`, `images`.
- `404 Not Found` if missing.

### POST /api/products
Create a product. **Admin only**.
- **Request** `ProductCreateDto`
```json
{
  "name": "string",
  "description": "string",
  "price": 100.0,
  "categoryId": 1,
  "imageUrl": "string"
}
```
- **Response** `201 Created` with location header `/api/products/{id}`

### PUT /api/products/{id}
Update a product. **Admin only**.
- **Response** `204 No Content` on success.

### DELETE /api/products/{id}
Delete a product. **Admin only**.
- **Response** `204 No Content`.

---

## Cart
All cart endpoints require authentication.

### GET /api/cart
Get user cart items.
- **Response** `200 OK`
```json
{
  "items": [
    {
      "cartItemId": 10,
      "productId": 1,
      "productName": "...",
      "quantity": 2,
      "unitPrice": 199.99,
      "imageUrl": "...",
      "lineTotal": 399.98
    }
  ],
  "totalAmount": 399.98
}
```

### POST /api/cart
Add product to cart (or update quantity if exists).
- **Request**
```json
{
  "productId": 1,
  "quantity": 2
}
```
- **Response** `200 OK` with updated cart item or `201 Created`.

### PUT /api/cart/{cartItemId}
Update quantity for a cart item.
- **Request** `{ "quantity": 3 }
- **Response** `204 No Content`

### DELETE /api/cart/{cartItemId}
Remove item from cart.
- **Response** `204 No Content`

---

## Orders
All order endpoints require authentication.

### POST /api/orders
Place an order from current user's cart. Server computes totals.
- **Request** (optional shipping info, or use user's saved address)
```json
{
  "addressId": 5,
  "paymentMethod": "Mock"
}
```
- **Response** `201 Created`
```json
{
  "orderId": 1001,
  "status": "Created",
  "totalAmount": 499.98
}
```

### GET /api/orders
List authenticated user's orders (paged)
- **Response** `200 OK` list of `OrderSummaryDto`.

### GET /api/orders/{id}
Get order details (user must own order unless Admin).
- **Response** `200 OK` `OrderDto` (including items)
- `404` if not found or unauthorized.

---

## Admin - Orders
**Admin role required**

### GET /api/admin/orders
List all orders for admin dashboard (paged).

### PUT /api/admin/orders/{id}/status
Update order status (e.g., Processing, Shipped, Delivered).
- **Request** `{ "status": "Shipped" }`
- **Response** `204 No Content`

---

## Users & Profiles

### GET /api/users/me
Get current user's profile.
- **Response** `200 OK`

### PUT /api/users/me
Update profile (name, phone, etc.)
- **Response** `204 No Content`

---

## Common Response Patterns
- **400 Bad Request**: validation failed (return details)
- **401 Unauthorized**: missing or invalid token
- **403 Forbidden**: insufficient role/permission
- **404 Not Found**: resource missing
- **500 Internal Server Error**: unexpected error

---

## Pagination
Endpoints returning lists implement pagination with `page` and `pageSize` query parameters and return `totalItems` and `totalPages` metadata in the response.

---

## Errors (example)
```json
{
  "status": 400,
  "errors": [
    { "field": "email", "message": "Invalid email address" }
  ]
}
```

---

## Notes & Learning Tips
- Keep response DTOs deliberately small: avoid returning heavy domain objects with navigation properties.
- Use swagger (Swashbuckle) to auto-generate a live API explorer.
- Implement sample responses in controllers to help frontend integration during early learning (mock data if needed).
