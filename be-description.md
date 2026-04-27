# Warehouse Backend Description (Full Scope, Detailed API Response)

## 1. Goal
- Build a production-ready warehouse management API.
- Enforce one unified API response envelope for all JSON endpoints.
- Apply strict auth, RBAC, ownership checks, soft-delete, and audit columns.
- Provide complete endpoint contracts for FE integration without ambiguity.

## 2. Confirmed Decisions
1. Pagination params: `page` + `pageSize`.
2. Sort param: `sort=field:dir`.
3. Token location: `accessToken` and `refreshToken` both in JSON response body.
4. HTTP style: POST, PUT, DELETE all return HTTP 200 with envelope body.
5. Error code catalog: full by domain.
6. Generate PDF: synchronous, returns URL in envelope.
7. Google login input: `idToken`.
8. Product prices response: grouped by `productId`, each product has price timeline.
9. Dashboard date filter: preset + custom range.
10. Orders list geo when snapshot null: return null, no fallback.
11. Delete behavior: soft-delete for all domains.
12. Money fields in response: float number.

## 3. Common API Envelope

### 3.1 Success
```json
{
  "success": true,
  "message": "Operation success",
  "data": {},
  "error": null,
  "meta": {}
}
```

### 3.2 Error
```json
{
  "success": false,
  "message": "Validation failed",
  "data": null,
  "error": {
    "code": "VALIDATION_ERROR",
    "details": []
  },
  "meta": {}
}
```

### 3.3 Pagination
```json
{
  "success": true,
  "message": "Get list successfully",
  "data": [],
  "error": null,
  "meta": {
    "page": 1,
    "pageSize": 10,
    "total": 100,
    "totalPages": 10
  }
}
```

## 4. Base Conventions

### 4.1 Prefix
- Base prefix: `/api`.

### 4.2 Auth
- Protected endpoints require `Authorization: Bearer <accessToken>`.

### 4.3 List Query Standard
- `page`: integer, default 1.
- `pageSize`: integer, default 10.
- `sort`: `field:dir`.
- `dir` values: `asc` or `desc`.
- `search` and domain filters per endpoint.

### 4.4 Soft Delete
- DELETE updates `deletedAt` and `deletedBy`.
- List/detail excludes deleted records by default.

### 4.5 Money
- All money values in response are float number.
- Examples: `totalAmount`, `discountAmount`, `finalAmount`, `basePrice`, `totalRevenue`.

## 5. Enums

### 5.1 UserRole
- `sales`
- `accountant`
- `manager`

### 5.2 UserStatus
- `ACTIVE`
- `INACTIVE`

### 5.3 LoginType
- `local`
- `google`

### 5.4 OrderStatus
- `OPEN`
- `SHIPPING`
- `TOTAL_COMPLETED`
- `DEBT_COMPLETED`
- `FAILURE`

### 5.5 Unit
- `thung`
- `chai`
- `cai`
- `lo`

## 6. Error Code Catalog

### 6.1 Common
- `VALIDATION_ERROR`
- `BAD_REQUEST`
- `UNAUTHORIZED`
- `FORBIDDEN`
- `NOT_FOUND`
- `CONFLICT`
- `INTERNAL_ERROR`

### 6.2 Auth
- `AUTH_INVALID_CREDENTIALS`
- `AUTH_TOKEN_EXPIRED`
- `AUTH_TOKEN_INVALID`
- `AUTH_REFRESH_TOKEN_INVALID`
- `AUTH_REFRESH_TOKEN_REVOKED`
- `AUTH_GOOGLE_TOKEN_INVALID`
- `AUTH_SESSION_NOT_FOUND`

### 6.3 Users
- `USER_EMAIL_ALREADY_EXISTS`
- `USER_PHONE_ALREADY_EXISTS`
- `USER_NOT_FOUND`
- `USER_STATUS_INVALID`

### 6.4 Customers
- `CUSTOMER_NOT_FOUND`
- `CUSTOMER_PHONE_ALREADY_EXISTS`

### 6.5 Products
- `PRODUCT_NOT_FOUND`
- `PRODUCT_NAME_ALREADY_EXISTS`

