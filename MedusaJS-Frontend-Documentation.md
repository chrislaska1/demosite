# MedusaJS B2B Ecommerce Frontend Documentation

## Project Overview

This is a Next.js-based B2B ecommerce frontend that integrates with a MedusaJS backend. The application includes standard ecommerce functionality plus B2B-specific features like company management, employee hierarchies, spending limits, approval workflows, and quote management.

### Tech Stack
- **Frontend**: Next.js 15 with App Router
- **Styling**: Tailwind CSS with @medusajs/ui components
- **Backend Integration**: MedusaJS JS SDK
- **Payments**: Stripe, PayPal
- **Analytics**: Vercel Analytics

## Backend Integration Architecture

### Configuration (`src/lib/config.ts`)
```typescript
import Medusa from "@medusajs/js-sdk"

const MEDUSA_BACKEND_URL = process.env.NEXT_PUBLIC_MEDUSA_BACKEND_URL || "http://localhost:9000"

export const sdk = new Medusa({
  baseUrl: MEDUSA_BACKEND_URL,
  debug: process.env.NODE_ENV === "development",
  publishableKey: process.env.NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY,
})
```

### Middleware & Region Handling (`src/middleware.ts`)
- **Region Detection**: Uses IP geolocation and URL country codes
- **Cart Management**: Handles cart ID cookies and checkout redirects
- **Cache Management**: Implements cache invalidation strategies
- **Authentication**: Manages auth tokens and session state

Key features:
- Automatic region detection via Vercel headers
- Cart persistence across sessions
- Region-specific product catalogs and pricing
- Checkout flow redirects with cart transfer

## Core Ecommerce Features

### 1. Product Catalog (`src/lib/data/products.ts`)

**Key Functions:**
- `getProductByHandle()` - Fetch single product by URL handle
- `listProducts()` - Paginated product listing with filters
- `listProductsWithSort()` - Sorted product listings
- `getProductsById()` - Bulk product fetching

**Features:**
- Region-based pricing and availability
- Variant management with calculated prices
- Inventory quantity tracking
- Product metadata and tagging
- SEO-friendly URL handling

### 2. Shopping Cart (`src/lib/data/cart.ts`)

**Cart Operations:**
- `retrieveCart()` - Get current cart with company/approval context
- `addToCart()` - Add single items
- `addToCartBulk()` - Bulk item addition
- `updateLineItem()` - Modify quantities
- `deleteLineItem()` - Remove items
- `emptyCart()` - Clear all items

**B2B Extensions:**
- Company-specific cart metadata
- Approval workflow integration
- Spending limit validation
- Cart-to-CSV export functionality

### 3. Checkout Process (`src/modules/checkout/`)

**Checkout Steps:**
1. **Contact Details** - Customer info and B2B metadata
2. **Shipping Address** - Delivery information
3. **Billing Address** - Payment address
4. **Shipping Methods** - Delivery options
5. **Payment** - Multiple payment providers
6. **Review** - Order confirmation

**B2B Checkout Features:**
- Invoice recipients and cost centers
- Requisition number tracking
- Door codes and delivery notes
- Approval workflow triggers

### 4. Order Management (`src/lib/data/orders.ts`)

**Order Operations:**
- `retrieveOrder()` - Fetch order details with items/payments
- `listOrders()` - Customer order history
- Order tracking and status updates
- Payment collection management

### 5. Payment Integration (`src/lib/data/payment.ts`)

**Payment Providers:**
- Stripe integration with React components
- PayPal integration
- Region-specific payment method filtering
- Payment session management

## B2B-Specific Features

### 1. Company Management (`src/lib/data/companies.ts`)

**Company Operations:**
- `createCompany()` - Company registration
- `updateCompany()` - Company profile management
- `retrieveCompany()` - Fetch company details with employees
- `updateApprovalSettings()` - Configure approval workflows

**Company Features:**
- Employee management and hierarchies
- Spending limits with reset frequencies
- Admin approval requirements
- Company-wide settings and branding

### 2. Employee Management

