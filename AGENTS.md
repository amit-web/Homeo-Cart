# Project: MedMart (PharmEasy-inspired pharmacy platform)

## Stack
- Frontend (client + admin): React (Vite), Tailwind CSS, Redux Toolkit + RTK Query, react-router-dom v6
- Backend: Node.js, Express, MongoDB (Mongoose)
- Auth: JWT (short-lived access token + httpOnly refresh cookie)
- Payments: Razorpay (test mode)
- File storage: Cloudinary (prescription images)

## Folder Structure (do not deviate from this)
```
medmart/
├── client/src/{api,components,pages,store,hooks,utils}
├── admin/src/{api,components,pages,store,hooks,utils}
├── server/src/{config,models,controllers,routes,middleware,services,utils}
```

## Conventions
- Controllers contain only request/response handling. Business logic goes in `services/`.
- Every POST/PUT route must validate input with Zod before touching the DB.
- Every protected route must use `authMiddleware` + `roleMiddleware('role')` — never rely on frontend-only role checks.
- Mongoose queries that are read-only lists must use `.lean()`.
- Never trust price, quantity, or stock values sent from the client — always recompute/verify server-side from the DB record.
- All list endpoints must support pagination (`page`, `limit` query params) — never return unbounded arrays.
- Payment verification (Razorpay signature) happens server-side only, using HMAC SHA256. Never mark an order "paid" based on a frontend callback alone.
- Roles: `customer`, `pharmacist`, `admin`. Use `roleMiddleware` checks matching these exact strings.

## Core Schemas (use these exact field names — don't invent variations)
- User: name, email, phone, passwordHash, role, addresses[], createdAt
- Medicine: name, genericName, manufacturer, category, requiresPrescription, composition, mrp, sellingPrice, discountPercent, stock, images[], description, isActive
- Prescription: userId, imageUrls[], status (pending/approved/rejected), reviewedBy, rejectionReason, linkedOrderId
- Order: userId, items[], prescriptionId, totalAmount, deliveryAddress, paymentStatus, paymentMeta, orderStatus, statusHistory[]

## Business Rules
- If cart contains any medicine with `requiresPrescription: true`, checkout must be blocked server-side until an `approved` Prescription is linked to the order.
- Order status transitions must be appended to `statusHistory` with a timestamp, not just overwritten.

## Style
- Functional React components only, with hooks. No class components.
- Tailwind utility classes only — no inline styles, no separate CSS files unless absolutely necessary.
- Async/await everywhere — no raw `.then()` chains in new code.
- Wrap all async controller functions in a try/catch or an `asyncHandler` wrapper — never let unhandled promise rejections crash the server.