### 6.6 Orders
- `ORDER_NOT_FOUND`
- `ORDER_STATUS_TRANSITION_INVALID`
- `ORDER_ITEM_INVALID`
- `ORDER_EMPTY_ITEMS`
- `ORDER_CALCULATION_INVALID`

### 6.7 Dashboard
- `DASHBOARD_FILTER_INVALID`

### 6.8 PDF
- `PDF_GENERATION_FAILED`

## 7. Data Objects

### 7.1 AuthTokenData
- `accessToken`: string
- `refreshToken`: string
- `tokenType`: string
- `expiresIn`: number
- `refreshExpiresIn`: number
- `user`:
  - `id`: number
  - `email`: string
  - `name`: string
  - `phone`: string or null
  - `role`: UserRole
  - `status`: UserStatus

### 7.2 UserData
- `id`, `email`, `name`, `phone`, `role`, `status`, `googleId`
- `createdAt`, `updatedAt`

### 7.3 CustomerData
- `id`, `name`, `phone`, `address`, `latitude`, `longitude`
- `createdAt`, `updatedAt`

### 7.4 ProductData
- `id`, `name`, `basePrice`
- `createdAt`, `updatedAt`

### 7.5 OrderItemData
- `id`, `productId`, `productName`
- `unit`, `quantity`
- `price`, `discount`, `commission`, `total`

### 7.6 OrderData
- `id`, `customerId`, `salesId`, `status`
- `customerName`, `customerPhone`, `customerAddress`
- `latitude`, `longitude`
- `totalAmount`, `discountAmount`, `finalAmount`
- `items`: OrderItemData[]
- `createdAt`, `updatedAt`

## 8. API Details By Endpoint

## 8.1 Auth APIs

### POST /api/auth/login
- Auth: Public
- Body:
  - `email`: string
  - `password`: string
- Success 200:
  - `message`: `Login successfully`
  - `data`: AuthTokenData
- Error codes:
  - `VALIDATION_ERROR`
  - `AUTH_INVALID_CREDENTIALS`
  - `USER_STATUS_INVALID`

### POST /api/auth/google
- Auth: Public
- Body:
  - `idToken`: string
- Success 200:
  - `message`: `Login with Google successfully`
  - `data`: AuthTokenData
- Error codes:
  - `VALIDATION_ERROR`
  - `AUTH_GOOGLE_TOKEN_INVALID`
  - `USER_STATUS_INVALID`

### GET /api/auth/me
- Auth: Required
- Success 200:
  - `message`: `Get profile successfully`
  - `data`:
    - `id`, `email`, `name`, `phone`, `role`, `status`, `googleId`
    - `createdAt`, `updatedAt`
- Error codes:
  - `UNAUTHORIZED`
  - `AUTH_TOKEN_EXPIRED`
  - `AUTH_TOKEN_INVALID`

### POST /api/auth/refresh
- Auth: Public
- Body:
  - `refreshToken`: string
- Success 200:
  - `message`: `Refresh token successfully`
  - `data`: AuthTokenData
- Error codes:
  - `VALIDATION_ERROR`
  - `AUTH_REFRESH_TOKEN_INVALID`
  - `AUTH_REFRESH_TOKEN_REVOKED`
  - `AUTH_SESSION_NOT_FOUND`

### POST /api/auth/logout
- Auth: Required
- Body:
  - `refreshToken`: string
- Success 200:
  - `message`: `Logout successfully`
  - `data`:
    - `loggedOut`: true
- Error codes:
  - `VALIDATION_ERROR`
  - `AUTH_REFRESH_TOKEN_INVALID`
  - `AUTH_SESSION_NOT_FOUND`

## 8.2 Users APIs

### GET /api/users
- Auth: manager
- Query:
  - `page`, `pageSize`
  - `role`
  - `status`
  - `search`
  - `sort`
- Success 200:
  - `message`: `Get users successfully`
  - `data` item:
    - `id`, `name`, `email`, `role`, `status`
    - `totalOrders`: number
    - `totalRevenue`: number
    - `createdAt`
  - `meta`: pagination
- Error codes:
  - `FORBIDDEN`
  - `VALIDATION_ERROR`

### GET /api/users/:id
- Auth: manager
- Success 200:
  - `message`: `Get user successfully`
  - `data`: UserData
