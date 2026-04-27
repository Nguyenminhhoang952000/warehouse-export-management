# Warehouse Management System Specification

## 1. Final Database Schema
All tables include strict audit columns: `created_at`, `updated_at`, `deleted_at` (for soft deletes), `created_by`, `updated_by`, `deleted_by`. Money fields are stored as `DECIMAL(15,2)`. Indexes are included for search and filtering optimization.

### 1.1 `users`
- `id` (PK)
- `email` (VARCHAR, Unique, Indexed)
- `password_hash` (VARCHAR, Nullable for Google logins)
- `name` (VARCHAR, Indexed)
- `phone` (VARCHAR, Unique, Nullable)
- `role` (ENUM: `SALES` | `ACCOUNTANT` | `MANAGER`, Indexed)
- `status` (ENUM: `ACTIVE` | `INACTIVE`)
- `google_id` (VARCHAR, Unique, Nullable, Indexed)

### 1.2 `user_sessions`
- `id` (PK)
- `user_id` (FK to `users`, Indexed)
- `refresh_token_hash` (VARCHAR, Not Null - Hashed for security)
- `login_type` (ENUM: `LOCAL` | `GOOGLE`)
- `user_agent` (VARCHAR)
- `ip` (VARCHAR)
- `expired_at` (TIMESTAMP, Indexed for automated cleanup)

### 1.3 `customers`
- `id` (PK)
- `name` (VARCHAR, Indexed)
- `phone` (VARCHAR, Unique, Indexed)
- `address` (TEXT)
- `latitude` VARCHAR
- `longitude` VARCHAR

### 1.4 `products`
- `id` (PK)
- `name` (VARCHAR, Unique, Indexed)
- `base_price` (DECIMAL(15,2), >= 0)
- `image_url` (TEXT, Nullable)

### 1.5 `product_units`
- `id` (PK)
- `product_id` (FK to `products`, Indexed)
- `name` (VARCHAR, Unique) *(e.g., thùng, chai, cái, lọ)*

### 1.6 `orders`
- `id` (PK)
- `customer_id` (FK to `customers`, Indexed)
- `sales_id` (FK to `users`, Indexed)
- `status` (ENUM: `OPEN` | `SHIPPING` | `TOTAL_COMPLETED` | `DEBT_COMPLETED` | `FAILURE`, Indexed)
- `total_amount` (DECIMAL(15,2), >= 0)
- `discount_amount` (DECIMAL(15,2), >= 0)
- `final_amount` (DECIMAL(15,2), >= 0)
- **Snapshot Fields** (Copied at creation for historical accuracy):
  - `customer_name` (VARCHAR)
  - `customer_phone` (VARCHAR)
  - `customer_address` (TEXT)
  - `latitude` VARCHAR
  - `longitude` VARCHAR
- **Indexes:** `(sales_id, status)`, `(customer_id, status)`, `(created_at)`

### 1.7 `order_items`
- `id` (PK)
- `order_id` (FK to `orders`, Indexed)
- `product_id` (FK to `products`, Indexed)
- `unit_id` (FK to `product_units`)
- `unit_name` (VARCHAR, Snapshot from `product_units` at creation)
- `quantity` (INT, > 0)
- `price` (DECIMAL(15,2), >= 0, Snapshot from `products.base_price`)
- `discount` (DECIMAL(15,2), >= 0, Per item)
- `commission` (DECIMAL(15,2), >= 0)
- `item_total` (DECIMAL(15,2), >= 0)

---

## 2. Final API Design
**Rules:**
- DB fields are strictly `snake_case`, but TypeORM must alias all fields to `camelCase` for cleaner usage in Backend code and JSON responses.
- ALL JSON responses use the Common API Envelope (`success`, `message`, `data`, `error`, `meta`).
- ALL money fields return as `NUMBER` (float).
- ALL list endpoints support: `page`, `pageSize`, `sort` (`field:dir`), and context-specific `search`.
- List responses exclude soft-deleted records by default.

### 2.1 Auth
- **`POST /api/auth/login`**: Accepts `email`, `password`.
- **`POST /api/auth/google`**: Accepts `idToken`.
- **`POST /api/auth/refresh`**: Accepts `refreshToken`.
- **`POST /api/auth/logout`**: Accepts `refreshToken` (Invalidates session).
- **`GET /api/auth/me`**: Returns current UserData.

### 2.2 Users
- **Search fields**: `name`, `email`
- `GET /api/users` *(Filters: role, status)*
- `GET /api/users/:id`
- `POST /api/users`
- `PUT /api/users/:id`
- `DELETE /api/users/:id`
- `GET /api/users/selection` *(Provides `id`, `name` without pagination overhead)*

