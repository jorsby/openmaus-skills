---
name: supabase
description: Use when doing ANY task involving Supabase. Triggers — Supabase products (Database, Auth, Edge Functions, Realtime, Storage, Vectors, Cron, Queues); client libraries and SSR integrations (supabase-js, @supabase/ssr) in Next.js, React, SvelteKit, Astro, Remix; auth issues (login, logout, sessions, JWT, cookies, getSession, getUser, getClaims, RLS); Supabase CLI or MCP server; schema changes, migrations, declarative schemas, security audits, Postgres extensions (pg_graphql, pg_cron, pg_vector); debugging and troubleshooting errors or unexpected behavior on Supabase projects (HTTP errors, Postgres errors, RLS surprises, permission denied, schema cache issues, timeouts, Edge Function crashes, Realtime drops, Storage failures) and reading or querying logs (Logs Explorer, ClickHouse).
metadata:
  author: supabase
  version: "0.1.2"
---

> OpenMaus içinde yönetilir. Köken: Multica studio, bridge; 6 Eylül 2026 dışa aktarımı.
> Yeni içeriği Skills ekranında inceleyip etkinleştir. Bu dosya eski kütüphaneden senkronize edilmez.
> Paket ekleri aşağıda “Paket eki” başlıklarında tam metindir. Metindeki göreli paket yolları bu bölümlere karşılık gelir. Yalnız ihtiyaç duyulan bölümü oku. Scriptler disk üzerinde kurulu değildir; çalıştırma gerekirse onaylanan iş kapsamında geçici dosyaya çıkarılıp doğrulanır.


# Supabase

## Core Principles

**1. Supabase changes frequently — verify against changelog and current docs before implementing.**
Do not rely on training data for Supabase features. Function signatures, config.toml settings, and API conventions change between versions.

First, fetch `https://supabase.com/changelog.md` (a lightweight summary index — not a heavy pull), scan for `breaking-change` tags relevant to your task, and follow the linked page for any that apply. Then look up the relevant topic using the documentation access methods below.

**2. Verify your work.**
After implementing any fix, run a test query to confirm the change works. A fix without verification is incomplete.

**3. Recover from errors, don't loop.**
If an approach fails after 2-3 attempts, stop and reconsider. Try a different method, check documentation, inspect the error more carefully, and review relevant logs when available. Supabase issues are not always solved by retrying the same command, and the answer is not always in the logs, but logs are often worth checking before proceeding.

**4. Exposing tables to the Data API:** Depending on the user's [Data API settings](https://supabase.com/dashboard/project/<ref>/integrations/data_api/settings), newly created tables may not be automatically exposed via the Data (REST) API. If this is the case, `anon` and `authenticated` roles will need to be explicitly granted access.

> Note that this is separate from RLS, which controls which _rows_ are visible once a table is accessible, not whether the table is accessible at all.