**Employee Operations:**
- `createEmployee()` - Add team members
- `updateEmployee()` - Modify employee details
- `deleteEmployee()` - Remove team members
- Role-based permissions (admin vs. employee)

**Employee Features:**
- Individual spending limits
- Company association
- Customer profile linking
- Permission-based UI rendering

### 3. Approval Workflows (`src/lib/data/approvals.ts`)

**Approval Process:**
- Cart approval requirements
- Admin notification system
- Approval status tracking
- Order approval before completion

**Approval Types:**
- Spending limit exceeded
- Admin-required purchases
- Custom approval rules

### 4. Quote Management (`src/lib/data/quotes.ts`)

**Quote Operations:**
- `createQuote()` - Generate quote from cart
- `fetchQuotes()` - List customer quotes
- `acceptQuote()` - Convert quote to cart
- `rejectQuote()` - Decline quote
- `createQuoteMessage()` - Quote communication

**Quote Features:**
- Quote-to-order conversion
- Negotiation messaging
- Quote expiration handling
- Admin quote management

## Data Layer Architecture

### Authentication & Authorization
**File**: `src/lib/data/customer.ts`

**Key Functions:**
- `signup()` - B2B customer registration with company creation
- `login()` - Authentication with cart transfer
- `signout()` - Session cleanup
- `retrieveCustomer()` - Get authenticated customer with employee data

**Features:**
- Employee-company association
- Cart transfer on login
- Role-based access control
- Company creation during signup

### Cookie Management
**File**: `src/lib/data/cookies.ts`

**Cookie Types:**
- `_medusa_cart_id` - Cart persistence
- `_medusa_cache_id` - Cache invalidation
- Authentication tokens
- Region preferences

### Cache Strategy
- **Force-cache** for static content
- **Revalidate tags** for dynamic updates
- **Region-specific** cache keys
- **Company-scoped** cache invalidation

## Frontend Component Structure

### Module Organization
```
src/modules/
├── cart/           # Cart UI components
├── checkout/       # Multi-step checkout flow
├── products/       # Product display components
├── account/        # Customer account management
├── companies/      # B2B company management
├── quotes/         # Quote management UI
├── orders/         # Order history and tracking
└── layout/         # Common layout components
```

### Key Component Categories

**Cart Components:**
- `cart-drawer/` - Slide-out cart interface
- `cart-totals/` - Price calculations
- `approval-status-banner/` - B2B approval indicators
- `cart-to-csv-button/` - Export functionality

**Checkout Components:**
- `payment/` - Payment method selection
- `shipping-address-form/` - Address collection
- `company-form/` - B2B checkout fields
- `review/` - Order confirmation

**Product Components:**
- Product listings with B2B pricing
- Variant selection interfaces
- Bulk order capabilities
- Company-specific product access

## Key Workflows

### 1. Customer Registration & Company Setup
1. Customer fills registration form
2. System creates customer account
3. Company entity is created automatically
4. Employee record links customer to company
5. Admin permissions assigned to first employee
6. Cart metadata updated with company ID

### 2. B2B Checkout Flow
1. Cart validation against spending limits
2. Company information collection
3. Approval workflow triggering (if required)
4. Payment processing with B2B metadata
5. Order creation with company association
6. Invoice and cost center tracking

### 3. Quote-to-Order Process
1. Customer creates quote from cart
2. Admin reviews and potentially modifies quote
3. Quote communication via messaging
4. Customer accepts/rejects quote
5. Accepted quote converts to cart
6. Standard checkout flow proceeds

### 4. Approval Workflow
1. Cart exceeds spending limits or requires approval
2. System creates approval request
3. Admin notification triggered
4. Approval status tracked in cart
5. Approved carts proceed to payment
6. Rejected carts return to customer

## Implementation Guide for Astro Migration

### 1. Backend Integration Setup

**SDK Configuration:**
```javascript
// lib/medusa.js
import Medusa from "@medusajs/js-sdk"

export const sdk = new Medusa({
  baseUrl: import.meta.env.PUBLIC_MEDUSA_BACKEND_URL,
  publishableKey: import.meta.env.PUBLIC_MEDUSA_PUBLISHABLE_KEY,
})
```

