# SaveIQ — Save smarter. Reach goals faster.

SaveIQ is a full-stack savings goal tracker built with React, TypeScript, Tailwind CSS, Express, Drizzle, MySQL-compatible schema definitions, Recharts, and Zod. It helps people set goals, record contributions, see progress, and receive educational pacing insights.

## Run locally

```bash
pnpm install
cp .env.example .env
pnpm dev
```

The development server listens on `http://localhost:3000` by default. The managed Webdev Preview uses the same port and requires the process to listen on `0.0.0.0`.

Useful commands:

```bash
pnpm check          # TypeScript validation
pnpm test           # Vitest suite
pnpm build          # Vite client build + bundled Express server
pnpm start          # Serve the production bundle
pnpm db:generate    # Generate a Drizzle migration from schema changes
pnpm db:migrate     # Apply checked-in migrations
 pnpm db:seed        # Write the demo account and sample goals to the configured database
```

## Demo account

The development server seeds a demo user at startup:

- Email: `demo@saveiq.app`
- Password: `SaveIQdemo2026!`

This credential is for local/demo use only. Change it before using any deployment outside the demo environment.

## Environment variables

See `.env.example`. The important values are `DATABASE_URL`, `SESSION_SECRET`, optional `SESSION_PEPPER`, `PUBLIC_ORIGIN`, `COOKIE_SECURE`, and the managed LLM values delivered by the platform. Provider credentials remain server-side.

## Architecture

`client/src/App.tsx` contains the responsive public marketing pages, auth screens, protected application shell, dashboard, goals, insights simulator, and settings/security surfaces. `server/_core/index.ts` owns the Express API, cookie sessions, CSRF, rate limiting, lockout, password reset, goal/transaction invariants, exports, audit logs, and deterministic insight fallback. `shared/saveiq.ts` holds Zod contracts and shared types. `drizzle/schema.ts` describes the durable MySQL tables and cascading relationships. The project serves `client/public/manus-routes.json` at the origin root.

The API uses a Drizzle/MySQL-backed state snapshot for the running demo and restores sessions, goals, transactions, audit records, reset records, rate-limit state, and insight cache entries across process restarts. The normalized Drizzle tables and checked-in migrations remain the durable data model for extracting high-volume queries into per-table repositories as the product scales horizontally.

## Security

Read [`SECURITY.md`](./SECURITY.md) for the auth design, password hashing, session handling, CSRF, security headers, rate limits, authorization, audit log, reset token, export, and deletion details. SaveIQ provides educational insights, not financial advice.

## Product assumptions

Pricing is a placeholder and does not collect payments. The privacy and terms pages are placeholders for legal review. TOTP 2FA is deferred in favor of the requested password/session security controls. Canonical and Open Graph absolute URLs should be configured with an explicit public origin before production publication.