- Error codes:
  - `FORBIDDEN`
  - `USER_NOT_FOUND`

### POST /api/users
- Auth: manager
- Body:
  - `email`, `password`, `name`, `phone`, `role`, `status`
- Success 200:
  - `message`: `Create user successfully`
  - `data`: UserData
- Error codes:
  - `FORBIDDEN`
  - `VALIDATION_ERROR`
  - `USER_EMAIL_ALREADY_EXISTS`
  - `USER_PHONE_ALREADY_EXISTS`

### PUT /api/users/:id
- Auth: manager
- Body:
  - `name`, `phone`, `role`, `status`, optional `password`
- Success 200:
  - `message`: `Update user successfully`
  - `data`: UserData
- Error codes:
  - `FORBIDDEN`
  - `VALIDATION_ERROR`
  - `USER_NOT_FOUND`
  - `USER_EMAIL_ALREADY_EXISTS`
  - `USER_PHONE_ALREADY_EXISTS`

### DELETE /api/users/:id
- Auth: manager
- Success 200:
  - `message`: `Delete user successfully`
  - `data`:
    - `deleted`: true
    - `id`: number
- Error codes:
  - `FORBIDDEN`
  - `USER_NOT_FOUND`

### GET /api/users/selection
- Auth: manager, accountant, sales
- Query:
  - `search`
  - `page`
  - `pageSize`
- Success 200:
  - `message`: `Get user selections successfully`
  - `data` item:
    - `id`
    - `name`
  - `meta`: pagination
- Error codes:
  - `VALIDATION_ERROR`
  - `FORBIDDEN`

## 8.3 Customers APIs

### GET /api/customers
- Auth: manager, sales, accountant
- Query:
  - `page`, `pageSize`, `search`, `sort`
- Success 200:
  - `message`: `Get customers successfully`
  - `data` item:
    - `id`, `name`, `phone`, `address`
    - `totalOrders`: number
    - `totalSpent`: number
    - `createdAt`
- Error codes:
  - `VALIDATION_ERROR`
  - `FORBIDDEN`

### GET /api/customers/:id
- Auth: manager, sales, accountant
- Success 200:
  - `message`: `Get customer successfully`
  - `data`: CustomerData
- Error codes:
  - `CUSTOMER_NOT_FOUND`
  - `FORBIDDEN`

### POST /api/customers
- Auth: manager, sales
- Body:
  - `name`, `phone`, `address`, `latitude`, `longitude`
- Success 200:
  - `message`: `Create customer successfully`
  - `data`: CustomerData
- Error codes:
  - `VALIDATION_ERROR`
  - `CUSTOMER_PHONE_ALREADY_EXISTS`
  - `FORBIDDEN`

### PUT /api/customers/:id
- Auth: manager, sales
- Body:
  - `name`, `phone`, `address`, `latitude`, `longitude`
- Success 200:
  - `message`: `Update customer successfully`
  - `data`: CustomerData
- Error codes:
  - `VALIDATION_ERROR`
  - `CUSTOMER_NOT_FOUND`
  - `CUSTOMER_PHONE_ALREADY_EXISTS`
  - `FORBIDDEN`

### DELETE /api/customers/:id
- Auth: manager
- Success 200:
  - `message`: `Delete customer successfully`
  - `data`:
    - `deleted`: true
    - `id`: number
- Error codes:
  - `CUSTOMER_NOT_FOUND`
  - `FORBIDDEN`

### GET /api/customers/selection
- Auth: manager, sales, accountant
- Query:
  - `search`, `page`, `pageSize`
- Success 200:
  - `message`: `Get customer selections successfully`
  - `data` item:
    - `id`
    - `name`
  - `meta`: pagination
- Error codes:
  - `VALIDATION_ERROR`
  - `FORBIDDEN`

## 8.4 Products APIs

### GET /api/products
- Auth: manager, sales, accountant
- Query:
  - `page`, `pageSize`, `search`, `sort`
- Success 200:
  - `message`: `Get products successfully`
  - `data` item:
    - `id`, `name`, `basePrice`
    - `totalRevenue`: number
    - `createdAt`
