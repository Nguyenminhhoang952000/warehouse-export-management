# Warehouse Frontend Description (Full Scope, Rewritten)

## 1. Goal
- Build a production-grade admin web app for warehouse operations.
- Integrate with backend APIs under /api.
- Keep architecture easy to customize at UI layer without touching feature code.

## 2. Non-Negotiable Rules
1. Ant Design is mandatory UI framework.
2. Do not import antd directly in pages, features, or business components.
3. All Antd usage must go through common wrapper components under src/common/ui.
4. Zustand is for client or UI state only.
5. TanStack Query is for server state only.
6. API sort query must follow backend contract: sort=field:dir.

## 3. Core Logic
- Authentication
  - Email/password login.
  - Google login.
  - Restore session on app load.
  - Refresh token flow and forced logout on invalid session.
- Dashboard
  - Revenue summary cards.
  - Orders by status.
  - Revenue over time.
  - Top products.
  - Top sales.
  - **RBAC Rule:** Sales roles can exclusively view their own dashboard figures.
- Users
  - Full CRUD.
  - Filter by role and status.
  - Search by name or email.
- Customers
  - Full CRUD.
  - Search by name or phone.
  - Show totalOrders and totalSpent.
- Products & Units
  - Full CRUD for Products.
  - Full CRUD for Product Units (manage specific units tied per product).
  - Search by name.
  - Historical prices from completed orders.
  - Show totalRevenue.
- Orders
  - Full CRUD with order items.
  - Selection of Product and associated Unit when building items.
  - Capture exact snapshot coordinates (latitude/longitude strings) during creation.
  - Filter by date range, status, sales, customer, keyword.
  - Show totals and status transitions.
  - Generate PDF action (product images are strictly omitted from the PDF).

## 4. Frontend Tech Stack
- React + TypeScript + Vite.
- React Router.
- TanStack Query.
- Zustand.
- React Hook Form + Zod.
- Axios.
- Ant Design.
- Recharts.
- Day.js.

## 5. Libraries
- react
- react-dom
- typescript
- vite
- react-router-dom
- @tanstack/react-query
- zustand
- axios
- react-hook-form
- zod
- @hookform/resolvers
- antd
- @ant-design/icons
- recharts
- dayjs

## 6. API Design on FE
- Shared API client in src/services/http.
- Request interceptor:
  - Attach access token.
  - Attach request id if needed.
- Response interceptor:
  - Normalize success envelope.
  - Normalize error envelope.
  - Handle 401 and refresh flow.
- Common response shape consumed by FE:
  - success, message, data, error, meta.
- Standard list query model:
  - page, pageSize, sort, search, plus domain filters.
- sort format:
  - sort=field:dir where dir is asc or desc.

## 7. Folder Structure (Frontend Classic)
- src/app
  - main bootstrap, router setup, providers
- src/pages
  - route pages only, minimal logic
- src/layouts
  - app layout, auth layout, role menu layout
- src/components
  - reusable business components
- src/common
  - ui wrappers, shared hooks, constants, utils, types
- src/common/ui
  - Antd wrappers only (Button, Input, Table, Modal, Form, Select, DatePicker, etc.)
- src/hooks
  - cross-feature hooks
- src/services
  - API services, http client, query keys
- src/store
  - Zustand stores for client or UI state only
- src/schemas
  - zod schemas shared across forms
- src/utils
  - formatters and helpers
- src/styles
  - global styles and theme overrides
- src/assets
  - static assets

## 8. Common UI Wrapper Strategy
- Goal:
  - If design customization changes, update only src/common/ui wrappers.
- Rules:
  - Business components import only from common wrappers.
  - Wrapper layer controls default size, spacing, variants, and theme behavior.
  - Wrapper layer can convert Antd props into project-specific props.

## 9. State Management Rules
- Zustand:
  - Use for sidebar state, modal state, local filter draft, auth client state.
  - Do not cache remote lists or details in Zustand.
- TanStack Query:
  - Own all remote fetch/mutation states.
  - Use centralized query keys.
  - Invalidate or update cache after mutation.

## 10. Architecture and Code Conventions
- No direct Axios call in page components.
- No direct antd import outside src/common/ui.
- Keep pages thin and delegate logic to hooks/services.
- All list pages support loading, empty, error, retry.
- All list pages support pagination, filter, sort, search.
- Keep permission checks in shared helper utilities.

