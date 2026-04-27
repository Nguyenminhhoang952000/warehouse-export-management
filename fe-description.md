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
- Users
  - Full CRUD.
  - Filter by role and status.
  - Search by name or email.
- Customers
  - Full CRUD.
  - Search by name or phone.
  - Show totalOrders and totalSpent.
- Products
  - Full CRUD.
  - Search by name.
  - Historical prices from completed orders.
  - Show totalRevenue.
- Orders
  - Full CRUD with order items.
  - Filter by date range, status, sales, customer, keyword.
  - Show totals and status transitions.
  - Generate PDF action.

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
