
# 🧩 Sequence Diagrams
E‑Commerce Platform — Key User Flows

---
## 1. **Login Sequence Diagram**
```mermaid
sequenceDiagram
    actor User
    User ->> UI: Enter username/password
    UI ->> AuthService: POST /auth/login
    AuthService ->> UserRepository: ValidateCredentials()
    UserRepository -->> AuthService: UserRecord
    AuthService ->> TokenService: GenerateJWT()
    TokenService -->> AuthService: JWT Token
    AuthService -->> UI: Login Success + JWT
    UI -->> User: Redirect to Dashboard
```

---
## 2. **Add to Cart Sequence Diagram**
```mermaid
sequenceDiagram
    actor User
    User ->> UI: Click "Add to Cart"
    UI ->> CartService: POST /cart/items
    CartService ->> ProductService: GET /products/{id}
    ProductService -->> CartService: Product Details
    CartService ->> CartRepository: AddItem()
    CartRepository -->> CartService: Success
    CartService -->> UI: Item Added
    UI -->> User: Show Cart Count Updated
```

---
## 3. **Checkout Sequence Diagram**
```mermaid
sequenceDiagram
    actor User
    User ->> UI: Click "Checkout"
    UI ->> OrderService: POST /order/checkout
    OrderService ->> CartService: GetCart()
    CartService -->> OrderService: Cart items
    OrderService ->> PaymentGateway: ProcessPayment()
    PaymentGateway -->> OrderService: Payment Success
    OrderService ->> OrderRepository: CreateOrder()
    OrderRepository -->> OrderService: OrderID
    OrderService -->> UI: Checkout Success
    UI -->> User: Show Order Confirmation
```

---
## 4. **Product Search Sequence Diagram**
```mermaid
sequenceDiagram
    actor User
    User ->> UI: Enter search query
    UI ->> ProductService: GET /products?search=query
    ProductService ->> SearchEngine: QueryIndex()
    SearchEngine -->> ProductService: Result List
    ProductService -->> UI: Product List
    UI -->> User: Display Search Results
```

