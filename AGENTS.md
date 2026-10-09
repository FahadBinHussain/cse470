# TaskFlow project notes

next.js 16 + typescript + tailwind v4 + drizzle + neon serverless postgres task/project manager. full-stack: next route handlers as the api, jwt auth (bcrypt + jsonwebtoken), nodemailer for reset emails (ethereal test by default).

## stack (per global defaults)
- **github repo name (2026-10-06):** canonical is `FahadBinHussain/cse470` (was `taskflow` earlier — the name got changed on the remote). a stale remote URL like `taskflow.git` keeps "working" only because github 301-redirects old repo names after a rename; don't rely on it. sync check: `gh repo view <owner>/<name> --jq .name` shows the true name. local `origin` set to `cse470.git` (verified 2026-10-06, `git fetch` clean, 0 ahead/behind).
- pnpm for everything (repo was npm initially; package-lock.json dropped, pnpm-lock.yaml used). do not reintroduce npm.
- `pnpm install` builds esbuild + unrs-resolver. if pnpm 11 says "ignored build scripts", the allowlist lives in `pnpm-workspace.yaml` under `onlyBuiltDependencies` (the old `package.json > pnpm` field is ignored by pnpm 11). the file also carries `minimumReleaseAgeExclude` for next 16.3.4-era packages, otherwise install fails on fresh releases.
- scripts: `pnpm dev` / `build` / `start` / `lint`, `pnpm db:push`, `pnpm db:studio`, `pnpm db:seed`.

## database
- neon project `cse470` (id `wild-night-29488804`, renamed from `taskflow` 2026-10-07 — id unchanged so DATABASE_URL/connection string unaffected) on the owning account's org (`org-round-voice-32039494`, region aws-ap-southeast-1). this account was 0-projects before this app.
- `drizzle.config.ts` and `src/db/seed.ts` load `.env.local` first then `.env` (dotenv), so env-sync restores work: `automata-private\bitwarden.com\env-sync.ps1 -Repo cse470` writes `.env.local` from the vault item `github.com/FahadBinHussain/cse470 / .env (development)` (renamed from `.../taskflow` 2026-10-07 when repo+folder took the cse470 name).
- `.env` / `.env.local` are gitignored; never commit them. real DATABASE_URL lives in the vault (rule 41).
- schema: users, password_resets, projects, project_members, project_invites, tasks, notifications, flagged_content. drizzle-kit push is the migration path (no drizzle migration files generated so far).
- seed creates 5 users (password `taskflow123` for all, admin = admin@taskflow.app), 4 projects, members, 6 tasks, 2 notifications.

## neon api gotcha (learned 2026-09-01)
creating a neon project under an org-scoped api key: the `org_id` must go INSIDE the `project` body object, not as a `?org_id=` query param (query param + top-level body both return 400 "org_id is required"). working create:
```powershell
$body = @{ project = @{ name='taskflow'; region_id='aws-ap-southeast-1'; pg_version=17; org_id='org-round-voice-32039494' } } | ConvertTo-Json -Depth 3
Invoke-WebRequest -Uri 'https://console.neon.tech/api/v2/projects' -Method Post -Headers $headers -Body $body -ContentType 'application/json'
```
connection_uri comes back in the 201 response (connection_uris[0].connection_uri). get the api key from the vault (`Read-VaultSecret -Email <neon-account-email> -NamePattern 'console.neon.tech*'`).
