---
name: workers-best-practices
description: Reviews and authors Cloudflare Workers code against production best practices. Load when writing new Workers, reviewing Worker code, configuring wrangler.jsonc, or checking for common Workers anti-patterns (streaming, floating promises, global state, secrets, bindings, observability). Biases towards retrieval from Cloudflare docs over pre-trained knowledge.
---

> OpenMaus içinde yönetilir. Köken: Multica studio, bridge; 6 Eylül 2026 dışa aktarımı.
> Yeni içeriği Skills ekranında inceleyip etkinleştir. Bu dosya eski kütüphaneden senkronize edilmez.
> Paket ekleri aşağıda “Paket eki” başlıklarında tam metindir. Metindeki göreli paket yolları bu bölümlere karşılık gelir. Yalnız ihtiyaç duyulan bölümü oku. Scriptler disk üzerinde kurulu değildir; çalıştırma gerekirse onaylanan iş kapsamında geçici dosyaya çıkarılıp doğrulanır.


Your knowledge of Cloudflare Workers APIs, types, and configuration may be outdated. **Prefer retrieval over pre-training** for any Workers code task — writing or reviewing.

## Retrieval Sources

Fetch the **latest** versions before writing or reviewing Workers code. Do not rely on baked-in knowledge for API signatures, config fields, or binding shapes.

| Source | How to retrieve | Use for |
|--------|----------------|---------|
| Workers best practices | Fetch `https://developers.cloudflare.com/workers/best-practices/workers-best-practices/` | Canonical rules, patterns, anti-patterns |
| Workers types | See `references/review.md` for retrieval steps | API signatures, handler types, binding types |
| Wrangler config schema | `node_modules/wrangler/config-schema.json` | Config fields, binding shapes, allowed values |
| Cloudflare docs | Search tool or `https://developers.cloudflare.com/workers/` | API reference, compatibility dates/flags |

## FIRST: Fetch Latest References

Before reviewing or writing Workers code, retrieve the current best practices page and relevant type definitions. If the project's `node_modules` has an older version, **prefer the latest published version**.

```bash
# Fetch latest workers types
mkdir -p /tmp/workers-types-latest && \
  npm pack @cloudflare/workers-types --pack-destination /tmp/workers-types-latest && \
  tar -xzf /tmp/workers-types-latest/cloudflare-workers-types-*.tgz -C /tmp/workers-types-latest
# Types at /tmp/workers-types-latest/package/index.d.ts
```

## Reference Documentation

- `references/rules.md` — all best practice rules with code examples and anti-patterns
- `references/review.md` — type validation, config validation, binding access patterns, review process

## Rules Quick Reference

### Configuration

| Rule | Summary |
|------|---------|
| Compatibility date | Set `compatibility_date` to today on new projects; update periodically on existing ones |
| nodejs_compat | Enable the `nodejs_compat` flag — many libraries depend on Node.js built-ins |
| wrangler types | Run `wrangler types` to generate `Env` — never hand-write binding interfaces |
| Secrets | Use `wrangler secret put`, never hardcode secrets in config or source |
| wrangler.jsonc | Use JSONC config for non-secret settings — newer features are JSON-only |

### Request & Response Handling

| Rule | Summary |
|------|---------|
| Streaming | Stream large/unknown payloads — never `await response.text()` on unbounded data |
| waitUntil | Use `ctx.waitUntil()` for post-response work; do not destructure `ctx` |

### Architecture

| Rule | Summary |
|------|---------|
| Bindings over REST | Use in-process bindings (KV, R2, D1, Queues) — not the Cloudflare REST API |
| Queues & Workflows | Move async/background work off the critical path |
| Service bindings | Use service bindings for Worker-to-Worker calls — not public HTTP |
| Hyperdrive | Always use Hyperdrive for external PostgreSQL/MySQL connections |

### Observability

| Rule | Summary |
|------|---------|
| Logs & Traces | Enable `observability` in config with `head_sampling_rate`; use structured JSON logging |

