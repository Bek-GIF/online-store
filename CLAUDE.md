# CLAUDE.md

## Project Overview

This is a full-stack e-commerce application with two roles:

- `USER`
- `ADMIN`

Users can browse products, search, filter, add products to cart and create orders.

Admins can manage products, categories, users and orders.

The project is designed as a real production-style educational project. Code should be clean, understandable and maintainable.

## Tech Stack

### Frontend

- Next.js
- TypeScript
- React
- React Query
- React Hook Form
- Axios
- Zustand
- SCSS / CSS

### Backend

- Node.js
- Express
- TypeScript
- Prisma
- PostgreSQL
- Supabase
- JWT
- bcrypt
- Zod

### Database

PostgreSQL hosted on Supabase.

Prisma is used for:

- database schema
- migrations
- database client
- relations
- queries

Never manually modify the production database structure without updating the Prisma schema.

## Project Structure

Use a monorepo-style structure:

```
project/
├── frontend/
│   ├── src/
│   │   ├── app/
│   │   ├── pages/
│   │   ├── widgets/
│   │   ├── features/
│   │   ├── entities/
│   │   ├── shared/
│   │   ├── services/
│   │   ├── store/
│   │   └── types/
│   ├── public/
│   └── package.json
│
├── backend/
│   ├── src/
│   │   ├── routes/
│   │   ├── controllers/
│   │   ├── services/
│   │   ├── middlewares/
│   │   ├── schemas/
│   │   ├── utils/
│   │   ├── types/
│   │   ├── config/
│   │   ├── lib/
│   │   ├── app.ts
│   │   └── server.ts
│   ├── prisma/
│   │   └── schema.prisma
│   └── package.json
│
├── CLAUDE.md
└── README.md
```

## Frontend Architecture

Use a simplified Mini FSD architecture.

Do NOT implement full enterprise-level FSD.

### Layers

```
app
widgets
features
entities
shared
```

### app

Application-level pages, layouts and providers.

```
app/
├── layout.tsx
├── page.tsx
├── products/
├── cart/
├── checkout/
├── profile/
├── orders/
└── admin/
```

### widgets

Large reusable page sections.

```
widgets/
├── header/
├── footer/
├── product-list/
├── product-details/
├── cart-summary/
├── order-list/
├── admin-sidebar/
└── dashboard-stats/
```

### features

User actions and business interactions.

```
features/
├── auth/
├── add-to-cart/
├── create-order/
├── update-cart/
├── product-search/
├── product-filter/
├── login/
├── register/
└── change-order-status/
```

### entities

Business entities.

```
entities/
├── product/
├── category/
├── user/
├── cart/
└── order/
```

An entity can contain:

```
product/
├── ui/
├── api/
├── model/
└── types.ts
```

### shared

Generic reusable code.

```
shared/
├── ui/
├── lib/
├── hooks/
├── api/
├── constants/
├── types/
└── config/
```

Examples:

```
shared/ui/Button
shared/ui/Input
shared/ui/Modal
shared/ui/Loader
shared/ui/EmptyState
```

Do not put business-specific logic inside `shared`.

## Frontend Rules

### React Query

Use React Query for server state:

- products
- categories
- users
- orders
- authentication-related server data

Examples: `useQuery`, `useMutation`, `useQueryClient`.

Do NOT store API data unnecessarily inside Zustand.

### Zustand

Use Zustand only for client-side state. Good examples:

- cart
- UI state
- sidebar state
- temporary client preferences

Do not use Zustand as a replacement for React Query.

### React Hook Form

Use React Hook Form for forms:

- login
- register
- product creation
- product editing
- category creation
- checkout
- profile editing

Use Zod validation where appropriate.

### API Communication

Create a centralized Axios instance:

```
shared/api/
└── axios.ts
```

Do not create a new Axios instance in every component.

API calls should be separated from UI components.

Bad:

```tsx
function Products() {
  axios.get(...)
}
```

Prefer placing API calls in:

```
entities/product/api/
features/auth/api/
```

Components should use hooks or API functions instead of directly calling Axios.

## Backend Architecture

Backend must use:

```
Route
  ↓
Controller
  ↓
Service
  ↓
Prisma
  ↓
Supabase PostgreSQL
```

### Routes

