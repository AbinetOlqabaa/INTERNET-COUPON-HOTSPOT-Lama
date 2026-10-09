# Changelog and Evidence

## Authentication, Multi-User RBAC, Administration & AI Telemetry Foundation (Completed)
- Built authentication & session security engine in `server/src/auth/service.ts`:
  - PBKDF2/SHA-512 password hashing with random 16-byte salt, 10,000 iterations, 64-byte key length.
  - Timing-safe comparison preventing timing leakage attacks.
  - 24-hour cryptographically secure random session tokens with immediate invalidation upon logout, password change, password reset, or account deactivation.
  - In-memory progressive anti-automation rate limiter: enforces 15-minute lockout after 5 consecutive failed login attempts.
  - Password lifecycle: secure authenticated password change (`POST /api/v1/auth/change-password`), non-enumerating forgot-password request (`POST /api/v1/auth/forgot-password`), 15-minute single-use SHA-256 hashed reset token validation (`POST /api/v1/auth/reset-password`).
  - First-run administrator bootstrap (`POST /api/v1/admin/bootstrap`): initializes initial `SUPER_ADMIN` safely when no administrator exists, locking permanently after initialization.
- Built Role-Based Access Control & User Administration in `server/src/admin/routes.ts`:
  - Four distinct roles: `SUPER_ADMIN`, `OWNER`, `STAFF`, `CUSTOMER`.
  - Server-enforced role middleware (`createRoleMiddleware`).
  - Safeguard protection: system strictly prohibits deactivating, demoting, or deleting the last active `SUPER_ADMIN`.
  - User CRUD operations: list with search, role filters, status filters, and pagination; view user; create staff/owner; update user; activate and deactivate accounts with immediate session revocation; and delete user.
  - Administrative overview metrics (`GET /api/v1/admin/overview`): total users, active/inactive distribution, role distribution, system uptime, and rate-limiting status.
  - Tamper-evident administrative audit trail (`GET /api/v1/admin/audit-logs`).
  - AI operational telemetry foundation (`GET /api/v1/admin/ai/insights`): aggregates security posture and operational findings without leaking secrets or raw credentials.
- Built responsive operator UI in `client/src/App.tsx`, `client/src/components/AuthView.tsx`, and `client/src/components/LogoutModal.tsx`:
  - Polished authentication screen supporting Sign In, Register, Forgot Password, Reset Password, and Admin Bootstrap.
  - Password visibility toggle (`showPassword` state with accessible show/hide buttons).
  - Responsive collapsible sidebar (desktop collapse between 250px and 68px, mobile sliding drawer with backdrop).
  - Role-aware navigation links hiding admin sections from unauthorized users.
  - Accessible modal dialog for logout confirmation with reliable backend session revocation.
- Added database migration `002_user_management_and_roles.sql` defining role fields, display name, owner foreign key, and `password_reset_tokens` table.
- Added comprehensive unit and integration test suites:
  - `server/src/auth/password_lifecycle.test.ts` (4 tests)
  - `server/src/admin/admin.test.ts` (6 tests)
  - `client/src/theme.test.ts` (5 tests)
- Evidence: 57/57 tests passing across 12 test suites; zero TypeScript errors; clean production build.

## Phase 7 — Payment Adapters, Sandbox, Signed Webhooks, Manual Payments, Ledger & Reconciliation (Completed)
- Built extensible payment adapter layer in `server/src/payments/`:
  - `SandboxPaymentAdapter`: simulated tokenized checkout URLs, immediate success simulation, deterministic failure and 3DS challenge simulations.
  - `ManualCashPaymentAdapter`: front-desk on-premise cash collection producing tamper-evident receipt numbers (`CASH-YYYYMMDD-XXXXXX`).
- Built payment intent management in `server/src/payments/service.ts`:
  - Strict idempotency key handling: returns cached intent on duplicate calls; rejects with HTTP 409 if conflicting amounts or currencies are submitted.
  - Integration with general ledger: automatic posting of sales entries upon completion.
  - Automatic session state coordination: transitions linked session to `payment_verified`.
- Built replay-protected signed webhook processing:
  - Validates HMAC-SHA256 signatures with 300-second timestamp tolerance window to block replay attacks.
  - Validates amount and currency match against intent to prevent tampering anomalies.
