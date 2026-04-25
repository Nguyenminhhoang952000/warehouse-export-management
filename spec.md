# Warehouse Spec

## 1) Database

### 1.1 Base audit columns (applies to all tables)
- createdAt
- updatedAt
- deletedAt
- createdBy
- updatedBy
- deletedBy

### 1.2 users
- id
- email
- passwordHash
- name
- phone
- role (sales | accountant | manager)
- status (ACTIVE | INACTIVE)
- googleId

### 1.3 user_sessions
- id
- userId
- refreshToken
- loginType (local | google)
- userAgent
- ip
- expiredAt

### 1.4 customers
- id
- name
- phone
- address
- latitude
- longitude

### 1.5 products
- id
- name
- basePrice

### 1.6 orders
- id
- customerId
- salesId
- status (OPEN | SHIPPING | TOTAL_COMPLETED | DEBT_COMPLETED | FAILURE)
- createdAt
- updatedAt
- totalAmount
- discountAmount
- finalAmount

Snapshot fields (copy từ customer tại thời điểm mua):
- customerName
- customerPhone
- customerAddress

### 1.7 order_items
- id
- orderId
- productId
- unit (thùng | chai | cái | lọ)
- quantity
- price (giá per unit, snapshot từ product khi tạo order)
- discount
- commission
- total

## 2) API (`/api/...`)

### 2.1 Auth
- POST /auth/login
- POST /auth/google
- GET /auth/me

### 2.2 Users
- GET /users
- GET /users/:id
- POST /users
- PUT /users/:id
- DELETE /users/:id
- GET /users/selection (list id + name, hỗ trợ search name)

### 2.3 Customers
- GET /customers
- GET /customers/:id
- POST /customers
- PUT /customers/:id
- DELETE /customers/:id
- GET /customers/selection (list id + name, hỗ trợ search name)

### 2.4 Products
- GET /products
- GET /products/:id
- POST /products
- PUT /products/:id
- DELETE /products/:id
- GET /products/selection (list id + name, hỗ trợ search name)
- GET /products/prices (list historical prices from completed orders)

### 2.5 Orders
- GET /orders
- GET /orders/:id
- POST /orders
- PUT /orders/:id
- DELETE /orders/:id
- POST /orders/:id/generate-pdf

### 2.6 Dashboard
- GET /dashboard

## 3) List Views and Filters

### 3.1 Orders list
Fields:
- id
- customer.name
- customer.phone
- sales.name
- totalItems
- totalQuantity
- totalAmount
- discountAmount
- finalAmount
- status
- latitude
- longitude
- address
- createdAt

Filters:
- page
- pageSize
- fromDate
- toDate
- status
- salesId
- customerId
- search (customer name / order id)

### 3.2 Customers list
Fields:
- id
- name
- phone
- address
- totalOrders
- totalSpent
- createdAt

Filters:
- page
- pageSize
- search (name / phone)

### 3.3 Products list
Fields:
- id
- name
- basePrice
- totalRevenue
- createdAt

Filters:
- page
- pageSize
- search (name)

### 3.4 Users list
Fields:
- id
- name
- role
- status
- totalOrders
- totalRevenue
- createdAt

Filters:
- page
- pageSize
- role
- status
- search (name / email)

### 3.5 Sorting
- Tất cả table list hỗ trợ sort bằng cách bấm mũi tên ở tên field, trừ field `id`.

## 4) Tech Stack
- Backend: NestJS
- Database: PostgreSQL
- Frontend: React (build static)
- Infra: Docker (full local + production build)
- Nginx:
	- `/api` proxy tới backend
	- `/` serve static frontend
- Services: app, db
- Domain / DNS: Cloudflare
- VPS minimum:
	- RAM > 2 GB
	- CPU 2 cores

## 5) Open Questions
1. User có thể tự tạo tài khoản, hay chỉ manager tạo?
	 - Suggest: chỉ manager được tạo user để tránh data thừa.
2. Phone/address đi theo từng order hay lấy từ customer hiện tại?
3. Dashboard cần hiển thị chính xác các widget nào?
	 - Suggest:
		 - Tổng doanh thu
		 - Số đơn theo status
		 - Doanh thu theo thời gian
		 - Top sản phẩm
		 - Top sales
4. Trên phiếu giao hàng, `unit` đi theo product hay theo order item?
5. Có cần config giá theo từng unit trong product settings không?
6. Công thức tính tiền chuẩn là gì?

## 6) Estimate Timeline (ET)

### 6.1 Database (1.5d)
- Design schema chuẩn production
- Tạo migration (tables + enum)
- Setup index
- Seed data mẫu

### 6.2 Auth (1d)
- Login (email/password)
- Login Google
- Refresh token
- Middleware auth + role

### 6.3 User Module (1.5d)
- CRUD user
- Role & permission
- User selection API
- Validate (email unique, role...)

### 6.4 Customer Module (1.5d)
- CRUD customer
- Customer selection API
- Search + pagination
- Map (lat/lng support)

### 6.5 Product Module (1.5d)
- CRUD product
- Product selection API
- Search + pagination

### 6.6 Order Module (2d)
- Create order (with items)
- Update order
- Delete order
- Get order detail
- List order (filter + sort + pagination)
- Calculate total (items, amount, discount...)
- Snapshot customer + product

### 6.7 PDF (1d)
- Generate invoice PDF
- Template HTML -> PDF
- Attach order data

### 6.8 Dashboard (1.5d)
- Tổng doanh thu
- Số đơn theo status
- Doanh thu theo thời gian
- Top sản phẩm
- Top sales

### 6.9 Search / Filter / Sort (1d)
- Global pagination
- Filter (date, status, user...)
- Sort (createdAt, revenue...)
- Search (name, phone, id...)

### 6.10 DevOps (1.5d)
- Dockerfile (dev + prod)
- Docker compose (app + db + nginx)
- Env config
- Nginx config
- VPS setup
- Domain + Cloudflare
- Build & deploy frontend
- SSL setup