Routes define endpoints and middleware.

```
routes/
├── auth.routes.ts
├── product.routes.ts
├── category.routes.ts
├── order.routes.ts
├── admin.routes.ts
└── user.routes.ts
```

Routes should NOT contain business logic.

```ts
router.post(
  "/",
  authMiddleware,
  roleMiddleware("ADMIN"),
  createProduct
);
```

### Controllers

Controllers handle HTTP-level logic:

- read request
- validate/receive input
- call service
- return response
- handle HTTP status

Controllers should NOT contain complex business logic (no large database/business logic inside controllers).

Prefer:

```ts
export async function createProduct(req: Request, res: Response) {
  const product = await productService.create(req.body);

  res.status(201).json(product);
}
```

### Services

Services contain business logic.

```
services/
├── auth.service.ts
├── product.service.ts
├── category.service.ts
├── order.service.ts
└── user.service.ts
```

Services are responsible for:

- database operations
- business rules
- calculations
- checking permissions/business conditions
- creating orders
- updating order status

```ts
export async function createOrder(userId: string, data: CreateOrderInput) {
  // validate cart
  // calculate total
  // create order
  // create order items
}
```

### Prisma

Prisma is the only database access layer.

Do not write raw SQL unless there is a strong technical reason.

```ts
prisma.product.findMany()
prisma.product.findUnique()
prisma.product.create()
prisma.product.update()
prisma.product.delete()
```

Prisma schema: `backend/prisma/schema.prisma`

Database migrations:

```shell
npx prisma migrate dev
```

Production migration:

```shell
npx prisma migrate deploy
```

Generate Prisma client when necessary:

```shell
npx prisma generate
```

Do not edit generated Prisma files manually.

### Supabase

Supabase is used as the PostgreSQL database provider.

The backend connects to Supabase PostgreSQL through Prisma.

Environment variables must be stored in `.env`:

```shell
DATABASE_URL=
DIRECT_URL=
JWT_ACCESS_SECRET=
JWT_REFRESH_SECRET=
PORT=5000
```

Never commit `.env`.

Never expose database credentials to the frontend.

## Database Models

The main entities are:

```
User
Product
Category
Cart
CartItem
Order
OrderItem
```

Relations must be properly defined in Prisma:

```
User
 ├── Orders
 └── Cart

Category
 └── Products

Order
 └── OrderItems

Product
 ├── Category
 └── OrderItems
```

## Authentication

Use:

- JWT Access Token
- JWT Refresh Token
- bcrypt

Authentication middleware: `authMiddleware`

Role middleware: `roleMiddleware`

```
USER
 ├── products
 ├── cart
 ├── checkout
 ├── orders
 └── profile

ADMIN
 ├── products
 ├── categories
 ├── users
 └── orders
```

Never trust the role sent by the frontend.

The backend must verify the authenticated user's role.

## API

### Auth

```
POST /api/auth/register
POST /api/auth/login
POST /api/auth/refresh
POST /api/auth/logout
GET  /api/auth/profile
```

### Products

```
GET    /api/products
GET    /api/products/:id
POST   /api/products
PATCH  /api/products/:id
DELETE /api/products/:id
```

Creating, editing and deleting products require `ADMIN`.

### Categories

```
GET    /api/categories
POST   /api/categories
PATCH  /api/categories/:id
DELETE /api/categories/:id
```

POST/PATCH/DELETE require `ADMIN`.

### Orders

```
POST  /api/orders
GET   /api/orders
GET   /api/orders/:id
```

Admin:

```
GET   /api/admin/orders
PATCH /api/admin/orders/:id/status
```

### Users

```
GET /api/admin/users
```

Admin only.

## Product Filtering

Products endpoint should support:

- `search`
- `category`
- `minPrice`
- `maxPrice`
- `sort`
- `page`
- `limit`

```
GET /api/products?search=iphone&category=phones&minPrice=100&maxPrice=1000&page=1&limit=12
```

Backend should handle filtering, sorting and pagination.

Do not download all products to the frontend and filter them there.

## Orders

When a user creates an order:

```
Cart
 ↓
Validate products
 ↓
Check stock
 ↓
Calculate total
 ↓
Create Order
 ↓
Create OrderItems
 ↓
Update stock
 ↓
Clear Cart
```