When a user reports a SQL-created table is unexpectedly inaccessible, check their Data API settings and whether the roles have been granted access via explicit `GRANT` SQL. When granting public (`anon`/`authenticated`) access, always enable RLS too. See [Exposing a Table to the Data API](https://supabase.com/docs/guides/api/securing-your-api.md) for the full setup workflow.

**5. RLS in exposed schemas.**
Enable RLS on every table in any exposed schema, which includes `public` by default. This is critical in Supabase because tables in exposed schemas can be reachable through the Data API when the `anon`/`authenticated` roles have access (see [Exposing a Table to the Data API](https://supabase.com/docs/guides/api/securing-your-api.md)). For private schemas, prefer RLS as defense in depth. After enabling RLS, create policies that match the actual access model rather than defaulting every table to the same `auth.uid()` pattern.

**6. Security checklist.**
When working on any Supabase task that touches auth, RLS, views, storage, or user data, run through this checklist. These are Supabase-specific security traps that silently create vulnerabilities:

- **Auth and session security**
  - **Never use `user_metadata` claims in JWT-based authorization decisions.** In Supabase, `raw_user_meta_data` is user-editable and can appear in `auth.jwt()`, so it is unsafe for RLS policies or any other authorization logic. Store authorization data in `raw_app_meta_data` / `app_metadata` instead.
  - **Deleting a user does not invalidate existing access tokens.** Sign out or revoke sessions first, keep JWT expiry short for sensitive apps, and for strict guarantees validate `session_id` against `auth.sessions` on sensitive operations.
  - **If you use `app_metadata` or `auth.jwt()` for authorization, remember JWT claims are not always fresh until the user's token is refreshed.**

- **API key and client exposure**
  - **Never expose the `service_role` or secret key in public clients.** Prefer publishable keys for frontend code. Legacy `anon` keys are only for compatibility. In Next.js, any `NEXT_PUBLIC_` env var is sent to the browser.

- **RLS, views, and privileged database code**
  - **Views bypass RLS by default.** In Postgres 15 and above, use `CREATE VIEW ... WITH (security_invoker = true)`. In older versions of Postgres, protect your views by revoking access from the `anon` and `authenticated` roles, or by putting them in an unexposed schema.
  - **UPDATE requires a SELECT policy.** In Postgres RLS, an UPDATE needs to first SELECT the row. Without a SELECT policy, updates silently return 0 rows — no error, just no change.
  - **`auth.role()` is deprecated — use the `TO` clause instead.** Supabase has deprecated `auth.role()` in favour of specifying the target role directly on the policy with `TO authenticated` or `TO anon`. Beyond deprecation, `auth.role() = 'authenticated'` breaks silently when anonymous sign-ins are enabled, because anonymous users carry the `authenticated` Postgres role and pass the check regardless of whether the user is genuinely signed in.
    ```sql
    -- Deprecated (do not use)
    create policy "example" on table_name for select
    using ( auth.role() = 'authenticated' );
    ```
  - **`TO authenticated` alone is authentication without authorization (BOLA / IDOR).** Using `TO authenticated` only checks the role — it does not restrict which rows a user can access. The correct pattern combines `TO authenticated` with an ownership predicate in `USING`:
    ```sql
    create policy "example" on table_name for select
    to authenticated
    using ( (select auth.uid()) = user_id );
    ```
  - **UPDATE policies require both `USING` and `WITH CHECK`.** Without `WITH CHECK`, a user can reassign a row's `user_id` to another user:
    ```sql
    create policy "example" on table_name for update
    to authenticated
    using ( (select auth.uid()) = user_id )
    with check ( (select auth.uid()) = user_id );
    ```
  - **`SECURITY DEFINER` functions bypass RLS.** A `SECURITY DEFINER` function runs with its creator's privileges — typically a role with `bypassrls` (e.g., `postgres`). Never add `SECURITY DEFINER` to resolve a permission error; it silently removes access control without fixing the underlying cause. Prefer `SECURITY INVOKER`.
  - **`SECURITY DEFINER` functions in `public` are callable by all roles.** Postgres grants `EXECUTE` to `PUBLIC` by default for every new function, so any `SECURITY DEFINER` function in `public` is a public API endpoint callable by `anon` and `authenticated` (which inherit from `PUBLIC`) without any additional grant. When `SECURITY DEFINER` is genuinely needed (e.g., bypassing RLS on an internal lookup table), keep the function in a non-exposed schema, always include an `auth.uid()` check in the function body, and run `supabase db advisors` after making changes.

- **Storage access control**
  - **Storage upsert requires INSERT + SELECT + UPDATE.** Granting only INSERT allows new uploads but file replacement (upsert) silently fails. You need all three.

- **Dependency and supply-chain security**
  - **Always pin package versions and commit lockfiles** when installing Supabase packages (`supabase-js`, `@supabase/ssr`, `supabase-py`, etc.). See the [npm security guide](https://supabase.com/docs/guides/security/npm-security.md) for the full checklist.

For any security concern not covered above, fetch the Supabase product security index: `https://supabase.com/docs/guides/security/product-security.md`

## Supabase CLI

Always discover commands via `--help` — never guess. The CLI structure changes between versions.

```bash
supabase --help                    # All top-level commands
supabase <group> --help            # Subcommands (e.g., supabase db --help)
supabase <group> <command> --help  # Flags for a specific command
```

**Supabase CLI Known gotchas:**

- `supabase db query` requires **CLI v2.79.0+** → use MCP `execute_sql` or `psql` as fallback
- `supabase db advisors` requires **CLI v2.81.3+** → use MCP `get_advisors` as fallback
- In imperative migration projects, create new hand-authored migration files with `supabase migration new <name>` first. Never invent a migration filename or rely on memory for the expected format. Declarative schema projects generate migrations from `supabase/schemas/`; see "Making and Committing Schema Changes" below.

**Version check and upgrade:** Run `supabase --version` to check. For CLI changelogs and version-specific features, consult the [CLI documentation](https://supabase.com/docs/reference/cli/introduction) or [GitHub releases](https://github.com/supabase/cli/releases).

## Supabase MCP Server

For setup instructions, server URL, and configuration, see the [MCP setup guide](https://supabase.com/docs/guides/getting-started/mcp).

**Troubleshooting connection issues** — follow these steps in order:

1. **Check if the server is reachable:**
   `curl -so /dev/null -w "%{http_code}" https://mcp.supabase.com/mcp`
   A `401` is expected (no token) and means the server is up. Timeout or "connection refused" means it may be down.

2. **Check `.mcp.json` configuration:**
   Verify the project root has a valid `.mcp.json` with the correct server URL. If missing, create one pointing to `https://mcp.supabase.com/mcp`.

3. **Authenticate the MCP server:**
   If the server is reachable and `.mcp.json` is correct but tools aren't visible, the user needs to authenticate. The Supabase MCP server uses OAuth 2.1 — tell the user to trigger the auth flow in their agent, complete it in the browser, and reload the session.

## Supabase Documentation

Before implementing any Supabase feature, find the relevant documentation. Use these methods in priority order:

1. **MCP `search_docs` tool** (preferred — returns relevant snippets directly)
2. **Fetch docs pages as markdown** — any docs page can be fetched by appending `.md` to the URL path.
3. **Web search** for Supabase-specific topics when you don't know which page to look at.

## Making and Committing Schema Changes

First decide which schema workflow the project uses.

### Option A: Declarative schemas

Use this when `supabase/schemas/` exists or `config.toml` sets `schema_paths`. Edit the desired schema state in those files, then generate and review the migration. Do not start by hand-writing a migration. See the [Declarative database schemas guide](https://supabase.com/docs/guides/local-development/declarative-database-schemas).

### Option B: Imperative migrations

Use this when the project does not use declarative schemas.

**To make schema changes, use `execute_sql` (MCP) or `supabase db query` (CLI).** These run SQL directly on the database without creating migration history entries, so you can iterate freely and generate a clean migration when ready.

Do NOT use `apply_migration` to change a local database schema — it writes a migration history entry on every call, which means you can't iterate, and `supabase db diff` / `supabase db pull` will produce empty or conflicting diffs. If you use it, you'll be stuck with whatever SQL you passed on the first try.

**When ready to commit** your changes to a migration file:

1. **Run advisors** → `supabase db advisors` (CLI v2.81.3+) or MCP `get_advisors`. Fix any issues.
2. **Review the Security Checklist above** if your changes involve views, functions, triggers, or storage.
3. **Generate the migration** → `supabase db pull <descriptive-name> --local --yes`
4. **Verify** → `supabase migration list --local`

## Debugging

When you get an error on a Supabase-related request, for example an error code from the Supabase REST API, Postgres database, or PostgREST, an empty result, getting blocked by RLS unexpectedly, or an error from a Supabase service like Auth, Realtime, Edge Functions, or Storage, you **must** fetch Supabase's [Monitoring and Debugging](https://supabase.com/docs/guides/monitoring-and-debugging.md) documentation before diagnosing or proposing a fix, rather than working from memory. The same docs also cover performance optimizations, such as slow queries and missing indexes.

## Reference Guides

- **Skill Feedback** → [references/skill-feedback.md](references/skill-feedback.md)
  **MUST read when** the user reports that this skill gave incorrect guidance or is missing information.


## Paket eki: `CHANGELOG.md`

Kaynak SHA256: `f4c9c7750ea2c9212a05c3f38bc04d66cb2fe037f6b9a3fb6279d53bef152f8e` · UTF-8 boyutu: 7355 bayt.

````text
# Changelog

## [0.1.7](https://github.com/supabase/agent-skills/compare/v0.1.6...supabase-v0.1.7) (2026-08-12)


### Features

* **supabase:** add debugging workflows to the supabase skill ([#112](https://github.com/supabase/agent-skills/issues/112)) ([3a4f0ce](https://github.com/supabase/agent-skills/commit/3a4f0ce0782e0cbdcf187c362e8d15d9e324462b))

## [0.1.6](https://github.com/supabase/agent-skills/compare/v0.1.5...supabase-v0.1.6) (2026-07-30)


### Features

* add declarative schema section to schema management ([#120](https://github.com/supabase/agent-skills/issues/120)) ([94cf44f](https://github.com/supabase/agent-skills/commit/94cf44ff3a123edcf54a9af373929798cddefed8))
* add instructions to check changelog ([#74](https://github.com/supabase/agent-skills/issues/74)) ([4bb13d8](https://github.com/supabase/agent-skills/commit/4bb13d858d19f1f848505a66f46fc9603fdcde95))
* add npm supply-chain security guidance to supabase skill ([#94](https://github.com/supabase/agent-skills/issues/94)) ([82df90a](https://github.com/supabase/agent-skills/commit/82df90a5de1cd84386d8bc192746e50343b86dc0))
* instructions on exposing tables to the data api ([#71](https://github.com/supabase/agent-skills/issues/71)) ([f15a5a4](https://github.com/supabase/agent-skills/commit/f15a5a40779072a530c9e53c3f14ec4131118ea6))
* using Supabase agent skills ([#12](https://github.com/supabase/agent-skills/issues/12)) ([7c2e389](https://github.com/supabase/agent-skills/commit/7c2e3894fddfde8eb6c77d2a8921904543b9be7a))


### Bug Fixes

* bump supabase skill to v0.1.1 and fix Data API broken link ([#72](https://github.com/supabase/agent-skills/issues/72)) ([5a6542e](https://github.com/supabase/agent-skills/commit/5a6542e08fc026d90c9a6a0f5a67749e9ceb9946))
* cover SECURITY DEFINER, auth.role() deprecation, and BOLA in security checklist ([#85](https://github.com/supabase/agent-skills/issues/85)) ([133f43e](https://github.com/supabase/agent-skills/commit/133f43e8c2ffc48823ff0630c692cabecea3e3a3))
* update Data API doc link and bump supabase skill to v0.1.1 ([#73](https://github.com/supabase/agent-skills/issues/73)) ([e5f7a7c](https://github.com/supabase/agent-skills/commit/e5f7a7cfd697765848ffd6a4505f3c02e1ee17ee))

## [0.1.5](https://github.com/supabase/agent-skills/compare/v0.1.4...v0.1.5) (2026-07-10)


### Features

* add declarative schema section to schema management ([#120](https://github.com/supabase/agent-skills/issues/120)) ([94cf44f](https://github.com/supabase/agent-skills/commit/94cf44ff3a123edcf54a9af373929798cddefed8))
* add instructions to check changelog ([#74](https://github.com/supabase/agent-skills/issues/74)) ([4bb13d8](https://github.com/supabase/agent-skills/commit/4bb13d858d19f1f848505a66f46fc9603fdcde95))
* add npm supply-chain security guidance to supabase skill ([#94](https://github.com/supabase/agent-skills/issues/94)) ([82df90a](https://github.com/supabase/agent-skills/commit/82df90a5de1cd84386d8bc192746e50343b86dc0))
* instructions on exposing tables to the data api ([#71](https://github.com/supabase/agent-skills/issues/71)) ([f15a5a4](https://github.com/supabase/agent-skills/commit/f15a5a40779072a530c9e53c3f14ec4131118ea6))
* using Supabase agent skills ([#12](https://github.com/supabase/agent-skills/issues/12)) ([7c2e389](https://github.com/supabase/agent-skills/commit/7c2e3894fddfde8eb6c77d2a8921904543b9be7a))


### Bug Fixes

* bump supabase skill to v0.1.1 and fix Data API broken link ([#72](https://github.com/supabase/agent-skills/issues/72)) ([5a6542e](https://github.com/supabase/agent-skills/commit/5a6542e08fc026d90c9a6a0f5a67749e9ceb9946))
* cover SECURITY DEFINER, auth.role() deprecation, and BOLA in security checklist ([#85](https://github.com/supabase/agent-skills/issues/85)) ([133f43e](https://github.com/supabase/agent-skills/commit/133f43e8c2ffc48823ff0630c692cabecea3e3a3))
* update Data API doc link and bump supabase skill to v0.1.1 ([#73](https://github.com/supabase/agent-skills/issues/73)) ([e5f7a7c](https://github.com/supabase/agent-skills/commit/e5f7a7cfd697765848ffd6a4505f3c02e1ee17ee))

## [0.1.4](https://github.com/supabase/agent-skills/compare/v0.1.3...v0.1.4) (2026-06-05)


### Features

* add instructions to check changelog ([#74](https://github.com/supabase/agent-skills/issues/74)) ([4bb13d8](https://github.com/supabase/agent-skills/commit/4bb13d858d19f1f848505a66f46fc9603fdcde95))
* add npm supply-chain security guidance to supabase skill ([#94](https://github.com/supabase/agent-skills/issues/94)) ([82df90a](https://github.com/supabase/agent-skills/commit/82df90a5de1cd84386d8bc192746e50343b86dc0))
* instructions on exposing tables to the data api ([#71](https://github.com/supabase/agent-skills/issues/71)) ([f15a5a4](https://github.com/supabase/agent-skills/commit/f15a5a40779072a530c9e53c3f14ec4131118ea6))
* using Supabase agent skills ([#12](https://github.com/supabase/agent-skills/issues/12)) ([7c2e389](https://github.com/supabase/agent-skills/commit/7c2e3894fddfde8eb6c77d2a8921904543b9be7a))


### Bug Fixes

* bump supabase skill to v0.1.1 and fix Data API broken link ([#72](https://github.com/supabase/agent-skills/issues/72)) ([5a6542e](https://github.com/supabase/agent-skills/commit/5a6542e08fc026d90c9a6a0f5a67749e9ceb9946))
* cover SECURITY DEFINER, auth.role() deprecation, and BOLA in security checklist ([#85](https://github.com/supabase/agent-skills/issues/85)) ([133f43e](https://github.com/supabase/agent-skills/commit/133f43e8c2ffc48823ff0630c692cabecea3e3a3))
* update Data API doc link and bump supabase skill to v0.1.1 ([#73](https://github.com/supabase/agent-skills/issues/73)) ([e5f7a7c](https://github.com/supabase/agent-skills/commit/e5f7a7cfd697765848ffd6a4505f3c02e1ee17ee))

## [0.1.3](https://github.com/supabase/agent-skills/compare/v0.1.2...v0.1.3) (2026-06-02)


### Features

* add instructions to check changelog ([#74](https://github.com/supabase/agent-skills/issues/74)) ([4bb13d8](https://github.com/supabase/agent-skills/commit/4bb13d858d19f1f848505a66f46fc9603fdcde95))
* add npm supply-chain security guidance to supabase skill ([#94](https://github.com/supabase/agent-skills/issues/94)) ([82df90a](https://github.com/supabase/agent-skills/commit/82df90a5de1cd84386d8bc192746e50343b86dc0))
* instructions on exposing tables to the data api ([#71](https://github.com/supabase/agent-skills/issues/71)) ([f15a5a4](https://github.com/supabase/agent-skills/commit/f15a5a40779072a530c9e53c3f14ec4131118ea6))
* using Supabase agent skills ([#12](https://github.com/supabase/agent-skills/issues/12)) ([7c2e389](https://github.com/supabase/agent-skills/commit/7c2e3894fddfde8eb6c77d2a8921904543b9be7a))


### Bug Fixes

* bump supabase skill to v0.1.1 and fix Data API broken link ([#72](https://github.com/supabase/agent-skills/issues/72)) ([5a6542e](https://github.com/supabase/agent-skills/commit/5a6542e08fc026d90c9a6a0f5a67749e9ceb9946))
* cover SECURITY DEFINER, auth.role() deprecation, and BOLA in security checklist ([#85](https://github.com/supabase/agent-skills/issues/85)) ([133f43e](https://github.com/supabase/agent-skills/commit/133f43e8c2ffc48823ff0630c692cabecea3e3a3))
* update Data API doc link and bump supabase skill to v0.1.1 ([#73](https://github.com/supabase/agent-skills/issues/73)) ([e5f7a7c](https://github.com/supabase/agent-skills/commit/e5f7a7cfd697765848ffd6a4505f3c02e1ee17ee))

````


## Paket eki: `assets/feedback-issue-template.md`

Kaynak SHA256: `ea211aa174c5a096fe76461f25143b8bc23f3bde3adc9e727cda38218cc8c282` · UTF-8 boyutu: 459 bayt.

````text
## What happened

**Task:** <!-- e.g., "Set up MFA on patient records" -->

**Skill said:** <!-- e.g., "Use auth.jwt()->'app_metadata' in the RLS policy" -->

**Expected:** <!-- e.g., "The function also needs SECURITY DEFINER + grant to supabase_auth_admin" -->

## Source

**File:** <!-- e.g., references/security-model.md -->

**Section:** <!-- e.g., "Trust Boundaries > user_metadata vs app_metadata" -->

## Fix suggestion

<!-- Leave blank if unsure -->

````


## Paket eki: `references/skill-feedback.md`

Kaynak SHA256: `09ad94c0291735dfc04c92557c8734fa259ec39e1d56bf2bd5c6a6529d838717` · UTF-8 boyutu: 1035 bayt.

````text
# Skill Feedback

Use this when the user reports that the skill gave incorrect guidance, is missing information, or could be improved. This is about the skill (agent instructions), not about Supabase the product.

## Steps

1. **Ask permission** — Ask the user if they'd like to submit feedback to the skill maintainers. If they decline, move on.

2. **Draft the issue** — Use the template at [assets/feedback-issue-template.md](../assets/feedback-issue-template.md) to structure the feedback. Fill in the fields based on the conversation. Always identify which specific reference file and section caused the problem.

3. **Submit** — Create a GitHub Issue on the `supabase/agent-skills` repository using the draft as the issue body. The title must follow this format: `user-feedback: <summary of the problem>`.

4. **Share the result** — Share the issue URL with the user after submission. If submission fails, give the user this link to create the issue manually:

```
https://github.com/supabase/agent-skills/issues/new
```

````