### Code Patterns

| Rule | Summary |
|------|---------|
| No global request state | Never store request-scoped data in module-level variables |
| Floating promises | Every Promise must be `await`ed, `return`ed, `void`ed, or passed to `ctx.waitUntil()` |

### Security

| Rule | Summary |
|------|---------|
| Web Crypto | Use `crypto.randomUUID()` / `crypto.getRandomValues()` — never `Math.random()` for security |
| No passThroughOnException | Use explicit try/catch with structured error responses |

## Anti-Patterns to Flag

| Anti-pattern | Why it matters |
|-------------|----------------|
| `await response.text()` on unbounded data | Memory exhaustion — 128 MB limit |
| Hardcoded secrets in source or config | Credential leak via version control |
| `Math.random()` for tokens/IDs | Predictable, not cryptographically secure |
| Bare `fetch()` without `await` or `waitUntil` | Floating promise — dropped result, swallowed error |
| Module-level mutable variables for request state | Cross-request data leaks, stale state, I/O errors |
| Cloudflare REST API from inside a Worker | Unnecessary network hop, auth overhead, added latency |
| `ctx.passThroughOnException()` as error handling | Hides bugs, makes debugging impossible |
| Hand-written `Env` interface | Drifts from actual wrangler config bindings |
| Direct string comparison for secret values | Timing side-channel — use `crypto.subtle.timingSafeEqual` |
| Destructuring `ctx` (`const { waitUntil } = ctx`) | Loses `this` binding — throws "Illegal invocation" at runtime |
| `any` on `Env` or handler params | Defeats type safety for all binding access |
| `as unknown as T` double-cast | Hides real type incompatibilities — fix the design |
| `implements` on platform base classes (instead of `extends`) | Legacy — loses `this.ctx`, `this.env`. Applies to DurableObject, WorkerEntrypoint, Workflow |
| `env.X` inside platform base class | Should be `this.env.X` in classes extending DurableObject, WorkerEntrypoint, etc. |

## Review Workflow

1. **Retrieve** — fetch latest best practices page, workers types, and wrangler schema
2. **Read full files** — not just diffs; context matters for binding access patterns
3. **Check types** — binding access, handler signatures, no `any`, no unsafe casts (see `references/review.md`)
4. **Check config** — compatibility_date, nodejs_compat, observability, secrets, binding-code consistency
5. **Check patterns** — streaming, floating promises, global state, serialization boundaries
6. **Check security** — crypto usage, secret handling, timing-safe comparisons, error handling
7. **Validate with tools** — `npx tsc --noEmit`, lint for `no-floating-promises`
8. **Reference rules** — see `references/rules.md` for each rule's correct pattern

## Scope

This skill covers Workers-specific best practices and code review. For related topics:

- **Durable Objects**: load the `durable-objects` skill
- **Workflows**: see [Rules of Workflows](https://developers.cloudflare.com/workflows/build/rules-of-workflows/)
- **Wrangler CLI commands**: load the `wrangler` skill

## Principles

- **Be certain.** Retrieve before flagging. If unsure about an API, config field, or pattern, fetch the docs first.
- **Provide evidence.** Reference line numbers, tool output, or docs links.
- **Focus on what developers will copy.** Workers code in examples and docs gets pasted into production.
- **Correctness over completeness.** A concise example that works beats a comprehensive one with errors.


## Paket eki: `references/review.md`

Kaynak SHA256: `3e32da345be0906d020e5235d01f2b6398a41216900b5370a267e3f17757ade0` · UTF-8 boyutu: 8145 bayt.

````text
# Code Review — Workers

How to review Workers code for type correctness, API usage, config validity, and best practices. This is self-contained — do not assume access to other skills.

## Retrieval

Prefer retrieval over pre-training. Types, config schemas, and APIs change with compatibility dates and new bindings.

### Workers types

Fetch the latest `@cloudflare/workers-types` before reviewing. The project may have an older version installed.

```bash
mkdir -p /tmp/workers-types-latest && \
  npm pack @cloudflare/workers-types --pack-destination /tmp/workers-types-latest && \
  tar -xzf /tmp/workers-types-latest/cloudflare-workers-types-*.tgz -C /tmp/workers-types-latest
# Types are at /tmp/workers-types-latest/package/index.d.ts
```

Search this file for the specific type, class, or interface under review. Do not guess type names.

Alternative: `npx wrangler types` generates a typed `Env` interface from the local wrangler config.

Fallback: read `node_modules/@cloudflare/workers-types/index.d.ts`. Note the installed version.

### Wrangler config schema

The authoritative schema is bundled with wrangler as `config-schema.json` (JSON Schema draft-07).

```bash
# Read from local node_modules
cat node_modules/wrangler/config-schema.json
```

Do not guess field names or structures — look them up.

### Cloudflare docs

Use the Cloudflare docs search tool if available, or fetch from `https://developers.cloudflare.com/workers/`. The best practices page lives at `/workers/best-practices/workers-best-practices/`.

---

## Type Validation

### Env interface

- Every binding must have a specific type. Flag `any`, `unknown`, `object`, or `Record<string, unknown>` on bindings.
- Binding types that accept generic parameters (Durable Object namespaces, Queues, Service bindings for RPC) must include them. Read the type definition to confirm which types are generic.
- Binding names must match the wrangler config exactly.
- Prefer generated types from `wrangler types` over hand-written interfaces.

### Handler and class signatures

Verify against current type definitions — do not assume signatures are stable.

- Correct import path (most Workers platform classes import from `"cloudflare:workers"`)
- Generic type parameter on base classes (e.g., `DurableObject<Env>`)
- Binding access pattern: `env.X` in module export handlers, `this.env.X` in classes extending platform base classes
- `ExecutionContext` as the third param in module export handlers (needed for `ctx.waitUntil()`)
- `fetch()` handlers must return `Promise<Response>`

### Binding access — the most common error

- **Module export handlers** (`fetch`, `scheduled`, `queue`, `email`): bindings via `env.X` parameter
- **Platform base classes** (`WorkerEntrypoint`, `DurableObject`, `Workflow`, `Agent`): bindings via `this.env.X`

Flag `env.X` inside a class extending a platform base class. Flag `this.env.X` inside a module export handler.

### Type integrity rules

| Rule | Detail |
|------|--------|
| No `any` | Never on binding types, handler params, or API responses |
| No double-casting | `as unknown as T` hides real incompatibilities — fix the underlying design |
| Justify suppressions | `@ts-ignore`/`@ts-expect-error` must include a comment explaining why |
| Prefer `satisfies` | Use `satisfies ExportedHandler<Env>` over `as` — validates without widening |
| Validate, do not assert | Schema or type guard for untyped data (JSON, parsed bodies), not `as` |

### Stale class patterns

Old patterns survive in codebases long after APIs change.

- **`extends` vs `implements`**: platform classes use `extends`, not `implements`. The `implements` pattern is legacy and loses `this.ctx`, `this.env`.
- **Import paths**: verify module specifiers match what types actually export. Common mistake: wrong path for `"cloudflare:workers"` vs `"cloudflare:workflows"`.
- **Renamed properties**: e.g., `this.state` to `this.ctx` in Durable Objects. Search types to confirm.
- **Constructor signatures**: base class constructors change. Verify expected parameters.

---

## Config Validation

### Required fields

For executable examples, verify: `name`, `compatibility_date`, `main`. Check the schema for current required fields.

### Config format

- **JSONC** (`wrangler.jsonc`) — preferred for new projects
- **JSON** (`wrangler.json`) — valid but no comments
- **TOML** (`wrangler.toml`) — legacy; acceptable in existing content, flag in new projects

### Binding-code consistency

1. Every `env.X` reference in code has a corresponding binding declaration in config
2. Every binding in config is referenced in code (warn on unused)
3. Names match exactly (case-sensitive)
4. For Durable Objects: `class_name` matches the exported class name

### Common config mistakes

| Check | What to look for |
|-------|-----------------|
| Stale `compatibility_date` | Should be recent; use `$today` placeholder in docs |
| Missing DO migrations | Every new DO class needs a migration entry |
| Binding name mismatch | Config `binding`/`name` must match `env.X` in code |
| Secrets in config | Never in `vars` — use `wrangler secret put` |
| Wrong binding key | Verify top-level key name against the schema |
| Missing entrypoint | `main` required for executable Workers |

---

## Anti-Patterns to Flag

See the full anti-patterns table in `SKILL.md`. The type-specific ones to watch for during review:

- **`any` on `Env` or handler params** — defeats type safety for all downstream binding access
- **`as unknown as T`** — hides real type incompatibilities; fix the underlying design
- **`@ts-ignore`/`@ts-expect-error` without explanation** — masks errors silently; require a justifying comment
- **`implements` instead of `extends` on platform base classes** — legacy pattern; loses `this.ctx`, `this.env`
- **`env.X` inside class body** — should be `this.env.X` in platform base classes
- **`this.env.X` in module export handler** — should be `env.X` parameter
- **Non-serializable values across boundaries** — `Response`, `Error` in step/queue compiles but fails at runtime

---

## Serialization Boundaries

Data crossing these boundaries must be structured-clone serializable:

- **Queue messages**: body passed to `.send()` or `.sendBatch()`
- **Workflow step return values**: persisted to durable storage
- **DO storage**: values in `storage.put()` or SQL
- **`postMessage()`**: WebSocket messages

Non-serializable types to flag: `Response`, `Request`, `Error`, functions, class instances with methods, `Map`/`Set`, `Symbol`.

Valid: plain objects, arrays, strings, numbers, booleans, null, `ArrayBuffer`, `Date`.

---

## Review Process

1. **Retrieve** — fetch latest workers types, wrangler schema, and best practices page
2. **Read full files** — not just diffs; context matters for binding access patterns
3. **Categorize code** — determines what to check:
   - **Illustrative** (concept demo, comments for most logic): verify correct API names and realistic signatures
   - **Demonstrative** (functional snippet, would work in context): verify syntax, correct APIs, correct binding access
   - **Executable** (standalone, runs without modification): verify compiles, runs, includes imports and config
4. **Check types** — binding access pattern, handler signatures, no `any`, no unsafe casts
5. **Check config** — compatibility_date, nodejs_compat, observability, secrets, binding-code consistency
6. **Check patterns** — streaming, floating promises, global state, serialization boundaries
7. **Check security** — crypto usage, secret handling, timing-safe comparisons, error handling
8. **Validate with tools** — `npx tsc --noEmit`, lint for `no-floating-promises`
9. **Assess risk** — HIGH (auth, crypto, bindings), MEDIUM (business logic, config), LOW (style, comments)

### Output format

```
**[SEVERITY]** Brief description
`file.ts:42` — explanation with evidence
Suggested fix: `code`
```

Severity: **CRITICAL** (security, data loss, crash) | **HIGH** (type error, wrong API, broken config) | **MEDIUM** (missing validation, edge case) | **LOW** (style, minor improvement)

````


## Paket eki: `references/rules.md`

Kaynak SHA256: `b0273b18c7fa40fadf33262e7da7c8f2a52f2f1240ce66d590e4c31663261380` · UTF-8 boyutu: 16483 bayt.

````text
# Workers Best Practices — Rules

Each rule has an imperative summary, what to check, the correct pattern, and an anti-pattern where applicable. Code examples are plain TypeScript — no MDX components.

When a rule involves config fields or API signatures that may evolve, a **Retrieve** callout reminds you to check the latest docs or types before flagging. All doc paths are relative to `https://developers.cloudflare.com`.

---

## Configuration

### Keep compatibility_date current

Set `compatibility_date` to today on new projects. Update periodically on existing ones to access new APIs and fixes.

**Check**: `compatibility_date` exists. Flag if older than 6 months.

```jsonc
// wrangler.jsonc
{
  "compatibility_date": "$today",  // Replace with today's date (YYYY-MM-DD)
  "compatibility_flags": ["nodejs_compat"]
}
```

**Retrieve**: current compatibility dates at `/workers/configuration/compatibility-dates/`.

### Enable nodejs_compat

The `nodejs_compat` flag enables Node.js built-in modules (`node:crypto`, `node:buffer`, `node:stream`). Many libraries require it. Missing this flag causes cryptic import errors at runtime.

**Check**: `compatibility_flags` includes `"nodejs_compat"`.

```jsonc
{
  "compatibility_flags": ["nodejs_compat"]
}
```

### Generate binding types with wrangler types

Never hand-write the `Env` interface. Run `wrangler types` to generate it from the wrangler config. Re-run after adding or renaming any binding.

**Check**: no manually defined `Env` or `interface Env` that duplicates wrangler config bindings. Look for `satisfies ExportedHandler<Env>` pattern on the default export.

```ts
// Generated by wrangler types — always matches actual config
export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    const value = await env.MY_KV.get("key");
    return new Response(value);
  },
} satisfies ExportedHandler<Env>;
```

Anti-pattern:
```ts
// Hand-written Env that drifts from actual bindings
interface Env {
  MY_KV: KVNamespace;  // What if the binding name changed?
}
```

### Store secrets with wrangler secret

Secrets must never appear in wrangler config or source code. Use `wrangler secret put` and access via `env` at runtime. Non-secret config goes in `vars`.

**Check**: no string literals that look like API keys, tokens, or credentials. Verify `.env` is in `.gitignore` for local dev.

```jsonc
{
  "vars": {
    "API_BASE_URL": "https://api.example.com"  // Non-secret: OK in config
  }
  // Secrets set via: wrangler secret put API_KEY
}
```

Anti-pattern:
```jsonc
{
  "vars": {
    "API_KEY": "sk-live-abc123..."  // Secret in version control
  }
}
```

### Use wrangler.jsonc for config

Prefer `wrangler.jsonc` over `wrangler.toml`. Newer features are JSON-only. JSONC supports comments for documenting config decisions.

**Check**: project uses `wrangler.jsonc` (or `wrangler.json`). Flag `wrangler.toml` in new projects.

---

## Request & Response Handling

### Stream request and response bodies

Workers have a 128 MB memory limit. Buffering entire bodies with `await response.text()` or `await request.arrayBuffer()` crashes on large payloads. Stream data through using `TransformStream` or pass `response.body` directly.

**Check**: any `await response.text()`, `await response.json()`, or `await response.arrayBuffer()` on data that could be large or unbounded. Small, bounded payloads (known-size JSON, config files) are fine to buffer.

Correct — stream through:
```ts
async fetch(request: Request, env: Env): Promise<Response> {
  const response = await fetch("https://api.example.com/large-dataset");
  return new Response(response.body, response);
}
```

Correct — concatenate multiple streams:
```ts
async fetch(request: Request, env: Env, ctx: ExecutionContext): Promise<Response> {
  const urls = ["https://api.example.com/part-1", "https://api.example.com/part-2"];
  const { readable, writable } = new TransformStream();

  // Track the pipeline promise — don't let it float
  ctx.waitUntil((async () => {
    for (const url of urls) {
      const response = await fetch(url);
      if (response.body) {
        await response.body.pipeTo(writable, { preventClose: true });
      }
    }
    await writable.close();
  })());

  return new Response(readable, {
    headers: { "Content-Type": "application/octet-stream" },
  });
}
```

Anti-pattern:
```ts
// Buffers entire body — crashes on large payloads
const response = await fetch("https://api.example.com/large-dataset");
const text = await response.text();
return new Response(text);
```

**Retrieve**: streaming APIs at `/workers/runtime-apis/streams/`.

### Use waitUntil for work after the response

`ctx.waitUntil()` performs background work (analytics, cache writes, webhooks) after the response is sent. Keeps response fast. 30-second time limit after response.

**Check**: background work uses `ctx.waitUntil()`, not inline `await`. Do not destructure `ctx` — it loses the `this` binding and throws "Illegal invocation".

```ts
async fetch(request: Request, env: Env, ctx: ExecutionContext): Promise<Response> {
  const data = await processRequest(request);

  ctx.waitUntil(logToAnalytics(env, data));
  ctx.waitUntil(updateCache(env, data));

  return Response.json(data);
}
```

Anti-pattern:
```ts
// Destructuring ctx loses the this binding
const { waitUntil } = ctx;  // "Illegal invocation" at runtime
waitUntil(somePromise);
```

---

## Architecture

### Use bindings for Cloudflare services, not REST APIs

Bindings (KV, R2, D1, Queues, Workflows) are direct, in-process references — no network hop, no authentication, no extra latency. Using the Cloudflare REST API from a Worker wastes time and adds complexity.

**Check**: no `fetch("https://api.cloudflare.com/client/v4/...")` calls for services available as bindings.

```ts
// Binding — direct, zero-cost
const object = await env.MY_BUCKET.get("my-file");
```

Anti-pattern:
```ts
// REST API from inside a Worker — unnecessary overhead
const response = await fetch(
  "https://api.cloudflare.com/client/v4/accounts/.../r2/buckets/.../objects/my-file",
  { headers: { Authorization: `Bearer ${env.CF_API_TOKEN}` } }
);
```

### Use Queues and Workflows for async and background work

Long-running, retriable, or non-urgent tasks should not block a request.

- **Queues**: decouple producer from consumer. Fan-out, buffering/batching, simple single-step background jobs. At-least-once delivery.
- **Workflows**: multi-step durable execution. Each step's return value is persisted; only failed steps retry. Can run for hours/days/weeks.
- **Both together**: Queue buffers high-throughput entry, consumer creates Workflow instances for complex processing.

**Check**: long-running work (email sends, webhooks, multi-step processes) is offloaded to Queues or Workflows, not done inline in the fetch handler.

```ts
async fetch(request: Request, env: Env): Promise<Response> {
  const order = await request.json<{ id: string; type: string }>();

  if (order.type === "simple") {
    await env.ORDER_QUEUE.send({ orderId: order.id, action: "send-email" });
  } else {
    await env.FULFILLMENT_WORKFLOW.create({ params: { orderId: order.id } });
  }

  return Response.json({ status: "accepted" }, { status: 202 });
}
```

**Retrieve**: `/queues/` and `/workflows/` for current APIs. For Workflow-specific rules, see [Rules of Workflows](https://developers.cloudflare.com/workflows/build/rules-of-workflows/).

### Use service bindings for Worker-to-Worker communication

Service bindings are zero-cost, bypass the public internet, and support type-safe RPC. Do not call another Worker via its public URL.

**Check**: Worker-to-Worker calls use `env.SERVICE_NAME.method()` (RPC) or `env.SERVICE_NAME.fetch()`, not `fetch("https://my-other-worker.example.com/...")`.

```ts
import { WorkerEntrypoint } from "cloudflare:workers";

export class AuthService extends WorkerEntrypoint {
  async verifyToken(token: string): Promise<{ userId: string; valid: boolean }> {
    return { userId: "user-123", valid: true };
  }
}

// Caller Worker
const auth = await env.AUTH_SERVICE.verifyToken(token);
```

**Retrieve**: verify `WorkerEntrypoint` import path and signature against latest `@cloudflare/workers-types`.

### Use Hyperdrive for external database connections

Hyperdrive maintains a regional connection pool, eliminating per-request TCP + TLS + auth cost (often 300-500ms). Create a new `Client` per request — Hyperdrive manages the underlying pool. Requires `nodejs_compat`.

**Check**: any `new Client()` or database connection that uses a direct connection string instead of `env.HYPERDRIVE.connectionString`.

```jsonc
{
  "hyperdrive": [{ "binding": "HYPERDRIVE", "id": "<YOUR_HYPERDRIVE_ID>" }]
}
```

```ts
import { Client } from "pg";

async fetch(request: Request, env: Env): Promise<Response> {
  const client = new Client({ connectionString: env.HYPERDRIVE.connectionString });
  await client.connect();
  const result = await client.query("SELECT id, name FROM users LIMIT 10");
  return Response.json(result.rows);
}
```

**Retrieve**: `/hyperdrive/` for current configuration and supported databases.

---

## Observability

### Enable Workers Logs and Traces

Enable `observability` in wrangler config before deploying to production. Use `head_sampling_rate` to control volume and cost. Use structured JSON logging — `console.log(JSON.stringify({...}))` — so logs are searchable. Use `console.error` for errors (appears at error severity in the dashboard).

**Check**: `observability.enabled` is `true` in config. Logging uses structured JSON, not string concatenation.

```jsonc
{
  "observability": {
    "enabled": true,
    "logs": { "head_sampling_rate": 1 },
    "traces": { "enabled": true, "head_sampling_rate": 0.01 }
  }
}
```

```ts
// Structured JSON — searchable and filterable
console.log(JSON.stringify({ message: "incoming request", method: request.method, path: url.pathname }));

// Error severity
console.error(JSON.stringify({ message: "request failed", error: e instanceof Error ? e.message : String(e) }));
```

Anti-pattern:
```ts
// Unstructured string logs — hard to query
console.log("Got a request to " + url.pathname);
```

**Retrieve**: `/workers/observability/logs/workers-logs/` and `/workers/observability/traces/` for current config options.

---

## Code Patterns

### Do not store request-scoped state in global scope

Workers reuse isolates across requests. Module-level mutable variables cause cross-request data leaks, stale state, and "Cannot perform I/O on behalf of a different request" errors.

**Check**: no mutable `let`/`var` at module scope that gets assigned inside a handler. Pass state through function arguments.

```ts
export default {
  async fetch(request: Request, env: Env, ctx: ExecutionContext): Promise<Response> {
    const userId = request.headers.get("X-User-Id");
    const result = await handleRequest(userId, env);
    return Response.json(result);
  },
} satisfies ExportedHandler<Env>;
```

Anti-pattern:
```ts
// Module-level mutable state — leaks between requests
let currentUser: string | null = null;

export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    currentUser = request.headers.get("X-User-Id");  // Visible to next request
    // ...
  },
};
```

### Always await or waitUntil Promises

A Promise that is not `await`ed, `return`ed, or passed to `ctx.waitUntil()` is a floating promise. Causes: dropped results, swallowed errors, unfinished work. The runtime may terminate the isolate before it completes.

**Check**: every `fetch()`, `env.*.put()`, `env.*.send()`, and any other async call is handled. Enable `no-floating-promises` lint rule.

```bash
# ESLint
npx eslint --rule '{"@typescript-eslint/no-floating-promises": "error"}' src/

# oxlint
npx oxlint --deny typescript/no-floating-promises src/
```

```ts
// Correct: await when you need the result
const response = await fetch("https://api.example.com/process", { method: "POST", body: JSON.stringify(data) });

// Correct: waitUntil when you don't need the result before responding
ctx.waitUntil(fetch("https://api.example.com/webhook", { method: "POST", body: JSON.stringify(data) }));
```

Anti-pattern:
```ts
// Floating promise — result dropped, error swallowed
fetch("https://api.example.com/webhook", { method: "POST", body: JSON.stringify(data) });
```

### Be aware of platform limits

Workers have a 10ms CPU time limit (Bundled) or 30s (Standard/Unbound). Heavy synchronous work — tight loops, large JSON parsing, compute-intensive crypto — can hit the CPU limit and terminate the request.

**Check**: compute-heavy operations that run synchronously. Consider breaking work into smaller chunks, offloading to Queues/Workflows, or using WebAssembly for CPU-intensive tasks.

**Retrieve**: current limits at `/workers/platform/limits/`.

---

## Security

### Use Web Crypto for secure token generation

Use `crypto.randomUUID()` for unique IDs and `crypto.getRandomValues()` for random bytes. `Math.random()` is not cryptographically secure.

For comparing secrets (API keys, HMAC signatures), use `crypto.subtle.timingSafeEqual()`. Hash both values to a fixed size first — do not short-circuit on length mismatch (leaks length via timing).

**Check**: no `Math.random()` for security-sensitive values. Secret comparisons use `timingSafeEqual` with fixed-size hashing.

```ts
// Secure random UUID
const sessionId = crypto.randomUUID();

// Secure random bytes
const tokenBytes = new Uint8Array(32);
crypto.getRandomValues(tokenBytes);
const token = Array.from(tokenBytes).map((b) => b.toString(16).padStart(2, "0")).join("");
```

```ts
// Constant-time comparison — hash first to avoid length leak
async function verifyToken(provided: string, expected: string): Promise<boolean> {
  const encoder = new TextEncoder();
  const [providedHash, expectedHash] = await Promise.all([
    crypto.subtle.digest("SHA-256", encoder.encode(provided)),
    crypto.subtle.digest("SHA-256", encoder.encode(expected)),
  ]);
  return crypto.subtle.timingSafeEqual(providedHash, expectedHash);
}
```

Anti-pattern:
```ts
// Predictable — not cryptographically secure
const token = Math.random().toString(36).substring(2);

// Timing side-channel — leaks information about the expected value
return provided === expected;
```

**Retrieve**: `/workers/runtime-apis/web-crypto/` for current API surface.

### Explicit error handling over passThroughOnException

`passThroughOnException()` is a fail-open mechanism that sends requests to the origin when the Worker throws. It hides bugs and makes debugging difficult. Use explicit try/catch with structured error responses.

**Check**: no `ctx.passThroughOnException()` calls. Error handling uses try/catch with structured JSON error responses and `console.error`.

```ts
async fetch(request: Request, env: Env, ctx: ExecutionContext): Promise<Response> {
  try {
    const result = await handleRequest(request, env);
    return Response.json(result);
  } catch (error) {
    const message = error instanceof Error ? error.message : "Unknown error";
    console.error(JSON.stringify({ message: "unhandled error", error: message, path: new URL(request.url).pathname }));
    return Response.json({ error: "Internal server error" }, { status: 500 });
  }
}
```

---

## Development & Testing

### Test with @cloudflare/vitest-pool-workers

Runs tests inside the Workers runtime with real bindings. Catches issues that Node.js-based tests miss.

**Known pitfall**: the Vitest pool auto-injects `nodejs_compat`, so tests pass even if your wrangler config is missing the flag. Always confirm your `wrangler.jsonc` includes `nodejs_compat` if your code depends on Node.js built-ins.

**Check**: test setup uses `@cloudflare/vitest-pool-workers`. Tests cover nullable returns (e.g., KV `.get()` returning `null`).

```ts
import { describe, it, expect } from "vitest";
import { env } from "cloudflare:test";

describe("KV operations", () => {
  it("should store and retrieve a value", async () => {
    await env.MY_KV.put("key", "value");
    const result = await env.MY_KV.get("key");
    expect(result).toBe("value");
  });

  it("should return null for missing keys", async () => {
    const result = await env.MY_KV.get("nonexistent");
    expect(result).toBeNull();
  });
});
```

**Retrieve**: `/workers/testing/vitest-integration/` for current setup and configuration.

````
