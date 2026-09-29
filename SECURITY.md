# SaveIQ security design

SaveIQ is a demo-ready full-stack application. Before production launch, the owner should review the dependency tree, complete a threat model, configure a real secret manager, and replace placeholder legal copy.

## Passwords

Email addresses are normalized before lookup. Passwords are never stored in plaintext. SaveIQ hashes passwords with `bcryptjs` using cost 12 and a unique salt per password. An optional `SESSION_PEPPER` can add a server-side secret to token hashing. Registration requires at least 10 characters, upper/lowercase letters, a number, a symbol, and rejects a small common-password list. Login returns the same `Invalid email or password` message for malformed credentials, unknown accounts, and wrong passwords.

## Sessions

Login creates a 32-byte cryptographically random token. Only a SHA-256 hash of the token is retained server-side. The raw token is delivered in an HttpOnly cookie named `saveiq_session`, with Secure enabled for HTTPS and SameSite=Lax. Sessions have a 30-minute idle timeout and seven-day absolute expiry. A successful login revokes existing active sessions and creates a fresh session. Logout and password changes revoke sessions. The Active Sessions page shows IP, user agent, last activity, and expiry without exposing tokens.

## CSRF and headers

A double-submit CSRF token is issued through `/api/csrf` in a non-HttpOnly cookie. Every state-changing request must send the same value as `x-csrf-token`. The server applies `X-Content-Type-Options: nosniff`, a restrictive Referrer Policy, Permissions Policy, CSP, and HSTS when the request is HTTPS. The CSP frame-ancestors list includes the supported Cloud Preview ancestors; it intentionally does not use `X-Frame-Options: DENY` because the managed Preview is an iframe.

## Abuse controls

Signup, login, password reset, and account-keyed requests use IP/account rate limits that are persisted with the application state snapshot. Failed logins increment a per-user failure counter; five failures trigger a 15-minute temporary lock. A multi-replica production deployment should move the counters to an atomic shared rate-limit service so concurrent increments are coordinated across replicas.

## Authorization and IDOR prevention

Protected routes require a valid server-side session. Every goal, transaction, session, audit, export, and account operation is scoped to the authenticated user from the session context. Client-provided user IDs are not trusted. Unknown or foreign goal/session identifiers return a generic not-found response instead of leaking existence.

## Password resets and audit logs

Reset tokens are 32-byte random values; only hashes are stored, tokens expire after 30 minutes, and a token is single-use. In development, the reset URL is logged to the server console instead of being emailed. Security events include signup, login success/failure, logout, password changes/resets, goal mutations, session revocation, and account deletion. The user can view their own audit history.

## Data deletion and exports

Settings provides JSON and CSV export. Account deletion requires the current password, revokes sessions, deletes goals and transactions, clears reset and audit records, and removes the user from the durable state snapshot. The Drizzle schema includes cascading foreign keys for the normalized managed database model.

## AI boundary

Only a minimum goal/transaction summary is sent to the server-side LLM helper when configured. Provider keys are read server-side and never embedded in client code. Structured output is validated with Zod. If the provider is unavailable or returns invalid output, a deterministic rule-based fallback keeps the feature usable. The product states: `SaveIQ provides educational insights, not financial advice.`

## Deferred scope

TOTP 2FA is intentionally deferred in this version in favor of completing the required password/session security controls. It should be added before a high-risk production launch if the product later stores sensitive financial data.