**Environment Variables:**
```
PUBLIC_MEDUSA_BACKEND_URL=http://localhost:9000
PUBLIC_MEDUSA_PUBLISHABLE_KEY=pk_...
MEDUSA_PUBLISHABLE_KEY=pk_...
```

### 2. Data Fetching Patterns

**Server-Side Data Fetching:**
```javascript
// pages/products/[handle].astro
---
import { getProductByHandle } from '../lib/data/products.js'

const { handle } = Astro.params
const product = await getProductByHandle(handle, 'us')
---
```

**Client-Side State Management:**
- Use Astro's client directives for interactive components
- Implement cart state with localStorage and reactive stores
- Handle authentication state across page loads

### 3. Authentication Implementation

**Server Middleware:**
```javascript
// middleware.js
export async function onRequest(context, next) {
  // Handle region detection
  // Manage cart cookies
  // Authenticate requests
  return next()
}
```

**Client-Side Auth:**
```javascript
// stores/auth.js
import { persistentAtom } from '@nanostores/persistent'

export const authToken = persistentAtom('auth-token', '')
export const customer = atom(null)
```

### 4. Component Architecture

**Astro Component Structure:**
```
src/
├── components/
│   ├── cart/              # Cart components
│   ├── checkout/          # Checkout flow
│   ├── products/          # Product displays
│   └── ui/                # Reusable UI components
├── layouts/
│   ├── BaseLayout.astro   # Main layout
│   └── StoreLayout.astro  # Ecommerce layout
├── pages/
│   ├── [countryCode]/     # Region-based routing
│   └── api/               # API endpoints
└── lib/
    ├── data/              # Backend integration
    ├── stores/            # State management
    └── utils/             # Helper functions
```

### 5. Key Migration Considerations

**Routing:**
- Implement country code routing in Astro
- Handle dynamic routes for products/categories
- Set up proper redirects and SEO

**State Management:**
- Replace React hooks with Astro-compatible solutions
- Use nanostores for reactive state
- Implement proper cart persistence

**Styling:**
- Migrate Tailwind configuration
- Replace @medusajs/ui components with Astro equivalents
- Ensure responsive design consistency

**Build Process:**
- Configure Astro for SSR/SSG hybrid
- Set up proper environment handling
- Implement cache strategies

### 6. Critical Features to Implement

**Must-Have Features:**
1. Product catalog with filtering/search
2. Shopping cart with persistence
3. Multi-step checkout process
4. Customer authentication
5. Order management
6. Payment integration

**B2B Features:**
1. Company registration and management
2. Employee hierarchies and permissions
3. Approval workflows
4. Quote management
5. Spending limits
6. Bulk ordering capabilities

**Additional Considerations:**
1. SEO optimization for product pages
2. Performance optimization for large catalogs
3. Mobile-responsive design
4. Accessibility compliance
5. Analytics integration
6. Error handling and logging

## API Endpoints Used

### Core Ecommerce APIs
- `GET /store/products` - Product listings
- `GET /store/products/{id}` - Product details
- `POST /store/carts` - Create cart
- `GET /store/carts/{id}` - Retrieve cart
- `POST /store/carts/{id}/line-items` - Add to cart
- `POST /store/carts/{id}/complete` - Place order
- `GET /store/orders` - Order history
- `GET /store/regions` - Available regions
- `GET /store/payment-providers` - Payment methods

### B2B-Specific APIs
- `POST /store/companies` - Create company
- `GET /store/companies/{id}` - Company details
- `POST /store/companies/{id}/employees` - Manage employees
- `POST /store/quotes` - Create quote
- `GET /store/quotes` - List quotes
- `POST /store/quotes/{id}/accept` - Accept quote
- Custom approval workflow endpoints

This documentation provides a comprehensive foundation for recreating the MedusaJS B2B ecommerce functionality in Astro while maintaining the same backend integration patterns and business logic.