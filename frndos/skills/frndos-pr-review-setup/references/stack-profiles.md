# Stack Profiles

Registry of supported stacks. Each profile supplies the values the templates need.

To add a new profile: add a section below + reference it from `SKILL.md` step 1 of `+review-md`.

---

## `php-laravel`

**Detection:** `composer.json` exists + `laravel/framework` in `require`.

| Placeholder | Value |
|-------------|-------|
| `{{STACK}}` | `Laravel 13 · PHP 8.4 · Pest 4 · Pint · Sanctum/JWT · Spatie Data` (adjust versions from `composer.json`) |
| `{{STYLE_TOOL}}` | `Pint` |
| `{{TYPE_TOOL}}` | `PHPStan/Larastan` |
| `{{TEST_TOOL}}` | `Pest 4` |
| `{{AUTH_PATHS}}` | `app/Http/Controllers/Auth/*`, `app/Services/Authentication*`, `config/auth.php`, `config/sanctum.php`, `config/jwt.php`, `routes/auth.php` |
| `{{SURFACE_AREA}}` | routes (`routes/api.php`, `routes/web.php`, `routes/auth.php`, `routes/channels.php`) · controllers (`app/Http/Controllers/**`) · models (`app/Models/**`) · services/actions/repositories (`app/Services/**`, `app/Actions/**`, `app/Repositories/**`) · jobs/events/listeners/observers (`app/Jobs/**`, `app/Events/**`, `app/Listeners/**`, `app/Observers/**`) · migrations (`database/migrations/**`) · resources/DTOs (`app/Http/Resources/**`, `app/Data/**`) · config (`config/**`) · mailables (`app/Mail/**`, `app/Notifications/**`) |

**Default official-skill suggestions** (offer to user, do not auto-add):
- `laravel-boost` review skill (bundled with `laravel/boost` package)
- `laravel-simplifier` agent (if installed via `npx skills add ...`)

**Rubric tables:** see [`review-md.template.md`](./review-md.template.md) — the embedded reference rubric IS the Laravel rubric.

---

## `ts-nextjs`

**Detection:** `package.json` + `"next"` in `dependencies` or `devDependencies`.

| Placeholder | Value |
|-------------|-------|
| `{{STACK}}` | `Next.js <ver> · TypeScript <ver> · React <ver> · <test framework e.g. Vitest> · ESLint · Prettier` |
| `{{STYLE_TOOL}}` | `Prettier + ESLint` |
| `{{TYPE_TOOL}}` | `tsc` |
| `{{TEST_TOOL}}` | from `package.json` `scripts.test` — Vitest / Jest / Playwright |
| `{{AUTH_PATHS}}` | `middleware.ts`, `app/api/auth/**`, `lib/auth*`, `app/(auth)/**`, NextAuth config files |
| `{{SURFACE_AREA}}` | routes (`app/**/route.ts`, `pages/api/**`) · pages (`app/**/page.tsx`) · server actions (files using `'use server'`) · components (`components/**`, `app/**/_components/**`) · hooks (`hooks/**`, `lib/use*`) · server utilities (`lib/server/**`, `server/**`) · types (`types/**`) · middleware (`middleware.ts`) · config (`next.config.*`, `tailwind.config.*`) |

**Default official-skill suggestions:**
- `next-best-practices` (skills.sh)
- `next-cache-components` (skills.sh)
- `tanstack-query-best-practices` (if `@tanstack/react-query` in deps)
- `web-design-guidelines` (skills.sh)

**Rubric adaptation notes** (when generating REVIEW.md for ts-nextjs, replace the Laravel rubric rows with these):
- Quality: server/client component boundary violations, `'use client'` over-broad, unsafe `any`/`@ts-ignore`, missing `key` props, prop-drilling > 3 levels, inline arrow fns in JSX prop, `useEffect` for derived state.
- Security: missing CSP, `dangerouslySetInnerHTML` without sanitisation, missing CSRF on POST routes, secrets in `NEXT_PUBLIC_*`, unsafe `<a target="_blank">` without `rel="noopener"`, missing auth on `app/api/**` routes, SQL injection in raw queries (Prisma `$queryRawUnsafe`), unvalidated `redirect()` targets, SSRF in server fetches without allowlist.
- Quantity: same generic rubric (LOC, mixed concerns, half-finished, abstractions for single caller).

