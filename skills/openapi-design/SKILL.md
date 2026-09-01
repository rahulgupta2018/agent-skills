---
name: openapi-design
description: >
  Designs and maintains OpenAPI 3.x/3.1 API specifications, especially multi-file specs split
  across a shared components file plus one file per service or bounded context. Activates when
  authoring, splitting, reviewing, or validating an OpenAPI/Swagger document (openapi.yaml,
  components.yaml, `$ref` wiring, security schemes, error envelopes, paths/schemas), or when
  generating a spec from an API inventory. Owns the structure, cross-file references, and lint
  conformance of the spec. Does not own implementing the API, generating server/client code, or
  designing the REST resource model itself.
license: MIT
metadata:
  author: Bluevix Labs
  version: "0.2.0"
  last_updated: 2026-08-30
  category: coding
---

# OpenAPI Design (multi-file 3.x / 3.1)

## Overview

Produces valid, lint-clean OpenAPI specifications and keeps them that way as they grow. The
default shape is a **shared `components.yaml` plus one spec file per service / bounded context**,
wired with cross-file `$ref`. The skill encodes the reference-resolution rules that trip people up
(especially security schemes), a standard error/response envelope, and a validation loop.

**Freedom level: MEDIUM** — the multi-file layout and reference rules are fixed; the resource
model, naming, and envelope shapes adapt to the API.

## When to Activate

Activate when:
- Authoring a new OpenAPI/Swagger document, or converting an API inventory/design into one.
- Splitting a monolithic spec into a components file plus per-service files, or wiring `$ref`s.
- Reviewing or debugging a spec that fails to bundle/lint/resolve references.
- Defining `securitySchemes`, error envelopes, pagination, idempotency, or standard responses.

**Do not activate** (adjacent skills own this):
- `typed-service-contracts` — owns the in-code Spec/Handler contract and Result types.
- `fullstack-developer` / `python-expert` / `java-quarkus-expert` — own implementing the API and
  generating server/client code from the spec.
- `technical-writer` — owns prose API guides, tutorials, and narrative reference docs.
- `code-reviewer` — owns security/quality review of the *implementation* behind the spec.

## Core Concepts

- **Two kinds of `$ref`.** A `$ref` to a *definition* (schema, parameter, header, response,
  requestBody) resolves across files fine: `./components.yaml#/components/schemas/Error`. A
  **security requirement is different** — see the top gotcha.
- **Split by ownership, not size.** One file per service / bounded context that owns those paths,
  plus one shared `components.yaml` (`paths: {}`) for cross-cutting schemas, params, headers, and
  standard responses. Each spec file is independently bundleable and lintable.
- **3.1 vs 3.0.** 3.1 is JSON-Schema-2020-12 aligned: nullable is `type: [string, "null"]` (not
  `nullable: true`); `examples` (plural, keyed or array) over singular `example`; `webhooks` is a
  top-level object. Pick one version across all files — don't mix.