- Built general ledger and reconciliation reporting:
  - `GET /api/v1/payments/ledger`: double-entry tracking of sales, refunds, and adjustments with net balance calculation.
  - `GET /api/v1/payments/reconcile`: audits payment intents against general ledger entries, confirming balanced status and detecting discrepancies.
  - `POST /api/v1/payments/:id/refund`: executes full or partial refunds with negative ledger debits, session state coordination, and duplicate refund prevention.
  - `POST /api/v1/payments/:id/dispute`: flags dispute states on payment intents and sessions.
- Added unit and integration test suite:
  - `server/src/payments/payments.test.ts` (10 tests)

## Phase 6 — Packages, Pricing Snapshots, Coupons & Abuse Controls (Completed)
- Built access package management in `server/src/packages/routes.ts`:
  - `POST /api/v1/packages` and `GET /api/v1/packages`.
  - Immutable historical price snapshot safety: updating package prices does not retroactively alter purchased sessions.
- Built voucher & coupon management in `server/src/coupons/routes.ts`:
  - `POST /api/v1/coupons` with uppercase normalization, usage limits, and expiration bounds.
  - `POST /api/v1/coupons/validate`: public redemption verification endpoint checking active status, expiry, and usage limits before returning package snapshot.
- Added comprehensive unit and integration tests:
  - `server/src/packages/packages.test.ts` (3 tests)
  - `server/src/coupons/coupons.test.ts` (3 tests)

## Phase 5 — Customers, Consent, Device/Session History, Search & Pagination (Completed)
- Built customer management in `server/src/customers/routes.ts`:
  - `POST /api/v1/customers` with terms consent and data retention tracking.
  - `GET /api/v1/customers` with multi-field search (phone, name, MAC) and pagination (`limit`, `offset`, `totalCount`, `hasMore`).
  - `GET /api/v1/customers/:id` returning customer details and complete historical session list.
  - `PUT /api/v1/customers/:id` for profile and consent updates.
  - `DELETE /api/v1/customers/:id` for privacy-compliant deletion.
- Added unit and integration tests in `server/src/customers/customers.test.ts` (4 tests).

## Phase 4 — Owner Profile, Settings, Capability Status & Dashboard Shell (Completed)
- Created owner routes in `server/src/owner/routes.ts`:
  - `GET /api/v1/owner/profile` (profile retrieval)
  - `PUT /api/v1/owner/profile` (profile update)
  - `GET /api/v1/owner/dashboard` (metrics: packages, active sessions, total revenue, customer count, and honest gateway mode disclosure)
- Mounted `/api/v1/owner` protected by `authMiddleware` in `server/src/app.ts`.
- Added unit and integration tests in `server/src/owner/owner.test.ts` (2 tests).

## Phase 3 — Authentication, Tenant Isolation, Permissions & Audit (Completed)
- Implemented `AuthService` in `server/src/auth/service.ts`:
  - PBKDF2/SHA-512 password hashing with cryptographically random per-user salt and timing-safe comparison.
  - Bearer session token issuance, validation, and revocation on logout.
- Created auth routes in `server/src/auth/routes.ts` (`/register`, `/login`, `/me`, `/logout`) and `createAuthMiddleware`.
- Added test suite in `server/src/auth/auth.test.ts` (5 tests).

## Phase 2 — Relational Schema, Repositories, Constraints & Backup Plan (Completed)
- Authored initial PostgreSQL 16 schema in `server/src/db/migrations/001_initial_schema.sql`.
- Created Zod entity schemas in `server/src/db/schema.ts`.
- Built `MemoryDatabase` repository registry in `server/src/db/repositories.ts`.
- Added unit test suite in `server/src/db/db.test.ts` (6 tests).

## Phase 1 — Product Contracts, Currency Math, Gateway Matrix & Privacy (Completed)
- Authored formal domain contracts in `server/src/contracts/index.ts`.
- Added contract unit test suite in `server/src/contracts/contracts.test.ts` (7 tests).

## Phase 0 — Kernel Baseline Audit and Repair (Completed)
- Baseline tests verified. Unified server runtime in `server.ts` bridges Express API (`/api/v1/*`) and Vite client on port 3000.