## 11. Lint Enforcement (Required)
- Add restricted import rule to block antd usage outside wrapper layer.
- Example policy:
  - Block import path antd and antd/* globally.
  - Disable that block only for files under src/common/ui.

## 12. Quality and Release
- Unit tests for hooks, stores, and utils.
- Component tests for high-risk forms and tables.
- E2E tests for login flow, permission boundaries, and order flow.
- Production build served by Nginx at root path.
- Requests under /api proxied to backend.

# 13. Screen Detailed Descriptions

## 13.1 Login

### 1. RBAC
- **Roles with Access:** Public (Unauthenticated).
- **Constraints:** Authenticated users attempting to access Login are immediately redirected to their respective entry pages (`/dashboard` for Manager/Accountant, `/orders` for Sales).

### 2. Form (Login)
- **Email:** `input`, required, strict email regex validation (`/^\S+@\S+\.\S+$/`).
- **Password:** `password`, required, minimum 6 characters.
- **Login Button:** Submits the local strategy payload.
- **Google Login Button:** Triggers Google OAuth popup or redirect.
- **API Field Mapping:** Form keys `email` and `password` map exactly to JSON body. Google OAuth outputs `idToken`.

### 3. Special UI Logic
- **Loading States:** Connect `isLoading` to both Login buttons during async call to prevent double submissions.
- **Error Handling:** Catch `401 AUTH_INVALID_CREDENTIALS` and show global toast: "Invalid email or password."
- **Redirects:** Post-login, update Zustand `authStore`. Automatically route to `/dashboard` for `MANAGER`/`ACCOUNTANT`, and `/orders` for `SALES`. 

### 4. API Mapping
- **Local Login:** `POST /api/auth/login` (Body: `{ email, password }`)
- **Google Login:** `POST /api/auth/google` (Body: `{ idToken }`)

### 5. Business Logic Notes
- Expected Response Envelope: `AuthTokenData`.
- Tokens (`accessToken` and `refreshTokenHash`) must be saved securely in memory or `localStorage`.

---

## 13.2 Dashboard

### 1. RBAC
- **Roles with Access:** MANAGER, ACCOUNTANT, SALES.
- **Constraints:** 
  - MANAGER and ACCOUNTANT can view aggregated company-wide metrics.
  - SALES can ONLY view metrics aggregated strictly from their own orders.
  - "Top Sales" chart is hidden ENTIRELY from SALES users.

### 2. List View (Dashboard Panels)
#### Filters
- **Date Preset:** `select` (Today, Week, Month, Year). Default: `month`. Query param: `preset`. Visible to all.
- **From Date/To Date:** `dateRangePicker`, optional. Query params: `fromDate`, `toDate`. Visible to all. 

#### Widgets (Data Cards)
- **Revenue Summary:** Total Revenue, Total Discount, Total Orders, Total Completed Orders. Formats: Currency and Integer.
- **Orders by Status:** Pie/Donut Chart relying on `ordersByStatus`.
- **Revenue Timeline:** Line/Bar chart relying on `revenueTimeline.bucket` (X-axis Date format) and `revenueTimeline.revenue` (Y-axis Currency format).
- **Top Products:** Table/List displaying `productName` and `revenue` (currency format).
- **Top Sales:** Table/List displaying `salesName` and `revenue` (currency format) — **(MANAGER/ACCOUNTANT ONLY)**.

### 3. Special UI Logic
- **Async Loading:** Global skeleton loaders for widgets during TanStack Query fetching.
- **Error Behavior:** Catch `DASHBOARD_FILTER_INVALID` if custom dates clash, presenting an inline error. 

### 4. API Mapping
- `GET /api/dashboard?preset={value}&fromDate={date}&toDate={date}`

### 5. Business Logic Notes
- The Backend applies the Sales filters automatically via the JWT session identity. The Frontend doesn't need to append `salesId` for the SALES role. 

---

## 13.3 Users

### 1. RBAC
- **Roles with Access:** MANAGER, ACCOUNTANT, SALES.
- **Constraints:** 
  - MANAGER has Full CRUD access.
  - ACCOUNTANT has Read-Only access (View list and details).
  - SALES has NO access to the Users module (Navigation item hidden), EXCEPT for utilizing the `/selection` API quietly in dropdowns elsewhere.

### 2. List View
#### Filters
- **Search:** `input` (name or email). Query param: `search`. Delay/Debounce: 300ms.
- **Role:** `select` (SALES | ACCOUNTANT | MANAGER). Query param: `role`.
- **Status:** `select` (ACTIVE | INACTIVE). Query param: `status`.

#### Table Columns
- **Name:** `name`, String.
- **Email:** `email`, String.
- **Role:** `role`, Tag formatted (uppercase).
- **Status:** `status`, Tag formatted (Active=Green, Inactive=Red/Gray).
- **Total Orders:** `totalOrders`, Number.
- **Total Revenue:** `totalRevenue`, Currency format.
- **Created At:** `createdAt`, Date format (`DD/MM/YYYY`).

#### Actions
- **View/Edit:** Button (MANAGER only). Opens Edit Form.
- **Delete:** Button (MANAGER only). Opens confirmation modal.

### 3. Create / Edit Form
- **Name:** `input`, required.
- **Email:** `input`, required, strict email regex.
- **Phone:** `input`, optional, numeric check.
- **Password:** `password`, required on Create; optional on Update.
- **Role:** `select` (SALES, ACCOUNTANT, MANAGER), required.
- **Status:** `select` (ACTIVE, INACTIVE), required. Default: ACTIVE.

### 4. Special UI Logic
- **Disabled Logic:** A user cannot delete their own account (UI validates `session.user.id !== row.id`). 
- **Error Handling:** `USER_EMAIL_ALREADY_EXISTS` attaches a field-level error onto the Email input.

### 5. API Mapping
- **List:** `GET /api/users?page=&pageSize=&search=&role=&status=&sort=`
- **Selection:** `GET /api/users/selection`
- **Create:** `POST /api/users` (Body: `{ email, password, name, phone, role, status }`)
- **Update:** `PUT /api/users/:id`
- **Delete:** `DELETE /api/users/:id`

### 6. Business Logic Notes
- DB enforces soft deletes; deleted users will passively fall off this table. 

---

## 13.4 Customers

### 1. RBAC
- **Roles with Access:** MANAGER, SALES, ACCOUNTANT.
- **Constraints:** MANAGER and SALES have Full CRUD. ACCOUNTANT is Read-Only.

### 2. List View
#### Filters
- **Search:** `input` (name or phone). Query param: `search`. Debounced 300ms.

#### Table Columns
- **Name:** `name`, String.
- **Phone:** `phone`, String.
- **Address:** `address`, String.
- **Total Orders:** `totalOrders`, Number.
- **Total Spent:** `totalSpent`, Currency.
- **Created At:** `createdAt`, Date.

#### Actions
- **Edit:** Button (MANAGER, SALES only). Opens Update Form via Drawer or dedicated View.
- **Delete:** Button (MANAGER uniquely permitted to delete). Open confirmation popup.

### 3. Create / Edit Form
- **Name:** `input`, required.
- **Phone:** `input`, required.
- **Address:** `textarea`, required.
- **Latitude:** `input`, optional (string).
- **Longitude:** `input`, optional (string).

### 4. Special UI Logic
- **Geolocation Feature (Google Maps):** Integrate an interactive Google Map component (e.g. using `@react-google-maps/api`). The UI provides a map where users can search for places or drop a pin. This action will automatically reverse-geocode to populate the `address` text box, while concurrently capturing the exact `latitude` and `longitude` coordinates into the hidden/read-only string inputs.

### 5. API Mapping
- **List:** `GET /api/customers?page=&pageSize=&search=&sort=`
- **Selection:** `GET /api/customers/selection`
- **Create:** `POST /api/customers`
- **Update:** `PUT /api/customers/:id`
- **Delete:** `DELETE /api/customers/:id`

### 6. Business Logic Notes
- Phone format uniquely constrained in DB (`CUSTOMER_PHONE_ALREADY_EXISTS`). Sales MUST verify phone duplicates before submission or elegantly capture the generated error inline.

---

## 13.5 Products & Units

### 1. RBAC
- **Roles with Access:** MANAGER, SALES, ACCOUNTANT.
- **Constraints:** MANAGER has Full CRUD. SALES and ACCOUNTANT are Read-Only (but can trigger "View Price History").

### 2. List View
#### Filters
- **Search:** `input` (name). Query param: `search`. Debounced.

#### Table Columns
- **Name:** `name`, String.
- **Base Price:** `basePrice`, Currency.
- **Image:** `imageUrl`, Image Avatar/Thumbnail (fallback visible if null).
- **Total Revenue:** `totalRevenue`, Currency format.

#### Actions
- **Edit Product:** Button (MANAGER only).
- **Manage Units:** Button (MANAGER only). Opens Units Sub-Modal.
- **Price History:** Button (All roles). Opens Price History Modal.

### 3. Create / Edit Form (Product)
- **Name:** `input`, required.
- **Base Price:** `number`, required, `min=0`.
- **Image URL:** `input`, optional, valid URL check.

### 4. Special UI Logic
- **Manage Units Modal:** 
  - Async load existing units via `GET /api/product-units?productId={id}`.
  - Inline Creation: Input for `name` + "Add Unit" button (`POST /api/product-units`).
  - Inline Delete: Trash icon by unit (`DELETE /api/product-units/:id`).
- **Price History Modal:** 
  - Async load data via `GET /api/products/prices?productId={id}` 
  - Render as a vertical timeline or table showing `date` (formatted), `price` (currency), `unitName`, and `orderId`.

### 5. API Mapping
- **Product List:** `GET /api/products` 
- **Product Create:** `POST /api/products`
- **Product Update:** `PUT /api/products/:id`
- **Product Selection:** `GET /api/products/selection`
- **Units List:** `GET /api/product-units?productId=`
- **Units Create:** `POST /api/product-units` (Body: `{ productId, name }`)
- **Units Delete:** `DELETE /api/product-units/:id`
- **Product Prices:** `GET /api/products/prices?productId=`

---

## 13.6 Orders

### 1. RBAC
- **Roles with Access:** MANAGER, SALES, ACCOUNTANT.
- **Constraints:** 
  - MANAGER: Full CRUD.
  - ACCOUNTANT: Read-Only.
  - SALES: Full CRUD **strictly restricted** to orders where `salesId === session.user.id`. 

### 2. List View
#### Filters
- **From Date/To Date:** `dateRangePicker`. Queries: `fromDate`, `toDate`.
- **Status:** `select` (OPEN, SHIPPING, TOTAL_COMPLETED, DEBT_COMPLETED, FAILURE). Query: `status`.
- **Customer:** `async select`, fetches `/api/customers/selection`. Query: `customerId`.
- **Sales Rep:** `async select`, fetches `/api/users/selection?role=SALES`. Query: `salesId`. (Hidden entirely/Disabled dynamically for SALES users).
- **Search:** `input` (customerName, or orderId). Query param: `search`. Debounced.

#### Table Columns
- **Order ID:** `id`, Number.
- **Customer:** `customerName`, String.
- **Sales:** `salesName`, String.
- **Status:** `status`, Tag formatted.
- **Total Selected Items:** `totalItems`, Number.
- **Final Amount:** `finalAmount`, Currency.
- **Created At:** `createdAt`, Date.

#### Actions
- **View/Edit:** Button. Sales can only trigger if ownership validates.
- **Delete:** Button (MANAGER uniquely permitted to delete).
- **Generate PDF:** Button (All roles permitted if viewing constraints hold).

### 3. Create / Edit Form
- **Customer:** `async select` (required). Fetches `/api/customers/selection`.
- **Status:** `select` (required). Default OPEN. Maps to `status`.
- **Latitude & Longitude:** Silent inputs, populated via `navigator.geolocation` at point of order generation, locking in user locality coordinates.
- **Order Items (Dynamic Field Array):**
  - **Product:** `async select`, fetches `/api/products/selection`. Maps `productId`. **UX trigger:** On select, render `imageUrl` next to picker and reveal "View Price History" popup trigger.
  - **Unit:** `select`, nested async fetch to `/api/product-units?productId=` specific to the selected product. Maps `unitId`.
  - **Quantity:** `number`, required, `min=1`.
  - **Price:** `number`, read-only or strictly pre-filled originating from `basePrice` snapshots.
  - **Discount:** `number`, `min=0`.
  - **Commission:** `number`, `min=0`.
- **Preview Calculations (Read-Only block):**
  - FE logic evaluates `itemTotal`, `totalAmount`, `discountAmount`, and `finalAmount` locally to provide immediate perceptual awareness before syncing with BE calculation block. 

### 4. Special UI Logic
- **View Price History Context:** Modal dynamically fed via `/api/products/prices` on individual item rows aids reps to discount properly without jumping screens. 
- **Status Transitions UI Lock:** Restrict status dropdown selections based on linear lifecycle state: `OPEN` -> `SHIPPING` -> (`TOTAL_COMPLETED` | `DEBT_COMPLETED` | `FAILURE`). Disallow backwards movement visually.

### 5. API Mapping
- **List:** `GET /api/orders`
- **Create:** `POST /api/orders` (Payload: `{ customerId, status, latitude, longitude, items: [ { productId, unitId, quantity, price, discount, commission } ] }`)
- **Update:** `PUT /api/orders/:id`
- **Delete:** `DELETE /api/orders/:id`
- **PDF Generation:** `POST /api/orders/:id/generate-pdf`

### 6. Business Logic Notes
- The FE localized order mathematics serve fundamentally as a "cosmetic UI draft preview." Actual calculation derivations that insert into the database exist purely in the backend transaction bounds to eliminate discrepancy or UI hacks.

---

## 13.7 Document / PDF Flow

### 1. RBAC
- **Roles with Access:** MANAGER, SALES, ACCOUNTANT.
- **Constraints:** Sales can only instantiate PDF creations on their strictly owned order payloads.

### 2. Actions & Logic
- **Invocation Context:** Exposed visually via "Generate Invoice" action inside Orders List actions or single Order detail view.
- **Loading Overlay:** Entire view locks while backend synchronously wraps the layout compilation with Puppeteer.
- **Visual Formatting Constraint:** Under the strictest design standards, `imageUrl` metadata does not enter the PDF construction templates.

### 3. API Mapping
- `POST /api/orders/:id/generate-pdf`

### 4. Business Logic Notes
- Post-resolution envelope supplies an S3/Local proxy pointing to `fileUrl`.
- Routine executes standard JavaScript invocation (`window.open(fileUrl, '_blank')`) allowing users natively print/share the artifact.
