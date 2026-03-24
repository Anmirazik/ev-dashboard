# ev-dashboard

Angular 14 frontend for the Open e-Mobility EV charging infrastructure platform.
Connects to **ev-server** REST API. Part of the monorepo at `../`.

## Commands

```bash
npm start              # Dev server at http://localhost:4200
npm run start:ssl      # Dev with SSL (needs ./ssl/ certs)
npm run build:prod     # Production build → dist/
npm run eslint         # Lint
npm run eslint:fix     # Lint + auto-fix
npm test               # Jest + Puppeteer e2e
```

## Backend Connection

Config lives in `src/assets/config.json` — **now committed to the repo**, pre-configured for local Docker setup. No manual copy needed.

Local dev setup (ev-server on Docker):
- protocol: `http`, host: `localhost`, port: `8081`
- `captchaSiteKey`: Google test reCAPTCHA site key (paired with the test secret key in `ev-server/docker/config.json`)

Default logins (seeded automatically on fresh Docker start, or insert manually if missing):
- Super admin: `super.admin@ev.com` / `Super.admin00` → http://localhost:3080
- SLF tenant: `slf.admin@ev.com` / `Slf.admin00` → http://slf.localhost:3080

Full clean restart of ev-server (wipes everything including mongo data — re-seeds users):
```bash
cd ../ev-server/docker && make clean && make clean-mongo-data && docker network prune -f && make SUBMODULES_INIT=false
```

## Architecture

```
src/app/
  authentication/   # Login, signup, password reset
  pages/            # Feature pages (charging stations, users, etc.)
  services/         # API service layer
  shared/           # Reusable components/dialogs
  types/            # TypeScript interfaces
  utils/            # Helpers
src/assets/
  config.json           # Runtime config (committed — pre-configured for local Docker)
  config-template.json  # Template for reference / custom environments
  i18n/                 # Translation files (en, fr, de, it, es, pt, cs)
```

## Gotchas

- `src/assets/config.json` is committed and pre-configured for local Docker — no copy needed. For production or custom environments, edit it or base a new one off `config-template.json`
- Node 16.x and npm 8.x required (see `engines` in package.json)
- `npm start` calls `npm version` first — expects a clean or versioned state
- SSL dev mode needs certs at `./ssl/localhost.key` and `./ssl/localhost.cert`

---

## Cross-Repo Connection: ev-dashboard ↔ ev-server

ev-dashboard is a pure REST client of ev-server. It has **no WebSocket** — it polls HTTP every `pollIntervalSecs` seconds (default 10s in `config.json`).

### Config Alignment (must match on both sides)

| ev-dashboard `config.json` key | ev-server `config.json` key | Purpose |
|-------------------------------|------------------------------|---------|
| `CentralSystemServer.protocol` | `CentralSystemRestService.protocol` | http or https |
| `CentralSystemServer.host` | `CentralSystemRestService.host` | Backend hostname |
| `CentralSystemServer.port` | `CentralSystemRestService.port` | Backend port (Docker: 8081) |
| `User.captchaSiteKey` | `CentralSystemRestService.captchaSecretKey` | reCAPTCHA — must be a matching Google pair |

### Key API Service File

**`src/app/services/central-server.service.ts`** (~3670 lines) — single file that contains every REST call. Organized as:
- `restServerAuthURL` → `/v1/auth/*` (login, register, password reset)
- `restServerSecuredURL` → `/v1/api/*` (all protected endpoints, Bearer token required)
- `restServerServiceUtilURL` → `/v1/util/*` (health check, constants — no auth)

### Endpoint Constants

All URL paths are stored as `RESTServerRoute` enum in **`src/app/types/Server.ts`**.

**If ev-server adds/renames/removes a route:**
1. Update `RESTServerRoute` enum in `src/app/types/Server.ts`
2. Update the corresponding method in `central-server.service.ts`
3. Search for all callers in `src/app/pages/` and `src/app/shared/`

### JWT Token

ev-dashboard decodes the JWT returned by ev-server but **never signs it**. The token shape is defined in `UserToken` (`src/app/types/User.ts`).

**Current claims consumed:**
```typescript
interface UserToken {
  tenantID: string;
  userID: string;
  role: string;        // 'S' | 'A' | 'B' | 'D'
  currency: string;
  language: string;
  locale: string;
}
```

Token is stored in `localStorage` under key `'token'` and injected as `Authorization: Bearer <token>` plus a `Tenant: <tenantID>` header on every secured request.

**If ev-server changes JWT claims:**
→ Update `UserToken` interface in `src/app/types/User.ts`
→ Update `loginSucceeded()` and claim accessors in `central-server.service.ts`

### Shared Data Models (duplicated — no shared package)

Types are copied between repos. Keep them in sync manually.

| ev-dashboard `src/app/types/` | ev-server `src/types/` |
|---|---|
| `ChargingStation.ts` | `ChargingStation.ts` |
| `Transaction.ts` | `Transaction.ts` |
| `User.ts` | `User.ts` |
| `Tag.ts` | `Tag.ts` |
| `Asset.ts` | `Asset.ts` |
| `Billing.ts` | `Billing.ts` |
| `Car.ts` | `Car.ts` |
| `Authorization.ts` | `Authorization.ts` |

**If ev-server adds/renames/removes a field on a model:**
→ Apply the same change to the matching file in `src/app/types/`
→ Run `npm run eslint` — type errors will point to affected components

### User Roles

| ev-server role char | ev-dashboard usage |
|---|---|
| `S` | `UserRole.SUPER_ADMIN` |
| `A` | `UserRole.ADMIN` |
| `B` | `UserRole.BASIC` |
| `D` | `UserRole.DEMO` |

Roles are checked in `src/app/services/authorization.service.ts` and route guards at `src/app/guard/route-guard.ts`.

**If ev-server adds a new role:**
→ Add to `UserRole` enum in `src/app/types/User.ts`
→ Update `authorization.service.ts` permission checks
→ Update `route-guard.ts` if route access needs to be role-restricted

### Response Envelope

ev-server sends list responses as `{ count: number, result: T[] }`. The table/data-source components in `src/app/shared/table/` hardcode this shape. Do not change it without updating both sides.

### What to check when making changes

| You change this in ev-dashboard | Check in ev-server |
|---|---|
| Add/call a new API endpoint | Add the route + handler in `src/server/rest/v1/` |
| Change a field sent in a request body | Update the request validator in `src/validator/` |
| Change `CentralSystemServer` port/host | Must match `CentralSystemRestService` in ev-server config |
| Change reCAPTCHA site key | Must be paired with ev-server's `captchaSecretKey` |
| Update a type in `src/app/types/` | Apply matching change in `ev-server/src/types/` |