- **The spec is the contract, not a mirror of internals.** An internal upstream hop that speaks a
  vendored schema (e.g. a proxy's own OpenAI-compatible surface) needs no authored path — say so
  in prose rather than inventing endpoints.

## Workflow

1. **Fix the version and layout.** Choose 3.0.x or 3.1.0 for every file. Create `components.yaml`
   with `openapi:`, `info:`, and `paths: {}`; create one spec file per service.
2. **Populate `components.yaml`** with the cross-cutting building blocks: `securitySchemes`,
   reusable `parameters` and `headers`, an `Error` schema, a `Job`/async schema if used, and a set
   of standard `responses` (`BadRequest`, `Unauthorized`, `Forbidden`, `NotFound`, `Conflict`,
   `TooManyRequests`, `InternalError`, `ServiceUnavailable`, …).
3. **Author each service file.** Set its `servers`, its default `security`, its `paths`, and local
   `components.schemas`. Reference shared pieces with relative cross-file `$ref`
   (`./components.yaml#/components/responses/Unauthorized`).
4. **Wire security correctly.** Define every referenced `securityScheme` **locally** in each file
   that names it (see gotcha 1). Set a root `security`; override per-operation with `security: []`
   for public endpoints.
5. **Validate.** Bundle + lint every service file (which pulls in `components.yaml`) with Redocly
   or Spectral. Fix errors; triage warnings (many are style-only).
6. **Keep in sync.** When a locally-duplicated `securityScheme` changes, update every copy; mark
   the canonical copy in `components.yaml`.

## Practical Guidance

- **Error envelope.** Pick one shape and reuse it everywhere via a shared `Error` schema and named
  responses. Put rate-limit headers (`RateLimit-Limit/Remaining/Reset`, `Retry-After`) on the
  `TooManyRequests` response so every 429 carries them.
- **Idempotency.** For non-safe billable calls, take an `Idempotency-Key` header (shared param) and
  document that reuse with a different body is a `409`/`422`.
- **Async / long jobs.** Return `202` with a `Job` (`status: queued|running|ready|failed`) and a
  poll endpoint, rather than blocking.
- **Correlation.** Accept and echo a correlation/trace header (shared param + response header) on
  every operation for observability.
- **Naming.** Custom headers use a consistent prefix (e.g. `x-*`); give every operation a stable
  `operationId` (codegen and tooling key off it).
- **Example payloads (make the docs readable).** Add `examples` so renderers (Redoc/Swagger UI)
  show real request/response samples, not just field lists. Two placements, both rendered: a
  **media-type** `examples` map (keyed `{summary, value}`, sibling to `schema:`) for per-operation
  request/response samples, and a **schema-level** `examples` array (3.1) for a reusable default
  sample that follows the schema everywhere it's `$ref`'d. Prefer schema-level for shared/admin
  models so every operation inherits a sample; use media-type keys to show variants (e.g. `simple`
  vs `streaming`). For an SSE `text/event-stream` body, use singular `example: |` with literal
  `data: {...}` frames ending `data: [DONE]`.

## Examples

**Cross-file reference to a shared response:**
```yaml
responses:
  "401": { $ref: "./components.yaml#/components/responses/Unauthorized" }
```

**Security scheme must be local even though the canonical copy is shared:**
```yaml
# data-plane.yaml
security:
  - virtualKey: []          # references by KEY NAME, resolved in THIS document only
components:
  securitySchemes:          # so define it locally (canonical copy in components.yaml)
    virtualKey: { type: http, scheme: bearer }
```

**Public endpoint overriding a root security default:**
```yaml
/.well-known/jwks.json:
  get:
    security: []            # explicitly unauthenticated
```

**Sample payloads that render as Redoc panels** — media-type keyed vs schema-level (3.1):
```yaml
# media-type: per-operation request/response variants
requestBody:
  content:
    application/json:
      schema: { $ref: "#/components/schemas/ChatCompletionRequest" }
      examples:
        simple:    { summary: One-shot, value: { model: platform/gpt4o, messages: [ { role: user, content: Hello } ] } }
        streaming: { summary: Streamed,  value: { model: platform/gpt4o, stream: true, messages: [ { role: user, content: Hi } ] } }
---
# schema-level: one default sample that follows the schema wherever it is $ref'd
Tenant:
  type: object
  properties: { id: { type: string }, name: { type: string } }
  examples:                 # 3.1 array form (NOT singular `example`)
    - { id: ten_01H…, name: Acme Housing }
```

## Guidelines

1. One OpenAPI version across all files; never mix 3.0 and 3.1.
2. `components.yaml` carries `paths: {}` and only cross-cutting definitions.
3. Cross-file `$ref` uses relative paths (`./components.yaml#/…`); no absolute filesystem paths.
4. Every referenced `securityScheme` is defined in the same file that names it.
5. Every operation has an `operationId`, a success response, and a shared error response set.
6. Validate with a real linter (Redocly/Spectral) before declaring the spec done; report the
   error/warning split.
7. Nullable in 3.1 is `type: [T, "null"]`; use `examples`, not `example`.

## Gotchas

1. **Security schemes don't `$ref` across files.** A `security` requirement (`- virtualKey: []`)
   references a scheme **by key name, resolved only within the same document** — you cannot point a
   requirement at a `securityScheme` defined in another file. Redocly's `security-defined` rule
   flags this as an error. Fix: define each scheme **locally** in every spec file that uses it, and
   keep the canonical copy in `components.yaml` marked "keep in sync". (Schemas, params, headers,
   responses, requestBodies all `$ref` across files fine — only security requirements are
   name-scoped.)
2. **Missing root/operation security reads as an error.** Linters require *every* operation to have
   security defined on it or at the root. Set a root `security`; use `security: []` to mark public
   endpoints explicitly rather than omitting it.
3. **Sandbox npm cache EPERM.** `npx @redocly/cli lint` can fail with `EPERM` on `~/.npm`. Set
   `npm_config_cache="$TMPDIR/npmcache"` to use a writable cache.
4. **3.0 vs 3.1 nullable.** `nullable: true` is 3.0-only; in 3.1 it silently does nothing — use
   `type: [T, "null"]`. Mixing the two versions across files breaks bundling.
5. **Warnings aren't errors.** Missing `license`, absent `4XX` on health/metrics probes, and
   per-tag descriptions are style warnings — fine to leave on internal design-doc specs. Don't
   chase a zero-warning count; fix the errors.

## Integration

- `typed-service-contracts` — the spec is the wire contract; that skill designs the in-code
  Spec/Handler and Result types behind each operation.
- `fullstack-developer` / language `*-expert` skills — consume the spec to implement handlers and
  generate typed clients.
- `technical-writer` — turns the validated spec into narrative guides and tutorials.
- `code-reviewer` — reviews the implementation the spec describes.

## References

- OpenAPI 3.1.0 specification: https://spec.openapis.org/oas/v3.1.0
- Redocly CLI (`lint`, `bundle`, rules): https://redocly.com/docs/cli
- Spectral (alternative linter): https://docs.stoplight.io/docs/spectral
- Related skill: `typed-service-contracts`