### 2.3 Customers
- **Search fields**: `name`, `phone`
- `GET /api/customers` *(Includes `totalOrders`, `totalSpent`)*
- `GET /api/customers/:id`
- `POST /api/customers`
- `PUT /api/customers/:id`
- `DELETE /api/customers/:id`
- `GET /api/customers/selection`

### 2.4 Products & Units
- **Search fields**: `name`
- `GET /api/products` *(Includes `totalRevenue`)*
- `GET /api/products/:id`
- `POST /api/products`
- `PUT /api/products/:id`
- `DELETE /api/products/:id`
- `GET /api/products/selection`
- `GET /api/product-units` *(List available units for forms)*
- `POST /api/product-units`
- `PUT /api/product-units/:id`
- `DELETE /api/product-units/:id`
- `GET /api/products/prices` *(Accepts `productId` filter. Returns price history timeline from completed orders)*

### 2.5 Orders
- **Search fields**: `customerName`, `id`
- `GET /api/orders` *(Filters: fromDate, toDate, status, salesId, customerId)*
- `GET /api/orders/:id`
- `POST /api/orders`
- `PUT /api/orders/:id`
- `DELETE /api/orders/:id`
- `POST /api/orders/:id/generate-pdf`

### 2.6 Dashboard
- `GET /api/dashboard` *(Filters: preset ranges or fromDate/toDate. Returns revenue metrics, stats by status, top products, top sales)*

---

## 3. Order Flow

### 3.1 Order Calculations (ENFORCED IN BACKEND)
- Input: `items` containing `productId`, `unitId`, `quantity`, `discount`, `commission`.
- Operation must run in a **Database Transaction**.
- **Formula applied per item:** `price` (from DB snapshot) -> `itemTotal = (quantity * price)`.
- **Formula applied to order:**
  - `totalAmount = sum(itemTotal)`
  - `discountAmount = sum(discount per item)`
  - `finalAmount = totalAmount - discountAmount`
- Backend ensures `quantity > 0`, `price >= 0`, `discount >= 0`, and `finalAmount >= 0`.

### 3.2 Create & Update
- **Create:** 
  - Frontend sends `customerId`, `items`, optional `latitude`/`longitude`.
  - Backend creates `order`, copies snapshot info (`customerName`, `customerPhone`, `customerAddress`, geolocation) and creates `order_items`. Default status is `OPEN`.
- **Update:** 
  - Backend wipes and replaces old `order_items` array (recalculates financial totals inside the transaction).
  - Validation ensures status transitions are valid.

### 3.3 Status Lifecycle Transitions (Strict)
- `OPEN` -> `SHIPPING`
- `SHIPPING` -> (`TOTAL_COMPLETED` | `DEBT_COMPLETED` | `FAILURE`)
- Invalid transitions are rejected with `ORDER_STATUS_TRANSITION_INVALID`.

### 3.4 Delivery & PDF
- Delivery staff uses the snapshotted geolocation coordinates `latitude` and `longitude` provided when the order was placed to ensure accurate historical delivery data.
- PDF generation `/api/orders/:id/generate-pdf` generates the invoice synchronously. Product images (`imageUrl`) **are strictly omitted** from the PDF template.

---

## 4. Price History Flow

### 4.1 Frontend Behavior
- During order creation/editing, when a product is selected, the UI displays its `imageUrl` alongside a "**View price history**" button.
- Clicking the button opens a modal.
- The UI triggers `GET /api/products/prices?productId={id}`.

### 4.2 Backend Behavior
- The `GET /api/products/prices` endpoint queries `order_items` paired with `orders`.
- Filters records where `order_items.productId = requestedId`.
- Restricts bounds to orders where `orders.status IN ('TOTAL_COMPLETED', 'DEBT_COMPLETED')`.
- Returns a timeline payload grouped with ISO date (`orders.createdAt`), `price`, `unitName`, and `orderId`, matching the unified API response model.

---

## 5. RBAC and Ownership Rules

### 5.1 Manager
- Full access to all endpoints across all systems (Auth, Users, Customers, Products, Orders, Dashboard, Configs).

### 5.2 Accountant
- Global **Read-Only** access for lists/details: Users, Customers, Products, Prices, Orders, Dashboard.
- Cannot `Create/Update/Delete` core entities.
- Allowed to generate Order PDFs.

### 5.3 Sales
- **Ownership Constraint:** Sales staff can only `GET`, `UPDATE`, and `DELETE` their own generated orders. Requests targeting orders belonging to different sales reps return `403 FORBIDDEN` or exclude unauthorized records from the list query automatically.
- **Customers:** Full CRUD capabilities and selection lists.
- **Products:** Read-only access and listing capabilities (including price history and product selections).
- **Users:** Read-only access via selection list.
- **Dashboard:** Set permissions so they can only view their own sales figures; they cannot view the sales figures of other sales staff.
- Allowed to generate Order PDFs for their authored orders.