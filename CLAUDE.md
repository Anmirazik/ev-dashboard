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

Config lives in `src/assets/config.json` (gitignored — copy from `config-template.json`).

Local dev setup (ev-server on Docker):
- protocol: `http`, host: `localhost`, port: `8081`

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
  config.json           # Runtime config (gitignored)
  config-template.json  # Template to copy from
  i18n/                 # Translation files (en, fr, de, it, es, pt, cs)
```

## Gotchas

- `src/assets/config.json` must exist before running — not committed, copy from template
- Node 16.x and npm 8.x required (see `engines` in package.json)
- `npm start` calls `npm version` first — expects a clean or versioned state
- SSL dev mode needs certs at `./ssl/localhost.key` and `./ssl/localhost.cert`