---

## `python-fastapi`

**Detection:** `pyproject.toml` or `requirements.txt` containing `fastapi`.

| Placeholder | Value |
|-------------|-------|
| `{{STACK}}` | `FastAPI <ver> · Python <ver> · Pydantic v2 · pytest · Ruff · mypy` |
| `{{STYLE_TOOL}}` | `Ruff` |
| `{{TYPE_TOOL}}` | `mypy` / `pyright` |
| `{{TEST_TOOL}}` | `pytest` |
| `{{AUTH_PATHS}}` | `app/auth/**`, `app/dependencies/auth*`, `app/security/**`, `config/auth*` |
| `{{SURFACE_AREA}}` | routes (`app/routers/**`, `app/api/**`) · services (`app/services/**`) · models (`app/models/**`, SQLAlchemy / Tortoise / etc.) · schemas (`app/schemas/**`, Pydantic) · dependencies (`app/dependencies/**`) · background tasks (`app/tasks/**`, Celery / arq) · migrations (`alembic/versions/**` or framework equivalent) · config (`app/config/**`) |

**Default official-skill suggestions:**
- _(no FastAPI-official review skill at time of writing — leave §1.5 empty unless user supplies)_

**Rubric adaptation notes:**
- Quality: bare `except:`, mutable default arguments, missing `async`/`await` in async route, blocking I/O in async handler, missing Pydantic response model, missing dependency injection (using globals).
- Security: SQL injection in raw queries, missing rate limiter on auth endpoints, secrets in `os.environ` lookups outside config, missing CORS allowlist, `eval`/`exec` on user input, pickle deserialisation of user input, missing auth dependency on protected routes.

---

## `python-data`

**Detection:** `pyproject.toml` with `clickhouse-driver` / `pandas` / `duckdb` / similar, NO `fastapi`.

| Placeholder | Value |
|-------------|-------|
| `{{STACK}}` | `Python <ver> · ClickHouse · pandas · pytest · Ruff · mypy` |
| `{{STYLE_TOOL}}` | `Ruff` |
| `{{TYPE_TOOL}}` | `mypy` |
| `{{TEST_TOOL}}` | `pytest` |
| `{{AUTH_PATHS}}` | _(usually n/a — internal data service. Confirm with user.)_ |
| `{{SURFACE_AREA}}` | pipelines (`app/pipelines/**`) · jobs (`app/jobs/**`, Airflow DAGs / Prefect flows) · queries (`app/queries/**`, SQL files) · models (`app/models/**`) · connectors (`app/connectors/**`) · schemas (`app/schemas/**`) · config (`app/config/**`) |

**Default official-skill suggestions:** none.

**Rubric adaptation notes:**
- Quality: SQL string concatenation in queries, missing `LIMIT` on exploratory queries, untested transforms, pandas `inplace=True` (deprecated patterns), missing `dtype` on read_csv.
- Security: SQL injection in ClickHouse queries (use parameter binding), credentials in connection strings logged, no row-count guard on `DELETE`/`UPDATE`, secrets in DAG default args.

---

## `custom`

When detection fails, ask the user for:
- Language + primary framework (one line)
- Style/lint tool name
- Type-checker name
- Test framework name
- Directory layout for: routes/entrypoints, business logic, data layer, tests, config
- Auth-related file paths (or "n/a")

Generate the template with these supplied values; leave the rubric tables as TODO blocks the user fills in, with a comment block at the top of REVIEW.md saying:

```
<!-- TODO: rubric tables below are starter — adapt to <framework> idiom. -->
```