OrderItem must store the product price at the moment of purchase.

Do not rely on the current Product price when displaying historical orders.

## Validation

Use Zod for request validation.

```
schemas/
├── auth.schema.ts
├── product.schema.ts
├── category.schema.ts
├── order.schema.ts
└── user.schema.ts
```

Validate:

- request body
- query parameters
- route parameters where appropriate

Never trust user input.

## Error Handling

Use centralized error handling: `middlewares/error.middleware.ts`.

Controllers should not duplicate large error-handling blocks.

Use meaningful HTTP status codes:

```
200 OK
201 Created
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
500 Internal Server Error
```

Return consistent error responses:

```json
{
  "message": "Product not found"
}
```

## Security Rules

Never:

- store plain-text passwords
- expose JWT secrets
- expose database credentials
- trust frontend roles
- return password hashes
- commit `.env`
- allow admin endpoints without role verification

Passwords must always be hashed with bcrypt.

## Code Style

- Use TypeScript everywhere.
- Prefer `const` over `let` when reassignment is unnecessary.
- Avoid `any` unless absolutely necessary.
- Use meaningful names (`const products = ...`, not `const x = ...`).
- Keep functions small and focused.
- Avoid unnecessary abstractions.
- Do not create complex architecture for simple features.

## Components

Prefer reusable components.

Bad: `HugeProductPage.tsx`

Better:

```
ProductGallery
ProductInfo
ProductActions
RelatedProducts
```

But do not split every small element into a separate component without reason.

## Styling

- Use SCSS/CSS.
- Keep styles close to their components.
- Avoid inline styles unless necessary.
- Do not introduce Tailwind unless explicitly requested.
- Use a consistent design system.

## API Response Rules

Keep API responses predictable.

```json
{
  "data": {}
}
```

For lists:

```json
{
  "data": [],
  "pagination": {
    "page": 1,
    "limit": 12,
    "total": 100,
    "totalPages": 9
  }
}
```

## Development Workflow

Before implementing a feature:

1. Understand the existing architecture.
2. Find related files.
3. Reuse existing components/functions.
4. Check Prisma schema if database changes are needed.
5. Implement backend.
6. Implement frontend.
7. Connect API.
8. Handle loading/error/empty states.
9. Test the feature.
10. Only then refactor if necessary.

Do not rewrite unrelated parts of the project.

## Database Changes

When a database change is required:

1. Update `schema.prisma`.
2. Create a Prisma migration.
3. Generate Prisma client.
4. Update services/types if necessary.
5. Test affected endpoints.

Never change the database schema only manually in Supabase without updating Prisma.

## Important Claude Code Rules

- Before writing code, inspect the existing project structure.
- Do not blindly create duplicate files.
- Do not install new dependencies unless necessary.
- Before installing a package, check whether an existing dependency can solve the problem.
- Do not change the technology stack without explicit permission.
- Do not replace:
  - React Query with another data library
  - React Hook Form with another form library
  - Prisma with another ORM
  - Supabase with another database
  - Mini FSD with another architecture
- Do not introduce Tailwind.
- Do not introduce Redux.
- Do not create unnecessary abstractions.
- Prefer simple, readable solutions.
- When modifying existing code, preserve the current architecture and coding style.

## Feature Implementation Pattern

### Backend

```
route
 ↓
controller
 ↓
service
 ↓
prisma
```

Example: `product.routes.ts`, `product.controller.ts`, `product.service.ts`, `schema.prisma`.

### Frontend

```
page
 ↓
widget / feature
 ↓
React Query
 ↓
API
```

Use Mini FSD layers appropriately.

## Before Finishing Any Task

Check:

- TypeScript errors
- ESLint errors if configured
- API errors
- loading states
- empty states
- error states
- authorization
- responsive behavior
- database relations
- Prisma migration status

Do not say a feature is complete if there are obvious TypeScript/build errors.

## General Principle

The project should feel like a real junior/middle full-stack application.

Priorities:

1. Correct functionality
2. Clean architecture
3. Type safety
4. Security
5. Maintainability
6. Good UI/UX
7. Simplicity

Do not over-engineer the project.

When there are multiple valid solutions, choose the simplest solution that fits the existing architecture.