- Error codes:
  - `VALIDATION_ERROR`
  - `FORBIDDEN`

### GET /api/products/:id
- Auth: manager, sales, accountant
- Success 200:
  - `message`: `Get product successfully`
  - `data`: ProductData
- Error codes:
  - `PRODUCT_NOT_FOUND`
  - `FORBIDDEN`

### POST /api/products
- Auth: manager
- Body:
  - `name`, `basePrice`
- Success 200:
  - `message`: `Create product successfully`
  - `data`: ProductData
- Error codes:
  - `VALIDATION_ERROR`
  - `PRODUCT_NAME_ALREADY_EXISTS`
  - `FORBIDDEN`

### PUT /api/products/:id
- Auth: manager
- Body:
  - `name`, `basePrice`
- Success 200:
  - `message`: `Update product successfully`
  - `data`: ProductData
- Error codes:
  - `VALIDATION_ERROR`
  - `PRODUCT_NOT_FOUND`
  - `PRODUCT_NAME_ALREADY_EXISTS`
  - `FORBIDDEN`

### DELETE /api/products/:id
- Auth: manager
- Success 200:
  - `message`: `Delete product successfully`
  - `data`:
    - `deleted`: true
    - `id`: number
- Error codes:
  - `PRODUCT_NOT_FOUND`
  - `FORBIDDEN`

### GET /api/products/selection
- Auth: manager, sales, accountant
- Query:
  - `search`, `page`, `pageSize`
- Success 200:
  - `message`: `Get product selections successfully`
  - `data` item:
    - `id`
    - `name`
  - `meta`: pagination
- Error codes:
  - `VALIDATION_ERROR`
  - `FORBIDDEN`

### GET /api/products/prices
- Auth: manager, sales, accountant
- Query:
  - `page`, `pageSize`
  - `fromDate`, `toDate` optional
  - `search` optional
  - `sort` optional
- Success 200:
  - `message`: `Get product prices successfully`
  - `data` item:
    - `productId`
    - `productName`
    - `timeline`:
      - `date`: ISO datetime
      - `price`: number
      - `unit`: Unit
      - `orderId`: number
  - `meta`: pagination
- Error codes:
  - `VALIDATION_ERROR`
  - `FORBIDDEN`

## 8.5 Orders APIs

### GET /api/orders
- Auth: manager, sales, accountant
- Query:
  - `page`, `pageSize`
  - `fromDate`, `toDate`
  - `status`
  - `salesId`
  - `customerId`
  - `search`
  - `sort`
- Success 200:
  - `message`: `Get orders successfully`
  - `data` item:
    - `id`
    - `customerName`
    - `customerPhone`
    - `salesName`
    - `totalItems`: number
    - `totalQuantity`: number
    - `totalAmount`: number
    - `discountAmount`: number
    - `finalAmount`: number
    - `status`
    - `latitude`: number or null
    - `longitude`: number or null
    - `address`
    - `createdAt`
  - `meta`: pagination
- Error codes:
  - `VALIDATION_ERROR`
  - `FORBIDDEN`

### GET /api/orders/:id
- Auth: manager, sales, accountant
- Success 200:
  - `message`: `Get order successfully`
  - `data`: OrderData
- Error codes:
  - `ORDER_NOT_FOUND`
  - `FORBIDDEN`

### POST /api/orders
- Auth: manager, sales
- Body:
  - `customerId`
  - `salesId`
  - `status` optional default `OPEN`
  - `latitude` optional
  - `longitude` optional
  - `items`:
    - `productId`
    - `unit`
    - `quantity`
    - `price`
    - `discount`
    - `commission`
- Success 200:
  - `message`: `Create order successfully`
  - `data`: OrderData
- Error codes:
  - `VALIDATION_ERROR`
  - `ORDER_EMPTY_ITEMS`
  - `ORDER_ITEM_INVALID`
  - `ORDER_CALCULATION_INVALID`
  - `CUSTOMER_NOT_FOUND`
  - `PRODUCT_NOT_FOUND`
  - `FORBIDDEN`

### PUT /api/orders/:id
- Auth: manager, sales
- Body:
  - `status` optional
  - `items` optional
  - monetary fields recomputed by BE
- Success 200:
  - `message`: `Update order successfully`
  - `data`: OrderData
- Error codes:
  - `VALIDATION_ERROR`
  - `ORDER_NOT_FOUND`
  - `ORDER_STATUS_TRANSITION_INVALID`
  - `ORDER_ITEM_INVALID`
  - `ORDER_CALCULATION_INVALID`
  - `FORBIDDEN`

### DELETE /api/orders/:id
- Auth: manager
- Success 200:
  - `message`: `Delete order successfully`
  - `data`:
    - `deleted`: true
    - `id`: number
- Error codes:
  - `ORDER_NOT_FOUND`
  - `FORBIDDEN`

### POST /api/orders/:id/generate-pdf
- Auth: manager, sales, accountant
- Success 200:
  - `message`: `Generate pdf successfully`
  - `data`:
    - `orderId`: number
    - `fileName`: string
    - `fileUrl`: string
    - `generatedAt`: ISO datetime
- Error codes:
  - `ORDER_NOT_FOUND`
  - `PDF_GENERATION_FAILED`
  - `FORBIDDEN`

## 8.6 Dashboard API

### GET /api/dashboard
- Auth: manager, accountant
- Query:
  - `preset` optional: `today`, `week`, `month`, `year`
  - `fromDate` optional
  - `toDate` optional
- Success 200:
  - `message`: `Get dashboard successfully`
  - `data`:
    - `revenueSummary`:
      - `totalRevenue`: number
      - `totalDiscount`: number
      - `totalOrders`: number
      - `totalCompletedOrders`: number
    - `ordersByStatus`:
      - `status`
      - `count`
    - `revenueTimeline`:
      - `bucket`: ISO date or datetime
      - `revenue`: number
      - `orders`: number
    - `topProducts`:
      - `productId`
      - `productName`
      - `revenue`: number
      - `quantity`: number
    - `topSales`:
      - `salesId`
      - `salesName`
      - `revenue`: number
      - `orders`: number
- Error codes:
  - `VALIDATION_ERROR`
  - `DASHBOARD_FILTER_INVALID`
  - `FORBIDDEN`

## 9. RBAC Matrix

1. manager
- Full access to all endpoints.

2. accountant
- Read access for users list/detail/selection, customers list/detail/selection, products list/detail/selection/prices, orders list/detail, dashboard.
- No create/update/delete in users/customers/products/orders.
- Can generate pdf.

3. sales
- Access: auth endpoints, customers CRUD and selection, products read and selection/prices, orders CRUD within ownership policy, users selection.
- No users CRUD.
- No dashboard.

## 10. Non-Functional Requirements
1. Global ValidationPipe with strict DTO validation.
2. Global response interceptor for success envelope.
3. Global exception filter for error envelope.
4. Structured logging with requestId.
5. Security headers, CORS allowlist, auth rate limiting.
6. Full audit columns on all entities.
7. CI pipeline: lint, test, build, migration check.
8. E2E contract tests for envelope and error code consistency.

## 11. Libraries will be used:

### 11.1 Core Framework
- `@nestjs/common`
- `@nestjs/core`
- `@nestjs/platform-express`
- `@nestjs/config`
- `reflect-metadata`
- `rxjs`

### 11.2 Database and ORM
- `@nestjs/typeorm`
- `typeorm`
- `pg`

### 11.3 Validation and Transformation
- `class-validator`
- `class-transformer`

### 11.4 Authentication and Authorization
- `@nestjs/passport`
- `passport`
- `@nestjs/jwt`
- `passport-jwt`
- `passport-google-oauth20`
- `bcrypt`

### 11.5 API Documentation
- `@nestjs/swagger`
- `swagger-ui-express`

### 11.6 Security and Hardening
- `helmet`
- `@nestjs/throttler`

### 11.7 Logging and Observability
- `nestjs-pino`
- `pino`
- `pino-http`

### 11.8 PDF Generation
- `puppeteer`

### 11.9 Utilities
- `dayjs`
- `decimal.js`

### 11.10 Testing
- `jest`
- `@nestjs/testing`
- `supertest`
- `ts-jest`

### 11.11 Development Tooling
- `typescript`
- `ts-node`
- `ts-node-dev`
- `eslint`
- `prettier`


